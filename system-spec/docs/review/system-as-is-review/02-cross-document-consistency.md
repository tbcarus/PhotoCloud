# Cross-document consistency

Проверка согласованности System пакета внутри себя: одно и то же понятие не должно иметь разный смысл в разных файлах 00…18.

Result:

* **CONSISTENT** — совпадение смысла во всех проверенных файлах;
* **MINOR_DIFFERENCE** — локальная неточность или неполнота, не меняющая системную модель;
* **CONTRADICTION** — взаимоисключающие утверждения.

Итог по 30 областям: **CONSISTENT 27**, **MINOR_DIFFERENCE 3**, **CONTRADICTION 0**.

| # | Area | Files checked | Result | Notes |
| --- | --- | --- | --- | --- |
| 1 | System boundary | 00, 01, 02, 13, 17 | CONSISTENT | Inside: Android client + local DB/settings/encrypted tokens, server application, PostgreSQL, server-owned storage. External: MediaStore, OS permissions/scheduling, сеть, SMTP, device filesystem. 01 корректно различает «managed data входят в persistence boundary» и «mounts/эксплуатационные процедуры не превращаются в функции приложения». Внешний authority внутренним state нигде не объявлен. Отсутствие System runtime layer подтверждено в 01, 08, 13, 18 |
| 2 | User / principal identity | 03, 04, 05, 06 | CONSISTENT | Везде: Server `User.id`, unique lowercase email, JWT `sub`=email, principal перезагружается из БД. 04 добавляет предел «совпадающее числовое значение на другом host не удостоверяет тот же объект»; 03 не противоречит |
| 3 | Device media identity | 03, 04, 07, 11, 18 | CONSISTENT | MediaStore `_ID` > 0 и content URI; authority видимости — OS/provider; name/path не identity; отдельного volume/device namespace нет. 11 и 18 (FLOW-008) одинаково трактуют исчезновение ID |
| 4 | Android local identity | 03, 04, 07, 08, 11 | CONSISTENT | `mediaStoreId` как local PK, без account/server/folder/device namespace; два local ID могут иметь один hash. 07 (переход MediaStore → local index) и 04 (identity matrix) совпадают |
| 5 | `serverFileId` | 03, 04, 05, 07, 08, 12, 18 | CONSISTENT | Записывается только из положительного `id` успешного upload; existing выполняет status-only write и не заполняет/не очищает поле; в обычном fresh path остаётся null; сбрасывается при size/mtime change (08, 11). Ни один файл не утверждает, что `SYNCED` подразумевает известный ID |
| 6 | Folder identity | 03, 04, 05, 07 | CONSISTENT | `Folder.id` в конкретной server DB; ROOT unique per user; sibling names case-insensitive unique; типы ROOT/CAMERA/FILES/USER; local `relativePath` не отображается в server folder tree (02, 04, 11) |
| 7 | Checksum | 02, 04, 05, 07, 12, 14 | CONSISTENT | Строго разделены Android checksum (SHA-256 одного URI read, не передаётся как authority) и Server checksum (SHA-256 принятого stream). Uniqueness tuple `user+folder+checksum` одинаков в 04, 05, 12. Нигде не утверждается равенство двух reads — это основа `SYS-MISMATCH-003` |
| 8 | Auth | 05, 06, 13, 18 | CONSISTENT | Access 20 мин, refresh 7 дней, BCrypt, enabled/banned только на login, activation code 3 дня. Значения совпадают в 00, 05, 06, 16 (`SYS-OPEN-008`) |
| 9 | Refresh | 05, 06, 09, 14, 18 | CONSISTENT | Plain request без Bearer; без rotation/продления/consumption; 401 invalid, 403 revoked; non-2xx → clear pair; IOException → pair сохраняется; 2xx без access → сохраняется; blank access принимается. Таблица outcomes в 06 совпадает со строками 09 и с `SYS-FLOW-002` |
| 10 | Logout | 05, 06, 10, 18 | CONSISTENT | Protected request; revoke одного refresh; clear после HTTP response **до** проверки успешности; access живёт до exp; Room/settings не очищаются; one-time `photo_sync` не отменяется. Идентично в 00, 06, 10, 18 (FLOW-010) |
| 11 | CAMERA | 00, 05, 07, 13, 14, 16, 18 | CONSISTENT | Lazy ROOT не создаёт CAMERA; поиск первого прямого потомка по `folderType`, не по имени; absence на pre-check — нормальный bootstrap; CAMERA создаётся первым default upload IMAGE/VIDEO. Нормальный bootstrap нигде не объявлен mismatch (прямо оговорено в 07 и 14) |
| 12 | Pre-check | 05, 07, 08, 09, 18 | CONSISTENT | Обязательный own `folderId`; до 500 raw, максимум 500 unique lowercase hashes из максимум 500 local rows; logical partition без file IDs; не reservation, не physical check, не связывает следующий upload. Защитная ветка неполной partition указана в 05 и 09 одинаково |
| 13 | Duplicate upload | 00, 05, 07, 12, 18 | CONSISTENT | Те же user/folder/checksum → HTTP 200 прежний DTO с прежними name/metadata/dates; не 409; bytes не проверяются и не восстанавливаются; тот же checksum в другой folder → новые logical/object/bytes. Одинаково в 00, 05 (таблица), 07 (стадия 11), 12, 18 (FLOW-012) |
| 14 | Upload target | 00, 05, 07, 14 | CONSISTENT | Explicit own folder без ограничения MIME; omitted `folderId` → server-detected IMAGE/VIDEO в CAMERA, остальное в FILES; решение по detected bytes, не по local MIME header; part выбирается один раз на весь upload run. Совпадает во всех четырёх файлах |
| 15 | `SYNCED` | 00, 04, 07, 08, 11, 12, 18 | CONSISTENT | Всюду Android-side local conclusion. Ни один файл не выводит из него physical bytes, известный ID, повторную verification, сохранность оригинала или актуальность account/host. 08 и 12 дают согласованные перечни non-guarantees |
| 16 | Server logical existence | 05, 07, 08, 12 | CONSISTENT | Matching `FileItem` в scope user+folder на момент запроса; 12 выделяет «Logical existence» как отдельный класс доказательства и отличает от physical/durability/reachability/client belief/server record |
| 17 | Physical bytes | 03, 07, 11, 12 | CONSISTENT | Отдельный authority (server filesystem); возможны rows без bytes и orphan bytes без rows; наличие path не удостоверяет актуальный checksum. Таблица комбинаций DB/FS в 12 согласуется с 03 и 11 |
| 18 | Retry | 09, 12, 13, 18 | CONSISTENT | Три механизма Android явно разделены (checksum eligibility / HTTP-auth / WorkManager); Server только возвращает результат и не назначает client policy — сказано в 02, 09 и 05. Фиктивная общая system retry policy не создана. Числа count<3 и 300000 мс совпадают в 00, 09, 13 |
| 19 | WorkManager / background | 09, 10, 13, 18 | CONSISTENT | One-time KEEP и periodic UPDATE с разными unique names могут пересекаться; 1 час — настройка, не SLA; CONNECTED допускает metered; battery-not-low только у periodic; backoff делегирован framework defaults и не объявлен client constant. Одинаково в 09, 10, 18 (FLOW-007/009) |
| 20 | Process death | 08, 10, 12, 18 | CONSISTENT | Persisted: local rows, encrypted pair, IP/port, work scheduling. Утрачены: in-memory URL, observer, last outcome, server in-flight flags. Маркеры HASHING/UPLOADING могут остаться без активного worker; recovery только при входе соответствующей стадии — идентично в 08, 09, 10, 12 |
| 21 | Reboot | 10, 18 | CONSISTENT | Persisted work не устраняет cold URL gate; app-specific boot restore отсутствует; реальные OS/OEM сроки оставлены `SYS-OPEN-016`. Факт пустого URL не превращён в гипотезу, а неизвестность timing не превращена в факт |
| 22 | BaseUrl | 00, 04, 10, 18 | CONSISTENT | In-memory, initial empty, Worker сам settings не читает; UI восстанавливает URL до ping, и успешный UI ping для restore не требуется; IP/port сохраняются после successful public test. Два разных утверждения (сохранение против восстановления) не конфликтуют и в 10 приведены в одной таблице корректно |
| 23 | Local deletion | 00, 11, 13, 18 | CONSISTENT | Успешный snapshot без ID → физическое удаление local row независимо от status; оригинал и server copy сохраняются; remote DELETE не вызывается; `LOCAL_DELETED` объявлен без активного writer и не выдан за tombstone (08, 11) |
| 24 | Reverse reconciliation | 02, 11, 13, 15 | CONSISTENT | Отсутствует во всех файлах; `SYS-CAP-018` NOT PRESENT; server rename/move/copy/delete представлены как внешние изменения server state, а не как Android commands (11, 18 финальная ремарка) |
| 25 | DB / filesystem consistency | 07, 09, 12 | CONSISTENT | Порядок new upload одинаков: temp+hash/size/type → resolve target → duplicate/name checks → final move → DB transaction → DTO после commit. Общей atomic transaction нет; компенсации best effort; startup repair отсутствует. 07 (mermaid и таблица), 09 и 12 совпадают |
| 26 | System capabilities | 13 против 02, 05, 07, 11, 12 | CONSISTENT | 26 статусов согласуются с профильными разделами; `UNUSED` не применён по объявленному правилу; server-only группы не переписаны как доступные Android capabilities; формулировка каждой capability семантически согласована со своим статусом |
| 27 | Mismatches | 14 против 00, 04, 05, 07, 09, 16 | CONSISTENT | Все четыре упоминаются с одинаковыми условиями во всех ссылающихся файлах; 00 перечисляет их сжато без искажения; не повышены до mismatch перечисленные в 14 contract limits — и они действительно не названы mismatch нигде в пакете |
| 28 | Risks | 15 против 00, 09, 12, 14 | MINOR_DIFFERENCE | Формулировки и severity согласованы, но колонка `Sources` содержит одинаковый boilerplate на всех 18 строках и опускает релевантные `RISK-SRV-*` — [REV-SYS-003](01-findings.md#rev-sys-003). Дополнительно последствия `RISK-SRV-002`/`RISK-SRV-005` не представлены строкой реестра, хотя факты есть в 05 и 06 — [REV-SYS-001](01-findings.md#rev-sys-001) |
| 29 | OPEN | 16 против 00, 06, 09, 11, 12 | MINOR_DIFFERENCE | Все 21 сформулированы как вопросы, ни один не отвечен, типы применены единообразно и объявленное отображение типов соблюдено. `OPEN-SRV-012`/`OPEN-SRV-017` не представлены ни пунктом, ни явным исключением — [REV-SYS-002](01-findings.md#rev-sys-002). Ссылки из 12 и 05 на `SYS-OPEN-014/015` разрешаются корректно |
| 30 | Flows | 18 против 05, 06, 07, 09, 10, 11, 12 | MINOR_DIFFERENCE | Ни один из 12 flows не вводит поведение, отсутствующее в основных разделах; все стадии, success end-states и failure-ветки прослеживаются. Единственная неточность — `Entry condition` в `SYS-FLOW-001`, расходящийся с собственной failure-строкой: [REV-SYS-004](01-findings.md#rev-sys-004) |

## Отдельно проверенные точки возможного смешения понятия «file»

Раздел 16 задания требует искать места, где слово «file» может ошибочно смешивать MediaStore media, Android DB record, server logical file, `StoredObject` и physical bytes.

| Проверенное место | Результат |
| --- | --- |
| 03 «Пять разных объектов в пути фото» | Различение введено явно и корректно: «Это не пять реплик одной записи» |
| 04 identity matrix | Пять идентификаторов разнесены по отдельным строкам с указанием creator/scope/persistence |
| 05 upload/pre-check | «logical file», «logical ID», «FileItem.id, не StoredObject.id» — смешения нет |
| 07 таблица переходов | Каждая строка указывает owner и source of truth отдельно для local record, logical FileItem, StoredObject и bytes |
| 08 SYNCED | «Из server ID не следует StoredObject ID» — различение сохранено |
| 12 комбинации DB/FS | Четыре комбинации описаны через logical record и bytes раздельно |
| 18 flows | `SYS-FLOW-005/006/012` используют «logical» там, где речь о записи, и «bytes» там, где о содержимом |

Случаев, где смешение действительно меняет смысл, не обнаружено. Finding не создавался.
