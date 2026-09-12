# System mismatch registry

Четыре известных Android integration mismatch переосмыслены на system boundary и сохранены one-to-one. Они являются согласованно описанными сторонами поведения, **не противоречиями двух frozen specifications** и не четырьмя воспроизведёнными инцидентами. Routes/basic shapes всех десяти operations согласованы.

## SYS-MISMATCH-001

**Sources:**

- Android `INT-AND-001`: [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).
- Server SRV-AUTH-004, SRV-SYNC-002: [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md).

**Subject:** Unscoped local state и server/account identity.

**Android As-Is:** Android DB использует mediaStoreId, без account/server/folder/device namespace; logout/URL change её не очищают. Pair также не связана с host.

**Server As-Is:** Server авторизует principal и ограничивает FileItem/Folder user scope; checksum query — user+folder. Числовой server ID имеет смысл в контексте своей DB.

**System consequence:** Старый SYNCED может скрывать отсутствие backup в текущем account/host. Pending и credentials идут с текущим контекстом, не связанным с local history.

**Condition:** Смена account/host при сохранённой local DB/pair; не утверждается инцидент в каждой установке.

**Current classification:** CONFIRMED_SEMANTIC_MISMATCH; scope disagreement, не route/DTO error.

## SYS-MISMATCH-002

**Sources:**

- Android `INT-AND-002`: [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).
- Server SRV-FILE-002, SRV-FOLDER-001/002/003: [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md).

**Subject:** CAMERA pre-check и fallback upload target.

**Android As-Is:** Pre-check использует CAMERA. Перед upload lookup повторяется; Error или absence дают omitted folderId. Ответный target не сверяется.

**Server As-Is:** Explicit own folder разрешена; omitted target определяется по принятым bytes: IMAGE/VIDEO→CAMERA, остальное→FILES. Lazy ROOT не создаёт CAMERA.

**System consequence:** Проверка existing могла относиться к CAMERA, а принятый объект — к FILES; client отмечает SYNCED. Возможен name409 вне CAMERA.

**Condition:** Successful CAMERA pre-check, затем Error второго lookup, upload прошёл local image/ guard, но Server detection не IMAGE/VIDEO. Обычный первый IMAGE bootstrap без CAMERA согласован.

**Current classification:** CONFIRMED_SEMANTIC_MISMATCH; условное различие target, не malformed multipart.

## SYS-MISMATCH-003

**Sources:**

- Android `INT-AND-003`: [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).
- Server SRV-MEDIA-001, SRV-FILE-001: [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md).

**Subject:** Checksum read и upload bytes race.

**Android As-Is:** SHA-256 вычисляется отдельным чтением content URI. Upload читает URI снова, принимает id>0 и игнорирует response checksum/size; immutable snapshot нет.

**Server As-Is:** Hash и size вычисляются из фактически принятых bytes. Client checksum upload authority не является.

**System consequence:** Local hash/pre-check evidence и подтверждённый remote object могут описывать разные bytes, несмотря на SYNCED с ID.

**Condition:** Изменение bytes между hashing/pre-check и реальным upload read; стабильное содержимое само по себе mismatch не создаёт.

**Current classification:** CONFIRMED_SEMANTIC_MISMATCH; несогласованная content identity при указанной гонке.

## SYS-MISMATCH-004

**Sources:**

- Android `INT-AND-004`: [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).
- Server SRV-AUTH-005: [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md).

**Subject:** Refresh 5xx и очистка client session.

**Android As-Is:** Любой non-2xx refresh очищает encrypted token pair; IOException сам authenticator pair не очищает.

**Server As-Is:** Invalid/unknown/expired refresh имеет401, revoked403. Successful refresh не потребляет/не ротирует token; server error не означает revoke.

**System consequence:** При временной5xx Android теряет local session без доказанной недействительности refresh. Background reconcile не вызывается автоматически.

**Condition:** Protected401 привёл к plain refresh, который вернул5xx при потенциально ещё valid server refresh.

**Current classification:** CONFIRMED_SEMANTIC_MISMATCH; session meaning, не нарушение назначенной Server client retry policy.


## Reconciliation outcome

Дополнительных cross-component mismatches и противоречий между frozen inputs при этом сопоставлении не обнаружено. Вывод ограничен этими источниками; live integration не проверялась.

Не повышены до mismatch: SYNCED без ID после existing; logical existing без physical proof; неиспользуемый Server API; default CAMERA bootstrap; ignored server metadata; отсутствующая общая retry policy. Это contract limits, risks или OPEN в зависимости от роли факта. Данные о framework errors, MediaStore redaction/visibility и deployed endpoint остаются unknown, не объявлены дополнительными подтверждёнными incompatibilities.

Registry не содержит fixes. Негативные последствия и неназначенные решения вынесены в [15](15-system-risks.md) и [16](16-system-open-questions.md).
