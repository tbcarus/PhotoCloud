# Сводка системы

PhotoCloud связывает фотографии Android-устройства с персональным файловым хранилищем сервера. Android находит доступные изображения, ведёт локальную очередь и отправляет содержимое. Server устанавливает principal, управляет пользовательскими папками и логическими записями, вычисляет свойства принятых байтов и хранит их в локальном filesystem. PostgreSQL хранит пользователей, сессии refresh и каталог объектов. Ни один из компонентов не хранит общую атомарную «сессию синхронизации» между устройством и сервером.

Текущий end-to-end контур — **one-way backup/upload**. Он не образует bidirectional synchronization: Android не получает remote changes, не строит remote gallery, не скачивает серверные копии и не передаёт удаления оригиналов. Слово backup здесь обозначает направление копирования. Оно не удостоверяет проверенную сохранность каждой копии или наличие внешнего резервирования сервера.

## Участники и охват

Android сканирует external MediaStore Images с локальным относительным путём `DCIM/%`, включая подпапки. Видео и изображения вне этой области не входят в текущий discovery. ОС определяет, какие строки и потоки доступны с текущими permissions. Клиент не редактирует и не удаляет оригиналы, не создаёт private snapshot байтов и не фиксирует их неизменность на время обработки.

Локальный persisted record содержит MediaStore ID, URI, metadata, checksum, status, optional serverFileId и failure metadata. Это индекс установки приложения, а не зеркало PostgreSQL. Текстовый Files monitor показывает локальные записи; его наличие не означает облачную галерею. Пользователь настраивает HTTP IPv4/port, входит в аккаунт и предоставляет media permission. Settings/Profile destinations остаются заглушками.

Server имеет более широкий файловый API: listing, download, rename, move, copy, delete и управление USER-папками. Android использует лишь десять HTTP operations. Неиспользование остальных возможностей само по себе не является mismatch. Сервер принимает разные типы байтов; это также не создаёт Android video backup.

## Вход и срок сессии

Регистрация отправляет email/password серверу. Сервер сохраняет disabled user и activation code, вызывает SMTP; подтверждение ссылки включает аккаунт. Android показывает ответ регистрации, но не делает auto-login и не имеет встроенного confirm/resend/reset flow. Успешное сообщение об отправке письма не доказывает доставку. Reset JSON API существует на Server, однако email reset page — 501.

Login проверяет credentials и enabled/banned на Server. Успех выдаёт access на 20 минут и refresh на 7 дней. Android требует две nonblank строки и хранит пару в encrypted preferences; пароль в preferences не сохраняется. Наличие пары в UI означает локальное logged-in состояние, а не доказанную валидность серверной сессии.

Protected requests получают текущий access Bearer. При 401 клиент может запросить новый access plain refresh-запросом и один раз повторить исходный запрос. Server читает persisted refresh, проверяет revoke/подпись/тип/срок/subject и выдаёт access без ротации или продления refresh. Любой non-2xx refresh, включая временную 5xx, очищает пару на Android; network IOException сама по себе её не очищает. Это один из четырёх системных mismatches.

Logout отзывает refresh на Server; access может действовать до exp. Android очищает пару после HTTP response, в том числе ошибочного. Ошибка сети до clear может оставить сессию локально. Logout не очищает media DB; ранее поставленная one-time работа не отменяется этим действием. Поэтому logout, server revoke и прекращение всех фоновых side effects — разные события.

## Путь новой фотографии

После появления доступного изображения UI scan либо фоновый worker получает полный snapshot MediaStore. Новый ID становится локальным PENDING. Размер или mtime, отличающиеся от предыдущего scan, сбрасывают checksum, serverFileId и обработку. Затем Android открывает URI, вычисляет SHA-256 и сохраняет CHECKSUM_READY.

Worker начинает pipeline только после наличия in-memory URL и успешного public ping. Он выполняет restore eligible failures, scan, hashing, pre-check и upload последовательно в рамках одного прогона. Это не глобальная блокировка: periodic worker, one-time worker и UI scan могут пересекаться.

Для pre-check Android получает ROOT и ищет первого прямого потомка с типом CAMERA. GET root лениво создаёт только ROOT. CAMERA не обязана существовать до первого default upload изображения или видео. Если её нет, клиент переводит ready hashes в PENDING_UPLOAD без запроса existing. Если CAMERA найдена, клиент отправляет до 500 уникальных checksum, полученных максимум из 500 локальных строк.

Server отвечает existing/missing для logical FileItem текущего user в конкретной folder. Existing не резервирует объект и не проверяет physical bytes. Android записывает SYNCED только обновлением status; serverFileId обычно остаётся null. Missing переводит строку в очередь upload.

Перед отправкой клиент повторно разрешает CAMERA. Успех даёт explicit folderId. Отсутствие CAMERA или ошибка этого второго lookup приводят к отсутствию folderId в multipart. Server тогда выбирает CAMERA для полученных IMAGE/VIDEO bytes и FILES для остальных. Решение основано на определённом сервером типе содержимого, а не на локальном MIME header. Local relativePath не превращается в server folder tree.

