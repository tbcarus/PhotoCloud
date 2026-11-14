# Системная модель PhotoCloud

Это каноническое определение системных сущностей, полей, типов и связей. Сценарии описывают операции над этой моделью. Имена RemoteFile, LocalMedia, DeviceMedia, PhysicalBytes и TokenPair обозначают системные представления существующих объектов, а не новые классы или хранилища.

Модель описывает текущий As-Is. В ней нет сущностей ACL, Device, Album, UserSettings или SyncSession. UploadSession реализована только на Server; Android пока её не использует.

<a id="system-types"></a>

## Типы и идентификаторы

| Тип | Значение |
| --- | --- |
| `String`, `Boolean` | Строка и логическое значение |
| `Integer`, `Number` | Целое число и число с возможной дробной частью |
| `DateTime` | Дата и время; Server передаёт локальное время без offset, Android хранит время media/ошибок в epoch milliseconds |
| `Instant` | Момент времени, независимый от часового пояса; Server хранит его со смещением и сравнивает как точку на шкале времени |
| `UserId`, `FolderId`, `FileId`, `StoredObjectId`, `MetadataId`, `SessionId`, `EmailRequestId`, `ShareId`, `NotificationId` | Идентификаторы записей на конкретном Server; одинаковые числа на разных Server не связывают объекты |
| `MediaId` | Положительный MediaStore ID в локальном индексе установки; отдельного volume/device namespace нет |
| `Checksum` | SHA-256 содержимого конкретного чтения: 64 шестнадцатеричных символа в нижнем регистре |
| `UUID` | UUID клиента (Server принимает его без изменения) либо серверный идентификатор UploadSession |
| `List<T>`, `Set<T>`, `Map<K,V>` | Список, множество и отображение значений |
| `T?` | Значение может отсутствовать; это не отдельное состояние обработки |

