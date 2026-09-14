# Библиотека файлов и папок

## 1. Экран Files в Android

**Status:** AS-IS  
**Availability:** ANDROID_ONLY  
**System:** [FILES-LOCAL](../system/05-file-library.md#files-local)

Экран Files сейчас не является удалённой фотогалереей. Он показывает известные Android фотографии, их состояние в очереди backup, краткие ошибки и общий результат последней обработки.

Пользователь видит состояние backup, но не содержимое удалённой библиотеки Server. На этом экране нет preview удалённых фотографий, ручной загрузки или скачивания, управления папками, Sync now/Retry/Cancel и полноценной истории синхронизации.

## 2. Просмотр и скачивание удалённой библиотеки

**Status:** AS-IS  
**Availability:** SERVER_ONLY  
**System:** [FILES-LIST](../system/05-file-library.md#files-list) · [FILES-METADATA](../system/05-file-library.md#files-metadata) · [FILES-DOWNLOAD](../system/05-file-library.md#files-download) · [FOLDERS-BROWSE](../system/05-file-library.md#folders-browse)

Server уже умеет выдавать список удалённых файлов, сведения о конкретном файле, содержимое файла для скачивания и прямые подпапки выбранной папки.

Android эти возможности пользователю сейчас не предоставляет, поэтому удалённая библиотека как пользовательская галерея в приложении отсутствует.

## 3. Управление папками удалённой библиотеки

**Status:** AS-IS  
**Availability:** SERVER_ONLY  
**System:** [FOLDERS-CREATE](../system/05-file-library.md#folders-create) · [FOLDERS-RENAME](../system/05-file-library.md#folders-rename) · [FOLDERS-MOVE](../system/05-file-library.md#folders-move) · [FOLDERS-DELETE](../system/05-file-library.md#folders-delete)

Server позволяет создавать пользовательские папки, переименовывать и перемещать их, а пустые пользовательские папки — удалять. Рекурсивного удаления непустого дерева сейчас нет.

Android пользовательского интерфейса для этих операций не предоставляет.

## 4. Организация удалённых файлов

**Status:** AS-IS  
**Availability:** SERVER_ONLY  
**System:** [FILES-RENAME](../system/05-file-library.md#files-rename) · [FILES-MOVE](../system/05-file-library.md#files-move)

Server позволяет переименовать удалённый файл или переместить его в другую папку. Эти изменения относятся к серверной библиотеке и автоматически не меняют исходную фотографию или её локальную запись на Android.

## 5. Копирование удалённого файла

**Status:** AS-IS  
**Availability:** SERVER_ONLY  
**System:** [FILES-COPY](../system/05-file-library.md#files-copy)

Server позволяет создать независимую копию удалённого файла. Это новая серверная копия, а не общая ссылка на исходный объект и не предоставление доступа другому пользователю.

## 6. Удаление удалённого файла

**Status:** AS-IS  
**Availability:** SERVER_ONLY  
**System:** [FILES-DELETE](../system/05-file-library.md#files-delete)

Server позволяет удалить файл из удалённой библиотеки. Это действие относится к серверной копии и автоматически не удаляет исходную фотографию на Android.

## 7. Корзина, восстановление и версии

**Status:** AS-IS  
**Availability:** NOT_PRESENT  
**System:** [FILES-TRASH](../system/05-file-library.md#files-trash)

Пользовательской корзины, восстановления удалённых файлов и управляемой истории версий сейчас нет. Удаление серверного файла не является перемещением в доступную пользователю корзину.

## 8. Двусторонняя синхронизация библиотеки

**Status:** AS-IS  
**Availability:** NOT_PRESENT  
**System:** [FILES-RECONCILE](../system/05-file-library.md#files-reconcile) · [BACKUP-DELETE](../system/04-backup-and-sync.md#backup-delete)

Изменения удалённой библиотеки не синхронизируются обратно в Android. Если файл на Server переименован, перемещён или удалён, обычное локальное состояние фотографии автоматически не пересматривается.

Аналогично удаление локальной фотографии не удаляет серверную копию. Поэтому локальная очередь и удалённая библиотека могут со временем расходиться; полноценного двустороннего протокола синхронизации сейчас нет.
