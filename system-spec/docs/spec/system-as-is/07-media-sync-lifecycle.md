# Media sync lifecycle

## Entry и стадии

Entry: доступное image в MediaStore/DCIM и последующий scan. Worker требует in-memory URL и successful public ping; затем restore → scan → checksum → CAMERA/pre-check → CAMERA/upload → local outcome. UI scan также индексирует media, но не создаёт отдельную кнопку ручного полного sync.

| Stage | Android state | Server operation/state | Persistent result | Failure behaviour |
| --- | --- | --- | --- | --- |
| 1. Media appears | Local row ещё нет | Нет | MediaStore/device bytes вне client DB | OS permission/visibility/event может отложить discovery |
| 2. Discover | Snapshot external Images/DCIM | Worker уже прошёл public ping; при UI scan отдельный путь | Пока snapshot в памяти | Query error/null cursor — scan Error, не пустой success |
| 3. Persist index | Новый PENDING; size/mtime change reset; stale delete | Нет | Local ID/URI/metadata; hash/ID null для нового/изменённого | Partial apply может оставить часть writes |
| 4. Checksum | PENDING → HASHING → CHECKSUM_READY | Нет | SHA-256 URI read; success очищает failure/count/time | Per-file FAILED либо stage retry; permission guard |
| 5. Resolve pre-check folder | Ready/non-null hashes | GET ROOT, direct children | ROOT может создать DB row; CAMERA ID в памяти клиента | Error → stage retry; upload этого run не начинается |
| 6. Pre-check | CHECKSUM_READY | POST own CAMERA folderId + batch hashes | Ответ не сохраняется на Server; local status writes по partition | Error → retry; прошлые batches сохраняются |
| 7. Existing path | Status-only SYNCED | Logical matching FileItem в user/folder | ID не приходит; прежний serverFileId не меняется, обычно null | Bytes не проверены; remote mutation после ответа не наблюдается |
| 8. Upload pending | Missing или CAMERA absent → PENDING_UPLOAD; recovery UPLOADING | Повторный CAMERA lookup | Local status persisted; folderId part только в памяти | Absence/error второго lookup → omitted folderId |
| 9. Upload request | MIME/pre-open guards → UPLOADING; URI открыт снова | Multipart; temp bytes, hash/size/MIME; target resolution | Temp physical, default folder может сохраниться отдельно | 400/409/413 permanent; прочие HTTP/IO transient у Android |
| 10. New remote media | UPLOADING до response | Final move → DB transaction StoredObject + FileItem + metadata | Physical bytes и DB commit раздельны | Best-effort cleanup; возможны orphan/missing bytes |
| 11. Duplicate upload | Та же очередь UPLOADING | Existing user/folder/checksum →200 прежний DTO | Новые logical/object/bytes не создаются | Existing bytes не проверяются/не чинятся |
| 12. Local conclusion | 2xx id>0 → SYNCED | Response logical FileItem.id | serverFileId + clear failure/count/time; local hash прежний | Потеря response/local write оставляет неопределённый для клиента server result |

## Главная последовательность

```mermaid
sequenceDiagram
    participant M as MediaStore
    participant A as Android pipeline
    participant L as Local DB
    participant S as Server
    participant D as Server DB
    participant F as File storage
    A->>S: Public ping (worker gate)
    S-->>A: Success
    A->>L: Restore eligible failures
    A->>M: Scan Images/DCIM
    M-->>A: Visible snapshot
    A->>L: Insert/reset/update/delete stale
    A->>M: Read URI for SHA-256
    A->>L: CHECKSUM_READY
    A->>S: Get ROOT, direct children
    break Pre-check folder lookup error
        A->>A: Stop run, request retry
    end
    alt CAMERA found
        A->>S: Exists(folderId, hashes)
        S->>D: Logical user/folder checksum query
        S-->>A: Existing / missing, no IDs
        A->>L: SYNCED status-only / PENDING_UPLOAD
    else CAMERA absent
        A->>L: Ready hashes to PENDING_UPLOAD
    end
    break Exists request error
        A->>A: Stop run, request retry
    end
    Note over S,F: Existing does not inspect physical bytes
    A->>L: Recover UPLOADING to PENDING_UPLOAD
    A->>S: Resolve CAMERA again
    S-->>A: ID, absence or error
    loop Pending uploads until complete or transient failure
        A->>M: Pre-open then reopen URI
        A->>L: UPLOADING
        A->>S: Multipart file + optional folderId
        S->>F: Temp write; compute received hash/size/type
        S->>D: Resolve target; inspect duplicate
        alt Logical duplicate
            S-->>A: 200 previous FileItemDto
        else New content
            S->>F: Move temp to final
            S->>D: Commit StoredObject + FileItem + optional metadata
            S-->>A: 200 FileItemDto
        end
        A->>L: 2xx id>0: SYNCED with serverFileId
        Note over A,L: Failure: FAILED or pending/retry; see stage table
    end
```

