# Findings — PhotoCloud System As-Is v1 Review

Четыре finding, все MINOR. BLOCKER 0, MAJOR 0.

Каждый finding опирается на конкретный Server section/ID, Android section/ID либо на внутреннее расхождение System пакета. Findings без source evidence не создавались. Стилистические замечания не оформлялись как findings.

Обозначения источников те же, что в проверяемом пакете: `Snn` — файл nn в `PhotoCloudServer/docs/spec/server-as-is/`, `Ann` — файл nn в `PhotoCloudClient/docs/spec/android-as-is/`.

---

## REV-SYS-001

**Severity:**
MINOR

**Type:**

* RISK_ERROR

**Location:**

[15-system-risks.md](../../spec/system-as-is/15-system-risks.md) — реестр `SYS-RISK-001…018` целиком; ближайшая по теме строка `SYS-RISK-013`.

**Current statement:**

`SYS-RISK-013` описывает «Logout/session concurrency и незавершённый фон» с последствием «Offline logout может сохранить pair; поздний refresh может вернуть старую; one-time/cancel не барьер side effects; server access живёт до exp» и ссылается на `RISK-SRV-003/004`, `RISK-AND-007/009/010/025`.

Отдельно [06-auth-session-lifecycle.md](../../spec/system-as-is/06-auth-session-lifecycle.md) верно фиксирует сам факт: «Access principal и roles берутся из актуального User, но enabled/banned повторно в access/refresh не проверяются. Reset пароля не отзывает ранее выданные токены. Это system-visible окно доступа, не новый auth policy requirement.» А [05-effective-api-contract.md](../../spec/system-as-is/05-effective-api-contract.md) фиксирует «password required, 4..20; Android не имеет аналогичной length/email validation до запроса».

Ни одна из 18 строк `SYS-RISK` не несёт последствия этих двух фактов.

**Frozen evidence:**

* `RISK-SRV-002` (S16, Severity **HIGH**): «Enabled/banned повторно не проверяются access/refresh; reset не отзывает существующие токены» → «Смена пароля или ban не прекращают ранее полученный доступ». Подтверждено `VER-SRV-013/014`.
* `RISK-SRV-005` (S16, Severity **HIGH**): «Нет active rate limiter; password policy только 4..20» → «Перебор, SMTP abuse и рост token/code rows; допустимы слабые пароли».
* S06: «Enabled/banned при этом не проверяются; они проверяются только login»; «Reset пароля меняет хеш, не отзывая токены»; «Password length 4..20 действует на register/login/reset; complexity rule не применяется».
* A07: клиент не валидирует формат email и длину/trim пароля; A16 `AND-AUTH-002`. Плечо Android лежит прямо на используемом auth path (`POST /auth/login`, `POST /auth/register` — 2 из 10 используемых operations, [05](../../spec/system-as-is/05-effective-api-contract.md)).
* A18 `RISK-AND-003` (BODY logging пропускает password/token bodies) и `RISK-AND-004` (HTTP без TLS) уже отражены в `SYS-RISK-011`/`SYS-RISK-012` — то есть смежные части того же credential-периметра в System реестре присутствуют, а эти две отсутствуют.

**Issue:**

Оба факта являются end-to-end и boundary-significant, а не component-only internals: они относятся к общему auth периметру, который защищает весь архив пользователя, и находятся на Android-используемом пути. Их системное последствие — Android-клиент продолжает отправлять личные media с access-токеном учётной записи, которую сервер уже заблокировал или пароль которой уже сменён, и сам архив защищён паролем длиной от четырёх символов без ограничения частоты попыток.

Пакет корректно приводит **факты** в 05 и 06, но не переводит их в **последствие** в реестре рисков, хотя именно реестр рисков заявлен местом для «возможных негативных последствий As-Is». Это не потеря факта (он есть) и не искажение источника, а неполнота классификации в SYS-RISK.

