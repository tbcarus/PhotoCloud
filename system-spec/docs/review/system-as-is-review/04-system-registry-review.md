# System registry review

Проверены все заявленные записи: 26 `SYS-CAP-*`, 12 `SYS-FLOW-*`, 4 `SYS-MISMATCH-*`, 18 `SYS-RISK-*`. Счёт, уникальность и непрерывность подтверждены пересчётом определений в файлах 13, 18, 14, 15.

## Capabilities — 26

Проверялось: (1) является ли запись системной capability, а не internal component feature; (2) соответствует ли статус фактам frozen specs; (3) корректны ли Server/Android source references; (4) нет ли дублирования с другой `SYS-CAP`.

| SYS-CAP | Classification correct | Sources correct | Notes |
| --- | --- | --- | --- |
| 001 Authenticate user | Да — IMPLEMENTED | Да | `SRV-AUTH-003/004` + `AND-AUTH-002/003/004` существуют и релевантны; оговорка о доступности endpoint/аккаунта уместна |
| 002 Register and activate account | Да — PARTIAL | Да | Confirm вне Android и недоказанная delivery обосновывают PARTIAL при `SRV-AUTH-002` IMPLEMENTED; `AND-AUTH-009` NOT PRESENT подтверждает клиентское плечо |
| 003 Refresh access and replay | Да — PARTIAL | Да | Ограничения (non-2xx clear, session races) прямо следуют из `AND-AUTH-005` PARTIAL и `SRV-AUTH-005` |
| 004 Logout current session | Да — PARTIAL | Да | `SRV-AUTH-017` (NOT PRESENT: отзыв access) корректно использован как предел capability, не как отдельная функция |
| 005 Discover local photos | Да — IMPLEMENTED | Да | Android-only стадия, но это entry системного pipeline. Server-колонка «Не участвует» соответствует правилу файла |
| 006 Persist local backup queue | Да — IMPLEMENTED | Да | Носитель системного состояния. `SRV-SYNC-004` использован как граница отсутствующей общей session — корректный приём |
| 007 Calculate content checksum | Да — IMPLEMENTED | Да | Два независимых вычисления описаны как две стороны одной системной capability, равенство не заявлено |
| 008 Resolve CAMERA / bootstrap | Да — IMPLEMENTED | Да | Четыре server ID + `AND-SYNC-002` релевантны; ограничение fallback вынесено в mismatch, а не в статус |
| 009 Detect logical existing media | Да — IMPLEMENTED | Да | Пределы («не physical check, ID не возвращается») в scope-колонке; `AND-SYNC-007` NOT PRESENT учтён |
| 010 Upload selected image stream | Да — IMPLEMENTED | Да | `SRV-FILE-001` + `SRV-MEDIA-001` + `AND-SYNC-004/006` |
| 011 Handle same-folder duplicate | Да — IMPLEMENTED | Да | `SRV-FILE-003` + `AND-SYNC-005`; «предыдущие metadata/bytes не заменяются» соответствует S07 |
| 012 Automatically retry backup | Да — PARTIAL | PARTIAL | `SRV-API-004` и `AND-SYNC-010/011` релевантны. `SRV-API-002` (unified controlled errors) к retry относится лишь косвенно — см. [REV-SYS-003](01-findings.md#rev-sys-003) |
| 013 Background backup | Да — PARTIAL | Да | `AND-BG-001/002/003/004` точны; `SRV-OPS-003` использован как граница отсутствия server jobs |
| 014 Recover interrupted local stages | Да — PARTIAL | Да | `SRV-STORAGE-004` PARTIAL + `AND-SYNC-012` + `AND-BG-004`; «server commit не атомарен с local mark» соответствует S07/A11 |
| 015 Reprocess changed local media | Да — PARTIAL | Да | `SRV-FILE-013` NOT PRESENT (trash/version) релевантен; `AND-MEDIA-002` PARTIAL — источник size/mtime эвристики |
| 016 Remove stale local index entries | Да — IMPLEMENTED | Да | Локальная операция, но системно значима именно отсутствием propagation; парна с 017, дублирования нет |
| 017 Propagate local deletion to server | Да — NOT PRESENT | Да | Capability сформулирована как propagation, которой нет вовсе → NOT PRESENT, а не PARTIAL. Наличие `SRV-FILE-010` указано как Server-сторона без объявления дефектом |
| 018 Bidirectional synchronization | Да — NOT PRESENT | Да | `SRV-SYNC-003` PARTIAL / `SRV-SYNC-004` NOT PRESENT + `AND-SYNC-014`; основание one-way характеристики |
| 019 Remote gallery/download on Android | Да — NOT PRESENT | Да | Формулировка «on Android» делает NOT PRESENT семантически верным при существующих `SRV-FILE-004/005/006` |
| 020 Remote organization through Android | Да — NOT PRESENT | Да | Формулировка «through Android» аналогично корректна; восемь server IDs перечислены как unused capability, не mismatch |
| 021 Video / outside-DCIM backup | Да — NOT PRESENT | Да | Приём VIDEO сервером явно отделён от отсутствия client discovery; `SRV-MEDIA-004` + `AND-MEDIA-005` |
| 022 Image metadata across system | Да — PARTIAL | Да | `SRV-MEDIA-002` PARTIAL / `SRV-MEDIA-003` + `AND-SYNC-008` NOT PRESENT |
| 023 End-to-end physical integrity verification/repair | Да — NOT PRESENT | Да | `SRV-FILE-003` + `SRV-STORAGE-006`; согласуется с `SYS-RISK-004` и `SYS-OPEN-015` |
| 024 Account/server-scoped backup history | Да — NOT PRESENT | Да | Server ownership существует, клиентский namespace отсутствует → end-to-end capability отсутствует. Ссылка на `INT-AND-001` уместна |
| 025 Resumable upload and byte progress | Да — NOT PRESENT | Да | `SRV-FILE-014` + `SRV-SYNC-004` + `AND-SYNC-015` |
| 026 User settings controls across system | Да — STUB | Да | Обе стороны — доступные заглушки (`SRV-PROFILE-003` 501, `AND-UI-005` STUB), поэтому STUB предпочтительнее NOT PRESENT. Согласуется с определением статусов в файле |

Дубликатов между capabilities не обнаружено; пары 016/017 и 019/020 разграничены формулировками. Правило раздела 37 задания (PARTIAL против NOT PRESENT при неиспользуемой Server capability) соблюдено: во всех таких случаях формулировка capability включает Android-сторону, что делает NOT PRESENT семантически согласованным.

**STATUS_ERROR: 0.**

## Flows — 12

Проверялось: entry condition, components, stages, success end-state, failure/recovery, source sections; и отдельно — не вводит ли index поведение, отсутствующее в разделах 05–12.

| SYS-FLOW | Valid system flow | Sources correct | Notes |
| --- | --- | --- | --- |
| 001 Login and session establishment | Да | Да | `Entry condition` включает условие успеха («activated/non-banned») и расходится с собственной failure-строкой — [REV-SYS-004](01-findings.md#rev-sys-004). Остальные поля верны |
| 002 Access token refresh | Да | Да | Все четыре outcome-ветки (replay current token, plain refresh, non-2xx clear, IO без clear) и guard `responseCount>=2` соответствуют A07 и S06 |
| 003 New local media discovery | Да | Да | Корректно отмечено, что Server участвует только как worker gate; «Query error не empty success» соответствует A05 |
| 004 Checksum and pre-check | Да | Да | Success end-state покрывает обе ветки (existing и missing/absence); failure — per-file FAILED против stage retry |
| 005 Upload new media | Да | Да | Порядок стадий (recover → repeat lookup → guards → multipart → temp/hash/type/target → final move → DB commit → DTO/local mark) совпадает с S07/S08 и [07](../../spec/system-as-is/07-media-sync-lifecycle.md) |
| 006 Existing media detection | Да | Да | «serverFileId обычно null; bytes не проверены» — точное соответствие A08/A17 и `SRV-SYNC-002` |
| 007 Background retry and failure restoration | Да | Да | Success end-state прямо оговаривает, что SUCCESS не означает отсутствие FAILED |
| 008 Local media deletion/disappearance | Да | Да | «Partial visibility не physical deletion proof» и «scan Error до apply не удаляет» соответствуют A05 |
| 009 Cold-process and reboot recovery | Да | Да | Факт пустого URL как факт, OS timing как unknown; «server restart repair не делает» соответствует S12 |
| 010 Logout and session exit | Да | Да | Разделены remote revoke, local clear и незавершённый фон; «access до exp» верно |
| 011 Changed local media reprocessing | Да | Да | «Unchanged size/mtime может скрыть bytes change» — предел эвристики, а не гарантия обнаружения версий |
| 012 Duplicate upload / lost acknowledgement recovery | Да | Да | Условность same-ID повтора («изменённый tuple/remote deletion снимает условие») соответствует S10 «Свойства повторного вызова» |

Ни один flow не вводит логику, отсутствующую в основных разделах. Финальная ремарка файла корректно уточняет, что смена account/host не является поддержанным migration flow, а remote rename/move/delete — внешние изменения server state, а не Android commands.

**FLOW_ERROR: 1** (REV-SYS-004, MINOR).

## Mismatches — 4

Проверялось: Android-факт, Server-факт, условие, системное последствие, классификация, traceability. Соответствие `INT-AND-001…004` не принималось как автоматически верное — проверялся сам перенос.

| SYS-MISMATCH | Android fact | Server fact | Condition | Result |
| --- | --- | --- | --- | --- |
| 001 Unscoped local state ↔ server/account identity | Верно: `mediaStoreId` без account/server/folder/device namespace; logout/URL change не очищают; pair без host binding (A03, A07) | Верно: principal-scoped авторизация, `FileItem`/`Folder` в user scope, checksum query по user+folder; числовой ID осмыслен в своей DB (S03, S05, S06) | Верно и условно: «Смена account/host при сохранённой local DB/pair; **не утверждается инцидент в каждой установке**» | **CONFIRMED без изменений.** Возможный cross-account сбой не превращён в наблюдавшийся incident. Классификация «scope disagreement, не route/DTO error» точна |
| 002 CAMERA pre-check ↔ fallback upload target | Верно: pre-check по CAMERA, повторный lookup, Error или absence → omitted `folderId`, target ответа не сверяется (A06, A08) | Верно: explicit own folder разрешена; omitted target → IMAGE/VIDEO в CAMERA, остальное в FILES по принятым bytes; lazy ROOT не создаёт CAMERA (S05, S07) | Верно и условно: успешный CAMERA pre-check → Error второго lookup → local `image/` guard пройден → server detection не IMAGE/VIDEO. Прямо оговорено: «Обычный первый IMAGE bootstrap без CAMERA согласован» | **CONFIRMED без изменений.** Нормальный CAMERA flow как mismatch не описан — требование раздела 41 задания выполнено |
| 003 Checksum read ↔ upload byte change | Верно: SHA-256 отдельным чтением URI, upload читает URI снова, принимает `id>0`, response checksum/size игнорирует, immutable snapshot отсутствует (A03, A05, A08) | Верно: hash и size по фактически принятым bytes; client checksum не является authority (S05, S07) | Верно и условно: «Изменение bytes между hashing/pre-check и реальным upload read; **стабильное содержимое само по себе mismatch не создаёт**» | **CONFIRMED без изменений.** Mismatch ограничен гонкой содержимого, как требует раздел 42 |
| 004 Refresh 5xx ↔ очистка client session | Верно: любой non-2xx refresh очищает pair; IOException сам pair не очищает (A07, A10) | Верно: invalid/unknown/expired → 401, revoked → 403; успешный refresh не потребляет/не ротирует token; server error сам revoke не означает (S05, S06, S10) | Верно: protected 401 → plain refresh → 5xx при потенциально ещё валидном refresh | **CONFIRMED без изменений.** Классифицирован как «session meaning, не нарушение назначенной Server client retry policy» — точно то различение, которое требует раздел 43 |

Раздел «Reconciliation outcome» перечисляет факты, **не** повышенные до mismatch: `SYNCED` без ID после existing, logical existing без physical proof, неиспользуемый Server API, default CAMERA bootstrap, игнорируемые server metadata, отсутствие общей retry policy. Review подтверждает, что ни один из них не является semantic incompatibility и что в пакете они действительно проведены как contract limits, risks или OPEN.

Независимая проверка на дополнительные cross-component contradictions, прямо видимые из двух frozen specs (принятие любого 2xx вместо точного статуса; clear tokens при logout 404; `null` children body как пустой список; text/plain `folderId`; 401 на public path при приложенном Bearer; opaque server dates), показала, что все они уже классифицированы Android spec как `CLIENT_ASSUMPTION_WEAKER_THAN_SERVER` или OPEN и корректно не возведены в mismatch. Новый поиск проблем не проводился.

**MISMATCH_ERROR: 0. Подтверждены без изменений: 4 из 4.**

## Risks — 18

Проверялось: factual source, системное последствие, severity, отсутствие unsupported incident claim, отсутствие скрытых remediation.

| SYS-RISK | System-level | Evidence | Severity coherent | Result |
| --- | --- | --- | --- | --- |
| 001 Unscoped local history/pair при switch | Да | `INT-AND-001`, `RISK-AND-002` (HIGH), `SRV-SYNC-002` | Да — HIGH | OK. Последствие сформулировано как возможное («Пропуск backup нового scope»), инцидент не заявлен |
| 002 Cold process без восстановленного URL | Да | `RISK-AND-001` (HIGH), `AND-BG-004` | Да — HIGH | OK. Общая ссылка `[S16]` без конкретного risk ID — REV-SYS-003 |
| 003 Очистка session на refresh 5xx | Да | `INT-AND-004`, `RISK-AND-006/008` (HIGH), `SRV-AUTH-005` | Да — HIGH | OK |
| 004 Logical existing/duplicate при missing bytes | Да | `RISK-AND-019` (HIGH), `SRV-FILE-003`, `SRV-SYNC-002` | Да — HIGH | OK по содержанию. Не указан прямо соответствующий `RISK-SRV-010` (HIGH) — REV-SYS-003 |
| 005 DB/FS/response/local mark неатомарны | Да | `RISK-SRV-007/008` (HIGH), `SRV-STORAGE-004`, `AND-SYNC-012` | Да — HIGH | OK. `RISK-SRV-012` не указан — REV-SYS-003 |
| 006 Изменение bytes между hash/upload | Да | `INT-AND-003`, `RISK-AND-016` (HIGH), `SRV-MEDIA-001` | Да — HIGH | OK. Покрывает и вторую ветку — незамеченное изменение при прежних size/mtime |
| 007 Fallback target после CAMERA pre-check | Да | `INT-AND-002`, `RISK-AND-017` (HIGH), `SRV-FILE-002` | Да — HIGH | OK. Ссылка на условие mismatch, а не подмена его описания |
| 008 Нет reverse reconciliation и delete propagation | Да | `RISK-AND-018` (HIGH), `SRV-SYNC-003/004`, `SRV-FILE-010`, `AND-SYNC-014` | Да — HIGH | OK. `RISK-SRV-036`/`RISK-SRV-012` не указаны — REV-SYS-003 |
| 009 Head-of-line и невосстанавливаемые FAILED | Да | `RISK-AND-013/014` (HIGH), S10 | Да — HIGH | OK. Корректно отмечено, что Server retry policy не назначает |
| 010 Concurrent scan/stage/restore writes | Да | `RISK-AND-011/012` (HIGH), A09, S05 (exists не reservation) | Да — HIGH | OK. Interleavings поданы как условные |
| 011 BODY logging на обеих сторонах | Да | `RISK-SRV-001` (HIGH), `RISK-AND-003` (HIGH), S13, A14 | Да — HIGH | OK. Точность формулировки подтверждена: речь о credentials/tokens/metadata, а не о байтах фото (server заменяет multipart/image/video в тексте на `[binary]`) |
| 012 HTTP-only Android endpoint | Да | `RISK-AND-004` (HIGH), `RISK-AND-023`, `RISK-SRV-024` | Да — HIGH | OK. «фактический proxy/LAN доступ неизвестен» сохраняет предел доказательности |
| 013 Logout/session concurrency и незавершённый фон | Да | `RISK-SRV-003/004`, `RISK-AND-007/009/010/025` (HIGH) | Да — HIGH | OK по содержанию. Не покрывает `RISK-SRV-002` — [REV-SYS-001](01-findings.md#rev-sys-001) |
| 014 Partial MediaStore visibility / восстановление установки | Да | `RISK-AND-015/024/029` (HIGH), S03 | Да — HIGH | OK. Корректно сказано, что cloud mapping от этого не защищает |
| 015 Observer loss, KEEP и constraints | Да | `RISK-AND-026` (MEDIUM), `RISK-AND-039` (LOW), A10 | Да — MEDIUM | OK. Отсутствие deadline не превращено в нарушение |
| 016 Ограниченная end-to-end диагностика и automated evidence | Да | `RISK-AND-036` (HIGH), `RISK-AND-027/035`, `RISK-SRV-030/038`, S13, S14, `AND-OPS-002` | Да — HIGH | OK. HIGH обоснован Android-плечом; Server tests не выданы за системное подтверждение |
| 017 Onboarding/recovery через незавершённый email flow | Да | `RISK-SRV-026/027`, `AND-AUTH-009`, A07 | Да — MEDIUM | OK. `RISK-SRV-027` на Server HIGH, но системное последствие (затруднённое, не утраченное восстановление) обосновывает MEDIUM — допустимое различие по правилу файла |
| 018 Upload resources / network / storage capacity | Да | `RISK-SRV-033`, `SRV-STORAGE-003`, `RISK-AND-021/022/031/038` | Да — MEDIUM | OK |

Ни один риск не является механическим копированием component risk: из 38 `RISK-SRV` и 39 `RISK-AND` System оставил 18 системно значимых, объединённых по последствию. Скрытых remediation, назначенных исправлений и заявлений о произошедших инцидентах не обнаружено; завершающие абзацы файла прямо оговаривают, что объёмы, частоты гонок и реальные задержки не измерялись, а retention-политика не является «исправлением риска».

**RISK_ERROR: 1** (REV-SYS-001, MINOR — неполнота реестра, не ошибка существующих строк).
