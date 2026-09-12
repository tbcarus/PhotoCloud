# System open questions

Реестр содержит только system-significant вопросы. Ответы, fixes и To-Be items не назначены. `SYSTEM_ARCHITECTURE_DECISION` соответствует архитектурным OPEN компонентов; `INTEGRATION_DECISION` отделяет решение о совместном contract от runtime unknown. `DEPLOYMENT_RUNTIME_UNKNOWN` объединяет deployed Server и Android runtime unknown с сохранением исходных ID; OS/provider вопросы отдельно имеют `OS_FRAMEWORK_UNKNOWN`.

## SYS-OPEN-001

**Type:** PRODUCT_DECISION

**Current As-Is:** SYNCED — local existing либо positive upload ID; logical record не удостоверяет bytes и ID optional.

**Question:** Какой смысл имеет пользовательское подтверждение «синхронизировано», в том числе при потерянном ответе или missing bytes?

**Why it matters:** Определяет интерпретацию client belief, logical и physical evidence.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-005; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-020.

## SYS-OPEN-002

**Type:** PRODUCT_DECISION

**Current As-Is:** Local disappearance удаляет только row; server mutations не наблюдаются; hard delete без tombstones, старая remote copy после local change остаётся.

**Question:** Как трактуются направление синхронизации, retention изменённых/удалённых фото и роль Android при remote delete/move?

**Why it matters:** Связывает local и remote lifecycle, не меняя известное отсутствие propagation.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-001/008; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-012/013.

## SYS-OPEN-003

**Type:** PRODUCT_DECISION

**Current As-Is:** Одна local DB/pair без account/host/device namespace; Server owner-scoped и разрешает несколько refresh rows.

**Question:** Каков ожидаемый сценарий нескольких аккаунтов, серверов и устройств одного пользователя?

**Why it matters:** Устанавливает scope ожиданий при mismatch001, не назначает storage schema.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-002/003; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-014/027.

## SYS-OPEN-004

**Type:** SYSTEM_ARCHITECTURE_DECISION

**Current As-Is:** Local ID/checksum/remote ID различны; content/session writes и scan/stages не одна transaction.

**Question:** Какие границы согласованности local asset ↔ remote identity считаются частью системной модели при смене session и concurrent processing?

**Why it matters:** Объединяет identity и concurrency decision; известные interleavings не объявляются unknown.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-003; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-028.

## SYS-OPEN-005

**Type:** INTEGRATION_DECISION

**Current As-Is:** CAMERA разрешается дважды; first absence bootstrap согласован, second lookup Error даёт default routing по bytes.

**Question:** Какой contract target/первого CAMERA pre-check закрепляется для системы при отсутствии CAMERA или неуспехе повторного lookup?

**Why it matters:** Отделяет известный lifecycle и mismatch002 от неназначенного решения о target semantics.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-004/011; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-011; INT-AND-002; [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).

## SYS-OPEN-006

**Type:** PRODUCT_DECISION

**Current As-Is:** Android Images/DCIM, Server разные MIME и service100MiB; client size guard/video discovery нет.

**Question:** Какой охват типов/локальных папок/размеров заявлен пользователю и как интерпретируются media за его пределами?

**Why it matters:** Server acceptance не равен охвату device backup.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-011/023; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-011/022.

## SYS-OPEN-007

**Type:** PRODUCT_DECISION

**Current As-Is:** Local dates/name/MIME и server EXIF/fallback/normalized name не mirror; explicit original/GPS access отсутствует.

**Question:** Какие общие время, имена и metadata видит пользователь и каковы ожидания побайтовой копии и передачи геопозиции?

**Why it matters:** Различает presentation/privacy decision и отдельно неизвестные OS bytes.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-013; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-021/026.

## SYS-OPEN-008

**Type:** PRODUCT_DECISION

**Current As-Is:** Access20min/refresh7days; no refresh rotation; logout отзывает refresh, client offline clear не гарантирован, refresh5xx clear известен.

**Question:** Какой смысл завершения logout, истечения session и момента прекращения доступа закреплён пользователю?

**Why it matters:** Связывает local exit, server revoke и background effects без ответа о будущей реализации.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-016; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-015; INT-AND-004; [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md).

## SYS-OPEN-009

**Type:** PRODUCT_DECISION

**Current As-Is:** CHECKSUM_IO retry ограничен, terminal FAILED/manual controls отсутствуют; фон зависит от cold URL и OS.

**Question:** Каковы ожидания ручного восстановления failures, автономности backup, задержки и уведомлений?

**Why it matters:** Кодовая policy установлена; обещания пользователю остаются решением.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-024; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-018/019.

## SYS-OPEN-010

**Type:** PRODUCT_DECISION

**Current As-Is:** Client HTTP IPv4/port; CONNECTED допускает metered network, periodic требует batteryNotLow.

**Question:** Какой transport/deployment сценарий и расход мобильного трафика/батареи заявляются продуктом?

**Why it matters:** Определяет пользовательские условия передачи личного media; фактический LAN/proxy отдельно unknown.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): Нет отдельного тождественного решения; [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md) boundary; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-016/017.

## SYS-OPEN-011

**Type:** PRODUCT_DECISION

**Current As-Is:** Android register/login есть, встроенных confirm/reset нет; Server resend/reset page501 и delivery tracking нет.

**Question:** Как пользователь завершает onboarding и recovery доступа, включая недоставленное письмо?

**Why it matters:** Доступ к backup зависит от внешнего mail flow и текущих заглушек.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-018; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-023.

## SYS-OPEN-012

**Type:** PRODUCT_DECISION