Отмечу отдельно: исключение `RISK-SRV-029` (HIGH, буферизация download-ответа) и `RISK-SRV-013` (HIGH, move → 500) из System реестра, напротив, обосновано — Android не использует ни download, ни move, и A17 это прямо оговаривает. Претензия относится только к `RISK-SRV-002`/`RISK-SRV-005`.

**Minimal correction:**

Расширить существующий `SYS-RISK-013` одним предложением последствия и добавить `RISK-SRV-002`/`RISK-SRV-005` в его Sources, либо добавить одну новую строку `SYS-RISK-019` вида «Ограничения credential/account boundary» с последствием «ban и смена пароля не прекращают ранее полученный access; отсутствие rate limiting при пароле 4..20 допускает перебор учётной записи, владеющей всем архивом» и Sources `[S06]`; `RISK-SRV-002/005`; `[A07]`; `AND-AUTH-002`. Новые IDs существующих не переиспользуют. Изменение локально и не затрагивает остальные 18 строк.

---

## REV-SYS-002

**Severity:**
MINOR

**Type:**

* OPEN_QUESTION_ERROR

**Location:**

[16-system-open-questions.md](../../spec/system-as-is/16-system-open-questions.md) — реестр `SYS-OPEN-001…021` и раздел «Consolidation boundary» в конце файла.

**Current statement:**

Раздел «Consolidation boundary» перечисляет намеренно не перенесённые Server OPEN: «Server-only low-level mapping/schema details, framework download Range (Android download не использует), folder tree internals и package maintenance не перенесены отдельными System OPEN. Server sharing/account-delete/profile decisions не расширены до выдуманных Android flows.»

`OPEN-SRV-012` и `OPEN-SRV-017` не представлены ни одним `SYS-OPEN` и не попадают ни под одну из перечисленных категорий исключения.

**Frozen evidence:**

* `OPEN-SRV-012` (S17, PRODUCT_DECISION) — «Конфликты имён»: «Sanitize, case-insensitive precheck вне CAMERA, без auto-rename/overwrite/Unicode normalization» → «Как трактуются совпадения имён, Unicode/case и конфликт при повторе с новым именем?»
* Системная значимость `OPEN-SRV-012` подтверждается самим пакетом: [05](../../spec/system-as-is/05-effective-api-contract.md) содержит строку «Новые bytes, занятое name вне CAMERA | 409 CONFLICT | FAILED HTTP_409, без auto restore», а [09](../../spec/system-as-is/09-error-retry-recovery.md) — «Upload 400/409/413 … Нет automatic restore … 409 name conflict вне CAMERA, не duplicate checksum». По A09/A11 `HTTP_409` — терминальный FAILED без auto restore и без manual retry, то есть безвозвратно непереданное фото. Условие возникновения — omitted target с default FILES (`SYS-MISMATCH-002`), то есть ветка уже описанной system boundary.
* `OPEN-SRV-017` (S17, PRODUCT_DECISION) — «Пароль и ограничения запросов»: «Password 4..20; rate limiting не подключён» → «Какая политика паролей и лимитов обращений закрепляется продуктом?» Это продуктовое решение того же периметра, о котором идёт речь в REV-SYS-001.
* Для сравнения: покрытие остальных реестров исключительно полное — перенесены 29 из 30 `OPEN-AND` (не перенесён только `OPEN-AND-024` о назначении UI-заглушек, что корректно как COMPONENT_ONLY) и 19 из 30 `OPEN-SRV`, причём 9 из 11 непокрытых прямо подпадают под объявленные исключения (`OPEN-SRV-007/009/010/014/015/019/020/029` и близкий к `SYS-OPEN-001/002` `OPEN-SRV-006`).

**Issue:**

Два продуктовых решения, лежащих на Android-используемом пути, отсутствуют и в реестре, и в списке обоснованных исключений. Поскольку файл 16 явно декларирует границу консолидации, читатель не может отличить «сознательно оставлено компоненту» от «пропущено». Для `OPEN-SRV-012` пропуск заметнее: его следствие — терминальный `HTTP_409` — в пакете описано дважды, а вопроса о его продуктовом смысле нет, тогда как `SYS-OPEN-009` спрашивает об ожиданиях ручного восстановления failures в целом и мог бы его вместить.

