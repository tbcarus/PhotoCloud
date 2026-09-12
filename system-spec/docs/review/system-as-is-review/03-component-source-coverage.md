# Component source coverage

Проверка того, что System As-Is покрывает **system-relevant** материал двух frozen specifications. Перенос component-only деталей не требуется и его отсутствие не считается пробелом.

Coverage:

* **COMPLETE** — system-relevant содержание области перенесено полностью;
* **PARTIAL** — перенесено с заметной, но обоснованной или несущественной неполнотой;
* **NOT_REQUIRED** — материал component-only, перенос не требуется;
* **MISSING** — system-relevant материал потерян.

Итог: **COMPLETE 24**, **PARTIAL 3**, **NOT_REQUIRED 6**, **MISSING 0**.

## Системные области

| System area | Server source | Android source | Coverage | Notes |
| --- | --- | --- | --- | --- |
| Границы и внешние зависимости | S01, S11, S12 | A01, A10, A13 | COMPLETE | Inside/external разделение верно; MediaStore, OS scheduling, сеть и SMTP оставлены внешними; managed DB/FS данные включены в системную persistence boundary |
| Ответственность компонентов | S03, S05, S06, S07, S10, S12, S15 | A01, A03, A05, A06, A07, A08, A10, A16, A17 | COMPLETE | 23 responsibility-строки в [02](../../spec/system-as-is/02-component-responsibilities.md); ownership каждой значимой ответственности (auth, credentials, token persistence, media discovery, checksum, local state, folders, logical media, physical bytes, dedup, retry, background, delete, rename/move/copy, remote listing, reconciliation) соответствует frozen specs |
| Доменная модель | S03, S07, S09 | A03, A09 | COMPLETE | Разделены User/principal, device media, Android local record, Folder, logical FileItem, StoredObject, physical bytes, access session, refresh session. Допустимая кардинальность `StoredObject`→`FileItem` описана без вывода sharing API |
| Identity, ownership, scope | S03, S05, S06, S07 | A03, A05, A07, A09, A17 | COMPLETE | Девять идентификаторов с creator/scope/uniqueness/persistence/relationship; scope-модель по семи измерениям; отдельная source-of-truth матрица с явной оговоркой, что authority не гарантирует согласованность других authorities |
| Effective API contract | S05, S06, S07, S10, S15 | A06, A07, A08, A11, A13, A17 | COMPLETE | Все 10 используемых operations сверены по method/path/auth/request/consumed response/status. Арифметика 36 = 10 + 19 + 7 проверена и сходится |
| Checksum pre-check semantics | S05, S07, S08 | A06, A08, A17 | COMPLETE | Folder scope, 500 raw/500 unique, lowercase+dedup, logical partition без file IDs, отсутствие reservation и physical проверки — всё перенесено |
| Upload semantics и default routing | S05, S07, S08 | A06, A08, A11, A17 | COMPLETE | Explicit target, omitted `folderId`, routing по detected bytes, 100 MiB/110MB, duplicate 200, 409 только вне CAMERA, DataIntegrityViolation reread |
| Auth/session lifecycle | S03, S05, S06, S09, S10 | A03, A06, A07, A10, A11, A14, A17 | COMPLETE | Цепочка credentials → login → access+refresh → persistence → use → 401 → refresh → retry воспроизведена; политики Server и Android не слиты в единую contract guarantee |
| Media sync lifecycle | S03, S05, S07, S09, S10, S12 | A03, A05, A06, A08, A09, A10, A11, A17 | COMPLETE | 12 стадий с persistent result и failure behaviour; ветки existing и upload разведены; CAMERA bootstrap отделён от fallback |
| System state mapping | S03, S09 | A03, A08, A09, A10, A11, A17 | COMPLETE | Derived phase объявлен descriptive в первой строке; persisted Android статусы и server evidence разнесены; concurrency interleavings перенесены как условные |
| Error / retry / recovery | S05, S07, S10, S12, S13 | A07, A08, A09, A10, A11, A17 | COMPLETE | 21 строка ownership ошибок; все значимые failure-ветки Android failure matrix присутствуют; retry owner указан отдельно от detection; фиктивная общая retry policy не создана |
| Startup / restart / reboot | S07, S09, S12 | A03, A07, A08, A09, A10, A13, A19 | COMPLETE | Persisted / in-memory / OS-unknown разделены строго; cold URL gate как факт, OS timing как OPEN |
| Delete / change / reconciliation | S03, S05, S07, S09, S15, S17 | A03, A05, A08, A09, A16, A17, A19 | COMPLETE | Local modification, local deletion, сохранение server copy, отсутствие reverse reconciliation, неиспользование server rename/move/delete — всё присутствует; one-way backup выведен из frozen specs, а не принят как решение |
| Data consistency и durability | S01, S03, S05, S07, S09, S10, S12, S17 | A03, A05, A08, A09, A10, A11, A17, A19 | COMPLETE | Все шесть требуемых boundaries присутствуют; шесть классов доказательств разделены; гарантии не сильнее frozen inputs |
| Capability matrix | S15 | A16, A17 | COMPLETE | 26 системных capabilities с корректными ссылками на component registries; server-only группы не размножены в псевдо-Android capabilities |
| Mismatch registry | S03, S05, S06, S07, S10 | A17 | COMPLETE | `INT-AND-001…004` перенесены one-to-one с сохранением условности; дополнительных mismatch в frozen specs review также не нашёл |
| Risk registry | S06, S07, S10, S13, S14, S16 | A10, A11, A14, A15, A18 | PARTIAL | 18 системно значимых рисков, не механическое объединение 38+39. Не представлены системные последствия `RISK-SRV-002` и `RISK-SRV-005` — [REV-SYS-001](01-findings.md#rev-sys-001). Точность ссылок — [REV-SYS-003](01-findings.md#rev-sys-003) |
| Open questions | S17 | A19 | PARTIAL | 29 из 30 `OPEN-AND` и 19 из 30 `OPEN-SRV` представлены; исключения в основном объявлены. `OPEN-SRV-012` и `OPEN-SRV-017` не представлены и не исключены явно — [REV-SYS-002](01-findings.md#rev-sys-002) |
| Traceability | S19 (не использован как источник claims), S20 | A21 (не использован), A22 | COMPLETE | Area mapping по 20 областям; fingerprints 34 файлов проверены пересчётом; authority direction корректна |
| Flow index | S05, S06, S07, S10, S12 | A05, A07, A08, A10, A11, A17 | PARTIAL | 12 system-level flows, ни один не вводит новую логику. Одна неточная entry condition — [REV-SYS-004](01-findings.md#rev-sys-004) |
| Observability и диагностика | S13, S16 | A14, A18 | COMPLETE | BODY logging на обеих сторонах, отсутствие общей persistent error history и корреляции error id перенесены в `SYS-RISK-011`, `SYS-RISK-016`, `SYS-OPEN-012`. Точно указано, что чувствительны credentials/tokens/metadata (server заменяет multipart/image/video в тексте на `[binary]`) |
| Testing evidence | S14 | A15 | COMPLETE | «Server tests не удостоверяют Android↔Server; Android product coverage отсутствует» — `SYS-RISK-016`, `SYS-OPEN-013`. Server tests не выданы за системное подтверждение |
| Configuration / limits | S11 | A13 | COMPLETE | Перенесены только system-значимые значения: 20 мин/7 дней/3 дня, 100 MiB/110MB, 500/100/20, 3 попытки/300000 мс, debounce 3 с, 1 час, HTTP IPv4/port. Полные property-таблицы обоснованно оставлены компонентам |
| Security / privacy периметр | S06, S16 | A14, A18 | PARTIAL | HTTP без TLS и BODY logging перенесены (`SYS-RISK-011/012`), media privacy и GPS — в `SYS-OPEN-007/018`. Неполнота credential/account периметра — тот же [REV-SYS-001](01-findings.md#rev-sys-001) |

## Материал, обоснованно не перенесённый

| Component материал | Source | Coverage | Notes |
| --- | --- | --- | --- |
| Server architecture, слои, packages, mapping, transactions | S02, S04 | NOT_REQUIRED | Component-only internals. Раздел 04 (schema, constraints, индексы, миграции) не является системной семантикой; результат — unique tuple `user+folder+checksum` — перенесён |
| Server framework download contract (Range/206/416, ETag, HEAD) | S05 «Framework surface», `OPEN-SRV-029` | NOT_REQUIRED | Android download не использует; пакет прямо это оговаривает в Consolidation boundary |
| Server move → 500 `DATABASE_CONSTRAINT_VIOLATION` | S05, S07, `RISK-SRV-013` | NOT_REQUIRED | Move не вызывается Android; A17 прямо фиксирует непереносимость. Исключение обосновано |
| Server download-ответ буферизуется целиком (возможный OOM) | `RISK-SRV-029` | NOT_REQUIRED | Относится к download, которого нет в Android inventory. Upload-часть ресурсного вопроса покрыта `SYS-RISK-018` и `SYS-OPEN-019` |
| Миграции 01–14, DROP `media_file`, rollback | S04, `RISK-SRV-019` | NOT_REQUIRED | Server deployment/maintenance. Фактическая неизвестность применённого состояния DB перенесена как `SYS-OPEN-014` |
| Server naming/path internals: sanitizer, extension fallback, длина 255/80/20, lexical path guard | S07, `RISK-SRV-020/023` | NOT_REQUIRED | Internals хранения. System сохраняет наблюдаемое следствие — normalized filename и игнорирование его клиентом |
| `FileItem.checksum` ↔ `StoredObject.checksum` синхронизируются кодом, не constraint | `RISK-SRV-015` (MEDIUM) | NOT_REQUIRED | Внутренняя целостность server DB. System фиксирует наблюдаемую границу «logical existing не доказывает bytes», что покрывает пользовательское следствие |
| Android UI: modal loader, немаскируемый пароль, permission lifecycle, port validation, таймстемпы | `RISK-AND-005/028/032/033/034`, A12 | NOT_REQUIRED | Локальные UI/component-ограничения без изменения системной семантики. `RISK-AND-005` частично отражён фразой в [06](../../spec/system-as-is/06-auth-session-lifecycle.md) о пароле как открытой строке формы |
| Android Room schema, миграции 1→2→3, UNUSED helpers | A04, `AND-DATA-002/005` | NOT_REQUIRED | Component internals; системное следствие destructive fallback перенесено через `RISK-AND-029` в `SYS-RISK-014` |
| Server/Android verification logs | S18, A20 | NOT_REQUIRED | Пакет корректно не использует `VER-*` записи как самостоятельные источники и принимает их результаты как frozen input |

## Проверка «ничего значимого не потеряно»

Отдельно проверены наиболее вероятные кандидаты на `LOST_SYSTEM_FACT`:

| Кандидат | Frozen source | Найдено в System |
| --- | --- | --- |
| Public ping предшествует local стадиям, поэтому при outage индекс/hash не обновляются фоновым путём | `RISK-AND-039` | [09](../../spec/system-as-is/09-error-retry-recovery.md) строка gate; `SYS-RISK-015` («outage gate задерживает local index») |
| Успех worker совместим с FAILED и оставшимися CHECKSUM_READY | A08, A09, `RISK-AND-027` | 00, 07, 08, 09, `SYS-RISK-016`, `SYS-FLOW-007` |
| Неполная exists partition оставляет hash в CHECKSUM_READY без собственного retry | A08, A11 | [05](../../spec/system-as-is/05-effective-api-contract.md) защитная ветка; 09 (no-progress) |
| Destructive fallback Room может уничтожить локальную очередь/историю | `RISK-AND-029` | `SYS-RISK-014`; `SYS-OPEN-020` |
| Enabled/banned не перепроверяются; reset не отзывает токены | `RISK-SRV-002` | Факт — в [06](../../spec/system-as-is/06-auth-session-lifecycle.md); последствие в SYS-RISK отсутствует → [REV-SYS-001](01-findings.md#rev-sys-001) |
| Нет rate limiter, пароль 4..20 | `RISK-SRV-005`, `OPEN-SRV-017` | Факт — в [05](../../spec/system-as-is/05-effective-api-contract.md); последствие и вопрос отсутствуют → [REV-SYS-001](01-findings.md#rev-sys-001), [REV-SYS-002](01-findings.md#rev-sys-002) |
| Terminal `HTTP_409` без auto restore и manual retry | A09, A11, `OPEN-SRV-012` | Факт — в 05 и 09; продуктовый вопрос отсутствует → [REV-SYS-002](01-findings.md#rev-sys-002) |
| Owner-delete удаляет все logical references объекта | S07, S08, `RISK-SRV-012` | [11](../../spec/system-as-is/11-delete-change-reconciliation.md); ссылка на конкретный risk ID отсутствует → [REV-SYS-003](01-findings.md#rev-sys-003) |
| UI scan — отдельный путь, не требующий server ping | A05, A10 | [07](../../spec/system-as-is/07-media-sync-lifecycle.md) стадия 2; [10](../../spec/system-as-is/10-startup-restart-reboot.md) Files bootstrap |
| Логика one-way backup не подменена bidirectional sync | A00, A01, A16 | 00, 11, `SYS-CAP-018` NOT PRESENT; product-смысл оставлен `SYS-OPEN-002` |

**MISSING: 0.** Ни один существенный system-level факт frozen specs не утрачен. Три случая неполноты касаются регистрации последствия/вопроса, а не самого факта, и оформлены как MINOR findings.
