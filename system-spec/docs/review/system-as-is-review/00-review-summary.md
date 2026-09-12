# PhotoCloud System As-Is v1 — Independent Review Summary

## Предмет review

| Аспект | Значение |
| --- | --- |
| Проверяемый пакет | `system-spec/docs/spec/system-as-is/` (README + 19 файлов 00…18) |
| Заявленный статус | `PhotoCloud System As-Is v1 — READY_FOR_REVIEW` |
| Component authority 1 | `PhotoCloudServer/docs/spec/server-as-is/` — `Server As-Is v1 — FROZEN` |
| Component authority 2 | `PhotoCloudClient/docs/spec/android-as-is/` — `Android As-Is v1 — FROZEN` |
| Дата review | 2026-09-12 |
| Тип работы | Независимый review системной спецификации |

Фактические имена каталогов в workspace — `PhotoCloudServer` и `PhotoCloudClient`; они соответствуют `server` и `android` из задания. System пакет сам фиксирует это соответствие в [17](../../spec/system-as-is/17-source-traceability.md) и не выдаёт его за третий input.

Review выполнен только по двум frozen пакетам и проверяемому System пакету. Production code (Java/Kotlin/SQL/Gradle/manifest/tests/конфигурация) не читался. Исторические Server/Android audits, reviews и handoff не использовались даже как дополнительное подтверждение. Новая интеграционная консолидация не проводилась.

## Verdict

# `PASS_WITH_MINOR_FIXES`

System As-Is v1 корректно описывает фактическое end-to-end поведение PhotoCloud, не искажает frozen component specifications, находится на правильном уровне абстракции и пригоден к заморозке после конечного набора из четырёх локальных MINOR исправлений. Нового Server или Android audit не требуется.

## Счёт findings

| Severity | Количество |
| --- | --- |
| BLOCKER | **0** |
| MAJOR | **0** |
| MINOR | **4** |
| Всего | **4** |

## Findings по типам

| Тип проблемы | Количество | ID |
| --- | --- | --- |
| UNSUPPORTED_SYSTEM_CLAIM | **0** | — |
| LOST_SYSTEM_FACT | **0** | — |
| SOURCE_DISTORTION | **0** | — |
| INTERNAL_CONTRADICTION | **0** | — |
| ABSTRACTION_ERROR | **0** | — |
| STATUS_ERROR | **0** | — |
| MISMATCH_ERROR | **0** | — |
| RISK_ERROR | **1** | REV-SYS-001 |
| OPEN_QUESTION_ERROR | **1** | REV-SYS-002 |
| FLOW_ERROR | **1** | REV-SYS-004 |
| TRACEABILITY_ERROR | **1** | REV-SYS-003 |
| TO_BE_LEAK | **0** | — |

Полные описания — [01-findings.md](01-findings.md).

## Что проверено и подтверждено

### Уровень абстракции — PASS

Пакет отвечает на вопрос «как работает система в целом», а не «как реализован компонент». Целевой поиск implementation-имён не обнаружил протечек `*Service`, `*Controller`, `*Repository`, `*Mapper`, `*Dao`, `*ViewModel`, package names, миграций или SQL internals в описательных разделах. Единственное упоминание `DAO`/`controller` находится в self-check строке 17, где прямо сказано, что такие каталоги **не** переносились.

Присутствующая техническая конкретика необходима для системной семантики и допустима: `MediaStore`, `PostgreSQL`, `WorkManager`, `Room`, `FileItem`, `StoredObject`, `Folder`, `mediaStoreId`, `serverFileId`, `BaseUrlProvider`, «Authenticator» как отличие refresh-пути от logout-пути. Обратной проблемы — чрезмерной абстракции с потерей contract semantics — также нет: folder scope pre-check, смысл `existing`, отсутствие доказательства physical bytes, omitted `folderId`, default routing по detected bytes, duplicate 200 и условия 409 изложены точно.

### To-Be leakage — отсутствует

Сплошной поиск предписывающих конструкций (`должен`, `нужно`, `следует`, `требуется`, `необходимо`, `рекоменд*`, `roadmap`, `SPEC-*`) по файлам 00…18 дал только описательные совпадения вида «из X не следует Y» и «нужен последующий worker». Ни одного future prescription, fix, roadmap-элемента или назначенного ответа на OPEN не найдено. Все OPEN сформулированы как вопросы.

### Гарантии и негативные утверждения

Нет ни одного вхождения `всегда`, `always`, `никогда`, `never`, `невозможно`, `не существует`, `исключено`. Раздел «Current guarantees» в [12](../../spec/system-as-is/12-data-consistency-and-durability.md) ограничен шестью конкретными механизмами и явно оговаривает, что это не гарантия «каждое фото сохранено». Все негативные утверждения (`SYNCED` не доказывает bytes, existing не reservation, DB commit не атомарен с bytes) прямо следуют из frozen evidence.

### Registry counts — подтверждены полностью

| Registry | Заявлено | Определений найдено | Уникальность | Непрерывность | Дубликаты между типами |
| --- | --- | --- | --- | --- | --- |
| `SYS-CAP-*` | 26 | 26 | OK | 001…026 без пропусков | нет |
| `SYS-FLOW-*` | 12 | 12 | OK | 001…012 без пропусков | нет |
| `SYS-MISMATCH-*` | 4 | 4 | OK | 001…004 без пропусков | нет |
| `SYS-RISK-*` | 18 | 18 | OK | 001…018 без пропусков | нет |
| `SYS-OPEN-*` | 21 | 21 | OK | 001…021 без пропусков | нет |

