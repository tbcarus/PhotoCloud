# Системная модель PhotoCloud

Это каноническое определение системных сущностей, полей, типов и связей. Сценарии описывают операции над этой моделью. Имена RemoteFile, LocalMedia, DeviceMedia, PhysicalBytes и TokenPair обозначают системные представления существующих объектов, а не новые классы или хранилища.

Модель описывает текущий As-Is. В ней нет сущностей Sharing, ACL, Device, Album, UserSettings, UploadSession или SyncSession: таких прикладных моделей сейчас нет.

<a id="system-types"></a>

## Типы и идентификаторы

| Тип | Значение |
| --- | --- |
| `String`, `Boolean` | Строка и логическое значение |
| `Integer`, `Number` | Целое число и число с возможной дробной частью |
| `DateTime` | Дата и время; Server передаёт локальное время без offset, Android хранит время media/ошибок в epoch milliseconds |
| `UserId`, `FolderId`, `FileId`, `StoredObjectId`, `MetadataId`, `SessionId`, `EmailRequestId` | Идентификаторы записей на конкретном Server; одинаковые числа на разных Server не связывают объекты |
| `MediaId` | Положительный MediaStore ID в локальном индексе установки; отдельного volume/device namespace нет |
| `Checksum` | SHA-256 содержимого конкретного чтения: 64 шестнадцатеричных символа в нижнем регистре |
| `List<T>`, `Set<T>`, `Map<K,V>` | Список, множество и отображение значений |
| `T?` | Значение может отсутствовать; это не отдельное состояние обработки |

