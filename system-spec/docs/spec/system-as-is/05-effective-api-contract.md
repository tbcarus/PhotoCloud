# Effective API contract

Это десять Android-used operations, сопоставленные с frozen Server API и Android integration section. Все пути ниже включают `/api/v1`. Configured Android base — `http://IPv4:port/`, без выбора hostname/IPv6/HTTPS/context path. Совпадение route/shape не означает согласованность всего lifecycle.

## Android-used contract

| Operation | Method/path | Android purpose / request → consumed response | Auth | System semantics |
| --- | --- | --- | --- | --- |
| Public connection test | GET /api/v1/test | Нет body → message; UI test и worker gate | Public, plain | Server 200 со static message; не DB/FS/SMTP readiness |
| Session test | GET /api/v1/test/auth | Нет body → message | Access | Server 200 после principal lookup; explicit UI test показывает Result, init verify игнорирует Result |
| Registration | POST /api/v1/auth/register | JSON email/password → message | Public, plain | 201; disabled User + activation code + SMTP; auto-login нет |
| Login | POST /api/v1/auth/login | JSON email/password → accessToken, refreshToken | Public, plain | 200; Android принимает только обе nonblank строки; persisted pair |
| Refresh | POST /api/v1/auth/refresh-token | JSON refreshToken → accessToken | Public, plain без Bearer | 200; прежний refresh сохраняется; non-2xx clear pair на Android |
| Logout | POST /api/v1/auth/logout | JSON refreshToken; body response не используется | Access + refresh ownership | 200 revoke одного refresh; Android clear после HTTP response даже non-2xx |
| ROOT resolution | GET /api/v1/folders/root | Нет body → FolderDto.id | Access | 200; lazy ROOT, не CAMERA; null body — client Error |
| CAMERA discovery | GET /api/v1/folders/{id}/children | root ID → FolderDto[]; первый exact folderType=CAMERA | Access, own parent | 200 direct children, не recursive; null body client трактует как empty |
| Checksum pre-check | POST /api/v1/files/checksums/exists | JSON folderId + checksums → existing/missing | Access, own folder | 200 logical partition; existing status-only SYNCED, missing PENDING_UPLOAD |
| Media upload | POST /api/v1/files/upload | Multipart file + optional folderId → FileItemDto.id | Access, own explicit/default folder | 200 новый/existing logical file; Android 2xx id>0 → SYNCED/serverFileId |

Android везде проверяет 2xx, не exact status; frozen Server задаёт register201 и остальные используемые successes200. Protected request может пройти refresh/replay при 401. Public client Bearer не добавляет, поэтому server rejection invalid Bearer на public path не является mismatch этого refresh flow.

## Request/response semantics

Register/login: server email required/Email; password required, 4..20; Android не имеет аналогичной length/email validation до запроса. Register conflict email — 409, validation — 400; login invalid/disabled/banned — 401. Login не возвращает expiresIn; Android его не ожидает. Refresh server invalid token — 401, revoked — 403; logout unknown refresh — 404, ownership — 403.

Folder и logical file IDs — Long/JSON integer. Android использует root id без отдельного positive-ID guard; upload success отдельно требует id>0. Folder response type определяет CAMERA, имя папки для выбора не используется.

Pre-check принимает обязательные own folderId и непустой список 1..500 raw checksum. Каждый hash — 64 hex, без trim; Server приводит к lowercase и dedup с порядком первого появления. Android берёт максимум 500 local rows, группирует lowercase hashes и отправляет не более 500 unique values. Response — полная existing/missing partition нормализованных hashes, **без file IDs**. Запрос читает FileItem в folder/user, не проверяет filesystem, не резервирует данные и не связывает следующий upload с этим snapshot.

Android existing выполняет только status update, оставляя прежний serverFileId/failure metadata без изменения. Missing становится PENDING_UPLOAD. В защитной ветке неуказанный hash остаётся CHECKSUM_READY; no-progress завершает стадию без самостоятельного retry. Это не утверждение, что frozen Server возвращает неполную partition.

Multipart `file` содержит filename из local displayName, local MIME и вновь открытый URI stream. `folderId` при наличии — decimal Long text/plain part; при null часть отсутствует. Android не передаёт authoritative checksum, local ID/URI/status, capturedAt или retry fields. Server hash/size/MIME выводит из полученных bytes. Android использует только positive response ID, не сравнивает checksum/size/folderId/name/dates/metadata.

Empty file — application400; service limit 104857600 bytes (100 MiB), ровно лимит допустим; servlet file/request limits — 110MB. Client max-size guard отсутствует. Missing file part, нечисловой folderId и другие framework cases не имеют гарантированного собственного ErrorResponse; runtime неизвестность — [SYS-OPEN-014](16-system-open-questions.md#sys-open-014).

## CAMERA и upload target

GET ROOT → direct children → exact CAMERA. Отсутствие CAMERA на pre-check переводит hashes в pending без exists: нормальный bootstrap. Lookup error на pre-check завершает стадию с retry и **не допускает upload в этом run**.

Перед upload lookup повторяется. Найденная CAMERA → explicit ID. Absence **или Error второго lookup** → omitted folderId, один выбор части на весь upload run. При omitted folderId Server определяет тип по bytes: IMAGE/VIDEO → lazy CAMERA под ROOT; остальное → lazy FILES. Explicit target может быть любой own folder и не ограничивает MIME. Local image/ MIME guard не удостоверяет detected IMAGE. Условное расхождение target — `SYS-MISMATCH-002`.

## Duplicate и 409

| Condition | Server outcome | Android effect |
| --- | --- | --- |
| Те же user/folder/checksum, любое входное имя | 200 старый FileItemDto, прежние name/metadata/dates; без bytes check/repair | Positive id принят как успех |
| Тот же checksum в другой folder | 200 новый FileItem + StoredObject + независимые bytes | Новый positive ID сохранён |
| Новые bytes, занятое logical name в CAMERA | 200 новая запись | Успех |
| Новые bytes, занятое name вне CAMERA | 409 CONFLICT | FAILED HTTP_409, без auto restore |
| DB uniqueness race upload | Cleanup собственного final, reread matching logical record; existing200 либо ошибка | Обычная обработка response |

Same-folder checksum duplicate **не 409**. Для Android name409 особенно значим при omitted target и default FILES. Повтор upload условно возвращает ту же identity только при сохранении principal/folder/bytes и logical record; нет Idempotency-Key, upload session и bytes repair.

## Errors affecting Android

Controlled Server errors имеют id/code/message/fieldErrors, но единый body не обещан для всех IO/framework failures. Android использует fieldErrors → message → code → HTTP N, с raw fallback для malformed JSON; id/code не сохраняются как structured history. Upload 400/409/413 — per-file FAILED; прочие HTTP, IOException или invalid success ID — PENDING_UPLOAD + stop + worker retry. Folder/pre-check errors — stage retry. Server Retry-After/retryable contract не задаёт; Android собственного общего retry limit не имеет.

## Server-only capability groups

Из 36 mappings 26 не вызываются Android: 19 с реализацией и 7 STUB. Реализованные группы: activation confirm; logout-all/others; password reset request/JSON confirm; profile read; USER-folder create/rename/move/delete; file list/get/download/rename/move/copy/delete/full checksum list; legacy upload alias. STUB: activation/reset resend, reset page GET/POST, profile PATCH, settings GET/PATCH.

Это inventory границы, не новый полный controller catalog. Android local Files monitor не использует remote file listing; Auth email из JWT не является profile GET. Отсутствие использования не создаёт дополнительный mismatch.


## Источники

[S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A13](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