Статусы в [13](../../spec/system-as-is/13-system-capability-matrix.md): 9 IMPLEMENTED, 8 PARTIAL, 8 NOT PRESENT, 1 STUB = 26. `UNUSED` не применён, что соответствует объявленному в файле правилу.

### Source references — ни одной битой ссылки

Все `SRV-*`, `RISK-SRV-*`, `OPEN-SRV-*`, `AND-*`, `RISK-AND-*`, `OPEN-AND-*` и `INT-AND-*`, упомянутые в System пакете (171 различный ID после раскрытия сокращённой записи `PREFIX-001/002`), существуют в соответствующем frozen реестре. Битых ID: **0**. Все Markdown-ссылки и якоря внутри пакета разрешаются: битых ссылок **0**.

### Source fingerprints — проверены пересчётом

Все **34** значения SHA-256 в [17](../../spec/system-as-is/17-source-traceability.md) пересчитаны и совпали побайтово с текущими frozen файлами (16 Server + 18 Android). Заявление о сравнении полного набора **46** входных Markdown-файлов согласуется с фактом: Server каталог содержит 22 файла, Android — 24. Fingerprints подтверждают только identity входов, не семантическую корректность; как доказательство корректности они в пакете и не используются.

### Четыре mismatch — перенесены верно

`SYS-MISMATCH-001…004` сохраняют `INT-AND-001…004` one-to-one. Проверены Android-факт, Server-факт, условие, системное последствие и классификация каждого. Условность сохранена во всех четырёх: нормальный CAMERA bootstrap не объявлен mismatch (002), стабильный файл не объявлен mismatch (003), возможный cross-account сбой не превращён в наблюдавшийся инцидент (001), refresh 5xx классифицирован как потеря session semantics, а не как нарушение назначенной сервером retry policy (004). **Подтверждены без изменений: 4 из 4.**

Дополнительных cross-component contradictions, прямо видимых из двух frozen специфаций, review не обнаружил. Вывод пакета «0 дополнительных mismatches, 0 противоречий frozen inputs» подтверждается.

### Server capability unused ≠ mismatch — правило соблюдено

26 неиспользуемых Server mappings (19 реализованных + 7 STUB) сгруппированы как inventory-граница в [05](../../spec/system-as-is/05-effective-api-contract.md) и отражены статусами `NOT PRESENT` соответствующих системных capabilities, а не объявлены дефектами. Арифметика проверена: 36 mappings − 10 используемых = 26; 29 реализованных − 10 = 19; плюс 7 STUB. Сходится.

### `SYNCED` — сильных утверждений не найдено

[08](../../spec/system-as-is/08-system-state-model.md) прямо перечисляет, что из `SYNCED` **не** следует: current physical bytes, известный ID, повторная remote verification, сохранность оригинала, актуальность account/host. System phase явно объявлен descriptive mapping, а не persisted enum, в первой же строке файла. Требование раздела 27 задания выполнено полностью.

## Остаточные MINOR исправления

| ID | Суть | Файл для правки |
| --- | --- | --- |
| REV-SYS-001 | Системные последствия `RISK-SRV-002`/`RISK-SRV-005` (ban/reset не прекращают доступ; нет rate limiter при пароле 4..20) отсутствуют в SYS-RISK, хотя сами факты в пакете есть | [15](../../spec/system-as-is/15-system-risks.md) |
| REV-SYS-002 | `OPEN-SRV-012` и `OPEN-SRV-017` не представлены ни SYS-OPEN, ни явным исключением в Consolidation boundary | [16](../../spec/system-as-is/16-system-open-questions.md) |
| REV-SYS-003 | Однообразные boilerplate-ссылки на всех 18 строках SYS-RISK; конкретные релевантные `RISK-SRV-*` не указаны | [15](../../spec/system-as-is/15-system-risks.md) |
| REV-SYS-004 | Entry condition `SYS-FLOW-001` включает условие успеха и расходится с собственной failure-строкой | [18](../../spec/system-as-is/18-system-flow-index.md) |

Ни одно исправление не затрагивает системную модель, identity model, sync lifecycle, authority model или Server↔Android contract. Все четыре локальны и выполняются правкой одного-двух абзацев/строк в трёх файлах.

## Готовность к freeze

После закрытия четырёх MINOR findings пакет пригоден к заморозке как:

```text
PhotoCloud System As-Is v1 — FROZEN
```

Дополнительного Server audit, Android audit, повторной интеграционной консолидации или чтения production code для закрытия не требуется. `Server As-Is v1` и `Android As-Is v1` остаются authoritative technical specifications своих компонентов.

## Состав review

| Файл | Содержание |
| --- | --- |
| [00-review-summary.md](00-review-summary.md) | Verdict, счёт, подтверждённые области |
| [01-findings.md](01-findings.md) | `REV-SYS-001…004` в заданном формате |
| [02-cross-document-consistency.md](02-cross-document-consistency.md) | 30 областей cross-check |
| [03-component-source-coverage.md](03-component-source-coverage.md) | Покрытие system-relevant материала frozen specs |
| [04-system-registry-review.md](04-system-registry-review.md) | 26 CAP, 12 FLOW, 4 MISMATCH, 18 RISK |
| [05-open-question-review.md](05-open-question-review.md) | 21 SYS-OPEN с review classification |
| [06-traceability-review.md](06-traceability-review.md) | Source IDs, fingerprints, authority direction |

System спецификация этим review не изменялась.