Upload повторно читает URI. Server принимает поток во временный файл, вычисляет собственные SHA-256/size, определяет MIME и извлекает image metadata best effort. Для нового содержимого он переносит bytes в final storage, затем сохраняет StoredObject и FileItem в DB transaction. Успешный DTO приходит после commit. Android принимает 2xx с положительным logical ID, сохраняет serverFileId и SYNCED; остальные свойства response не сверяет.

Те же user/folder/checksum при duplicate upload возвращают старый DTO через HTTP 200, с прежними именем, metadata и датами. Это успешный путь Android. Duplicate не возвращает 409 и не восстанавливает потерянные bytes. Те же bytes в другой folder создают отдельную logical identity, StoredObject и физическую копию. Вне CAMERA новое содержимое с занятым именем может дать 409; в CAMERA совпадение имени допускается.

## Что означает результат

`SYNCED` — **Android-side local conclusion** после existing либо upload id>0. Из него не следуют известный serverFileId, текущее существование оригинала, повторная remote verification или наличие читаемых server bytes. Положительный ID в response также не связывает локальный checksum с актуально принятыми байтами: клиент их независимо читал и не сравнивает ответный checksum.

Источники истины разделены: MediaStore отвечает за текущую видимость device media, Android DB — за local queue/client belief, Server — за auth, PostgreSQL — за logical remote identity и hierarchy, filesystem — за фактические bytes. Эти authorities не синхронизированы общей транзакцией. Успешный ping не проверяет DB/storage/SMTP readiness. Успех worker может сосуществовать с per-file FAILED и не означает завершённый backup всей библиотеки.

## Ошибки, фон и перезапуск

Per-file checksum I/O допускает ограниченный возврат: count<3, прошло не менее пяти минут, строка выбрана следующим worker. Другие FAILED автоматически не восстанавливаются; manual retry отсутствует. Upload 400/409/413 сохраняет FAILED и позволяет обработать следующую строку. Прочие HTTP, IOException и 2xx без положительного ID возвращают PENDING_UPLOAD, завершают upload run и запрашивают повтор pipeline. Одна повторяемая transient ошибка нового файла может задерживать более старые pending.

Observer работает только в памяти процесса, debounce — три секунды. Periodic request настроен на час с network/battery constraints, но это не SLA запуска. После process death завершённые local writes сохраняются, in-memory URL и observer теряются. Headless worker не восстанавливает URL из settings и заканчивается SERVER_UNAVAILABLE+retry до UI restore. Reboot не устраняет эту границу; реальные сроки OS/OEM исполнения остаются открытыми.

На Server DB и filesystem не имеют общей атомарной transaction. Компенсации ограничены и выполняются best effort. Crash или частичный отказ может оставить orphan bytes либо записи без доступных bytes. Restart не запускает repair/reconciliation. Внешние backup/restore, mounts и durability deployment не установлены frozen inputs.

## Изменения и главные ограничения

Исчезновение ID из успешного локального snapshot удаляет только local record. Это может быть физическое удаление, перемещение за пределы DCIM или потеря видимости. Server copy сохраняется. Изменение size/mtime запускает обработку заново; прежняя remote copy не удаляется. Remote delete/move/rename не меняет автоматически local SYNCED. Система не поддерживает двустороннее разрешение конфликтов, tombstones, resumable upload и подтверждённую end-to-end историю сохранности.

Четыре `SYS-MISMATCH-001…004` сохраняют `INT-AND-001…004`: unscoped Android state при смене account/host; CAMERA pre-check и fallback target; раздельные чтения checksum/upload; очистка session при refresh 5xx. Условия этих веток известны, частота реальных инцидентов не установлена. Дополнительных semantic mismatches и противоречий frozen inputs при составлении не обнаружено.

Системные риски включают пропуск backup при смене scope, потерю автономности после холодного старта, ложную уверенность в physical copy, неатомарные DB/FS side effects, задержку очереди и раскрытие credentials через HTTP/BODY logging. Отсутствие Android product/e2e automated coverage ограничивает доказанность совместного runtime; Server tests не заменяют такой проверки.

OPEN отделены от этих фактов: продуктовый смысл SYNCED и retention, ожидания нескольких устройств/аккаунтов, media/metadata scope, завершение session/recovery, границы согласованности, транспорт deployment, OS visibility/scheduling и внешний процесс восстановления server bytes. Ответы и fixes здесь не назначены. Пакет готов к независимому review, но не заморожен.


## Источники

[S00](../../../../PhotoCloudServer/docs/spec/server-as-is/00-server-as-is-summary.md); [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [S13](../../../../PhotoCloudServer/docs/spec/server-as-is/13-observability.md); [S14](../../../../PhotoCloudServer/docs/spec/server-as-is/14-testing-current-state.md); [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md); [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md); [A01](../../../../PhotoCloudClient/docs/spec/android-as-is/01-system-boundary.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A14](../../../../PhotoCloudClient/docs/spec/android-as-is/14-security-privacy.md); [A15](../../../../PhotoCloudClient/docs/spec/android-as-is/15-testing-current-state.md); [A16](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
