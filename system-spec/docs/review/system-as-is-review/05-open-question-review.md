# Open question review

Проверены все 21 `SYS-OPEN-*` из [16-system-open-questions.md](../../spec/system-as-is/16-system-open-questions.md).

Review classification:

* **REAL_OPEN** — вопрос действительно не решён и системно значим;
* **ALREADY_ANSWERED_SERVER / _ANDROID / _SYSTEM** — ответ уже установлен соответствующей frozen/System спецификацией;
* **WRONG_TYPE** — тип назначен неверно;
* **MIXED** — объединяет разные по природе вопросы;
* **DUPLICATE** — повторяет другой `SYS-OPEN`;
* **COMPONENT_ONLY** — касается только внутренней реализации компонента и не влияет на system semantics;
* **UNCLEAR** — формулировка не позволяет определить предмет.

Review **не отвечает** ни на один OPEN, даже когда решение кажется очевидным. Проверяется корректность вопроса, а не выбор ответа.

Итог: **REAL_OPEN 20**, **MIXED 1**, остальные категории — 0.

| SYS-OPEN | Type | Review classification | Notes |
| --- | --- | --- | --- |
| 001 Смысл пользовательского «синхронизировано» | PRODUCT_DECISION | REAL_OPEN | Тип верен: это продуктовая интерпретация, а не неизвестное поведение кода. Current As-Is точно фиксирует уже установленное (`SYNCED` = existing либо `id>0`, ID optional, logical record не удостоверяет bytes). Источники `OPEN-SRV-005` и `OPEN-AND-020` существуют и релевантны |
| 002 Направление sync, retention изменённых/удалённых, роль Android при remote delete/move | PRODUCT_DECISION | REAL_OPEN | Объединяет `OPEN-SRV-001/008` и `OPEN-AND-012/013` по одному решению о lifecycle — консолидация обоснована, не размывает предмет. Прямо оговорено, что вопрос «не меняет известное отсутствие propagation» |
| 003 Несколько аккаунтов, серверов и устройств | PRODUCT_DECISION | REAL_OPEN | Корректно связан с `SYS-MISMATCH-001` как вопрос об ожиданиях, а не как назначение storage schema. Источники `OPEN-SRV-002/003`, `OPEN-AND-014/027` |
| 004 Границы согласованности local asset ↔ remote identity при смене session и concurrent processing | SYSTEM_ARCHITECTURE_DECISION | REAL_OPEN | Тип верен — архитектурное решение об инварианте. Соответствует `OPEN-AND-028` (ARCHITECTURE_DECISION) и `OPEN-SRV-003`. Формулировка «известные interleavings не объявляются unknown» удерживает границу между фактом и решением |
| 005 Contract target и первого CAMERA pre-check | INTEGRATION_DECISION | REAL_OPEN | Единственный `INTEGRATION_DECISION`, и тип здесь уместнее продуктового: предмет — совместный contract двух компонентов. Отделяет известный lifecycle и `SYS-MISMATCH-002` от неназначенного решения. Источники `OPEN-SRV-004/011`, `OPEN-AND-011`, `INT-AND-002` |
| 006 Охват типов/папок/размеров | PRODUCT_DECISION | REAL_OPEN | Верно отмечено, что «Server acceptance не равен охвату device backup». Источники `OPEN-SRV-011/023`, `OPEN-AND-011/022` |
| 007 Общие время, имена, metadata; побайтовая копия и геопозиция | PRODUCT_DECISION | REAL_OPEN | Корректно разделяет presentation/privacy **решение** от отдельно неизвестных OS bytes (вынесенных в `SYS-OPEN-018`) — разделение, а не дублирование. Источники `OPEN-SRV-013`, `OPEN-AND-021/026` |
| 008 Смысл завершения logout и истечения session | PRODUCT_DECISION | REAL_OPEN | Current As-Is перечисляет уже установленное (20 мин/7 дней, no rotation, refresh 5xx clear) и не переоткрывает его. Источники `OPEN-SRV-016`, `OPEN-AND-015`, `INT-AND-004` |
| 009 Ручное восстановление failures, автономность, задержка, уведомления | PRODUCT_DECISION | REAL_OPEN | Тип верен. Отмечу как место для минимальной правки по [REV-SYS-002](01-findings.md#rev-sys-002): именно сюда естественно добавить ссылку на `OPEN-SRV-012` (продуктовый смысл терминального `HTTP_409`). Ссылка `OPEN-SRV-024` релевантна лишь частично, `OPEN-AND-018/019` — точно |
| 010 Transport/deployment и расход трафика/батареи | PRODUCT_DECISION | REAL_OPEN | Отдельно отмечаю добросовестность формулировки источников: «`[S17]`: Нет отдельного тождественного решения» — вместо подгонки нерелевантного `OPEN-SRV-027` (который является DEPLOYMENT_UNKNOWN, а не продуктовым решением). Это корректная дисциплина traceability |
| 011 Завершение onboarding и recovery доступа | PRODUCT_DECISION | REAL_OPEN | Источники `OPEN-SRV-018`, `OPEN-AND-023` точны; связь с `SYS-RISK-017` согласована |
| 012 Диагностические данные, доступ к логам, сроки хранения | PRODUCT_DECISION | REAL_OPEN | Вопрос системно значим (соединяет privacy и возможность расследовать end-to-end failure). Server-плечо опирается на `[S13]` и на `OPEN-SRV-022` с пометкой «(retention boundary)»; `OPEN-SRV-022` относится к backup/retention данных, а не логов, — ссылка растянута, но снабжена оговоркой. Android-плечо `OPEN-AND-025` точно. Не finding, но учтено в [06-traceability-review.md](06-traceability-review.md) как weak reference |
| 013 Поддерживаемые устройства, объёмы, критерии проверки | PRODUCT_DECISION | REAL_OPEN | Корректно отделяет static As-Is от runtime acceptance. Источники `OPEN-SRV-023`, `[S14]`, `OPEN-AND-009/029` |
| 014 Развёрнутый endpoint/revision, состояние DB, transport, framework error responses | DEPLOYMENT_RUNTIME_UNKNOWN | REAL_OPEN | Тип верен и соответствует объявленному в файле отображению. Четыре Server источника (`OPEN-SRV-025/027/028/030`) и `OPEN-AND-010` все существуют и являются DEPLOYMENT/FRAMEWORK_UNKNOWN. Важно, что оговорено: «normal folderId text/plain не неизвестен» — то есть уже верифицированное не возвращено в неизвестность |
| 015 Реальные volumes/свойства FS и существование внешнего процесса восстановления bytes | DEPLOYMENT_RUNTIME_UNKNOWN | REAL_OPEN | Корректное разделение с `SYS-OPEN-021`: здесь — фактический вопрос о существовании внешнего процесса, там — архитектурная policy. Это в точности то разделение, которое сам `OPEN-SRV-022` требует («Наличие реального внешнего backup … — DEPLOYMENT_UNKNOWN; а backup/retention/RPO policy — архитектурное решение»). Источники `OPEN-SRV-022/026`, `OPEN-AND-030`. Не DUPLICATE по отношению к 021 |
| 016 Сроки scheduling/events после reboot, force-stop, Doze | OS_FRAMEWORK_UNKNOWN | REAL_OPEN | Тип верен (`OPEN-AND-001` — OS_FRAMEWORK_UNKNOWN). Явно оговорено, что вопрос «не переоткрывает установленный cold-start факт» — требование не возвращать решённое в unknown соблюдено |
| 017 Видимость и permission state в selected-access режиме | OS_FRAMEWORK_UNKNOWN | REAL_OPEN | Системно значим, а не component-only: от него зависит, удалит ли успешный неполный snapshot local history при сохранившихся оригиналах и remote copies. Источник `OPEN-AND-002` |
| 018 Какие bytes/GPS возвращает OS/provider | OS_FRAMEWORK_UNKNOWN | REAL_OPEN | Системно значим — определяет полноту копии. Верная оговорка «Nullable server GPS не доказывает redaction». Источник `OPEN-AND-003` |
| 019 Read/buffering/length effects и момент прекращения upload side effects | DEPLOYMENT_RUNTIME_UNKNOWN | **MIXED** | Тип согласован с объявленным отображением (Android runtime unknown → DEPLOYMENT_RUNTIME_UNKNOWN), поэтому не WRONG_TYPE. Но пункт объединяет разное по природе: (а) фактический объём дополнительных чтений/буферизации и RAM peak (`OPEN-AND-004`) — по существу component-only ресурсный вопрос; (б) расхождение `SIZE` с фактической длиной потока (`OPEN-AND-005`) и момент фактического прекращения side effects при cancel/constraint loss (`OPEN-AND-007`) — системно значимо, поскольку влияет на повторяемый IO failure, head-of-line и на то, когда запросы реально перестают доходить до сервера. Обоснование «Ресурсы и повторяемый IO failure влияют на queue» связывает их через одно системное следствие, что делает объединение защитимым. Finding не создан: разделение желательно, но неразделение не искажает факты |
| 020 Восстановление key/pair/local IDs/queue после restore/переноса/upgrade | DEPLOYMENT_RUNTIME_UNKNOWN | REAL_OPEN | Системно значим: утрата или перенос local belief меняет полноту backup без изменения Server records. Источники `OPEN-AND-006/008` |
| 021 Физическая storage boundary, backup/retention/RPO policy | SYSTEM_ARCHITECTURE_DECISION | REAL_OPEN | Тип верен (`OPEN-SRV-021/022` — ARCHITECTURE_DECISION). Явная ремарка «Отделяет архитектурную policy от вопроса о реально существующем внешнем процессе в SYS-OPEN-015» предотвращает прочтение как дубликата |

## Проверка на COMPONENT_ONLY

Раздел 50 задания требует отдельно убедиться, что в System SPEC не попали OPEN, касающиеся только внутренней реализации одного компонента.

| Проверка | Результат |
| --- | --- |
| Есть ли `SYS-OPEN`, целиком относящийся к внутренней реализации Server? | Нет |
| Есть ли `SYS-OPEN`, целиком относящийся к внутренней реализации Android/UI? | Нет. Ближайший кандидат — 019, но его системно значимая часть преобладает → MIXED, не COMPONENT_ONLY |
| Исключены ли component-only OPEN обоих компонентов? | Да, и в основном явно. `OPEN-AND-024` (назначение UI-заглушек) — единственный неперенесённый Android OPEN, корректно component-only. Из 11 неперенесённых `OPEN-SRV` девять подпадают под объявленные в файле исключения (sharing, account delete, tree delete, listing/presentation, profile, framework download, клиенты) |
| Не переоткрыты ли уже решённые component-вопросы? | Нет. Раздел «Consolidation boundary» прямо перечисляет установленное и не скопированное как неизвестность: identity, routes, CAMERA bootstrap, no rotation, existing/duplicate, retry. Проверено выборочно по 06, 07, 16 — установленные факты нигде не возвращены в статус unknown |

## Формулировки

Проверено требование разделов 58–59 задания: допустимо «Должна ли будущая система поддерживать X?», недопустимо «Система должна поддерживать X».

Все 21 пункта сформулированы как вопросы («Какой смысл…», «Каков ожидаемый…», «Каковы фактические…», «Как трактуются…»). Ни один не содержит предписания. Ни на один не дан ответ, в том числе там, где ответ мог бы показаться очевидным (направление sync, multi-account scope, retention удалённых фото, HTTPS). Поля `Current As-Is` описывают только установленное, `Why it matters` — только значение вопроса.

**OPEN_QUESTION_ERROR по существующим пунктам: 0.** Единственный finding по этому реестру — [REV-SYS-002](01-findings.md#rev-sys-002) — касается двух **непредставленных** Server OPEN, а не ошибок в 21 имеющихся.
