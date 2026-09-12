# Traceability review

Проверено: существование source IDs, релевантность их смысла, прослеживаемость основных system claims, source fingerprints, направление authority и отсутствие зависимости от исторических audit-материалов.

## Сводка

| Проверка | Результат |
| --- | --- |
| Source IDs exist | **PASS** — 0 битых из 171 различного ID |
| Source meaning relevant | **PASS с замечаниями** — 4 weak/misleading ссылки, ни одной неверной |
| Major system claims traceable | **PASS** — все 16 self-check вопросов пакета подтверждаются профильными разделами |
| Source fingerprints | **PASS** — 34 из 34 SHA-256 совпали при пересчёте |
| No component authority inversion | **PASS** — направление authority не нарушено ни в одном разделе |
| No historical audit dependency | **PASS** — исторические audits/reviews/handoff не использованы как источники |
| Markdown links / anchors | **PASS** — 0 битых ссылок, все 7 якорей разрешаются |

## Broken source references

**Ни одной.**

Проверка выполнена извлечением всех вхождений `SRV-*`, `RISK-SRV-*`, `OPEN-SRV-*`, `AND-*`, `RISK-AND-*`, `OPEN-AND-*`, `INT-AND-*` из файлов 00…18 с раскрытием сокращённой записи вида `PREFIX-001/002/003` и диапазонов `PREFIX-001…004`, и сверкой полученных 171 ID с определениями в authoritative реестрах:

| Namespace | Определено во frozen | Адресно упомянуто System | Битых |
| --- | --- | --- | --- |
| `SRV-*` capability (S15) | 77 | 41 | 0 |
| `RISK-SRV-*` (S16) | 38 | 11 | 0 |
| `OPEN-SRV-*` (S17) | 30 | 19 | 0 |
| `AND-*` capability (A16) | 58 | 36 | 0 |
| `RISK-AND-*` (A18) | 39 | 31 | 0 |
| `OPEN-AND-*` (A19) | 30 | 29 | 0 |
| `INT-AND-*` (A17) | 4 | 4 | 0 |

Неполное адресное упоминание само по себе дефектом не является: значительная часть component IDs — internals, STUB и неиспользуемые возможности, сгруппированные в System пакете нарративно (см. [03-component-source-coverage.md](03-component-source-coverage.md)). Существенно, что **ни один** упомянутый ID не отсутствует и не переименован.

Дополнительно проверены все Markdown-ссылки внутри пакета (включая относительные ссылки через `../../../../` в оба component каталога) — битых нет. Все семь используемых внутренних якорей (`#sys-mismatch-001`, `#sys-open-002/008/014/015`, `#source-of-truth`, `#server-only-capability-groups`) соответствуют реальным заголовкам.

## Weak / misleading references

Четыре случая. Ни один не делает утверждение ложным; все касаются точности указания источника.