В сценариях hash означает checksum, а mtime — LocalMedia.lastModified. Checksum не является ID локального или удалённого файла. Несколько LocalMedia, RemoteFile и StoredObject могут иметь одинаковый checksum. Тройка владелец + папка + checksum определяет duplicate только в Camera ([BACKUP-DUPLICATE](04-backup-and-sync.md#backup-duplicate)); в обычной библиотеке checksum не определяет уникальность ([имена библиотеки](01-system-model.md#library-names)).

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

User владеет Folder, RemoteFile, StoredObject, EmailRequest и выданными Share и может быть получателем Share и [Notification](01-system-model.md#notification). RefreshSession связана с ним по email, без прямой ссылки на User. Текущие роли пользователя определяют полномочия защищённого запроса; влияние enabled/banned на уже выданные токены описывает [AUTH-BLOCK](02-account-and-auth.md#auth-block).

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
| `createdAt` | DateTime | Время создания, от которого отсчитывается срок действия кода |

### Связи и срок

EmailRequest относится к одному User; у пользователя может быть несколько кодов. Отдельного поля expiresAt нет. Срок действия в днях задаётся конфигурацией Server отдельно для ACTIVATE и PASSWORD_RESET; по умолчанию оба равны 3 дням, изменение одного не влияет на другой. Значение меньше 1 дня — ошибка конфигурации: Server не запускается. Код действителен, пока createdAt + срок его типа позже текущего времени; назначение и used проверяются отдельно. Использование кодов описано в [AUTH-ACT](02-account-and-auth.md#auth-act) и [AUTH-RESET](02-account-and-auth.md#auth-reset).

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
| `revokedAt` | DateTime? | Время одиночного logout или замены при refresh; массовый отзыв это поле не заполняет |

### Связи

Login создаёт новую RefreshSession для User. Android сохраняет её token вместе с access. Успешный reset пароля массово отзывает все RefreshSession пользователя ([AUTH-RESET](02-account-and-auth.md#auth-reset)). Refresh, если до exp остаётся больше порога замены, запись не меняет и срок не продлевает. Иначе он в одной транзакции отзывает её (revoked, revokedAt) и создаёт новую RefreshSession того же пользователя с полным сроком; связь между отозванной и новой записью не хранится. Истечение срока само не удаляет запись. Уникальность сохранённого token не установлена. Операции — [AUTH-LOGIN](02-account-and-auth.md#auth-login), [AUTH-REFRESH](02-account-and-auth.md#auth-refresh), [AUTH-LOGOUT](02-account-and-auth.md#auth-logout).

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
| `exp` | DateTime | Подписанный срок окончания: iat + срок типа из конфигурации Server; по умолчанию access — 1440 минут (24 часа), refresh — 180 дней |
| `jti` | String | UUID только для refresh; у access такого claim нет |

### Связи

AccessToken предъявляется от имени User в Bearer-запросе. RefreshTokenValue связан с RefreshSession по полной строке. Уже выданный access не отзывается через logout; остальные условия доступа описаны в account/auth.

Конфигурация Server задаёт срок access в минутах, срок refresh и порог его замены в днях: по умолчанию 1440 минут, 180 и 30 дней. Значения должны быть положительными, а порог — меньше срока refresh, иначе Server не запускается. Сроки вычисляются без переполнения.

<a id="token-type"></a>

## TokenType

`ACCESS` используется для защищённых операций; `REFRESH` — для получения нового access. Типы не взаимозаменяемы: refresh в Bearer не аутентифицирует запрос, access не принимается как refresh.

<a id="token-pair"></a>

## TokenPair

Одна локальная пара токенов Android.

**Компонентное представление:** Android — Tokens в зашифрованном хранилище и памяти; Server возвращает пару при login, а при refresh новый access и, только при замене refresh, новый refresh.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `accessToken` | String | Строка AccessToken |
| `refreshToken` | String | Строка RefreshTokenValue |

### Связи

Пара целиком может отсутствовать. Её наличие определяет локальное состояние входа, но не подтверждает валидность токенов. Полей accountId/serverId/host у пары нет; последствия — [SYSTEM-SCOPE](07-system-rules-and-recovery.md#system-scope). Login принимает только две непустые строки; refresh не принимает пустой access и, если поле refresh передано, пустую строку refresh. Ответ refresh без поля refresh даёт пару из нового access и прежнего refresh, с полем — новую пару.

Чтение и все изменения пары идут под одной блокировкой хранилища, вместе с ними обновляется наблюдаемое состояние входа. Login сохраняет, а logout очищает пару безусловно. Refresh заменяет или очищает пару атомарно и только если в хранилище всё ещё ровно та пара (access и refresh), с которой он начинался. Поэтому запоздалый refresh не стирает и не перезаписывает новую сессию и не восстанавливает пару после logout. Refresh очищает пару только при окончательном отказе Server ([AUTH-REFRESH](02-account-and-auth.md#auth-refresh)).

Пара с новым refresh записывается синхронно, на рабочем потоке, обе строки одной записью: прежний refresh Server уже отозвал. Если хранилище не подтвердило запись на диск, refresh завершается временной ошибкой; в памяти при этом уже новая пара, и следующая успешная запись сохраняет её. Пара без смены refresh и очистка пишутся асинхронно, как прежде.

<a id="device-media"></a>

## DeviceMedia

Изображение, видимое Android через MediaStore, и его читаемый поток. Это объект устройства, не запись очереди и не удалённый файл.

**Компонентное представление:** Android читает MediaStore Images; нормализованный снимок MediaStoreImage используется одним scan. Снимок полный, если запрос выполнен, прочитаны все его строки и основное хранилище подключено. Server объект устройства не хранит.

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

DeviceMedia индексируется в LocalMedia по mediaStoreId. ОС, provider и разрешения определяют видимость и доступный поток. URI не закрепляет неизменяемую версию bytes: checksum и upload открывают его независимо.

Checksum, проверка открытия перед upload и сама отправка выбирают поток по одному правилу. При выданном необязательном ACCESS_MEDIA_LOCATION Android запрашивает оригинал (MediaStore.setRequireOriginal, файловый дескриптор) — исходные байты вместе с EXIF/GPS. Без этого разрешения, а также если Android отказал в оригинале (разрешение отозвано после проверки), читается обычный поток MediaStore: на Android 10+ для чужих файлов, например снимков камеры, он может быть копией той же длины с затёртыми геоданными. Отказ в оригинале не делает файл ошибочным; отсутствие файла, ошибка чтения и отказ в доступе к самому изображению при обычном открытии не скрываются. Отдельная приватная копия потока и модель selected-photo отсутствуют.

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
| `createdAt` | DateTime | Локальное время изображения из DeviceMedia; определяет порядок локального списка (по убыванию). Рабочие партии hash, pre-check и upload выбираются по mediaStoreId по возрастанию |
| `lastModified` | DateTime | Локальное время изменения; участвует в обнаружении изменения |
| `checksum` | Checksum? | SHA-256 потока, прочитанного Android по правилу [DeviceMedia](01-system-model.md#device-media): оригинала или доступной версии; после discovery/изменения отсутствует |
| `status` | SyncState | Сохранённое состояние обработки; исходно PENDING |
| `failureReason` | FailureReason? | Классификация последней сохраняемой ошибки файла |
| `retryCount` | Integer | Число накопленных markFailed, исходно 0; не число HTTP/worker попыток. Обнуляется успешным hash/upload, изменением содержимого и ручным повтором FAILED ([BACKUP-RETRY](04-backup-and-sync.md#backup-retry)) |
| `lastFailureAt` | DateTime? | Время последнего markFailed по часам устройства; определяет cooldown |
| `uploadMismatchCount` | Integer | Число подтверждённых несовпадений checksum ответа upload с локальным, исходно 0. Успешный hash его не обнуляет; обнуляют SYNCED, новая версия, обнаруженная scan, и ручной повтор FAILED ([BACKUP-UPLOAD](04-backup-and-sync.md#backup-upload)) |

### Связи и границы

LocalMedia ссылается на DeviceMedia через mediaStoreId/uri. serverFileId может связывать её с RemoteFile, но не сопровождается account/server/folder namespace. Два local ID с одинаковыми bytes остаются разными записями. В LocalMedia нет folderId, server/account identity, remote filename/MIME/size/capturedAt/uploadedAt/metadata, upload session или byte progress.

Изменение size/lastModified сбрасывает checksum, serverFileId и сведения об ошибке. Отсутствие ID в полном снимке при полном доступе к изображениям удаляет запись. Переходы и исключения — [BACKUP-DISCOVER](04-backup-and-sync.md#backup-discover), [BACKUP-CHANGE](04-backup-and-sync.md#backup-change), [BACKUP-RETRY](04-backup-and-sync.md#backup-retry).

<a id="backup-source"></a>

## BackupSource

Сохранённая Android запись выбранной папки устройства — источника автоматического резервирования на конкретном Server и в конкретном аккаунте ([BACKUP-SOURCE-CONNECT](04-backup-and-sync.md#backup-source-connect)).

**Компонентное представление:** Android — BackupSource (Room); на Server ей соответствует корень источника — [Folder](01-system-model.md#folder) с тем же backupSource.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | Integer | Локальный ID записи |
| `sourceUuid` | UUID | Идентификатор источника, создаётся один раз до первого запроса регистрации и не меняется; на Server — Folder.backupSource. Уникален в локальной БД |
| `treeUri` | String | URI выбранного дерева Storage Access Framework с сохранённым разрешением на чтение |
| `localName` | String | Имя выбранной папки на устройстве |
| `serverName` | String | Имя корня источника в AutoUploads; задаёт пользователь, приложение его не исправляет |
| `serverRootFolderId` | FolderId? | ID корня источника на Server; только после подтверждённой регистрации |
| `serverUrl` | String | Адрес Server на момент подключения, построенный из сохранённых настроек ([SETTINGS-NETWORK](03-profile-and-settings.md#settings-network)) |
| `accountId` | String | Subject access token пользователя на момент подключения |
| `registrationState` | SourceRegistrationState | Состояние регистрации на Server |
| `registrationError` | String? | Причина последнего неуспеха регистрации либо обнаруженного прогоном отключения для показа; при отключении пользователем на этом устройстве пусто |
| `accessState` | SourceAccess | Последнее проверенное состояние доступа к дереву |
| `createdAt` | DateTime | Время создания записи по часам устройства |
| `scanGeneration` | Integer | Номер текущего обхода дерева; 0 — обхода не было ([BACKUP-SOURCE-SYNC](04-backup-and-sync.md#backup-source-sync)) |
| `scanErrors` | Integer | Папки, не прочитанные в текущем обходе; при ненулевом значении обход неполный |
| `lastScanAt` | DateTime? | Завершение последнего обхода по часам устройства |

### Связи и границы

Запись принадлежит паре serverUrl + accountId: при выходе или смене Server она не удаляется и не показывается другому аккаунту, при возврате к прежним адресу и пользователю снова видна. Состояние регистрации и доступ к дереву независимы. Источник содержит [BackupFolder](01-system-model.md#backup-folder) и [BackupFile](01-system-model.md#backup-file) своего дерева; MediaFile/LocalMedia источник не использует.

<a id="source-registration-state"></a>

## SourceRegistrationState

**Компонентное представление:** Android — RegistrationState, поле BackupSource.registrationState.

| Значение | Смысл |
| --- | --- |
| `PENDING_REGISTRATION` | Подтверждённого ID нет; результат последнего запроса мог остаться неизвестным, поэтому serverName не меняется |
| `REGISTRATION_FAILED` | Server окончательно отказал (409, 400): источник с этим UUID не создан, serverName можно изменить |
| `ACTIVE` | Server подтвердил источник, serverRootFolderId сохранён; повторной регистрации нет |
| `REMOTE_DISABLED` | Server больше не считает корень активным источником этого UUID: отключение подтверждено ответом 204 на этом устройстве ([BACKUP-SOURCE-DISABLE](04-backup-and-sync.md#backup-source-disable)) либо прогон обнаружил, что backupSource снят или папки нет в AutoUploads. Резервирование остановлено, регистрация не повторяется, запись и очередь сохраняются |

<a id="source-access"></a>

## SourceAccess

**Компонентное представление:** Android — SourceAccess, поле BackupSource.accessState.

| Значение | Смысл |
| --- | --- |
| `AVAILABLE` | Разрешение сохранено, корень дерева читается |
| `UNAVAILABLE` | Разрешение сохранено, но провайдер или папка не ответили: временная недоступность, не отзыв |
| `PERMISSION_LOST` | Сохранённого разрешения на дерево нет |

<a id="backup-folder"></a>

## BackupFolder

Папка дерева источника и её связь с Folder на Server.

**Компонентное представление:** Android — BackupFolder (Room); на Server — USER-папка дерева источника.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | Integer | Локальный ID |
| `sourceId` | Integer | BackupSource, к которому относится папка |
| `parentId` | Integer? | Родительская BackupFolder; нет только у корня источника |
| `name` | String | Имя папки на устройстве |
| `relativePath` | String | Путь от выбранной папки, `""` у корня; идентичность внутри источника, уникален в нём |
| `documentId` | String | Документ SAF при последнем чтении родителя; после переименования может смениться и идентичностью не считается |
| `serverFolderId` | FolderId? | Папка на Server; у корня — BackupSource.serverRootFolderId |
| `problem` | String? | Причина, по которой папку нельзя создать на Server; сбрасывается, когда папка снова найдена новым обходом |
| `seenGeneration` | Integer | Последний обход, нашедший папку |
| `listedGeneration` | Integer | Последний обход, прочитавший её содержимое |

<a id="backup-file"></a>

## BackupFile

Файл дерева источника и его связь с RemoteFile.

**Компонентное представление:** Android — BackupFile (Room).

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | Integer | Локальный ID; задаёт порядок очереди (FIFO) |
| `sourceId` | Integer | BackupSource файла |
| `folderId` | Integer | BackupFolder, содержащая файл |
| `name` | String | Имя файла на устройстве и в upload |
| `relativePath` | String | Путь от выбранной папки; идентичность внутри источника, уникален в нём. Одинаковые пути разных источников независимы |
| `documentId` | String | Документ SAF при последнем обходе |
| `mimeType` | String? | MIME, сообщённый провайдером |
| `size` | Integer? | Размер, сообщённый провайдером; null — неизвестен |
| `lastModified` | DateTime? | Время изменения, сообщённое провайдером; null — неизвестно |
| `checksum` | Checksum? | SHA-256 последнего чтения содержимого; записывается до отправки, поэтому после потерянного ответа опознаёт свою копию на Server |
| `serverFileId` | FileId? | RemoteFile под исходным именем, связанный с этим путём; связь создаётся только с копией, содержимое которой читалось из этого пути (ответ upload или совпадение checksum). После подтверждённого переименования прежней версии — null до загрузки новой |
| `syncedChecksum` | Checksum? | Checksum копии на Server, на которую указывает serverFileId |
| `status` | BackupFileStatus | Состояние обработки |
| `failureReason` | String? | Причина последнего пропуска, конфликта или ошибки |
| `retryCount` | Integer | Число таких исходов подряд; успешная обработка обнуляет |
| `lastFailureAt` | DateTime? | Время последнего такого исхода по часам устройства |
| `seenGeneration` | Integer | Последний обход, нашедший файл |
| `missingSince` | DateTime? | Файл не найден полным обходом без ошибок; серверная копия сохраняется |
| `archiveName` | String? | Имя прежней версии, выбранное до PATCH; при повторах не меняется, при занятом имени заменяется следующим до нового PATCH. Очищается после подтверждения новой версии |
| `archiveFileId` | FileId? | ID прежней версии, которая переименовывается; после переименования ID не меняется |
| `archiveChecksum` | Checksum? | Checksum прежней версии: сверяется при восстановлении |

Этап архивирования определяется status (CHANGED → ARCHIVING → ARCHIVED → SYNCED) вместе с archive-полями; исходные имя и папка — `name` и BackupFolder.serverFolderId.

<a id="backup-file-status"></a>

## BackupFileStatus

**Компонентное представление:** Android — BackupFileStatus, поле BackupFile.status.

| Значение | Смысл |
| --- | --- |
| `PENDING` | Нужно прочитать содержимое: новый файл, изменились или неизвестны сведения, повтор ошибки |
| `UPLOADING` | Начатые hash и upload; в начале следующего прогона возвращается в PENDING |
| `SYNCED` | На Server есть копия с syncedChecksum, равным checksum последнего чтения |
| `CHANGED` | Содержимое отличается от копии serverFileId: архивирование прежней версии ещё не начато |
| `ARCHIVING` | Архивное имя записано, результат PATCH может быть неизвестен; перед повтором состояние прежней версии читается с Server по archiveFileId |
| `ARCHIVED` | Переименование прежней версии подтверждено, новая версия под исходным именем ещё не подтверждена |
| `SKIPPED` | Отправить нельзя до изменения файла: пустой, больше 100 МиБ, недопустимое имя, документ без локального содержимого, отказ Server 400/413 |
| `CONFLICT` | Данные Server не совпали с ожидаемыми: имя занято папкой или файлом, содержимое которого не читалось из этого пути; прежняя версия не найдена по ID или её папка, имя либо checksum другие; не нашлось свободного архивного имени. Server не меняется; повтор — после изменения файла |
| `FAILED` | Ошибка чтения, папка назначения не создана или файл изменился во время передачи: повтор при следующем обходе |

<a id="sync-state"></a>

## SyncState

**Компонентное представление:** Android — MediaFileStatus, поле LocalMedia.status.

| Значение | Смысл обычного последовательного пути |
| --- | --- |
| `PENDING` | Ожидает вычисления checksum |
| `HASHING` | Сохранённый признак начатого вычисления checksum |
| `CHECKSUM_READY` | Checksum готова для pre-check; локальная обработка завершена, запись передана Загрузчику |
| `PENDING_UPLOAD` | Ожидает отправки |
| `UPLOADING` | Сохранённый признак начатой отправки |
| `SYNCED` | Локальный вывод об успехе по критерию ниже |
| `FAILED` | Ошибка файла; автоматический возврат ограничен CHECKSUM_IO |
| `LOCAL_DELETED` | Значение существует, но активный путь его не записывает |

HASHING/UPLOADING могут сохраняться без работающей задачи. Из любого состояния возможен сброс при изменении size/lastModified или удаление строки при исчезновении из полного снимка. PENDING, HASHING и переход в CHECKSUM_READY или локальный FAILED выполняет Поставщик; переходы из CHECKSUM_READY, PENDING_UPLOAD и UPLOADING — Загрузчик. Результаты hash, pre-check и upload применяются условно: только к строке в ожидаемом состоянии с прежними checksum, size и lastModified. Это не глобальная атомарная машина состояний: безусловные записи scan и restore могут нарушать обычные сочетания status/checksum/ID и сохранять сведения об ошибке при status, отличном от FAILED.

Ручная сверка ([CameraCheck](01-system-model.md#camera-check)) до решения пользователя меняет SyncState только одним условным переходом CHECKSUM_READY → SYNCED по ответу existing; подтверждение загрузки переводит отсутствующие SYNCED той же версии в CHECKSUM_READY, а FAILED — по правилу ручного повтора в PENDING ([BACKUP-CAMERA-CHECK](04-backup-and-sync.md#backup-camera-check)).

<a id="synced"></a>

### Значение SYNCED

SYNCED — сохранённый вывод Android после logical existing либо upload 2xx с положительным FileId и checksum ответа, равным local checksum. Existing меняет только status: serverFileId остаётся прежним, в обычном новом пути — null. Upload при обычном завершении записывает ID.

После upload local checksum равна SHA-256, которую Server вычислил по принятым байтам ([BACKUP-UPLOAD](04-backup-and-sync.md#backup-upload)). Состояние не гарантирует читаемость и неизменность физических байтов Server, равенство local checksum текущему содержимому файла (изменение без смены size/lastModified не обнаруживается), актуальность remote ID/папки, наличие оригинала сейчас или backup в текущем account/host. Обычный SYNCED повторно с Server не сверяется; это делает только ручная сверка ([BACKUP-CAMERA-CHECK](04-backup-and-sync.md#backup-camera-check)), и до подтверждения пользователя SYNCED не понижает. Успешный прогон Worker не означает, что все строки SYNCED. Нерешённый продуктовый смысл подтверждения остаётся рядом с [BACKUP-NEW](04-backup-and-sync.md#backup-new).

<a id="camera-check"></a>

## CameraCheck

Текущая или последняя ручная сверка Camera с Server ([BACKUP-CAMERA-CHECK](04-backup-and-sync.md#backup-camera-check)); одна запись, история проверок не хранится. Пока запись не в FINISHED, новые Camera upload приостановлены.

**Компонентное представление:** Android — CameraCheck, таблица Room `camera_check`; на Server не хранится.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `runId` | String | Идентификатор запуска; записи стадий применяются только к тому же запуску |
| `state` | CameraCheckState | RUNNING, ERROR, AWAITING_DECISION, FINISHED |
| `stage` | CameraCheckStage | SCAN, HASH, SERVER, DONE — стадия, с которой проверка продолжится |
| `error` | CameraCheckError? | Причина ERROR: NO_MEDIA_PERMISSION, SCAN_INCOMPLETE, SCAN_FAILED, NOT_SIGNED_IN, NO_SERVER_ADDRESS, SERVER_UNAVAILABLE, SERVER_ERROR, AUTH, BAD_RESPONSE, DESTINATION_CHANGED |
| `serverUrl` | String | Адрес Server на момент запуска (из сохранённых IP/port) |
| `accountId` | String | Subject AccessToken на момент запуска |
| `cameraFolderId` | FolderId? | CAMERA, для которой получены ответы; задаётся первой серверной партией |
| `startedAt` | DateTime | Время запуска по часам устройства |
| `deferred` | Boolean | Пользователь выбрал «Позже»: результат свёрнут, пауза сохраняется |
| `finishedAt` | DateTime? | Время завершения ручного режима |
| `checked`, `confirmed`, `missing`, `unverifiable` | Integer | Итог серверной стадии: проверено, найдено, не найдено, не удалось проверить |
| `queuedForUpload` | Integer? | Сколько отсутствующих файлов возвращено в обычную синхронизацию подтверждением; null — подтверждения не было |

Назначение проверки — serverUrl + accountId + cameraFolderId. Ответы для другого назначения не применяются; модели нескольких аккаунтов нет.

<a id="camera-check-item"></a>

## CameraCheckItem

Строка набора текущей проверки: какая версия LocalMedia и по какому checksum сверяется.

**Компонентное представление:** Android — CameraCheckItem, таблица Room `camera_check_items`.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `mediaStoreId` | MediaId | LocalMedia набора |
| `size`, `lastModified` | Integer, DateTime | Версия файла, к которой относится checksum и результат |
| `checksum` | Checksum? | Из LocalMedia той же версии или вычисленный сверкой для FAILED без checksum; null — ещё не получен |
| `result` | EXISTS / MISSING / UNVERIFIABLE? | Достоверный итог строки; null — итога ещё нет |

Набор создаётся из всех LocalMedia, кроме LOCAL_DELETED, после полного scan и удаляется отменой, отказом, подтверждением или завершением без отсутствующих файлов. Счётчики проверки вычисляются по набору, поэтому повтор партии их не искажает.

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
| `CHANGED_DURING_UPLOAD` | Checksum ответа upload снова не совпал с local checksum после трёх автоматических восстановлений |
| `UNKNOWN` | Прочая ошибка отдельного файла при hashing |

Автоматически восстанавливается только CHECKSUM_IO при retryCount < 3 и достаточном cooldown; CHANGED_DURING_UPLOAD ждёт ручного повтора или новой версии файла. Исторический FAILED может не иметь причины. Временная ошибка upload не записывает FailureReason и не увеличивает retryCount; условия и сброс полей — [BACKUP-RETRY](04-backup-and-sync.md#backup-retry).

<a id="folder"></a>

## Folder

Папка Server, независимая от каталогов устройства. Папке обычной библиотеки соответствует физический каталог с тем же именем ([PhysicalBytes](01-system-model.md#physical-bytes)).

**Компонентное представление:** Server — Folder; Android использует FolderDto для получения CAMERA и просмотра библиотеки, дерево папок не сохраняет.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | FolderId | Идентификатор папки |
| `user` | User | Владелец |
| `parent` | Folder? | Родитель; создаваемая ROOT не имеет родителя |
| `name` | String | Логическое имя |
| `folderType` | FolderType | Категория, неизменяемая через API |
| `backupSource` | UUID? | Идентификатор автоматического источника Android, переданный при регистрации ([BACKUP-SOURCE-REGISTER](04-backup-and-sync.md#backup-source-register)); задан только у корня активного источника — USER-папки, прямого потомка AUTO_UPLOADS; уникален в пределах владельца, у разных пользователей может совпадать. Отключение источника его очищает ([BACKUP-SOURCE-DISABLE](04-backup-and-sync.md#backup-source-disable)), иначе через API не меняется |
| `createdAt` | DateTime | Время создания |
| `updatedAt` | DateTime | Время последнего обновления |

### Связи

Folder принадлежит User, содержит RemoteFile и имеет прямых потомков Folder. Отдельного сохраняемого поля children нет. По сохранённым данным различаются системный контейнер (folderType=AUTO_UPLOADS), корень активного источника (USER с backupSource), элемент активного дерева (папка, у которой она сама или предок до AUTO_UPLOADS имеет backupSource, и RemoteFile такой папки) и обычная папка. Активность определяется только наличием backupSource на корне и на вложенных объектах не дублируется. Отключённый источник — прямой потомок AUTO_UPLOADS без backupSource — обычная USER-папка; ограничения активного дерева — [FOLDERS-MANAGED-TREE](05-file-library.md#folders-managed-tree). Обычное дерево создаётся в пределах пользователя; структура хранения сама не гарантирует совпадение владельца parent или отсутствие длинных циклов: это обеспечивают операции под блокировкой ([структурные изменения](01-system-model.md#library-names)). Ограничения системных папок и перемещений находятся в [операциях папок](05-file-library.md#folders-browse).

<a id="folder-type"></a>

## FolderType

| Значение | Смысл |
| --- | --- |
| `ROOT` | Единственная корневая папка пользователя, создаётся лениво с именем root |
| `CAMERA` | Системная папка Camera: назначение backup Android и default routing IMAGE/VIDEO |
| `AUTO_UPLOADS` | Системный контейнер AutoUploads: единственный у пользователя прямой потомок ROOT, содержит только корни автоматических источников |
| `USER` | Пользовательская папка, создаваемая под ROOT/USER; корень источника и его вложенные папки — тоже USER |

CAMERA — системная конечная папка. ROOT, CAMERA и AUTO_UPLOADS нельзя переименовать, переместить или удалить через текущий API. Создание CAMERA и получение её ID Android описывает [BACKUP-TARGET](04-backup-and-sync.md#backup-target). AUTO_UPLOADS создаётся при первой регистрации источника ([BACKUP-SOURCE-REGISTER](04-backup-and-sync.md#backup-source-register)); прямыми потомками в нём могут быть только корни источников, созданные регистрацией, и отключённые источники, ещё не перенесённые из него; файлов в самом контейнере нет. Дерево активного источника меняет только механизм резервирования этого источника ([FOLDERS-MANAGED-TREE](05-file-library.md#folders-managed-tree)); отключённый источник подчиняется обычным операциям библиотеки и может быть перенесён из AUTO_UPLOADS, но не обратно.

<a id="library-names"></a>

## Camera, обычная библиотека и имена

Обычная библиотека — ROOT, USER-папки и контейнер AUTO_UPLOADS с деревьями источников: имена, конфликты и физическое размещение в них — как у остальной библиотеки. Camera — папка типа CAMERA со своими правилами: одинаковые имена допустимы, duplicate определяется по владельцу, папке и checksum, а физическое имя и путь задаются при создании StoredObject. Перенос между Camera и библиотекой запрещён, копирование возможно только из Camera в библиотеку ([FILES-MOVE](05-file-library.md#files-move), [FILES-COPY](05-file-library.md#files-copy)).

В каталоге обычной библиотеки файлы и папки используют одно пространство имён: имя принадлежит только одному объекту. Сравнение выполняет PostgreSQL через `lower()` по Folder.name и RemoteFile.originalName, поэтому результат зависит от locale БД; в Java отдельного сравнения имён нет. Имя включает расширение, регистр сохраняется как передан. Переименование объекта с изменением только регистра допустимо. В ROOT имя Camera зарезервировано независимо от регистра, даже если системная Camera ещё не создана: такое имя даёт 409. Имя AutoUploads не зарезервировано: существующие папка или файл с этим именем без учёта регистра не становятся системным контейнером, и регистрация источника получает 409.

Правила имени (нарушение → 400, автоматических исправлений нет):

| Правило | Значение |
| --- | --- |
| Содержимое | Не пусто и не только из пробелов |
| Символы | Нет `/ \ : * ? " < > |`, управляющих символов и одиночных суррогатов |
| Форма | Unicode NFC |
| Окончание | Не точка и не пробел; `.` и `..` недопустимы, начальная точка допустима |
| Имена устройств Windows | Часть до первой точки не CON, PRN, AUX, NUL, COM0–COM9, LPT0–LPT9, их варианты с ¹²³, CONIN$, CONOUT$ |
| Длина | Не более 255 байт UTF-8 |

Правила проверяет Server; на Camera-файлы они не распространяются: их имя очищается прежним способом ([BACKUP-METADATA](04-backup-and-sync.md#backup-metadata)).

Структурные изменения одного пользователя выполняются по очереди: транзакция блокирует строку его ROOT (`SELECT ... FOR NO KEY UPDATE`), проверяет обе таблицы и только затем пишет. В эту очередь входят создание, переименование, перемещение и удаление папок и файлов, upload, copy, получение Camera и регистрация источника. Физические изменения библиотеки выполняются внутри этой же транзакции, после записи в БД и до COMMIT; приём потока, расчёт checksum и копирование байтов во временный файл идут вне блокировки. Блокировка живёт до конца этой транзакции, включая публикацию байтов. Уникальность имён файлов и duplicate Camera обеспечиваются этой очередью, а не ограничением БД: PostgreSQL не может построить частичный индекс по типу родительской папки. Ограничения БД есть у имён папок одного родителя, у единственного AUTO_UPLOADS пользователя, у backupSource в пределах пользователя и у наличия backupSource только у USER; то, что корень источника — прямой потомок AUTO_UPLOADS, обеспечивают операции.

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
| `capturedAt` | DateTime | Локальные календарные дата и время съёмки из EXIF DateTimeOriginal, иначе DateTimeDigitized, без пересчёта часовых поясов; без пригодной EXIF-даты — uploadedAt ([BACKUP-METADATA](04-backup-and-sync.md#backup-metadata)) |
| `uploadedAt` | DateTime | Время создания удалённой записи; при copy новое |
| `deletedAt` | DateTime? | Поле существует, но текущий API его не заполняет и удаляет запись физически |
| `metadata` | FileMetadata? | Доступные сведения об изображении |

### Связи

User → Folder → RemoteFile → StoredObject → PhysicalBytes. У RemoteFile не более одной FileMetadata. Обычный upload/copy создаёт записи одного владельца, но сама модель допускает несовпадение владельцев связанных записей и несколько ссылок на StoredObject. Совместный доступ выражается только [Share](01-system-model.md#share) и таких ссылок не создаёт.

Отдельных uploaded/status или version chain у RemoteFile нет. Rename меняет originalName, move — folder, copy создаёт новую identity. Правила имён — [Camera, обычная библиотека и имена](01-system-model.md#library-names), детали конфликтов и удаления остаются в [библиотеке файлов](05-file-library.md).

<a id="stored-object"></a>

## StoredObject

Сохранённые сведения о физическом объекте; наличие записи не удостоверяет наличие bytes.

**Компонентное представление:** Server — StoredObject; Android отдельное представление не хранит.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | StoredObjectId | Идентификатор физического объекта в каталоге, не FileId |
| `user` | User | Владелец физических bytes |
| `filePath` | String? | Относительный каталог физического ключа от storage root; задаётся при создании и затем не меняется. Заполнен только у объектов Camera: `{userId}/camera/{YYYY}` по году capturedAt ([BACKUP-METADATA](04-backup-and-sync.md#backup-metadata)). У файла обычной библиотеки отсутствует: путь вычисляется из дерева папок и имени |
| `filename` | String? | Физическое имя объекта Camera `{исходное имя}_{UUID}.{fileExtension}`, без `.{fileExtension}` при пустом расширении; повтор расширения намеренно позволяет узнать исходное имя. Отсутствует вместе с filePath |
| `fileExtension` | String? | Расширение физического файла Camera; может быть пустой строкой. Отсутствует вместе с filePath |
| `checksum` | Checksum | SHA-256 прочитанного Server содержимого |
| `size` | Integer | Фактически прочитанный размер в байтах |
| `detectedMimeType` | String | MIME, определённый по bytes |
| `fileType` | FileType | Классификация определённого MIME |
| `createdAt` | DateTime | Время создания записи |

### Связи

StoredObject принадлежит User, на него могут ссылаться несколько RemoteFile. Обычные upload/copy создают отдельный объект и независимые bytes; copy допускается только для StoredObject текущего пользователя ([FILES-COPY](05-file-library.md#files-copy)). Checksum не является уникальным ключом этой записи. Сочетание пути и имени уникально на файловой системе за счёт исключающей публикации байтов при upload/copy: занятый путь не перезаписывается ([SYSTEM-STORAGE](07-system-rules-and-recovery.md#system-storage)); ограничения уникальности в БД нет. Прикладного обновления физических полей, refcount или сборки неиспользуемых объектов нет. Ветки owner/non-owner удаления описаны в [FILES-DELETE](05-file-library.md#files-delete).

<a id="physical-bytes"></a>

## PhysicalBytes

Содержимое в файловой системе Server, отдельно от StoredObject.

**Компонентное представление:** Server filesystem; отдельной DB entity нет. Android не имеет прямого доступа к этому хранилищу.

### Системные свойства

| Свойство | Системный тип | Значение |
| --- | --- | --- |
| Размещение | String | Camera: путь от настроенного storage root из StoredObject.filePath/filename. Обычная библиотека: `{userId}/files/{имена папок от ROOT, без самого ROOT}/{RemoteFile.originalName}`, без UUID; путь не хранится и вычисляется из дерева Folder |
| Содержимое | Поток байтов | Текущие фактические bytes по этому пути; upload публикует принятый поток без перекодирования и изменения метаданных |
| Доступность | Наличие и читаемость | Проверяется при обращении к файловой системе, не сохраняется как флаг StoredObject |

### Связи

StoredObject описывает размещение PhysicalBytes. Bytes могут остаться без записей, а записи — без читаемых bytes; общего атомарного сохранения DB/filesystem нет. Эти свойства не являются новыми полями Server entity. Последствия и recovery — [SYSTEM-STORAGE](07-system-rules-and-recovery.md#system-storage).

<a id="owner-marker"></a>

## OwnerMarker

Файл-маркер владельца каталога пользователя: помогает администратору определить владельца при ручном просмотре диска.

**Компонентное представление:** JSON-файл `{userId}/{safeEmail}.json` в корне каталога пользователя под storage root; отдельной DB entity нет. Android к нему доступа не имеет.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `userId` | UserId | Владелец каталога |
| `email` | String | Полный User.email на момент создания маркера |
| `userCreatedAt` | DateTime | User.createdAt; ISO 8601 без offset |
| `markerCreatedAt` | DateTime | Момент фактического создания маркера; ISO 8601 без offset |

Других полей нет: пароль, его хеш, токены, коды, роли и состояние аккаунта в маркер не записываются.

### Связи и правила

Имя файла строится из email: Unicode NFC, символы, недопустимые в Windows, и одиночные суррогаты заменяются на `_`, затем имя укорачивается по границам символов так, чтобы вместе с окончанием `.json` занимать не более 255 байт UTF-8. Сырой email путём не используется; перед записью проверяется, что файл лежит непосредственно в каталоге `{userId}`. Точный адрес хранится только внутри JSON.

Маркер создаётся при первой успешной физической публикации байтов пользователя через upload или copy, если файла с таким именем ещё нет; регистрация и активация его не создают. Существующий файл не перезаписывается, повторная запись не меняет markerCreatedAt. Ошибка создания маркера не прерывает upload/copy. Маркер — справка для администратора, а не источник авторизации или замена БД: актуальные данные хранит User. Правило для будущей смены email — [PROFILE-ABSENT](03-profile-and-settings.md#profile-absent).

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

<a id="share"></a>

## Share

Разрешение на один объект владельца — папку или файл. Server проверяет по нему чтение USER-доступа ([ACCESS-READ](06-sharing-and-access.md#access-read)); владелец выдаёт и отзывает USER-доступ через API ([SHARING-USER-MANAGE](06-sharing-and-access.md#sharing-user-manage)), получатель видит и снимает свои доступы ([SHARED-WITH-ME](05-file-library.md#shared-with-me)). Владелец создаёт LINK-доступ через API ([SHARING-LINK-MANAGE](06-sharing-and-access.md#sharing-link-manage)), чтение по нему без входа — [ACCESS-LINK-READ](06-sharing-and-access.md#access-link-read); папку по ссылке браузер показывает страницей просмотра ([FILES-PUBLIC-READ](05-file-library.md#files-public-read)), ссылками владелец управляет и в Android ([поделиться](05-file-library.md#files-browse-sharing)).

**Компонентное представление:** Server — Share; Android его не использует.

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | ShareId | Идентификатор разрешения |
| `owner` | User | Владелец объекта. Само поле право не доказывает: проверка чтения сравнивает его с фактическим Folder.user/RemoteFile.user и игнорирует несовпадающую запись |
| `folder` | Folder? | Целевая папка |
| `fileItem` | RemoteFile? | Целевой файл |
| `kind` | [ShareKind](01-system-model.md#share-kind) | Вид разрешения |
| `recipient` | User? | Получатель; есть только у USER и не совпадает с owner |
| `tokenHash` | String? | SHA-256 секрета ссылки, 64 шестнадцатеричных символа; есть только у LINK, уникален среди всех Share. По нему ищется ссылка |
| `encryptedToken` | String? | Секрет ссылки, зашифрованный AES-GCM ключом конфигурации Server: Base64 от случайного nonce и шифртекста; есть только у LINK. Нужен только для повторной выдачи URL владельцу; открытым текстом секрет не хранится |
| `expiresAt` | Instant? | Окончание действия; отсутствует — бессрочно. Разрешение действует, пока текущее время строго меньше expiresAt |
| `createdAt` | Instant | Время создания записи |

### Связи и правила

Задан ровно один из folder и fileItem. USER содержит recipient и не содержит tokenHash и encryptedToken, LINK — наоборот. На пару объект + получатель — не более одного USER; LINK на один объект может быть несколько. Эти правила, уникальность tokenHash и запрет разрешения самому себе проверяет БД; совпадение owner с владельцем объекта БД не проверяет.

Удаление целевого объекта (в том числе вместе с папкой-предком), владельца или получателя удаляет Share. Отзыв владельцем и отказ получателя удаляют запись физически; истёкшая запись остаётся, отдельного поля состояния нет — оно вычисляется по expiresAt. Share не создаёт RemoteFile, StoredObject или байты получателя: разрешение на папку действует на её текущее поддерево Folder. Объектом разрешения не могут быть ROOT и AUTO_UPLOADS: API выдачи их отклоняет, а проверка чтения не учитывает такие записи, если они появятся. CAMERA, обычные USER-папки и корни активных источников допустимы.

<a id="share-kind"></a>

## ShareKind

| Значение | Смысл |
| --- | --- |
| `USER` | Разрешение зарегистрированному получателю |
| `LINK` | Разрешение по секрету ссылки без входа; только чтение |

<a id="notification"></a>

## Notification

Сохранённое сообщение одному пользователю о событии на Server. Server создаёт уведомление о новом USER-доступе ([SHARING-USER-MANAGE](06-sharing-and-access.md#sharing-user-manage)); получатель читает уведомления и отмечает прочтение в Android и через API ([NOTIFICATIONS-READ](08-notifications.md#notifications-read)). Правила подсистемы — [уведомления](08-notifications.md#notifications-rules). Доставки push и email нет.

**Компонентное представление:** Server — Notification; Android — NotificationDto только в памяти процесса, без Room и настроек ([NOTIFICATIONS-READ](08-notifications.md#notifications-read-android)).

### Поля

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | NotificationId | Идентификатор уведомления |
| `recipient` | User | Получатель; единственный пользователь, которому уведомление видно |
| `type` | [NotificationType](01-system-model.md#notification-type) | Вид события |
| `title` | String | Заголовок, сформированный Server |
| `body` | String | Текст, сформированный Server |
| `targetType` | [NotificationTargetType](01-system-model.md#notification-target-type)? | Вид объекта события |
| `targetId` | FileId? или FolderId? | ID объекта события; FileId при targetType=FILE, FolderId при FOLDER |
| `createdAt` | Instant | Время создания по часам Server |
| `readAt` | Instant? | Время первого прочтения; отсутствует — не прочитано |

### Связи и правила

targetType и targetId заданы оба или отсутствуют оба; у SHARE_GRANTED они обязательны. Эти правила проверяет БД. title и body — снимок на момент события: переименование и удаление объекта их не меняют, секреты, токены и пути хранения в них не входят. targetId — только указатель для навигации: он не ссылается на объект ограничением БД и не даёт права доступа, чтение объекта по-прежнему проверяет [ACCESS-READ](06-sharing-and-access.md#access-read). Удаление объекта, отзыв, истечение или отказ от Share уведомление не удаляют; удаление получателя удаляет его уведомления, как и другие данные пользователя. readAt устанавливается один раз и затем не меняется.

<a id="notification-type"></a>

## NotificationType

Хранится строковым именем.

| Значение | Смысл |
| --- | --- |
| `SHARE_GRANTED` | Получателю создан новый USER-доступ к файлу или папке |
| `BACKUP_FAILED` | Зарезервировано; Server такие уведомления не создаёт |
| `SYSTEM_MESSAGE` | Зарезервировано; Server такие уведомления не создаёт |

<a id="notification-target-type"></a>

## NotificationTargetType

`FILE`, `FOLDER` — вид объекта, на который указывает Notification.targetId. Хранится строковым именем.

<a id="api-projections"></a>

## Представления операций

Это временные запросы/ответы существующих операций, а не дополнительные сохраняемые сущности. Они используют определения полей выше.

| Представление | Связь с моделью и системный тип |
| --- | --- |
| UserDto | User.id/email/displayName/enabled/banned/roles/createdAt/lastUpdate/lastLoginAt с теми же типами; passwordHash не возвращается |
| FolderDto | Folder.id/name/folderType/backupSource/createdAt/updatedAt; `parentId: FolderId?` представляет Folder.parent. Android допускает отсутствующие даты и читает ID/name/type/backupSource. Получателю чужой папки по Share backupSource не выдаётся ([ACCESS-READ](06-sharing-and-access.md#access-read)) |
| FileItemDto | RemoteFile.id/capturedAt/uploadedAt/deletedAt/metadata; `folderId: FolderId` представляет folder; `originalFilename: String` — originalName; `mimeType: String`, `size: Integer`, `checksum: Checksum`, `fileType: FileType` берутся из StoredObject |
| FileMetadataDto | Поля FileMetadata от width до longitude с теми же nullable типами; id/fileItem не выдаются |
| FileChecksumDto | `id: FileId`, `originalFilename: String` из RemoteFile и `checksum: Checksum` из StoredObject; folderId отсутствует |
| LoginResponse / RefreshResponse | При login строки TokenPair; при refresh `accessToken: String` и, только при замене refresh, `refreshToken: String`; без замены поля refreshToken в ответе нет |
| Auth input | `email: String`, `password: String` — ввод пользователя; password является временным исходным паролем для проверки/хеширования, не полем хранения User |
| Code confirmation | `code: String` — EmailRequest.code; при reset также новый `password: String` |
| Folder operations | `parentId: FolderId?` для create; `targetParentId: FolderId` для move; `name: String` изменяет Folder.name; регистрация источника — `name: String` и `sourceUuid: UUID` — будущие Folder.name и Folder.backupSource; `backupSource: UUID?` в create и rename — UUID активного источника, от имени механизма резервирования которого выполняется операция |
| File operations | `targetFolderId: FolderId` для move, `FolderId?` для copy; `originalName: String` для rename, `String?` для copy — новое RemoteFile.originalName; `backupSource: UUID?` в rename — как у Folder operations |
| Upload | `file: Поток байтов` с filename и MIME; `folderId: FolderId?` задаёт папку либо default routing; `backupSource: UUID?` — как у Folder operations. Backup Android берёт filename и MIME из LocalMedia и передаёт ID CAMERA; ручная загрузка — имя и MIME документа Storage Access Framework и ID открытой папки ROOT или USER |
| Share operations | Выдача: `fileId: FileId?` или `folderId: FolderId?` — ровно один, будущие Share.fileItem/folder; `emails: List<String>` — адреса получателей, определяющие Share.recipient; `expiresAt: Instant?` — Share.expiresAt. Изменение срока: `expiresAt: Instant?`, где явный null снимает срок, а отсутствие поля — ошибка ([SHARING-USER-MANAGE](06-sharing-and-access.md#sharing-user-manage)) |
| ShareDto | Share.id/expiresAt/createdAt; `resourceType: FILE\|FOLDER`, `resourceId` и `resourceName` — id и имя Share.fileItem либо Share.folder; `recipient` — ShareUserDto получателя; `status: ACTIVE\|EXPIRED` — вычисляется по expiresAt в момент ответа и не хранится. tokenHash и encryptedToken не выдаются |
| SharedResourceDto | `shareId` — Share.id; `resourceType`, `resourceId`, `name` — как у ShareDto; `file: FileItemDto?` — только у файла; `owner` — ShareUserDto владельца; `expiresAt` и `sharedAt` — Share.expiresAt и createdAt. Folder.backupSource, tokenHash и encryptedToken не выдаются ([SHARED-WITH-ME](05-file-library.md#shared-with-me)) |
| ShareUserDto | User.id/email/displayName владельца или получателя доступа |
| Link operations | Создание: `fileId: FileId?` или `folderId: FolderId?` — ровно один, `expiresAt: Instant?`; изменение срока — как у Share operations ([SHARING-LINK-MANAGE](06-sharing-and-access.md#sharing-link-manage)) |
| LinkShareDto | Share.id/expiresAt/createdAt; `resourceType`, `resourceId`, `resourceName` — как у ShareDto; `url: String?` — полный адрес ссылки из публичного адреса Server и расшифрованного секрета, отсутствует, если секрет не расшифровывается текущим ключом; `status` — как у ShareDto. tokenHash и encryptedToken не выдаются |
| NotificationDto | Notification.id/type/title/body/targetType/targetId/createdAt/readAt с теми же типами; recipient не выдаётся ([NOTIFICATIONS-READ](08-notifications.md#notifications-read)) |
| Unread notification count | `count: Integer` — число Notification текущего пользователя без readAt |
| PublicResourceDto | Корень ссылки: `type: FILE\|FOLDER`, `id` и `name` объекта, `file: PublicFileDto?` — только у файла, `expiresAt` — Share.expiresAt ([ACCESS-LINK-READ](06-sharing-and-access.md#access-link-read)) |
| PublicFileDto | RemoteFile.id, `name` — originalName; `mimeType`, `size`, `fileType` из StoredObject. Checksum, FileMetadata, folder, пути хранения и владелец не выдаются |
| PublicFolderDto | Folder.id/name; folderType, backupSource и parent не выдаются |
| Test/message response | `message: String` — текст результата операции, без сохраняемой domain entity |

В FileItemDto не раскрываются StoredObject.id/filePath/filename. Android допускает nullable поля FileItemDto и использует для успеха upload backup только id > 0; Library показывает originalFilename, fileType, size, capturedAt и uploadedAt, а ручная передача сравнивает с локальным файлом size и checksum. Даты остаются строками, metadata — `Map<String, Any?>?`. Это терпимость декодирования ответа, не изменение обязательности Server полей. Ответные свойства не становятся полями LocalMedia.

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

**Компонентное представление:** Server — `PageResponse<FileItemDto>`; Android — `FilePageDto`, живёт один запрос.

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

**Компонентное представление:** сохранённые IP/port (единственный источник настройки) и BaseUrlProvider в памяти процесса, который формирует из них адрес по единому правилу.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `ip` | String? | Сохранённый IPv4, без значения по умолчанию |
| `port` | String? | Сохранённая строка порта, без значения по умолчанию |
| `baseUrl` | String | Текущий адрес HTTP в памяти; исходно пустой. Восстанавливается из ip/port при создании Network UI и перед каждым прогоном Загрузчика; пуст при отсутствии или некорректности настройки |

TokenPair и LocalMedia не привязаны к этой настройке. Lifecycle — [SETTINGS-NETWORK](03-profile-and-settings.md#settings-network) и [BACKUP-RESTART](04-backup-and-sync.md#backup-restart).

<a id="sync-results"></a>

## Результаты стадий и прогона

**Компонентное представление:** временные Android ScanResult, ChecksumResult, ChecksumPrecheckResult и UploadResult. Счётчики — Integer, не поля LocalMedia и не постоянная история.

| Представление | Поля | Значение |
| --- | --- | --- |
| ScanResult | `scanned`, `insertedOrUpdated`, `deletedStale` | Прочитанные строки, размер применяемого снимка и удалённые отсутствующие строки (0 для неполного снимка); insertedOrUpdated не считает только реальные изменения |
| ChecksumResult | `processed`, `succeeded`, `failed` | Обработанные строки и результаты hashing |
| ChecksumPrecheckResult | `checked`, `existing`, `pendingUpload`, `unchanged` | Результаты pre-check в локальных строках, не уникальных hashes |
| UploadResult | `attempted`, `succeeded`, `failed`, `leftPending` | Результаты upload; leftPending относится к встреченной временной ошибке, не размеру всей очереди |

<a id="sync-run-results"></a>

## SyncRunResults

Сохранённые итоги последних прогонов Поставщика и Загрузчика; переживают перезапуск процесса.

**Компонентное представление:** Android — SyncRunResults в SyncStatusStore, файл DataStore `sync_status`, отдельный от настроек Server; на Server не сохраняется.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `providerProblem` | SyncProblem? | Ошибка последнего прогона Поставщика (ERROR: scan, hash или исключение) и время её возникновения |
| `lastScanAt` | DateTime? | Конец последнего прогона Поставщика с успешным scan |
| `uploaderProblem` | SyncProblem? | Проблема последнего прогона Загрузчика: SERVER_UNAVAILABLE (ping), ERROR (pre-check или исключение), RETRY_SCHEDULED (временная ошибка upload), и время её возникновения |
| `lastUploadAt` | DateTime? | Конец последнего прогона Загрузчика, в котором Server подтвердил хотя бы один отправленный файл; existing pre-check отправкой не считается |

Каждая часть пишет только свои поля, один раз в конце прогона: успешное завершение снимает проблему этой части, проблему другой не трогает. Повтор той же проблемы сохраняет прежнее время возникновения; DataStore при неизменных данных файл не перезаписывает. Время — часы устройства, с точностью до длительности прогона. Без полного доступа к фото, без пары и без адреса Server итог не записывается: эти условия проверяются в момент отображения. «Выполняется» не сохраняется. Ошибка записи только логируется и на синхронизацию не влияет. Время сбоя внутри стадии upload, о котором прогон не получил счётчиков, отправкой не считается.

<a id="backup-state"></a>

## BackupState

Пользовательское состояние резервного копирования, вычисляемое на Android при каждом изменении источников; не хранится.

**Компонентное представление:** Android — BackupState, функция backupStateOf и BackupStateMonitor; единственный источник состояния backup на Files ([FILES-LOCAL](05-file-library.md#files-local)).

Источники: счётчики LocalMedia (агрегирующий запрос Room: total, synced = SYNCED, preparing = PENDING + HASHING, toUpload = CHECKSUM_READY + PENDING_UPLOAD + UPLOADING, failed = FAILED; LOCAL_DELETED не учитывается, total равен сумме групп), RUNNING разовых и periodic задач Поставщика и Загрузчика в WorkManager, фактический полный доступ к фото, наличие TokenPair, адрес из сохранённых IP/port по правилу [SETTINGS-NETWORK](03-profile-and-settings.md#settings-network) и SyncRunResults. Опросов нет.

| Фаза | Условие (в порядке проверки) |
| --- | --- |
| `ATTENTION_REQUIRED` | Нет полного доступа к фото, нет пары или нет корректного адреса |
| `RUNNING` | Выполняется прогон любой части |
| `WAITING` | preparing + toUpload > 0 |
| `ATTENTION_REQUIRED` | Есть ошибка Поставщика или FAILED |
| `WAITING` | Успешной проверки фото ещё не было (lastScanAt отсутствует) |
| `SYNCED` | Иначе: проверка состоялась, все обнаруженные фото SYNCED |

Причины (issues) сопровождают любую фазу, по убыванию важности: нет доступа, нет входа, нет адреса, ошибка Поставщика, проверка фото ещё не выполнялась, недоступность Server или временная ошибка Загрузчика, FAILED. «Проверка не выполнялась» показывается при наличии доступа, без ошибки Поставщика и пока он не выполняется: пустой индекс до первого scan не означает, что фото нет. Сохранённая проблема — последний известный итог, а не подтверждённое текущее состояние: ошибка Поставщика показывается, пока он не выполняется и доступ есть; проблема Загрузчика — пока он не выполняется, нет причин первых трёх видов и toUpload > 0. Успешное завершение Worker само по себе не даёт SYNCED. Время возникновения сохранённой проблемы, lastScanAt и lastUploadAt входят в состояние.

<a id="sync-status"></a>

## SyncStatus и состояние работы

SyncStatus — проблема прогона части: `SERVER_UNAVAILABLE`, `ERROR`, `RETRY_SCHEDULED`; успешный прогон проблемы не имеет и снимает её ([SyncRunResults](01-system-model.md#sync-run-results)). Это итог прогона, независимый от SyncState каждой строки: прогон без проблемы может сосуществовать с FAILED и с оставшейся очередью.

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
| Разрешения на Files и Settings | Boolean, полный/частичный/нет доступ | Фактическое состояние Android, перечитывается при открытии экрана и возврате в приложение; не сохраняется |
| Список Files | `List<LocalMedia>` | Все локальные строки |
| Состояние backup на Files | [BackupState](01-system-model.md#backup-state) | Фаза, причины, счётчики и время; разрешение на фото экран передаёт при открытии и каждом возврате |
| Проверка копий на Files | [CameraCheck](01-system-model.md#camera-check) и счётчики набора | Читается из Room при каждом изменении; закрытие экрана проверку не останавливает |
| Сообщение операции | String? | Результат явного действия/ошибки для UI |

Устойчивой истории ошибок, byte progress и UI управления retry нет. Поведение — [FILES-LOCAL](05-file-library.md#files-local).

<a id="upload-session"></a>

## UploadSession

**Компонентное представление:** Server — самостоятельная запись `upload_sessions` и служебные файлы сборки/блока. **Availability: SERVER_ONLY.** Android пока не хранит и не использует эту сущность.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | UUID | Устойчивый UUID, созданный Server |
| `userId`, `clientRequestId` | UserId, UUID | Владелец и ключ идемпотентности; пара уникальна |
| `folderId`, `backupSource` | FolderId, UUID? | Исходное назначение; удаление папки не удаляет сессию, публикация повторно проверяет назначение |
| `originalFilename` | String | Запрошенное имя; действуют правила Camera/Library |
| `size`, `checksum` | Integer, Checksum | Ожидаемые размер в байтах и SHA-256 полного файла |
| `confirmedOffset` | Integer | Подтверждённые байты; место начала следующего блока |
| `state` | String | UPLOADING, FINALIZING, COMPLETED, CANCELLED, FAILED, EXPIRED |
| `createdAt`, `lastActivityAt`, `finishedAt` | Instant, Instant, Instant? | UTC, микросекунды; GET не меняет время активности |
| `result`, `error` | RemoteFile?, String? | Сохранённая квитанция FileItemDto либо терминальная причина; квитанция не меняется при последующих операциях Library |

Сервер также хранит внутренний путь незавершённой публикации и признак физической очистки. Они не возвращаются клиенту. Служебные байты находятся в `.upload-sessions` под storage root, не являются RemoteFile/StoredObject и недоступны Library, download и sharing. После успеха FileItem, StoredObject и результат сессии фиксируются вместе. После терминального перехода запись сохраняется 30 суток, либо дольше до успешной очистки временных байтов.

Протокол — [BACKUP-RESUMABLE](04-backup-and-sync.md#backup-resumable), публикация и восстановление — [SYSTEM-UPLOAD-RECOVERY](07-system-rules-and-recovery.md#system-upload-recovery).

<a id="error-response"></a>

## ErrorResponse

Один из форматов ответа Server при ошибке; не все framework/IO отказы используют его.

**Компонентное представление:** Server — ErrorResponse; Android форматирует известные поля для сообщения, не сохраняет структурированную историю.

| Поле | Системный тип | Значение |
| --- | --- | --- |
| `id` | String | UUID ошибки, не идентификатор сущности или sync attempt |
| `code` | String | Прикладной код ошибки; FOLDER_MANAGED — операция над деревом активного источника не разрешена ([FOLDERS-MANAGED-TREE](05-file-library.md#folders-managed-tree)) |
| `message` | String | Описание |
| `fieldErrors` | `Map<String,String>?` | Ошибки отдельных входных полей |

ErrorResponse не тождественен FailureReason или SyncStatus. Ответы STUB содержат message без этого полного формата. Диагностика — [SYSTEM-DIAGNOSTICS](07-system-rules-and-recovery.md#system-diagnostics).
