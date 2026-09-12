# Errors, retry и recovery

Общей retry policy системы нет. Android различает local per-file failure, stage error, framework retry и runtime UI outcome. Server описывает error response и ограниченные компенсации, но не назначает client retry policy, retryable flag или Retry-After.

## Error ownership

| Failure | Detected by | Persisted where | Retry owner | Recovery guarantee |
| --- | --- | --- | --- | --- |
| Blank runtime URL / public ping failure | Android gate | Media state gate не меняет; outcome только память, work scheduling отдельно | WorkManager по Android Result.retry | До успешного gate scan/hash/upload не начинаются; storage readiness не проверен |
| Нет permission до scan/hash | Android guard | Эта стадия media writes не делает; restore до scan мог уже пройти | Новый scan/run после grant; отдельного retry этой ветки нет | Bare WM success, прежний outcome может остаться |
| Query exception/null cursor | Android | Apply ещё не начат | Whole-worker retry | Старый индекс этим query не удаляется |
| Неполный successful snapshot | Android воспринимает как snapshot success | Stale row DELETE local DB | Следующий scan может снова индексировать | Не доказывает physical deletion; remote copy не удаляет |
| Ошибка apply local index | Android | Предыдущие local statements могут сохраниться | Worker retry | Нет rollback всего snapshot |
| Hash stream missing/permission/unknown | Android URI read | FAILED reason, count++, time | Автоматического restore нет | Новый grant без content change сам FAILED не сбрасывает |
| Hash IOException | Android | FAILED CHECKSUM_IO/count/time | Eligible restore следующего worker | Ограниченный count/cooldown, не таймер и не global exactly-once |
| CAMERA lookup / exists error на pre-check | Android по HTTP/parser/network | Прежние batches/rows сохраняются | Whole-worker retry | Upload этого run не начинается |
| CAMERA lookup error перед upload | Android | Lookup failure отдельно не persisted | Upload продолжается без folderId | Target может стать FILES по server bytes type |
| Local MIME не image/ или upload pre-open missing/permission | Android | FAILED NOT_IMAGE/FILE_NOT_FOUND/PERMISSION | Нет automatic retry этих FAILED | Следующий файл обрабатывается |
| Upload400/409/413 | Server response, Android classifier | FAILED HTTP_400/409/413/count/time в local DB | Нет automatic restore | Следующий файл; 409 name conflict вне CAMERA, не duplicate checksum |
| Upload прочий HTTP, в т.ч. 401/403/404/429/5xx | Android | PENDING_UPLOAD; новых failure/count/time нет | Whole-worker retry | Stop upload run; изменение причины не гарантировано |
| Upload IOException или 2xx без id>0 | Android | PENDING_UPLOAD | Whole-worker retry | Server мог завершить commit; повтор не repair bytes |
| Generic exception/process death после UPLOADING | Android при exception; смерть не даёт результата | Может остаться UPLOADING | Stage recovery при следующем достижении upload | Сначала проходят gate/scan/hash/pre-check |
| Access401 | Server validation | Access сам не server-persisted | Android authenticator | Один refresh/replay либо replay current token; без бесконечной цепочки |
| Refresh non-2xx | Android по Server response | Clear encrypted pair; server revoke из 5xx не следует | Caller-specific; новый login доступен пользователю | Временная5xx может потерять локальную session |
| Refresh IOException / logout IOException до clear | Android | Pair может сохраниться | Caller/user; сервер общей политики не назначает | Local exit не гарантирован |
| Upload DB/FS partial failure | Server | Temp/final и DB могут разойтись | Best-effort synchronous compensation | Нет durable repair queue; new attempt не гарантирует repair |
| Response потерян после server commit / local mark failure | Network/Android observation | Remote commit и незавершённый local status | Android retry | Same tuple duplicate200 может дать ID; сохранение tuple не закреплено |
| FS delete failure после DB commit (server-only operation) | Server | DB удалена; bytes orphan, error log | Автоматического retry нет | Ответ204; повтор по прежнему ID404 не повторяет cleanup |

## Три разных retry механизма Android

**Checksum:** выбираются FAILED CHECKSUM_IO, count<3, lastFailureAt<=now−300000; максимум500 строк один раз после successful ping перед scan. Restore очищает reason/time и ставит PENDING, сохраняя count/hash/ID. Первый failure делает count1; ещё два последовательных неуспеха приводят к count3 и прекращению auto restore. Success hash/upload или content change обнуляет count. Пять минут — eligibility по wall clock, не scheduled timer; нужен последующий worker. Concurrent restore не даёт глобальной гарантии числа выполнений.

**HTTP/auth:** refresh реагирует на protected401. Upload400/409/413 terminal для обычной очереди; остальные HTTP и IO возвращают pending и останавливают цикл. Эти transient failures не увеличивают local retryCount. Android не использует structured server code/Retry-After для policy. Детерминированные403/404/5xx могут повторяться без изменения условий.

**WorkManager:** Result.retry повторяет весь pipeline; собственного attempt limit/backoff/runAttemptCount branching нет. Frozen Android описывает framework defaults exponential30s с пределом delay5h; фактические задержки зависят от framework/OS/constraints, не являются SLA. One-time и periodic имеют разные unique names и могут пересекаться. WM success не исключает local FAILED.

## Head-of-line

Upload выбирает newest-first, последовательно. Первый transient response/IO/invalid success ID возвращает эту строку в pending и завершает upload run. Другие файлы остаются на последующий запуск. Если один новый файл воспроизводимо ошибается, более старые pending могут задерживаться. Это текущее условное следствие очереди, не назначенный fix.

## Server recovery boundary

Server считает bytes и сохраняет DB/FS синхронно. При DB failure после final move пытается удалить свой final; при uniqueness violation перечитывает logical duplicate и может вернуть200. Cleanup best effort: partial IO/crash может оставить bytes; post-commit error может оставить DB rows без bytes. Неизвестный результат нельзя автоматически трактовать как отсутствие server side effect.

Нет общей transaction DB/FS, Idempotency-Key, persisted request result, upload/sync session, repair/scrub/orphan jobs или startup reconciliation. Same-folder duplicate не проверяет и не восстанавливает bytes. Ограничения durability — [12](12-data-consistency-and-durability.md).

Error id/code/detail не образуют общую persisted диагностику Android↔Server. Android хранит короткую per-file reason либо runtime outcome; controlled ErrorResponse не покрывает весь framework wire. Product meaning recovery и неизвестные runtime cases — [16](16-system-open-questions.md).


## Источники

[S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [S13](../../../../PhotoCloudServer/docs/spec/server-as-is/13-observability.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