Это descriptive sequence с успешной серверной веткой upload; ошибки final/DB/response раскрыты в [09](09-error-retry-recovery.md) и [12](12-data-consistency-and-durability.md). Android не вызывает отдельные create StoredObject/FileItem endpoints.

## Ownership, identity и recovery каждого перехода

| Transition | Owner / identity / source of truth | Request/response и success | Persistence boundary / recovery boundary |
| --- | --- | --- | --- |
| MediaStore → local index | OS media _ID/URI → Android PK=mediaStoreId; authority текущей видимости MediaStore | Query snapshot, insert/reset/update/stale delete | Local DB writes отдельно от snapshot; следующий scan повторяет наблюдение, а не remote reconciliation |
| Local record → checksum | Android; тот же local ID и hash конкретного stream | URI read → lowercase SHA-256 | HASHING/hash persisted; recovery HASHING→PENDING только при входе checksum stage |
| Checksum → server pre-check | Client hash, server user/folder scope | folderId+hashes → logical existing/missing | Нет reservation/server write; batch local writes сохраняются независимо |
| Pre-check → existing conclusion | Server DB свидетельствует logical existence, Android владеет status | Existing → SYNCED без ID | Нет physical verification или автоматического repeated existing для SYNCED |
| Missing/absence → upload | Android очередь; серверный target из explicit ID либо detected bytes | Multipart и current principal | Hash/read/target не snapshot; повтор целого upload, не byte resume |
| Upload → logical FileItem | Server генерирует FileItem.id; user/folder/checksum uniqueness | New/duplicate DTO200 | DB commit до response; потерянный ACK повторяется условно с тем же tuple |
| Logical FileItem ↔ StoredObject | Server; отдельные DB IDs, FileItem reference | Внутри нового upload сохраняются вместе в DB transaction | DB rollback/compensation не общая transaction с FS |
| StoredObject → physical bytes | Server metadata описывает path/hash/size; filesystem хранит bytes | Final move предшествует DB save | Crash/cleanup может разделить rows/bytes; startup repair отсутствует |
| Response → local SYNCED | Android serverFileId=logical response ID | 2xx id>0, без hash/size/target comparison | Remote commit/local mark неатомарны; recover UPLOADING лишь при достижении upload stage |

## CAMERA bootstrap и fallback

ROOT создаётся лениво; direct children не создаёт CAMERA. Pre-check absence и upload absence — нормальная цепь bootstrap. Первый default IMAGE/VIDEO upload создаёт CAMERA; последующие файлы того же upload run могут всё ещё идти без folderId, поскольку part выбирается один раз до цикла.

При successful CAMERA pre-check и последующей lookup Error upload также идёт без folderId. Если Server обнаружит не IMAGE/VIDEO, фактический target будет FILES. Android response folderId игнорирует и может поставить SYNCED. Это условие `SYS-MISMATCH-002`; стабильный IMAGE bootstrap сам по себе mismatch не создаёт.

## Bytes, duplicate и завершение

Android checksum и upload читают исходный URI независимо; Server считает checksum своих принятых bytes. Private immutable snapshot, проверка response hash/size и content generation отсутствуют. Изменение между reads — `SYS-MISMATCH-003`.

Existing означает logical presence на момент запроса; SYNCED может иметь null ID; physical bytes могут отсутствовать. Это три отдельных факта. Duplicate200 может заполнить ID, но physical proof всё равно не возникает.

New upload сохраняет bytes/DB через раздельные boundaries. Same-folder duplicate возвращает прежние metadata/name/time без bytes repair. Copy/cross-folder upload не делают shared physical dedup. Server image metadata best effort; Android local dates в capturedAt не отправляет и response metadata не сохраняет.

Внутри одного worker партии checksum100, pre-check500 и upload20 повторяются до окончания очереди либо ошибки/no-progress; 20 — не общий лимит uploads за run. Per-file FAILED не обязательно делает worker retry. WorkManager SUCCESS не означает, что вся библиотека SYNCED.


## Источники

[S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
