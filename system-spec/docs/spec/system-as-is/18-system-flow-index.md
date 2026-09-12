# System flow index

Компактный индекс связывает entry, component boundaries, завершение и recovery. Success end-state ограничен описанными conditions; phases не вводят system enum. Отдельный System runtime отсутствует.

## SYS-FLOW-001 — Login and session establishment

| Field | As-Is |
| --- | --- |
| Entry condition | Настроен URL; пользователь отправляет email/password |
| Components | User → Android → Server/DB → encrypted storage |
| Main stages | POST login; Server validation/refresh INSERT; pair nonblank; local save/reconcile |
| Success end-state | Server refresh row и encrypted access/refresh pair; UI presence не validity proof |
| Failure/recovery | Unknown email, неверный пароль, disabled и banned наблюдаются клиентом как 401 / login failure; login failure показывает error; previous pair сама не clear; повтор login новая refresh row |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md) |
| System detail | [06-auth-session-lifecycle.md](06-auth-session-lifecycle.md) |

## SYS-FLOW-002 — Access token refresh

| Field | As-Is |
| --- | --- |
| Entry condition | Protected request получил401, guard допускает refresh |
| Components | Android auth/storage ↔ Server/DB |
| Main stages | Current-token replay либо plain refresh; validate persisted refresh; access save; original replay |
| Success end-state | Новый access и прежний refresh; операция получает final response |
| Failure/recovery | 4xx/5xx clear; IO сам не clear; повтор401 guard; mismatch004 |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) |
| System detail | [06-auth-session-lifecycle.md](06-auth-session-lifecycle.md) |

## SYS-FLOW-003 — New local media discovery

| Field | As-Is |
| --- | --- |
| Entry condition | Visible Images/DCIM ID и scan trigger |
| Components | MediaStore → Android/local DB; Server ping только worker gate |
| Main stages | Query full snapshot; normalize; insert/reset/update/stale delete |
| Success end-state | Новая PENDING row; metadata persisted |
| Failure/recovery | Query error не empty success; partial snapshot/apply границы; worker retry для scan Error |
| Source sections | [S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md) |
| System detail | [07-media-sync-lifecycle.md](07-media-sync-lifecycle.md) |

## SYS-FLOW-004 — Checksum and pre-check

| Field | As-Is |
| --- | --- |
| Entry condition | PENDING/HASHING и затем ready/non-null hashes |
| Components | Android/local DB/MediaStore ↔ Server folder/FileItem DB |
| Main stages | Hash URI; ROOT/direct children; CAMERA exists batch; partition |
| Success end-state | CHECKSUM_READY превращается в SYNCED(existing) или PENDING_UPLOAD(missing/absence) |
| Failure/recovery | Per-file FAILED; folder/exists error stop+retry; old batches сохраняются |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) |
| System detail | [07-media-sync-lifecycle.md](07-media-sync-lifecycle.md) |

## SYS-FLOW-005 — Upload new media

| Field | As-Is |
| --- | --- |
| Entry condition | PENDING_UPLOAD; предыдущие worker stages прошли |
| Components | Android/MediaStore/local DB → Server/DB/FS |
| Main stages | Recover UPLOADING; repeat CAMERA lookup; guards; multipart; temp/hash/type/target; final move; DB commit; DTO/local mark |
| Success end-state | New FileItem+StoredObject+bytes по нормальному пути; local SYNCED с ID |
| Failure/recovery | 400/409/413 FAILED; прочиеHTTP/IO stop+retry; DB/FS/local ACK неатомарны |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) |
| System detail | [07-media-sync-lifecycle.md](07-media-sync-lifecycle.md) |

## SYS-FLOW-006 — Existing media detection

| Field | As-Is |
| --- | --- |
| Entry condition | CAMERA существует, pre-check hash совпал с logical record |
| Components | Android/local DB ↔ Server DB |
| Main stages | Exists query по user/folder/hash; existing partition; status-only write |
| Success end-state | Local SYNCED; serverFileId обычно null; bytes не проверены |
| Failure/recovery | Network/stage retry; remote loss/mutation не наблюдаются обычным SYNCED |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) |
| System detail | [07-media-sync-lifecycle.md](07-media-sync-lifecycle.md) |

## SYS-FLOW-007 — Background retry and failure restoration

