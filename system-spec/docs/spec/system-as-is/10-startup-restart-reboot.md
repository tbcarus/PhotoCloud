# Startup, restart и reboot

## Что сохраняется

| State | Persistence / owner | После cold process start |
| --- | --- | --- |
| Local media rows, checksum/status/ID/failures | Android local DB | Завершённые writes сохраняются; SYNCED не revalidates remote |
| Access/refresh pair | Android encrypted preferences | Завершённая persistence сохраняется; validity устанавливает Server, restore/Keystore runtime отдельно OPEN |
| IP/port | Android settings persistence | Сохранены после successful UI public test |
| BaseUrlProvider | Память процесса; initial empty | Worker сам settings не читает |
| Observer registration/debounce | Память процесса | Потеряны; явный reconcile регистрирует снова при готовности |
| Last sync outcome | Память процесса | Потерян |
| Work requests | WorkManager scheduling persistence | Наличие request не восстанавливает app-specific URL; timing OS-dependent |
| Server users/folders/files/refresh | PostgreSQL | Персистентность DB отдельно от FS и client observation |
| Server final/temp bytes | Filesystem | Остатки возможны; автоматического startup repair нет |
| Server in-flight upload flags | Память server request | Потеряны; нет durable upload recovery session |

## Сценарии

| Entry | Current behaviour | Recovery boundary / unknown |
| --- | --- | --- |
| UI start | Network UI restore IP/port, назначение URL до ping; scheduling reconcile | Асинхронное restore оставляет краткое окно empty URL; успешный UI ping для restore не нужен |
| Files bootstrap/grant | Permission/URL/tokens влияют на local scan и reconcile | Worker сам permission dialog не показывает |
| Успешный login / logout Result | Explicit reconcile проверяет tokens+URL+permission | Нет subscription на все изменения состояния |
| Process death | Observer/URL/outcome теряются; writes остаются | HASHING/UPLOADING markers могут остаться без active worker |
| Headless cold worker | Empty runtime URL → SERVER_UNAVAILABLE + retry | Scan/hash/upload не начинаются до UI restore, даже если IP/port persisted |
| Periodic run в тёплом процессе | Полный pipeline после URL/public ping; configured1h, CONNECTED+batteryNotLow | Не обещан ровно час; нет Wi-Fi-only/charging requirement |
| Observer event | Images descendants event → debounce3s → one-time KEEP, CONNECTED | Observer только пока процесс жив; event не гарантирует немедленный scan |
| Reboot | App-specific boot restore URL/observer отсутствует | Persisted work не устраняет cold gate; OS/OEM schedules остаются unknown |
| Force-stop/Doze/constraint loss | Client explicit execution guarantee отсутствует | Реальные сроки/остановка — SYS-OPEN-016 и SYS-OPEN-019 |
| Server restart | Незавершённые request context потеряны | Нет restart reconciliation temp/final/DB и retries cleanup |

## Порядок восстановления

UI восстановление URL предшествует успешному прохождению worker gate. Eligible FAILED restore выполняется после ping. Оставшиеся HASHING сбрасываются при входе checksum stage после permission guard; UPLOADING — при входе upload stage после успешного прохождения предыдущих стадий. Это не startup-wide reset и не немедленная реакция на смерть процесса.

Если Server завершил upload, а Android умер до local mark, поздний повтор того же user/folder/bytes может получить duplicate200 и записать ID. Изменившиеся host/principal/bytes/target нарушают предпосылку такого повтора. Если Server потерял bytes, duplicate200 их не восстановит.

## Scheduling coherence и observer loss

Reconcile включает periodic+observer только при pair, непустом URL и permission; иначе отменяет periodic и останавливает observer. Authenticator clear не вызывает reconcile. Logout не отменяет ранее поставленный one-time request; он может продолжить local stages без entry auth gate.

KEEP не добавляет follow-up, если одноимённая one-time работа не закончена, в том числе в backoff. Событие после scan текущего run может ждать другого события или periodic. Dirty flag отсутствует. Отмена blocking HTTP не связана с Call.cancel, поэтому cancel не означает подтверждённое прекращение всех side effects.

Факты пустого URL и потери observer уже установлены. Неизвестность реального reboot/OEM timing не превращает их в гипотезу.


## Источники

[S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A13](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
