# Ответственность компонентов

| Responsibility | Android | Server | Boundary |
| --- | --- | --- | --- |
| Authentication UI | Register/login, Test Auth, Logout; форма и local session indicator | HTTP auth outcomes | UI presence tokens не удостоверяет principal |
| Credentials | Ввод и передача email/password; password не persisted в preferences | Проверка credentials, BCrypt hash, enabled/banned при login | Plain auth requests; пароль остаётся в памяти формы |
| Token persistence | Одна encrypted access/refresh pair | Полный refresh JWT + revoke state; access rows нет | Нет общего account/host namespace клиента |
| Token validation | Отправка current Bearer; refresh после 401 | Signature/type/expiry, principal и ownership | Local JWT sub — отображение, не authorization |
| Media discovery | Images external/DCIM, permissions, snapshot | Device inventory не получает | ОС — authority доступных media |
| Checksum | SHA-256 отдельного URI read, локально persisted | SHA-256 принятого upload stream | Равенство двух reads не закреплено |
| Local state | Очередь/status/failures/optional server ID | Не принимает Android status/URI/local ID | Нет server sync-session record |
| Folder hierarchy | ROOT/direct children lookup, CAMERA selection | Владелец иерархии, lazy ROOT/CAMERA/FILES, USER mutations | Local path не зеркалируется в remote folders |
| Logical file identity | Сохраняет positive upload ID; existing ID не получает | FileItem ID, user/folder/checksum uniqueness | Local ID и logical remote ID различны |
| Physical bytes | Читает original URI, не создаёт private snapshot | Temp/final bytes, StoredObject, storage ownership | DB record не физическое доказательство |
| Dedup | Pre-check hashes, duplicate200 принимается | Same-user/folder/checksum logical dedup | Нет cross-folder physical dedup |
| Retry | Per-file checksum eligibility, HTTP outcomes, WorkManager retry | Ошибки, conditional duplicate reread, best-effort compensation | Общей retry policy/Idempotency-Key нет |
| Background execution | Observer/periodic/one-time scheduling | Прикладных фоновых jobs нет | OS определяет фактический запуск |
| Metadata | Local name/path/size/times/MIME | Detected MIME/size, filename normalization, EXIF/GPS best effort | Response metadata игнорируются клиентом |
| Deletion | Stale local row DELETE; оригинал не удаляет | Hard delete API, physical cleanup после DB commit | Android remote DELETE не вызывает |
| Rename/move/copy | Обновляет local metadata при scan; remote mutations не вызывает | Logical rename/move; copy создаёт независимые bytes | Reverse propagation отсутствует |
| Remote listing | Только direct folder children для CAMERA | File listing/get/full checksums/download | Files monitor отображает local DB |
| Synchronization | One-way discover/hash/pre-check/upload | Folder checksum query + отдельные операции хранения | Нет reverse reconciliation/delta/tombstones |

Server-only implemented groups: confirmation, logout-all/others, JSON password reset, profile read, folder CRUD, file browse/get/download/mutations/full checksum list, legacy upload alias. Server STUB groups: resend, reset page, profile edit/settings. Android Settings/Profile — собственные UI заглушки, не вызывающие эти endpoints. Неиспользование возможностей не классифицируется как mismatch; системный статус определяется доступностью полного рассматриваемого flow в [13](13-system-capability-matrix.md).


## Источники

[S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md); [A01](../../../../PhotoCloudClient/docs/spec/android-as-is/01-system-boundary.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A16](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
