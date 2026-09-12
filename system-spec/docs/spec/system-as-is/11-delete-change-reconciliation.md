# Delete, change и reconciliation

## Local media change

Scan сопоставляет MediaStore по positive _ID. Size или lastModified отличаются → PENDING, clear checksum/serverFileId/failure reason/count/time; metadata обновляются. Новый checksum проходит обычный pre-check/upload. Если новые bytes имеют новый hash, может появиться новый FileItem; прежний remote FileItem Android не удаляет. Если hash прежний, possible existing/duplicate path использует прежнюю logical запись.

Rename/path change с тем же ID обновляет local metadata; сброс hash/status зависит от size/mtime, не имени/path самого по себе. Server name не переименовывается. Изменение bytes с прежними size/mtime может остаться незамеченным; это предел эвристики, а не гарантия version detection. Client dates берутся из MediaStore, server capturedAt из EXIF/fallback; общей timeline reconciliation нет.

## Local disappearance

| Event | Android DB | Device bytes | Server copy |
| --- | --- | --- | --- |
| ID отсутствует в полном успешном snapshot | Row физически удаляется независимо от status | Клиент их не удаляет | Remote DELETE отсутствует; копия сохраняется |
| Media moved вне DCIM / visibility narrowed | Тот же stale DELETE | Физическое существование возможно | Remote copy без изменений |
| Malformed row пропущена при успешном scan | Может удалить ранее видимую row | Не доказано удаление | Remote copy без изменений |
| Query error/null cursor до apply | Этот failed scan не делает stale DELETE | Не меняются клиентом | Не меняется этим scan |
| ID снова видим после stale DELETE | Новая PENDING row | Media доступно для чтения | Existing/duplicate может снова обнаружить logical запись |

LOCAL_DELETED не записывается, tombstone/remote deletion task нет. «Stale row удалена» не равно «пользователь удалил оригинал». Возврат строки после потери visibility может потерять локальную историю и вызвать повторный hash/pre-check, но не обязательно новые server bytes.

## Server-side changes

Android не использует file list/get/download/rename/move/copy/delete и не получает delta/events. Обычный SYNCED не перепроверяется:

| Server action через другой HTTP client | Remote result | Android observation |
| --- | --- | --- |
| Rename | Logical name меняется, physical path прежний | Local displayName и SYNCED сохраняются |
| Move | FileItem folder меняется, bytes прежние | Нет inbound folder/state update |
| Copy | Новый FileItem + StoredObject + bytes | Local record копии не появляется |
| Hard delete | Logical rows удалены; physical cleanup best effort | Local SYNCED/serverFileId могут устареть |
| Потеря physical bytes при сохранившейся DB | Logical listing/exists остаются | SYNCED не изменяется; pre-check также не physical check |

При позднем local content reset после remote move hash может стать missing в CAMERA и привести к новой independent copy в CAMERA. Это условное следствие folder-scoped query; Android сам не инициирует такую reverse reconciliation и не знает, куда remote object был перемещён.

На Server owner-delete удаляет все logical references объекта и StoredObject, затем bytes после commit; non-owner-reference branch удаляет только собственную logical reference. API создания shared/cross-owner references нет. Это system-visible data lifecycle, а не Android sharing capability. DeletedAt текущим API не используется, trash/restore/version history отсутствуют.

## Backup semantics

Фактическая модель — one-way upload с logical dedup в user/folder. Из локального удаления не следует удаление облачной копии; из server deletion не следует повторный upload неизменного local SYNCED. Нет общего change feed, conflict resolution, version/tombstone protocol или двустороннего зеркалирования.

Product retention и направление propagation остаются [SYS-OPEN-002](16-system-open-questions.md#sys-open-002). Отсутствие propagation описывает As-Is, без назначения будущей политики удаления.


## Источники

[S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md); [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md); [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A16](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md). Обозначения и область authority: [17-source-traceability.md](17-source-traceability.md).
