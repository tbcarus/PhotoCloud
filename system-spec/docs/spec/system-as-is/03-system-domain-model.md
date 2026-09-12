# Системная доменная модель

| Entity | Identity | Owner | Persistence | Component authority | Relationships |
| --- | --- | --- | --- | --- | --- |
| User / principal | Server User.id; unique email, JWT sub=email | Серверная учётная запись пользователя | PostgreSQL User; principal разрешается для request | Server | Владеет Folder, FileItem, StoredObject; refresh связан email |
| Device-local media | MediaStore _ID и content URI | Device/OS visibility и пользователь; cloud ownership не задаёт | MediaStore и внешние device bytes | Android описывает доступ; OS предоставляет объект | Индексируется local record; поток читается независимо на hash/upload |
| Android local media record | mediaStoreId >0 как local DB PK | Установка приложения, без account/server namespace | Android DB | Android | URI, checksum, status, optional serverFileId; несколько local IDs могут иметь один hash |
| Server Folder | Folder.id | Server User | PostgreSQL | Server | Parent ID; ROOT/CAMERA/FILES/USER; содержит logical FileItem |
| Server logical File | FileItem.id | Server User, в normal flow owner папки совпадает | PostgreSQL | Server | Folder + StoredObject; checksum; logical name/dates/optional metadata |
| Server Stored Object | StoredObject.id | Server User как владелец bytes | PostgreSQL physical metadata | Server | Relative storage path/name, server hash/size/MIME/type; referenced by FileItem |
| Physical bytes | Resolved server storage path/name | Server storage под user scope | Server filesystem | Server storage, отдельно от DB | Может существовать без DB reference либо отсутствовать при наличии DB rows |
| Access Session | Подписанная ACCESS JWT, sub/exp; отдельного session ID/row нет | Principal пользователя | Android encrypted pair; сервер access не persisted | Server validation, Android custody | Login/refresh выдаёт access; request Bearer |
| Refresh Session | JWT string с jti; отдельный DB row ID | Пользователь по email/userName | Android encrypted pair + full JWT/revoke row PostgreSQL | Server validity; Android custody | Login создаёт row; refresh читает её, logout меняет revoked |
| Email activation/recovery code | UUID code | User, назначение ACTIVATE/PASSWORD_RESET | PostgreSQL code/type/used/time | Server | SMTP link; activation открывает возможность login |

**Access Session** и **Refresh Session** здесь — системные понятия жизненного цикла доступа. Они не вводят Device/SyncSession entities. Refresh действительно persisted на обеих сторонах, access — только у клиента и в предъявляемом JWT.

## Пять разных объектов в пути фото

Device media → Android local record → Server logical FileItem → StoredObject metadata → physical bytes. Это не пять реплик одной записи: local record описывает исходный URI и очередь; FileItem — remote placement/identity; StoredObject — сведения о remote bytes; filesystem содержит содержимое.

Обычный новый upload создаёт FileItem + StoredObject + bytes. Same-folder duplicate возвращает прежний FileItem. Cross-folder upload/copy создаёт отдельные объекты и независимые bytes. StoredObject может иметь несколько logical references по модели данных, но active upload/copy не создают sharing/cross-owner links. Из допустимой кардинальности не выводится sharing API.

Folder tree логическая: rename/move FileItem не меняют physical path. Copy — новая identity, а не новая версия прежнего ID. Hard delete не оставляет tombstone; поле deletedAt не является действующей корзиной. Optional image metadata может отсутствовать при успешном upload.


## Источники

[S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