Review не отвечает на эти вопросы и не требует их переноса в конкретную формулировку — достаточно одного из двух вариантов ниже.

**Minimal correction:**

Любой из двух вариантов, на выбор автора пакета:

1. Добавить ссылку на `OPEN-SRV-012` в Sources существующего `SYS-OPEN-009` (или `SYS-OPEN-005`) и ссылку на `OPEN-SRV-017` в Sources существующего `SYS-OPEN-008`, дополнив «Current As-Is» соответствующих пунктов одной фразой; либо
2. Дополнить раздел «Consolidation boundary» одним предложением, прямо называющим `OPEN-SRV-012` и `OPEN-SRV-017` как оставленные в component registry, с указанием причины.

Новых `SYS-OPEN` создавать не обязательно. Существующие IDs и типы не меняются.

---

## REV-SYS-003

**Severity:**
MINOR

**Type:**

* TRACEABILITY_ERROR

**Location:**

[15-system-risks.md](../../spec/system-as-is/15-system-risks.md) — колонка `Sources` всех 18 строк.

**Current statement:**

Каждая из 18 строк реестра заканчивается одинаковым набором ссылок: `[S16]`, затем `[A18]`, `[A16]`, `[A17]`. Например `SYS-RISK-017` (email onboarding/recovery) и `SYS-RISK-018` (upload resources/capacity) обе несут `[A16]` (Android capability matrix) и `[A17]` (server–client consistency) наравне со всеми остальными строками.

Семь строк из восемнадцати — `SYS-RISK-002`, `004`, `006`, `008`, `009`, `010`, `014`, `015` — ссылаются на `[S16]` без указания конкретного `RISK-SRV-*` ID.

**Frozen evidence:**

Конкретные, точно соответствующие Server risk IDs существуют и не указаны:

* `SYS-RISK-004` («Logical existing/duplicate при missing bytes») — прямое соответствие `RISK-SRV-010` (S16, HIGH): «Exists и duplicate upload проверяют только DB; download не пересчитывает checksum» → «Existing/200 не подтверждают физическую сохранность и не восстанавливают bytes». Указаны только `SRV-FILE-003`, `SRV-SYNC-002` (capability IDs) и общий `[S16]`.
* `SYS-RISK-005` («DB/FS/response/local mark неатомарны») и `SYS-RISK-008` («Нет reverse reconciliation и delete propagation») — соотносятся с `RISK-SRV-012` (HIGH, owner-delete уничтожает все references; прямой SQL-delete не вызывает FS cleanup). Не указан.
* `SYS-RISK-008` — также соотносится с `RISK-SRV-036` (MEDIUM: «нет delta/tombstones/devices; CAMERA ID отсутствует до default upload»), это ближайший Server risk по теме. Не указан.
* `SYS-RISK-006` («Изменение bytes между hash/upload») — Server-плечо опирается на `SRV-MEDIA-001` и `[S07]`; конкретного `RISK-SRV-*` здесь действительно нет, и общая ссылка `[S16]` в этой строке вводит в заблуждение сильнее, чем её отсутствие.

Из 38 `RISK-SRV-*` System реестр адресно ссылается на 11; из 39 `RISK-AND-*` — на 31. Асимметрия сама по себе не дефект (Server risks в большей части component-only), но она усиливает эффект: там, где релевантный Server ID есть, он часто заменён общим указанием на файл из 38 рисков.

**Issue:**

Ссылка на файл целиком вместо конкретного ID не позволяет проверить перенос: чтобы подтвердить `SYS-RISK-004`, читателю приходится самостоятельно искать соответствующий риск среди 38. Одновременно повторение одного и того же хвоста `[A18] [A16] [A17]` на всех 18 строках делает эти ссылки неразличающими — они не сообщают, какой именно Android материал обосновывает конкретную строку.