| № | Место | Проблема | Отнесено к |
| --- | --- | --- | --- |
| 1 | [15](../../spec/system-as-is/15-system-risks.md), колонка `Sources`, все 18 строк | Одинаковый хвост `[S16]`, `[A18]`, `[A16]`, `[A17]` повторяется на каждой строке независимо от релевантности, из-за чего ссылки не различают строки. Например `SYS-RISK-017` (email onboarding) и `SYS-RISK-018` (upload resources) несут `[A17]` наравне со всеми | [REV-SYS-003](01-findings.md#rev-sys-003) |
| 2 | [15](../../spec/system-as-is/15-system-risks.md), строки 002, 004, 006, 008, 009, 010, 014, 015 | Ссылка на `[S16]` — файл из 38 рисков — без указания конкретного `RISK-SRV-*`, хотя для части строк точное соответствие существует: `RISK-SRV-010` для `SYS-RISK-004`, `RISK-SRV-012` для `SYS-RISK-005`/`008`, `RISK-SRV-036` для `SYS-RISK-008`. Для `SYS-RISK-006` адресного Server risk нет, и там общий `[S16]` вводит в заблуждение сильнее его отсутствия | [REV-SYS-003](01-findings.md#rev-sys-003) |
| 3 | [13](../../spec/system-as-is/13-system-capability-matrix.md), `SYS-CAP-012` | `SRV-API-002` (unified controlled errors, PARTIAL) указан как источник для «Automatically retry backup». К retry относится `SRV-API-004` (Idempotency-Key/retry queue NOT PRESENT), указанный там же; `SRV-API-002` — о формате ошибок | [REV-SYS-003](01-findings.md#rev-sys-003) |
| 4 | [16](../../spec/system-as-is/16-system-open-questions.md), `SYS-OPEN-012` | `OPEN-SRV-022` указан как «(retention boundary)» для вопроса о диагностических данных, доступе к логам и сроках их хранения. `OPEN-SRV-022` относится к backup/retention **данных** (DB+FS, RPO), а не логов. Ссылка растянута, но снабжена явной оговоркой, и Server-плечо дополнительно опирается на `[S13]`, где отсутствие log retention зафиксировано. Отдельный finding не создавался | — |

## Missing major traceability

**Ни одного.**

Все проверенные существенные system claims прослеживаются до конкретного frozen источника через профильный раздел и Area mapping таблицу файла 17:

| System claim | Прослеживается до |
| --- | --- |
| Access 20 мин / refresh 7 дней / no rotation | S06 таблица JWT; `SRV-AUTH-004/005`; A07 |
| Refresh non-2xx очищает pair, IOException — нет | A07 таблица outcomes; `INT-AND-004`; S05/S06/S10 |
| Logout clear после HTTP response до проверки успешности | A07; `SRV-AUTH-006`; S06 |
| `existing` — logical, без reservation и physical check | S05 `POST /files/checksums/exists`; S08; `SRV-SYNC-002`; A08/A17 |
| Duplicate same-folder → HTTP 200 прежний DTO, без bytes repair | S07; S05 таблица конфликтов; `SRV-FILE-003`; A08/A17 |
| Omitted `folderId` → CAMERA для IMAGE/VIDEO, иначе FILES, по detected bytes | S07; S05; `SRV-FILE-002`; `INT-AND-002` |
| 409 только вне CAMERA | S07; S05; A11; A17 |
| `SYNCED` не доказывает bytes/ID/оригинал/scope | A03/A08/A09; `SRV-SYNC-002`; `SRV-FILE-003`; `OPEN-AND-020` |
| DB и FS без общей atomic transaction; startup repair отсутствует | S07 «Согласованность и компенсация»; S10; S12; `SRV-STORAGE-004` |
| Cold headless worker останавливается на пустом URL | A10; A13; `RISK-AND-001`; `AND-BG-004` |
| Local stale DELETE не удаляет оригинал и server copy | A05; A16 `AND-MEDIA-003/007`; S07 |
| CHECKSUM_IO restore: count<3, 300000 мс, до 500 строк, один snapshot | A11; A13; A04 `getRetryableFailed` |
| 36 mappings = 10 used + 19 implemented-unused + 7 STUB | S05 индекс endpoints; A06; A17 |
| Server не назначает client retry policy / Retry-After / Idempotency-Key | S10 «Автоматических retries… нет»; `SRV-API-004` |
| Одна local DB/pair без account/host namespace | A03; A14; `AND-DATA-003`; `INT-AND-001` |

## Source fingerprints

Файл 17 содержит SHA-256 для 34 прочитанных authoritative документов (16 Server + 18 Android) и утверждает, что полный набор 46 входных Markdown-файлов сравнивался до и после составления.

Проверка пересчётом: **34 из 34 значений совпали побайтово** с текущим содержимым frozen файлов. Заявление о 46 файлах согласуется с фактическим составом каталогов — Server содержит 22 `.md`, Android 24 `.md`, итого 46.

| Аспект | Результат |
| --- | --- |
| Относятся ли fingerprints к **текущим** frozen Server/Android specs? | Да, подтверждено пересчётом |
| Совпадает ли заявленное число входных файлов с фактическим? | Да: 22 + 24 = 46 |
| Используется ли hash как доказательство семантической корректности? | Нет. Файл 17 прямо оговаривает: «неиспользуемые internal/verification документы не становятся источниками claims от самого факта хеширования». Это верная трактовка: hash подтверждает только identity источника |
| Заявлена ли Android production revision как переаудированная? | Нет: «Android frozen README указывает production revision `2b9ab698…`; System не переаудирует эту revision. Наличие FROZEN не удостоверяет deployed runtime» — корректная оговорка |

## No component authority inversion

Проверено соблюдение иерархии истины (разделы 5 и 74 задания).

| Проверка | Результат |
| --- | --- |
| Server-only assertions проверяются против Server As-Is | OK. Разделы 05, 06, 11, 12 выводят серверное поведение только из `S01/S03/S05/S06/S07/S09/S10/S12` |
| Android-only assertions проверяются против Android As-Is | OK. Разделы 07, 08, 09, 10 выводят клиентское поведение только из `A03/A05/A06/A07/A08/A09/A10/A11` |
| Cross-component assertions опираются на оба input | OK. Все четыре `SYS-MISMATCH` и разделы 05/12 указывают обе стороны |
| Не назначен ли один input «победителем» в mismatch | Нет. Файл 17 прямо фиксирует: «один input не назначается „победителем“ в semantic mismatch»; проверено по всем четырём mismatch — каждый описывает обе стороны как корректные в своём контексте |
| Не оспариваются ли закрытые component facts | Нет. Факты, установленные через `VER-AND-*` и `VER-SRV-*`, приняты как frozen input; ни одного случая переоткрытия не найдено |
| Не объявлен ли System выше component specs как technical authority | Нет. README: «Server internals → Server As-Is v1. Android internals → Android As-Is v1» — направление сохранено |
| System-derived концепции обозначены как derived | Да. «System phase является descriptive mapping и не является persisted state machine» ([08](../../spec/system-as-is/08-system-state-model.md), первая строка); «Mermaid nodes обозначают компоненты/границы, а не новый runtime» ([01](../../spec/system-as-is/01-system-boundary.md)); «phases не вводят system enum» ([18](../../spec/system-as-is/18-system-flow-index.md)); «System runtime layer… отсутствуют» |

### Derived concepts — отдельная проверка

Раздел 71 задания требует найти концепции, которых нет как runtime entities в компонентах, и убедиться, что они объявлены derived/descriptive.

| Derived concept | Объявлен descriptive | Где |
| --- | --- | --- |
| System phase | Да, явно и в первой строке файла | [08](../../spec/system-as-is/08-system-state-model.md) |
| System capability (`SYS-CAP-*`) | Да — статусы определены как оценка объёма механизма, не как runtime enum; «Это не оценка runtime test pass» | README, [13](../../spec/system-as-is/13-system-capability-matrix.md) |
| System flow (`SYS-FLOW-*`) | Да — «Отдельный System runtime отсутствует», «phases не вводят system enum» | [18](../../spec/system-as-is/18-system-flow-index.md) |
| Access Session / Refresh Session | Да — «системные понятия жизненного цикла доступа. Они не вводят Device/SyncSession entities»; отдельно уточнено, что refresh действительно persisted на обеих сторонах, а access — только у клиента | [03](../../spec/system-as-is/03-system-domain-model.md) |
| «Пять разных объектов в пути фото» | Да — «Это не пять реплик одной записи» | [03](../../spec/system-as-is/03-system-domain-model.md) |

Ни один derived concept не выдаётся за persisted runtime fact. Findings по этому основанию не создавались.

## No historical audit dependency

| Проверка | Результат |
| --- | --- |
| Используются ли Server Audit A/B как источники | Нет |
| Используются ли Android Audit A/B как источники | Нет |
| Используются ли Server/Android review-пакеты как источники | Нет |
| Используется ли historical handoff как действующий contract | Нет. Android input сам отвергает handoff («Здесь не используется historical handoff как действующий contract», A17), и System это не восстанавливает |
| Используются ли `VER-*` записи как самостоятельные источники | Нет. Файл 17: «исторические пакеты, verification logs и production evidence paths не использованы как самостоятельные источники» |
| Используются ли сведения об audit-происхождении | Только для статуса входов (FROZEN), что файл 17 прямо оговаривает |
| Не выбран ли третий input | Нет. Файл 17: «Историческая embedded ссылка Android на sibling `server-as-is` не выбирается как третий input». Отдельно проверено: существующий пустой каталог `docs/spec/system-as-is/` вне выходного пути источником не является и не изменялся |

## Passed traceability areas

Области, в которых прослеживаемость проверена и замечаний нет:

* Input authority и workspace mapping — соответствие логических путей задания фактическим `PhotoCloudServer`/`PhotoCloudClient` зафиксировано явно;
* Area mapping по 20 системным областям с указанием Server source, Android source и System output;
* Registry reconciliation — объявленные счёта 26/12/4/18/21 совпали с фактическими определениями;
* Completeness self-check — все 16 контрольных вопросов ведут к существующим разделам и registry-записям, проверено выборочно по 07, 08, 11, 12, 18;
* Все `SYS-MISMATCH` — обе стороны указаны с существующими и релевантными ID;
* Все `SYS-OPEN` — Server- и Android-плечи указаны, при отсутствии соответствия это сказано прямо (`SYS-OPEN-010`);
* Все `SYS-FLOW` — `Source sections` ведут в разделы, действительно содержащие описанные стадии;
* Все `SYS-CAP` — component capability IDs существуют; отношение «unused Server capability ≠ mismatch» выражено статусом, а не дефектом;
* Fingerprints — 34 значения, все верны;
* Внутренние ссылки и якоря пакета — все разрешаются.

## Вывод по traceability

Прослеживаемость пакета достаточна для заморозки. Единственная остаточная правка — точность колонки `Sources` в реестре рисков и одна ссылка в матрице capabilities ([REV-SYS-003](01-findings.md#rev-sys-003), MINOR). Ни одно утверждение System As-Is не оказалось непрослеживаемым до frozen источника, и ни один указанный источник не оказался несуществующим.