В сценариях hash означает checksum, а mtime — LocalMedia.lastModified. Checksum не является ID локального или удалённого файла. Несколько LocalMedia, RemoteFile и StoredObject могут иметь одинаковый checksum. Для операции duplicate существенна тройка владелец + папка + checksum, описанная в [BACKUP-DUPLICATE](04-backup-and-sync.md#backup-duplicate).

<a id="user"></a>

## User

Учётная запись на конкретном Server.

**Компонентное представление:** Server — User; Android не хранит полную сущность пользователя и не запрашивает профиль. Email на Auth берётся из access-токена.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | UserId | Идентификатор аккаунта |
| `email` | String | Уникальный email и имя входа; регистрация приводит его к нижнему регистру |
| `passwordHash` | String | BCrypt-хеш пароля; соответствует Server User.password. Исходный пароль в User не хранится |
| `displayName` | String? | Отображаемое имя; регистрация его не принимает |
| `enabled` | Boolean | Аккаунт активирован; исходно false |
| `banned` | Boolean | Запрет нового входа; исходно false |
| `roles` | `Set<Role>` | Роли аккаунта; регистрация назначает USER |
| `createdAt` | DateTime? | Время создания |
| `lastUpdate` | DateTime? | Время последнего обновления, в том числе при login; это фактическое имя поля, отдельного User.updatedAt нет |
| `lastLoginAt` | DateTime? | Время последнего успешного входа |

### Связи

User владеет Folder, RemoteFile, StoredObject и EmailRequest. RefreshSession связана с ним по email, без прямой ссылки на User. Текущие роли пользователя определяют полномочия защищённого запроса; влияние enabled/banned на уже выданные токены описывает [AUTH-BLOCK](02-account-and-auth.md#auth-block).

<a id="role"></a>

## Role

Значения: `USER`, `ADMIN`. ADMIN не предоставляет обход проверки владельца или отдельную действующую административную операцию.

<a id="email-request"></a>

## EmailRequest

Сохранённый механизм активации или сброса пароля.

**Компонентное представление:** Server — EmailRequest; Android соответствующую запись не хранит и не выполняет подтверждение.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | EmailRequestId | Идентификатор записи |
| `code` | String | Уникальная строка UUID, передаваемая в ссылке/подтверждении |
| `type` | EmailRequestType | Назначение кода |
| `used` | Boolean | Отметка использования; исходно false |
| `user` | User | Аккаунт, к которому относится код |
| `createdAt` | DateTime | Время создания, от которого отсчитываются 3 дня действия |

### Связи и срок

EmailRequest относится к одному User; у пользователя может быть несколько кодов. Отдельного поля expiresAt нет. Проверка срока сравнивает createdAt + 3 дня с текущим временем; назначение и used проверяются отдельно. Использование кодов описано в [AUTH-ACT](02-account-and-auth.md#auth-act) и [AUTH-RESET](02-account-and-auth.md#auth-reset).

<a id="email-request-type"></a>

## EmailRequestType

`ACTIVATE` активирует аккаунт; `PASSWORD_RESET` меняет пароль после подтверждения. Одинаковый формат code не делает назначения взаимозаменяемыми.

<a id="refresh-session"></a>

## RefreshSession

Сохранённая серверная refresh-сессия.

**Компонентное представление:** Server — RefreshToken; Android хранит её token в TokenPair.refreshToken, без полной серверной записи.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | SessionId | Идентификатор серверной записи, не file ID и не JWT jti |
| `token` | String | Полный подписанный refresh JWT; поиск выполняется по этой строке |
| `userName` | String | Email пользователя; связь с аккаунтом по имени |
| `expires` | DateTime | Копия exp JWT в серверном локальном времени; проверка срока использует подписанный exp |
| `revoked` | Boolean | Отметка отзыва; исходно false |
| `revokedAt` | DateTime? | Время одиночного logout; массовый отзыв это поле не заполняет |

### Связи

Login создаёт новую RefreshSession для User. Android сохраняет её token вместе с access. Refresh не меняет запись, не ротирует и не продлевает refresh; истечение срока само не удаляет запись. Уникальность сохранённого token не установлена. Операции — [AUTH-LOGIN](02-account-and-auth.md#auth-login), [AUTH-REFRESH](02-account-and-auth.md#auth-refresh), [AUTH-LOGOUT](02-account-and-auth.md#auth-logout).

<a id="access-token"></a>

## AccessToken и RefreshTokenValue

Подписанные значения доступа; AccessToken соответствует понятию access session, но отдельной серверной строки или session ID для access нет. RefreshTokenValue — значение RefreshSession.token, а не ещё одна сохраняемая сущность.

**Компонентное представление:** Server выдаёт JWT; Android хранит строки и декодирует subject для отображения.

### Поля значения JWT

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `sub` | String | Email пользователя, также называемый subject |
| `roles` | `Set<Role>` | Роли при выдаче; access-проверка использует текущие User.roles |
| `token_type` | TokenType | ACCESS или REFRESH |
| `iat` | DateTime | Время выдачи |
| `exp` | DateTime | Подписанный срок окончания: access — 20 минут, refresh — 7 дней |
| `jti` | String | UUID только для refresh; у access такого claim нет |

### Связи

AccessToken предъявляется от имени User в Bearer-запросе. RefreshTokenValue связан с RefreshSession по полной строке. Уже выданный access не отзывается через logout; остальные условия доступа описаны в account/auth.

<a id="token-type"></a>

## TokenType

`ACCESS` используется для защищённых операций; `REFRESH` — для получения нового access. Типы не взаимозаменяемы.

<a id="token-pair"></a>

## TokenPair

Одна локальная пара токенов Android.

**Компонентное представление:** Android — Tokens в зашифрованном хранилище и памяти; Server возвращает пару при login, а при refresh только новый access.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `accessToken` | String | Строка AccessToken |
| `refreshToken` | String | Строка RefreshTokenValue |

### Связи

Пара целиком может отсутствовать. Её наличие определяет локальное состояние входа, но не подтверждает валидность токенов. Полей accountId/serverId/host у пары нет; последствия — [SYSTEM-SCOPE](07-system-rules-and-recovery.md#system-scope). Login принимает только две непустые строки; refresh отдельно не отклоняет пустую строку access.

<a id="device-media"></a>

## DeviceMedia

Изображение, видимое Android через MediaStore, и его читаемый поток. Это объект устройства, не запись очереди и не удалённый файл.

**Компонентное представление:** Android читает MediaStore Images; нормализованный снимок MediaStoreImage используется одним scan. Server объект устройства не хранит.

### Данные доступного изображения

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `mediaStoreId` | MediaId | MediaStore _ID, по которому строится локальная identity |
| `uri` | String | Content URI Images с ID; адрес открытия потока |
| `displayName` | String | Имя из DISPLAY_NAME; при отсутствии используется image_ID |
| `relativePath` | String? | RELATIVE_PATH; пустое значение становится null, это не Server Folder |
| `mimeType` | String | MIME_TYPE; при отсутствии image/* |
| `size` | Integer | Заявленный SIZE в байтах; неизвестный становится 0 |
| `createdAt` | DateTime | Нормализованное время: положительное DATE_TAKEN, затем DATE_MODIFIED, DATE_ADDED, иначе now |
| `lastModified` | DateTime | Положительное DATE_MODIFIED, иначе createdAt |

DATE_TAKEN поступает в миллисекундах, DATE_MODIFIED/DATE_ADDED — в секундах и преобразуются в миллисекунды. Это исходные значения ОС для вычисления двух дат, а не дополнительные поля LocalMedia. Положительность size сканер не проверяет.

### Связи и байты устройства

DeviceMedia индексируется в LocalMedia по mediaStoreId. ОС, provider и разрешения определяют видимость и доступный поток. URI не закрепляет неизменяемую версию bytes: checksum и upload открывают его независимо. Отдельная приватная копия потока и модель selected-photo/location access отсутствуют.

<a id="local-media"></a>

## LocalMedia

Сохранённая Android запись изображения и его обработки.

**Компонентное представление:** Android — MediaFile; Server не хранит эту очередь.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `mediaStoreId` | MediaId | Первичный локальный ID из DeviceMedia |
| `serverFileId` | FileId? | Положительный RemoteFile.id успешного upload; existing не получает и не меняет ID |
| `uri` | String | Текущий адрес потока из DeviceMedia |
| `displayName` | String | Локальное имя для списка и multipart filename |
| `relativePath` | String? | Локальный путь из DeviceMedia |
| `mimeType` | String | Локальный MIME для проверки image/ и Content-Type части |
| `size` | Integer | Заявленный локальный размер; участвует в обнаружении изменения и длине upload |
| `createdAt` | DateTime | Локальное время изображения из DeviceMedia; определяет порядок очереди/списка |
| `lastModified` | DateTime | Локальное время изменения; участвует в обнаружении изменения |
| `checksum` | Checksum? | SHA-256 прочитанного Android потока; после discovery/изменения отсутствует |
| `status` | SyncState | Сохранённое состояние обработки; исходно PENDING |
| `failureReason` | FailureReason? | Классификация последней сохраняемой ошибки файла |
| `retryCount` | Integer | Число накопленных markFailed, исходно 0; не число HTTP/worker попыток |
| `lastFailureAt` | DateTime? | Время последнего markFailed по часам устройства; определяет cooldown |

### Связи и границы

LocalMedia ссылается на DeviceMedia через mediaStoreId/uri. serverFileId может связывать её с RemoteFile, но не сопровождается account/server/folder namespace. Два local ID с одинаковыми bytes остаются разными записями. В LocalMedia нет folderId, server/account identity, remote filename/MIME/size/capturedAt/uploadedAt/metadata, upload session или byte progress.

Изменение size/lastModified сбрасывает checksum, serverFileId и сведения об ошибке. Отсутствие ID в успешном scan удаляет запись. Переходы и исключения — [BACKUP-DISCOVER](04-backup-and-sync.md#backup-discover), [BACKUP-CHANGE](04-backup-and-sync.md#backup-change), [BACKUP-RETRY](04-backup-and-sync.md#backup-retry).

<a id="sync-state"></a>

## SyncState

**Компонентное представление:** Android — MediaFileStatus, поле LocalMedia.status.

| Значение | Смысл обычного последовательного пути |
| --- | --- |
| `PENDING` | Ожидает вычисления checksum |
| `HASHING` | Сохранённый признак начатого вычисления checksum |
| `CHECKSUM_READY` | Checksum готова для pre-check |
| `PENDING_UPLOAD` | Ожидает отправки |
| `UPLOADING` | Сохранённый признак начатой отправки |
| `SYNCED` | Локальный вывод об успехе по критерию ниже |
| `FAILED` | Ошибка файла; автоматический возврат ограничен CHECKSUM_IO |
| `LOCAL_DELETED` | Значение существует, но активный путь его не записывает |

HASHING/UPLOADING могут сохраняться без работающей задачи. Из любого состояния возможен сброс при изменении size/lastModified или удаление строки при исчезновении из успешного снимка. Это не глобальная атомарная машина состояний: поздние записи scan/restore/hash/pre-check/upload могут нарушать обычные сочетания status/checksum/ID и сохранять сведения об ошибке при status, отличном от FAILED.

<a id="synced"></a>

### Значение SYNCED

SYNCED — сохранённый вывод Android после logical existing либо upload 2xx с положительным FileId. Existing меняет только status: serverFileId остаётся прежним, в обычном новом пути — null. Upload при обычном завершении записывает ID.

Состояние не гарантирует читаемость и неизменность физических байтов Server, равенство local checksum отправленному содержимому, актуальность remote ID/папки, наличие оригинала сейчас или backup в текущем account/host. Обычный SYNCED повторно с Server не сверяется. Общий SUCCESS не означает, что все строки SYNCED. Нерешённый продуктовый смысл подтверждения остаётся рядом с [BACKUP-NEW](04-backup-and-sync.md#backup-new).

<a id="failure-reason"></a>

## FailureReason

Классификация LocalMedia.failureReason; исходный HTTP body здесь не хранится.

| Значение | Когда записывается |
| --- | --- |
| `CHECKSUM_IO` | IOException при hashing после отдельной обработки отсутствующего файла |
| `FILE_NOT_FOUND` | Нет потока/файла при hashing или проверке открытия перед upload |
| `PERMISSION` | Отказ доступа при hashing или проверке открытия |
| `NOT_IMAGE` | Локальный MIME не начинается с image/ перед upload |
| `HTTP_400`, `HTTP_409`, `HTTP_413` | Соответствующий окончательный HTTP-ответ upload |
| `UNKNOWN` | Прочая ошибка отдельного файла при hashing |

Автоматически восстанавливается только CHECKSUM_IO при retryCount < 3 и достаточном cooldown. Исторический FAILED может не иметь причины. Временная ошибка upload не записывает FailureReason и не увеличивает retryCount; условия и сброс полей — [BACKUP-RETRY](04-backup-and-sync.md#backup-retry).

<a id="folder"></a>

## Folder

Логическая папка Server, независимо от каталогов устройства и физического хранилища.

**Компонентное представление:** Server — Folder; Android использует FolderDto для поиска ROOT/CAMERA, не сохраняет дерево папок.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | FolderId | Идентификатор папки |
| `user` | User | Владелец |
| `parent` | Folder? | Родитель; создаваемая ROOT не имеет родителя |
| `name` | String | Логическое имя |
| `folderType` | FolderType | Категория, неизменяемая через API |
| `createdAt` | DateTime | Время создания |
| `updatedAt` | DateTime | Время последнего обновления |

### Связи

Folder принадлежит User, содержит RemoteFile и имеет прямых потомков Folder. Отдельного сохраняемого поля children нет. Обычное дерево создаётся в пределах пользователя; структура хранения сама не гарантирует совпадение владельца parent или отсутствие длинных циклов. Ограничения имён, системных папок и перемещений находятся в [операциях папок](05-file-library.md#folders-browse).

<a id="folder-type"></a>

## FolderType

| Значение | Смысл |
| --- | --- |
| `ROOT` | Единственная корневая папка пользователя, создаётся лениво с именем root |
| `CAMERA` | Системная папка Camera для default routing IMAGE/VIDEO |
| `FILES` | Системная папка Files для default routing остальных типов |
| `USER` | Пользовательская папка, создаваемая под ROOT/USER |

CAMERA и FILES — системные конечные папки. ROOT/CAMERA/FILES нельзя переименовать, переместить или удалить через текущий API. Создание системных папок при upload и выбор Android описывает [BACKUP-TARGET](04-backup-and-sync.md#backup-target).

<a id="remote-file"></a>

## RemoteFile

Логическая запись файла в папке пользователя.

**Компонентное представление:** Server — FileItem; Android читает FileItemDto после upload и сохраняет только положительный id в LocalMedia.serverFileId.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | FileId | Идентификатор для всех file operations |
| `user` | User | Владелец логической записи |
| `folder` | Folder | Текущее логическое размещение |
| `storedObject` | StoredObject | Сведения о физическом объекте |
| `checksum` | Checksum | Сохранённая копия StoredObject.checksum |
| `originalName` | String | Логическое имя; API возвращает его как originalFilename |
| `capturedAt` | DateTime | EXIF-время съёмки либо uploadedAt |
| `uploadedAt` | DateTime | Время создания удалённой записи; при copy новое |
| `deletedAt` | DateTime? | Поле существует, но текущий API его не заполняет и удаляет запись физически |
| `metadata` | FileMetadata? | Доступные сведения об изображении |

### Связи

User → Folder → RemoteFile → StoredObject → PhysicalBytes. У RemoteFile не более одной FileMetadata. Обычный upload/copy создаёт записи одного владельца, но сама модель допускает несовпадение владельцев связанных записей и несколько ссылок на StoredObject. Из этого не следует sharing.

Отдельных uploaded/status или version chain у RemoteFile нет. Rename меняет originalName, move — folder, copy создаёт новую identity. Детали конфликтов и удаления остаются в [библиотеке файлов](05-file-library.md).

<a id="stored-object"></a>

## StoredObject

Сохранённые сведения о физическом объекте; наличие записи не удостоверяет наличие bytes.

**Компонентное представление:** Server — StoredObject; Android отдельное представление не хранит.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | StoredObjectId | Идентификатор физического объекта в каталоге, не FileId |
| `user` | User | Владелец физических bytes |
| `filePath` | String | Относительный каталог размещения |
| `filename` | String | Физическое имя |
| `fileExtension` | String | Расширение физического файла; может быть пустой строкой |
| `checksum` | Checksum | SHA-256 прочитанного Server содержимого |
| `size` | Integer | Фактически прочитанный размер в байтах |
| `detectedMimeType` | String | MIME, определённый по bytes |
| `fileType` | FileType | Классификация определённого MIME |
| `createdAt` | DateTime | Время создания записи |

### Связи

StoredObject принадлежит User, на него могут ссылаться несколько RemoteFile. Обычные upload/copy создают отдельный объект и независимые bytes. Checksum, сочетание пути и имени не являются гарантированными уникальными ключами этой записи. Прикладного обновления физических полей, refcount или сборки неиспользуемых объектов нет. Ветки owner/non-owner удаления описаны в [FILES-DELETE](05-file-library.md#files-delete).

<a id="physical-bytes"></a>

## PhysicalBytes

Содержимое в файловой системе Server, отдельно от StoredObject.

**Компонентное представление:** Server filesystem; отдельной DB entity нет. Android не имеет прямого доступа к этому хранилищу.

### Системные свойства

| Свойство | Системный тип | Значение |
| --- | --- | --- |
| Размещение | String | Разрешённый путь от настроенного storage root с использованием StoredObject.filePath/filename |
| Содержимое | Поток байтов | Текущие фактические bytes по этому пути |
| Доступность | Наличие и читаемость | Проверяется при обращении к файловой системе, не сохраняется как флаг StoredObject |

### Связи

StoredObject описывает размещение PhysicalBytes. Bytes могут остаться без записей, а записи — без читаемых bytes; общего атомарного сохранения DB/filesystem нет. Эти свойства не являются новыми полями Server entity. Последствия и recovery — [SYSTEM-STORAGE](07-system-rules-and-recovery.md#system-storage).

<a id="file-metadata"></a>

## FileMetadata

Необязательные сведения, извлечённые Server из изображения.

**Компонентное представление:** Server — FileMetadata; Android получает nullable metadata в FileItemDto, но не интерпретирует и не сохраняет её.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | MetadataId | Идентификатор записи |
| `fileItem` | RemoteFile | Единственный логический файл этой записи metadata |
| `width` | Integer? | Ширина JPEG/PNG |
| `height` | Integer? | Высота JPEG/PNG |
| `durationSec` | Integer? | Длительность; текущий extractor не заполняет |
| `cameraMake` | String? | Производитель камеры из EXIF |
| `cameraModel` | String? | Модель камеры из EXIF |
| `lensModel` | String? | Модель объектива |
| `exposureTime` | String? | Строковое EXIF-значение выдержки |
| `fNumber` | Number? | Диафрагменное число |
| `iso` | Integer? | Светочувствительность |
| `focalLength` | Number? | Фокусное расстояние |
| `latitude` | Number? | Широта GPS |
| `longitude` | Number? | Долгота GPS |

### Связи

Для fNumber/focalLength сохраняется до 4 знаков после запятой, для latitude/longitude — до 7; фиксированный текстовый масштаб чисел в JSON не обещается.

FileMetadata относится к одному RemoteFile, копируется при copy и удаляется вместе с ним. Запись создаётся только при наличии извлечённых полей; одного capturedAt для этого недостаточно. capturedAt находится в RemoteFile. Полнота и отсутствие данных описаны в [BACKUP-METADATA](04-backup-and-sync.md#backup-metadata).

<a id="file-type"></a>

## FileType

`IMAGE`, `VIDEO`, `AUDIO`, `DOCUMENT`, `ARCHIVE`, `OTHER` — категории MIME на Server, не состояния синхронизации. Server принимает и другие типы помимо изображений; Android discovery/backup охватывает только изображения в своей области.

<a id="api-projections"></a>

## Представления операций

Это временные запросы/ответы существующих операций, а не дополнительные сохраняемые сущности. Они используют определения полей выше.

| Представление | Связь с моделью и системный тип |
| --- | --- |
| UserDto | User.id/email/displayName/enabled/banned/roles/createdAt/lastUpdate/lastLoginAt с теми же типами; passwordHash не возвращается |
| FolderDto | Folder.id/name/folderType/createdAt/updatedAt; `parentId: FolderId?` представляет Folder.parent. Android допускает отсутствующие даты и читает ID/type |
| FileItemDto | RemoteFile.id/capturedAt/uploadedAt/deletedAt/metadata; `folderId: FolderId` представляет folder; `originalFilename: String` — originalName; `mimeType: String`, `size: Integer`, `checksum: Checksum`, `fileType: FileType` берутся из StoredObject |
| FileMetadataDto | Поля FileMetadata от width до longitude с теми же nullable типами; id/fileItem не выдаются |
| FileChecksumDto | `id: FileId`, `originalFilename: String` из RemoteFile и `checksum: Checksum` из StoredObject; folderId отсутствует |
| LoginResponse / RefreshResponse | При login строки TokenPair; при refresh только `accessToken: String` |
| Auth input | `email: String`, `password: String` — ввод пользователя; password является временным исходным паролем для проверки/хеширования, не полем хранения User |
| Code confirmation | `code: String` — EmailRequest.code; при reset также новый `password: String` |
| Folder operations | `parentId: FolderId?` для create; `targetParentId: FolderId` для move; `name: String` изменяет Folder.name |
| File operations | `targetFolderId: FolderId` для move, `FolderId?` для copy; `originalName: String` для rename, `String?` для copy — новое RemoteFile.originalName |
| Upload | `file: Поток байтов` с filename из LocalMedia.displayName и MIME из LocalMedia.mimeType; `folderId: FolderId?` задаёт папку либо default routing |
| Test/message response | `message: String` — текст результата операции, без сохраняемой domain entity |

В FileItemDto не раскрываются StoredObject.id/filePath/filename. Android допускает nullable поля FileItemDto и использует для успеха только id > 0; даты остаются строками, metadata — `Map<String, Any?>?`. Это терпимость декодирования ответа, не изменение обязательности Server полей. Ответные свойства не становятся полями LocalMedia.

<a id="checksum-partition"></a>

## ChecksumPartition

**Компонентное представление:** ChecksumExistsRequest/Response на обеих сторонах; живёт один запрос.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `folderId` | FolderId | Папка запроса, проверяемая в контексте пользователя |
| `checksums` | `List<Checksum>` | Входные суммы; API принимает также верхний регистр hex и нормализует |
| `existing` | `List<Checksum>` | Уникальные суммы с логической записью в этой папке |
| `missing` | `List<Checksum>` | Уникальные суммы без такой записи |

Ответ содержит только existing/missing, без folderId или FileId. Порядок внутри групп соответствует первому появлению во входе. Физические bytes не проверяются; условия в [BACKUP-EXISTING](04-backup-and-sync.md#backup-existing).

<a id="file-page"></a>

## FilePage

**Компонентное представление:** Server — `PageResponse<FileItemDto>`; Android этот список не запрашивает.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `items` | `List<FileItemDto>` | Файлы страницы |
| `page` | Integer | Номер страницы с нуля |
| `size` | Integer | Размер страницы |
| `totalElements` | Integer | Всего файлов в выборке |
| `totalPages` | Integer | Всего страниц |
| `hasNext` | Boolean | Есть следующая страница |
| `hasPrevious` | Boolean | Есть предыдущая страница |

Связь с RemoteFile задаётся items. Фильтр folderId и порядок — [FILES-LIST](05-file-library.md#files-list).

<a id="network-state"></a>

## NetworkState

Системное представление существующей настройки соединения Android; не UserSettings Server.

**Компонентное представление:** сохранённые IP/port и отдельный BaseUrlProvider в памяти процесса.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `ip` | String? | Сохранённый IPv4, без значения по умолчанию |
| `port` | String? | Сохранённая строка порта, без значения по умолчанию |
| `baseUrl` | String | Текущий адрес HTTP в памяти; исходно пустой |

TokenPair и LocalMedia не привязаны к этой настройке. Восстановление URL требует Network UI; lifecycle — [SETTINGS-NETWORK](03-profile-and-settings.md#settings-network) и [BACKUP-RESTART](04-backup-and-sync.md#backup-restart).

<a id="sync-results"></a>

## Результаты стадий и прогона

**Компонентное представление:** временные Android ScanResult, ChecksumResult, ChecksumPrecheckResult, UploadResult и SyncResult. Счётчики — Integer, не поля LocalMedia и не постоянная история.

| Представление | Поля | Значение |
| --- | --- | --- |
| ScanResult | `scanned`, `insertedOrUpdated`, `deletedStale` | Прочитанные строки, размер применяемого снимка и удалённые отсутствующие строки; insertedOrUpdated не считает только реальные изменения |
| ChecksumResult | `processed`, `succeeded`, `failed` | Обработанные строки и результаты hashing |
| ChecksumPrecheckResult | `checked`, `existing`, `pendingUpload`, `unchanged` | Результаты pre-check в локальных строках, не уникальных hashes |
| UploadResult | `attempted`, `succeeded`, `failed`, `leftPending` | Результаты upload; leftPending относится к встреченной временной ошибке, не размеру всей очереди |
| SyncResult | Nullable результаты четырёх стадий | Данные одного прогона; не общая server sync session |

<a id="sync-status-record"></a>

## SyncStatusRecord

Последний общий результат Android в памяти процесса.

**Компонентное представление:** Android — SyncStatusRecord в SyncStatusStore; на Server не сохраняется.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `status` | SyncStatus | Общий outcome |
| `timestamp` | DateTime | Время результата; UI карточка его не показывает |

Весь record исходно отсутствует и теряется с процессом. PermissionDenied worker не заменяет его. Связи с постоянной историей попыток или конкретным RemoteFile нет.

<a id="sync-status"></a>

## SyncStatus и состояние работы

SyncStatus принимает `SUCCESS`, `SERVER_UNAVAILABLE`, `ERROR`, `RETRY_SCHEDULED`. Это результат прогона, независимый от SyncState каждой строки: SUCCESS может сосуществовать с FAILED.

WorkInfo относится к Android WorkManager, а не LocalMedia. Значимые здесь `RUNNING`, `ENQUEUED`, `BLOCKED` описывают выполнение или ожидание работы. Worker success/retry, состояние work и SyncStatus не взаимозаменяемы. `CONNECTED` означает требование доступной сети, `batteryNotLow` — достаточного заряда; это условия задачи, не поля LocalMedia или пользовательские настройки. Условия фонового запуска находятся в [BACKUP-BACKGROUND](04-backup-and-sync.md#backup-background).

<a id="ui-state"></a>

## Состояние экранов Android

Это существующие данные формы и представления в памяти, без отдельной модели профиля/настроек Server.

| Данные | Системный тип | Значение |
| --- | --- | --- |
| Email/password формы Auth | String | Ввод пользователя; пароль не маскируется, остаётся после login/logout и не сохраняется в preferences |
| `isLoggedIn` | Boolean | Наличие TokenPair, не результат verifySession |
| IP/port формы Network | String | Ввод для формирования NetworkState |
| `isLoading` | Boolean | Выполняемая операция Auth/Network; ошибка создания соединения может оставить индикатор |
| `isScanning` | Boolean | UI scan на Files |
| `isSyncing` | Boolean | Только RUNNING one-time photo_sync; periodic не учитывается |
| `LastScanResult` | Результат UI scan? | Success с ScanResult, PermissionDenied либо Error; worker его не обновляет |
| Список и counters Files | `List<LocalMedia>`, Integer | Все локальные строки, Total и число строк семи используемых SyncState |
| Сообщение операции | String? | Результат явного действия/ошибки для UI |

Экран Files также показывает SyncStatusRecord. Устойчивой истории ошибок, byte progress и UI управления retry нет. Поведение — [FILES-LOCAL](05-file-library.md#files-local).

<a id="error-response"></a>

## ErrorResponse

Один из форматов ответа Server при ошибке; не все framework/IO отказы используют его.

**Компонентное представление:** Server — ErrorResponse; Android форматирует известные поля для сообщения, не сохраняет структурированную историю.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | String | UUID ошибки, не идентификатор сущности или sync attempt |
| `code` | String | Прикладной код ошибки |
| `message` | String | Описание |
| `fieldErrors` | `Map<String,String>?` | Ошибки отдельных входных полей |

ErrorResponse не тождественен FailureReason или SyncStatus. Ответы STUB содержат message без этого полного формата. Диагностика — [SYSTEM-DIAGNOSTICS](07-system-rules-and-recovery.md#system-diagnostics).
