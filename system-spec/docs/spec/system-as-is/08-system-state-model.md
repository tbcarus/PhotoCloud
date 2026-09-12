# System state mapping

**System phase является descriptive mapping и не является persisted state machine.** Отдельного system enum/runtime нет. Android действительно сохраняет media status, включая HASHING и UPLOADING. Server не имеет upload/sync enum: evidence — DB rows, поля и physical bytes.

| System phase (derived) | Android persisted state | Server state/evidence | Meaning |
| --- | --- | --- | --- |
| Discovered | Новая local row PENDING | Не установлен | Устройство обнаружено локально |
| Pending checksum | PENDING либо оставшийся HASHING до recovery | Не установлен | Проверка содержимого ещё не завершена |
| Checksum known | CHECKSUM_READY + checksum в обычном flow | Не установлен | Известен hash одного URI read |
| Server-existing | После existing обычно SYNCED | Matching user/folder/checksum на момент query | Logical evidence; не bytes proof |
| Upload pending | PENDING_UPLOAD | Missing в прежнем query либо absence CAMERA; текущее remote state неизвестно | Есть local очередь, отсутствие remote сейчас не гарантировано |
| Upload in progress/interrupted | UPLOADING | Не принят, temp, committed new/duplicate — любой из этих случаев возможен | Persisted marker не доказывает активный worker |
| Uploaded/logically present | До local write ещё может быть UPLOADING | New/duplicate DTO, logical commit | Server success может быть потерян по сети |
| Locally marked synced | SYNCED, ID optional | Только историческое existing/accepted ID evidence | Client belief; актуальный remote state не наблюдается |
| Failed | FAILED с reason/count/time либо pending после transient | Нет общего failed state; commit мог произойти до потерянного response | Локальная классификация либо необходимость повторного run |

## Persisted transitions

Обычная очередь: PENDING → HASHING → CHECKSUM_READY → SYNCED(existing) либо PENDING_UPLOAD → UPLOADING → SYNCED(upload). Hash/permanent upload failure → FAILED. Transient upload → PENDING_UPLOAD и stop. Eligible CHECKSUM_IO FAILED → PENDING. HASHING→PENDING и UPLOADING→PENDING_UPLOAD выполняются при входе соответствующей стадии.

Из любого local status size/mtime change может вернуть PENDING с clear hash/ID/failure fields; успешный snapshot без ID удаляет row. LOCAL_DELETED объявлен в Android, но active writer отсутствует: это не tombstone и не runtime phase текущего pipeline.

## SYNCED: точное значение

- Existing: status-only update; API ID не возвращает, обычно local serverFileId=null; уже имевшийся ID сохраняется.
- Upload: 2xx и id>0; serverFileId записывается, failure/count/time очищаются; response checksum/size/folder не сверяются.
- Обычная очередь повторно SYNCED не проверяет. Remote change не имеет входящего перехода.

Из SYNCED не следуют current physical bytes, известный ID, повторная remote verification, сохранность original media или актуальность account/host. Из server ID не следует StoredObject ID. Подробные non-guarantees — [12](12-data-consistency-and-durability.md).

## Concurrency и другие state owners

Большинство local writes адресуются только по local ID без old-status/content-version guard. Scan, restore и workers могут пересекаться. Например, existing(hashA) может прийти после scan reset в PENDING/null hash и записать SYNCED status-only. Concurrent stale DELETE может удалить row до upload response; local UPDATE тогда ничего не обновит. Это условные interleavings из Android frozen state model, а не новая гарантированная последовательность.

WorkManager RUNNING/ENQUEUED/BLOCKED — framework state. Android last SyncStatus (SUCCESS/SERVER_UNAVAILABLE/ERROR/RETRY_SCHEDULED + timestamp) — только память процесса. Ни одно не является полем FileItem или общим system state. SUCCESS может содержать per-file FAILED; PermissionDenied на scan/hash возвращает bare success без нового outcome; старый UI outcome может остаться.

Server combinations «rows+bytes», «rows без bytes», «orphan bytes», «нет rows/bytes» описаны в [12](12-data-consistency-and-durability.md), без присвоения им новых persisted enum.


## Источники

[S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
