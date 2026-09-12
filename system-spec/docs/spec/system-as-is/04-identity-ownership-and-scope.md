# Identity, ownership и scope

## Identity matrix

| Identity / ID | Creator | Scope | Uniqueness | Persistence | Relationship |
| --- | --- | --- | --- | --- | --- |
| Device media _ID/content URI | MediaStore; Android строит URI по _ID | External Images collection, доступная установке | Android принимает positive _ID; глобальная cross-device/volume уникальность не заявлена | MediaStore; URI/ID копируются в local DB | Один _ID сопоставляется между scan; name/path не identity |
| Android local DB mediaStoreId | Android использует _ID, не генерирует новый ключ | Установка приложения | PK local record; отдельного device/volume/account/host ID нет | Android DB | MediaStore metadata + queue + optional serverFileId |
| Android checksum | Android SHA-256 URI read | Содержимое конкретного чтения | Не unique local PK; два media ID могут совпасть по hash | Local record, nullable | Запрос existing; не передаётся как upload authority |
| Server checksum | Server SHA-256 принятого stream | Содержимое server upload | FileItem unique в комбинации user/folder/checksum; StoredObject checksum не unique | StoredObject и копия FileItem | Hex64 lowercase; не logical ID и не global dedup key |
| serverFileId | Android копирует response FileItem.id>0 | Исторически тот host/principal, который ответил; namespace не сохранён | Нет отдельной local uniqueness guarantee | Nullable local field | Успех upload записывает; existing status-only не заполняет/не очищает |
| Server FileItem.id | Server DB | Конкретная server DB; lookup owner-scoped | DB PK | PostgreSQL | Все file endpoints используют этот logical ID, не object ID |
| StoredObject.id | Server DB | Конкретная server DB | DB PK; checksum/path не unique constraints | PostgreSQL | Метаданные physical object, не wire file ID |
| Folder.id | Server DB при lazy/default/USER creation | Конкретная DB; пользовательская hierarchy | DB PK; ROOT unique per user, siblings names case-insensitive unique | PostgreSQL | Explicit upload target или CAMERA/FILES default |
| User.id / email | Server registration/DB | Конкретная server DB | PK; email unique, registration lowercase | PostgreSQL | Principal/ownership; JWT subject=email |
| Refresh JWT / row ID | Server login/DB | Server signing context и userName | JWT jti; row PK, token column без unique constraint | Server DB и Android encrypted pair | Не содержит Android local DB namespace |

## Scope model

| Scope | Фактическое значение |
| --- | --- |
| Device | MediaStore виден через OS grants; сервер не регистрирует device, клиент не хранит отдельный device namespace |
| Android DB | Одна база установки; scope не меняется при login/logout/URL change |
| Account | Одна текущая token pair; ранее накопленные rows не привязаны к account |
| Server host | URL из IPv4/port settings, mutable runtime value; tokens/local rows не привязаны к host |
| User | Server извлекает principal и проверяет FileItem/Folder id + userId; чужой и missing дают 404 |
| Folder | Pre-check и duplicate проверяют конкретную own folder; local relativePath не является folder mapping |
| Checksum dedup | Logical tuple user + folder + SHA-256; не across hosts/users/folders, не глобальная physical dedup |

Server IDs — authority в соответствующей DB и owner context; совпадающее числовое значение на другом host не удостоверяет тот же объект. Local DB не сохраняет этот context и не хранит folderId. Это системное следствие `INT-AND-001` → [SYS-MISMATCH-001](14-system-mismatches.md#sys-mismatch-001), а не основание назначить новую схему.

После logout или успешной смены URL старые SYNCED остаются; обычная очередь не проверяет их на новом endpoint/account. Pending обрабатываются текущей парой; protected requests могут направить старые credentials новому host. Refresh читает URL в момент refresh, который может отличаться от host исходного request.

## Source of truth

| Concern | Current authority | Предел |
| --- | --- | --- |
| Device media existence/visibility | MediaStore / OS provider | Отсутствие из snapshot не доказывает physical deletion |
| Android sync state | Android local DB | Это persisted client belief, не Server state |
| User authentication / remote ownership | Server validation и User/refresh records | Local presence tokens и JWT display не подтверждают действительность |
| Folder hierarchy | Server DB | Не device folders и не physical storage directories |
| Logical remote media | Server DB FileItem/StoredObject relationships | Не проверяет bytes |
| Physical remote bytes | Server filesystem | Наличие path не удостоверяет актуальный checksum без проверки |
| Uploaded hash/size | Server вычисление по принятому stream и сохранённые metadata | Не доказательство неизменности subsequent bytes |
| Client belief of sync | Local status/optional serverFileId | Не remote verification и не bound account/host identity |

Authority означает, где устанавливается конкретный факт, а не автоматическую согласованность authorities. Полная consistency boundary и текущие non-guarantees — [12](12-data-consistency-and-durability.md).


## Источники

[S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