Это дефект прослеживаемости, а не фактическая ошибка: все проверенные утверждения самих рисков подтверждаются frozen evidence, и ни одной битой ссылки в пакете нет. Severity MINOR именно поэтому.

Тот же характер, но меньший масштаб, имеет одна ссылка в [13](../../spec/system-as-is/13-system-capability-matrix.md): `SYS-CAP-012` («Automatically retry backup») ссылается на `SRV-API-002` (unified controlled errors, PARTIAL), тогда как релевантен только `SRV-API-004` (Idempotency-Key/retry queue NOT PRESENT), указанный там же. Исправляется тем же приёмом.

**Minimal correction:**

В колонке `Sources` файла 15 указать конкретные `RISK-SRV-*` там, где они существуют — как минимум `RISK-SRV-010` в `SYS-RISK-004`, `RISK-SRV-012` в `SYS-RISK-005` и `SYS-RISK-008`, `RISK-SRV-036` в `SYS-RISK-008` — и убрать общий `[S16]` из строк, где адресного Server risk нет. Сократить хвост `[A18] [A16] [A17]` до фактически использованных для каждой строки. Дополнительно снять `SRV-API-002` из `SYS-CAP-012` в файле 13. Формулировки самих рисков, их severity и IDs не меняются.

---

## REV-SYS-004

**Severity:**
MINOR

**Type:**

* FLOW_ERROR
* INTERNAL_CONTRADICTION (локальная, внутри одной строки реестра)

**Location:**

[18-system-flow-index.md](../../spec/system-as-is/18-system-flow-index.md) — `SYS-FLOW-001 — Login and session establishment`, поля `Entry condition` и `Failure/recovery`.

**Current statement:**

| Field | Текст пакета |
| --- | --- |
| Entry condition | «Настроен URL; activated/non-banned user вводит credentials» |
| Failure/recovery | «Login failure показывает error; previous pair сама не clear; повтор login новая refresh row» |

**Frozen evidence:**

* S06: «Login читает user, проверяет password, enabled и banned; любой из этих отказов даёт 401 INVALID_CREDENTIALS.» То есть неактивированный и заблокированный пользователь **входит** в этот flow и получает 401 — activation и отсутствие ban являются условиями успеха, а не условиями входа.
* S05, `POST /api/v1/auth/login`: «401 INVALID_CREDENTIALS для unknown email, bad password, disabled/banned».
* A07: «AuthRepository.login выполняет plain request и требует body с двумя nonblank token strings. … Неудачный login сам не очищает предыдущую пару.» Клиент отправляет запрос, не проверяя ни формат email, ни состояние аккаунта.
* Сам System пакет описывает это верно в других местах: [06](../../spec/system-as-is/06-auth-session-lifecycle.md) — «Login проверяет password/enabled/banned»; [05](../../spec/system-as-is/05-effective-api-contract.md) — «login invalid/disabled/banned — 401». Расхождение локально и ограничено одной строкой файла 18.

**Issue:**

`Entry condition` смешивает предпосылку входа в flow с предпосылкой его успешного завершения. В прочтении «flow начинается только для activated/non-banned пользователя» собственная строка `Failure/recovery` того же пункта становится недостижимой: если бы вход был ограничен активированными и незаблокированными пользователями, ветки «Login failure» в этом flow не существовало бы. Кроме того, такое чтение скрывает системно значимый факт, что disabled/banned/unknown email объединены сервером в один неразличимый 401 — единственный наблюдаемый клиентом результат.

Это единственная неточность entry condition среди двенадцати flows; остальные одиннадцать проверены и корректны, и ни один flow не вводит поведение, отсутствующее в основных разделах 05–12.

**Minimal correction:**

Заменить `Entry condition` на формулировку, описывающую только вход, например: «Настроен URL; пользователь отправляет email/password». Условие успеха уже выражено полем `Success end-state`; при необходимости дополнить `Failure/recovery` фразой «unknown email, неверный пароль, disabled и banned неразличимы для клиента как 401». Правка касается одной-двух ячеек и не затрагивает остальные flows, их IDs или источники.