**Current As-Is:** BODY logs содержат sensitive data; client outcome в памяти, server/Android общей persistent error history нет.

**Question:** Какие диагностические данные, доступ к логам и сроки хранения относятся к системной эксплуатации?

**Why it matters:** Соединяет privacy и возможность расследовать end-to-end failure.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S13](../../../../PhotoCloudServer/docs/spec/server-as-is/13-observability.md) observability; OPEN-SRV-022 (retention boundary); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-025.

## SYS-OPEN-013

**Type:** PRODUCT_DECISION

**Current As-Is:** Server tests не удостоверяют Android↔Server; Android product automated coverage отсутствует, реальная нагрузка/совместимость не измерена.

**Question:** Каковы поддерживаемые устройства, объёмы архива и критерии продуктовой проверки/доступности?

**Why it matters:** Отделяет static As-Is от runtime acceptance и capacity expectations.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-023; [S14](../../../../PhotoCloudServer/docs/spec/server-as-is/14-testing-current-state.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-009/029.

## SYS-OPEN-014

**Type:** DEPLOYMENT_RUNTIME_UNKNOWN

**Current As-Is:** Frozen APIs согласованы; live revision/DB/proxy/LAN доступ и framework error statuses/body не проверены.

**Question:** Какой endpoint/revision, применённое состояние DB и transport/forwarded-origin реально действуют, какие error/binding responses наблюдаются?

**Why it matters:** Определяет применимость frozen wire к реальной паре Android+Server; normal folderId text/plain не неизвестен.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-025/027/028/030; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-010.

## SYS-OPEN-015

**Type:** DEPLOYMENT_RUNTIME_UNKNOWN

**Current As-Is:** DB и FS раздельны; app repair/backup jobs нет; root/temp/cwd и внешняя эксплуатация не установлены.

**Question:** Каковы реальные storage volumes/свойства FS и существует ли внешний процесс согласованного backup, обнаружения и восстановления потерянных bytes; кто им управляет?

**Why it matters:** Наличие внешнего recovery не выводится ни из app absence, ни из existing200.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-022/026; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-030.

## SYS-OPEN-016

**Type:** OS_FRAMEWORK_UNKNOWN

**Current As-Is:** Observer в памяти, cold URL empty известен; configured requests не обещают deadlines.

**Question:** Каковы фактические сроки и последовательности scheduling/events после reboot, force-stop, Doze и изменения constraints на поддерживаемых OS/OEM?

**Why it matters:** Определяет наблюдаемую задержку; не переоткрывает установленный cold-start факт.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md): device scheduling не на Server; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-001.

## SYS-OPEN-017

**Type:** OS_FRAMEWORK_UNKNOWN

**Current As-Is:** Images permission без отдельной selected-photo модели; successful snapshot определяет stale DELETE.

**Question:** Какую видимость и permission state даёт текущая сборка в selected-access режиме конкретной ОС?

**Why it matters:** Partial visibility может удалить local history при сохранённых originals и remote copies.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md): device visibility неизвестна Server; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-002.

## SYS-OPEN-018

**Type:** OS_FRAMEWORK_UNKNOWN

**Current As-Is:** Обычный content URI без location permission/setRequireOriginal; Server получает именно доставленный stream.

**Question:** Какие bytes/GPS возвращает OS/provider при фактических grants для geotagged media?

**Why it matters:** Nullable server GPS не доказывает redaction; полнота копии зависит от исходного stream.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md): metadata по received bytes; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-003.

## SYS-OPEN-019

**Type:** DEPLOYMENT_RUNTIME_UNKNOWN

**Current As-Is:** BODY logger может дополнительно читать stream; contentLength из SIZE; cancellation не связана с Call.cancel.

**Question:** Каковы фактические read/buffering/length effects и момент прекращения upload side effects при cancel/constraint loss на установленном runtime?

**Why it matters:** Ресурсы и повторяемый IO failure влияют на queue; факт отдельного URI read уже установлен.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md)/[S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md): accepted stream и upload boundary; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-004/005/007.

## SYS-OPEN-020

**Type:** DEPLOYMENT_RUNTIME_UNKNOWN

**Current As-Is:** Tokens/Room/settings сохраняются раздельно; device/account namespace нет, backup restore и исторические DB состояния не проверены.

**Question:** Как восстанавливаются key/pair/local IDs/queue на реальных установках после restore/переноса/upgrade?

**Why it matters:** Утрата или перенос local belief может изменить полноту backup без изменения Server records.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md)/[S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md): remote identity не local installation; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-006/008.

## SYS-OPEN-021

**Type:** SYSTEM_ARCHITECTURE_DECISION

**Current As-Is:** Server хранит local bytes отдельно от DB; приложениями не задана согласованная backup/restore и retention/RPO policy.

**Question:** Какая физическая storage boundary, политика согласованного backup/retention и допустимое окно потери данных относятся к системе?

**Why it matters:** Отделяет архитектурную policy от вопроса о реально существующем внешнем процессе в SYS-OPEN-015.

**Sources:** [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md): OPEN-SRV-021/022; [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md): OPEN-AND-030 (внешняя recovery boundary).


## Consolidation boundary

Server OPEN о «ещё не установленном Android behaviour» не скопированы как фактическая неизвестность: identity, routes, CAMERA bootstrap, no rotation, existing/duplicate и retry уже установлены двумя inputs. Сохранены только вопросы решения или runtime evidence.

Server-only low-level mapping/schema details, framework download Range (Android download не использует), folder tree internals и package maintenance не перенесены отдельными System OPEN. Server sharing/account-delete/profile decisions не расширены до выдуманных Android flows. Полный компонентный backlog остаётся в authoritative component OPEN registries.
