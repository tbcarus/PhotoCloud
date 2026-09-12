# Data consistency и durability

## Consistency boundaries

| Boundary | Связь и evidence | Возможный разрыв | Recovery As-Is |
| --- | --- | --- | --- |
| Android DB ↔ MediaStore | Snapshot local ID/metadata; size/mtime heuristic | Partial visibility, scan/apply partial writes, stale events; bytes mutable | Следующий scan; stale local DELETE, content reset; не byte-version proof |
| Android DB ↔ Server | Existing(hash,folder) или upload ID → local status | Remote commit до local mark, unscoped pair/DB, remote mutations, concurrent local writes | Pending retry; SYNCED не reverse reconcile |
| Server DB ↔ filesystem | StoredObject path/hash/size + FileItem reference ↔ final bytes | Move/DB/cleanup/crash не одна transaction | Synchronous best-effort cleanup; repair/orphan jobs отсутствуют |
| Android checksum ↔ uploaded bytes | Два независимых URI reads, Server считает принятый stream | Content change/redaction/length changes; response checksum не сравнивается | Автоматической integrity reconciliation нет |
| Server logical existing ↔ physical bytes | Exists/duplicate смотрят FileItem user/folder/checksum | Rows могут остаться без readable bytes | Existing/duplicate не проверяют/не чинят; download отдельно может404 |
| Local SYNCED ↔ current remote state | Историческое local conclusion | Remote delete/move/loss, scope switch, null/stale ID | Нет revalidation обычного SYNCED |

## Разные доказательства

- **Logical existence:** DB содержит matching FileItem; query привязан к user/folder и времени запроса.
- **Physical existence/readability:** bytes присутствуют/читаются на filesystem; это не устанавливается checksum pre-check.
- **Durability:** переживание crash/потери storage; не выводится из API200, SQL commit или попытки ATOMIC_MOVE.
- **Current reachability:** endpoint либо URI доступен сейчас; public ping не проверяет DB/storage/SMTP.
- **Client belief:** Android status и optional ID; ни Server state enum, ни сертификат сохранности.
- **Server record:** сохранённые metadata/hash/size; без checksum scrub не доказывает последующую неизменность bytes.

Source-of-truth matrix — [04](04-identity-ownership-and-scope.md#source-of-truth). Authorities устанавливают разные факты и могут расходиться.

## Server persistence sequence

New upload: temp stream/hash/size/type → resolve target (default folder может сохраниться отдельно) → duplicate/name checks → final move → DB transaction StoredObject/FileItem/optional metadata → DTO. Logical FileItem и StoredObject сохраняются в одной DB transaction; filesystem в неё не входит.

Final move пытается ATOMIC_MOVE, с ordinary move fallback. Это атомарность перемещения пути при поддержке, не crash-durability contract. Общий durable journal/fsync/repair mechanism отсутствует. При DB rollback Server пытается удалить свой final; cleanup failure оставляет orphan. При post-commit runtime error catch может удалить bytes, сохранив rows. При partial IO до успешного return target не гарантированно очищается; kill/power failure обходит catch.

| DB / FS combination | System meaning | Что может видеть Android |
| --- | --- | --- |
| FileItem/StoredObject + readable bytes | Обычный remote stored object в данный момент | Existing либо upload ID; Android самостоятельно readability не проверяет |
| FileItem/StoredObject + missing/unreadable bytes | Logical presence без physical availability | Existing/duplicate могут дать SYNCED; Server download404 не используется Android |
| Нет logical record, bytes остались | Orphan, по прежнему logical ID недоступен | Pre-check может missing; upload создаст новый объект |
| StoredObject без FileItem | Object metadata без logical reference, возможно вне обычного flow | Отдельного object API нет |
| Нет rows и bytes | Отсутствие либо завершённое удаление | Нет inbound event; старый local SYNCED может сохраниться |

## Current guarantees в подтверждённом объёме

| Guarantee | Условия и точный предел |
| --- | --- |
| Remote ownership enforcement | FileItem/Folder API lookup id+user; missing/foreign404. Это API boundary, не доказательство всех межтабличных owner invariants для внешних DB writes |
| Logical dedup uniqueness | Server tuple user/folder/checksum; ordinary duplicate возвращает200 прежний DTO, не новый object |
| Server upload content accounting | Server вычисляет SHA-256 и size из принятого stream; не принимает client checksum как authority |
| Normal new-upload persistence order | Final IO предшествует DB transaction; successful DTO нового объекта строится после commit |
| Client success criteria | Existing → status-only SYNCED; upload2xx id>0 → ID/SYNCED и clear failure fields в обычном завершённом local write |
| Local completed state persistence | Завершённые DB writes переживают process death; HASHING/UPLOADING восстанавливаются только при входе соответствующей стадии |

Это конкретные механизмы, а не безусловная гарантия «каждое фото сохранено». Наличие IMPLEMENTED в component/system matrix не заменяет runtime validation.

## Current non-guarantees

- SYNCED не гарантирует serverFileId, читаемые server bytes, текущий original или repeated remote verification.
- Existing и duplicate200 не доказывают physical bytes и не восстанавливают их.
- Positive upload ID не удостоверяет равенство local hash и server accepted bytes; response hash/size/target игнорируются.
- Server DB commit не атомарен с physical bytes или Android mark; потерянный ответ не доказывает rollback.
- Повтор upload не global exactly-once: user/folder/bytes и logical record могут измениться.
- Local delete не удаляет server copy; server mutation не меняет автоматически client belief.
- Worker SUCCESS не означает отсутствие FAILED; configured periodic interval не гарантирует deadline.
- Process/reboot persistence work requests не гарантирует headless URL recovery.
- Внешние backup/restore, RPO, volume durability, состояние deployed DB и current reachability не установлены двумя frozen specs.

## Operational recovery boundary

Server startup не выполняет сверку DB/FS; нет repair, scrub, retention и orphan cleanup jobs. Из отсутствия app jobs не следует отсутствие внешнего эксплуатационного процесса: он неизвестен, [SYS-OPEN-015](16-system-open-questions.md#sys-open-015). Настоящий пакет не проверяет live bytes и не формулирует будущие гарантии.


## Источники

[S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