| Field | As-Is |
| --- | --- |
| Entry condition | OS исполняет scheduled/retry work, URL/ping gate проходит |
| Components | OS/WorkManager → Android → MediaStore/Server |
| Main stages | Eligible CHECKSUM_IO restore; full scan/hash/pre-check/upload; per-file outcomes |
| Success end-state | Очередь продвинута; SUCCESS не означает отсутствие FAILED |
| Failure/recovery | CHECKSUM_IO count/cooldown; transient head-of-line; constraints/cold URL откладывают recovery |
| Source sections | [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) |
| System detail | [09-error-retry-recovery.md](09-error-retry-recovery.md) |

## SYS-FLOW-008 — Local media deletion/disappearance

| Field | As-Is |
| --- | --- |
| Entry condition | ID отсутствует в successful local snapshot |
| Components | MediaStore → Android DB; Server без DELETE call |
| Main stages | Snapshot diff; local row hard delete |
| Success end-state | Local record отсутствует, server copy сохраняется |
| Failure/recovery | Partial visibility не physical deletion proof; reappearance создаёт PENDING; scan Error до apply не удаляет |
| Source sections | [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) |
| System detail | [11-delete-change-reconciliation.md](11-delete-change-reconciliation.md) |

## SYS-FLOW-009 — Cold-process and reboot recovery

| Field | As-Is |
| --- | --- |
| Entry condition | Process death/reboot; work/settings/rows могли persisted |
| Components | OS/WorkManager → Android; Server участвует после gate |
| Main stages | Empty runtime URL stops headless run; UI restores settings→URL; explicit reconcile; later stage resets |
| Success end-state | После restore и успешных gates worker может продолжить persisted queue |
| Failure/recovery | До UI restore SERVER_UNAVAILABLE+retry; timing OS unknown; server restart repair не делает |
| Source sections | [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A13](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md) |
| System detail | [10-startup-restart-reboot.md](10-startup-restart-reboot.md) |

## SYS-FLOW-010 — Logout and session exit

| Field | As-Is |
| --- | --- |
| Entry condition | URL и pair есть, пользователь выбирает Logout |
| Components | Android/storage/scheduling ↔ Server refresh DB |
| Main stages | Protected logout; revoke own refresh; HTTP response→local clear; reconcile |
| Success end-state | Remote refresh revoked при200, pair cleared; periodic/observer stopped при reconcile; access до exp |
| Failure/recovery | Network до clear может оставить pair; non-2xx response clear+error; one-time/in-flight могут продолжаться |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) |
| System detail | [06-auth-session-lifecycle.md](06-auth-session-lifecycle.md) |

## SYS-FLOW-011 — Changed local media reprocessing

| Field | As-Is |
| --- | --- |
| Entry condition | Тот же ID имеет иной size/mtime при scan |
| Components | MediaStore → Android/local DB → Server |
| Main stages | Reset hash/ID/status/failures; hash; existing либо upload |
| Success end-state | Новый local conclusion; possible new remote identity; old remote copy не удаляется |
| Failure/recovery | Unchanged size/mtime может скрыть bytes change; hash/upload race; обычные retry limits |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) |
| System detail | [11-delete-change-reconciliation.md](11-delete-change-reconciliation.md) |

## SYS-FLOW-012 — Duplicate upload / lost acknowledgement recovery

| Field | As-Is |
| --- | --- |
| Entry condition | Upload повторяется с теми же bytes/user/folder и сохранённой logical записью |
| Components | Android/local DB ↔ Server/DB/FS |
| Main stages | Server читает incoming bytes; finds tuple; temp cleanup best effort; previous DTO200; local positive ID mark |
| Success end-state | Прежний FileItem ID принят клиентом; новых remote objects нет |
| Failure/recovery | Missing bytes не repair; изменённый tuple/remote deletion снимает условие same-ID; lost local write допускает следующий retry |
| Source sections | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) |
| System detail | [09-error-retry-recovery.md](09-error-retry-recovery.md) |


Смена account/host не является отдельным поддержанным migration flow: её As-Is consequence отражено в [04](04-identity-ownership-and-scope.md) и SYS-MISMATCH-001. Remote rename/move/delete представлены как внешние изменения server state в [11](11-delete-change-reconciliation.md), а не как несуществующие Android commands.
