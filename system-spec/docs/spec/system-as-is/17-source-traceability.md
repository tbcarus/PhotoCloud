# Source traceability и self-check

## Input authority и workspace mapping

| Logical input | Authoritative actual path | Version / status |
| --- | --- | --- |
| server/docs/spec/server-as-is/ | [PhotoCloudServer/docs/spec/server-as-is/README.md](../../../../PhotoCloudServer/docs/spec/server-as-is/README.md) | PhotoCloud Server As-Is v1 — FROZEN; [S20](../../../../PhotoCloudServer/docs/spec/server-as-is/20-freeze-record.md) |
| android/docs/spec/android-as-is/ | [PhotoCloudClient/docs/spec/android-as-is/README.md](../../../../PhotoCloudClient/docs/spec/android-as-is/README.md) | PhotoCloud Android As-Is v1 — FROZEN; [A22](../../../../PhotoCloudClient/docs/spec/android-as-is/22-freeze-record.md) |
| system-spec/docs/spec/system-as-is/ | [System README](README.md) | PhotoCloud System As-Is v1 — FROZEN |

Snn означает файл nn в фактическом Server input; Ann — файл nn в Android input. Ссылки ведут прямо в два authoritative directories. Историческая embedded ссылка Android на sibling server-as-is не выбирается как третий input. Сведения об audit/review происхождении в frozen README/freeze используются только для статуса входов; исторические пакеты, verification logs и production evidence paths не использованы как самостоятельные источники.

Freeze даты входов — 2026-09-12. Android frozen README указывает production revision `2b9ab698c2068d1e20eeb53aa7547557c3e17b54`; System не переаудирует эту revision. Наличие FROZEN не удостоверяет deployed runtime.

Server-only assertions относятся к Server authority; Android-only — к Android authority. Cross-component conclusions получены сопоставлением обеих. При недостатке runtime evidence вывод оставлен OPEN; один input не назначается «победителем» в semantic mismatch. Отдельных противоречий frozen facts не обнаружено.

## Area mapping

| System area | Server As-Is source | Android As-Is source | System output |
| --- | --- | --- | --- |
| Boundary / responsibilities | [S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | [A01](../../../../PhotoCloudClient/docs/spec/android-as-is/01-system-boundary.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [01](01-system-boundary.md); [02](02-component-responsibilities.md) |
| Identity / ownership / scope | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [03](03-system-domain-model.md); [04](04-identity-ownership-and-scope.md) |
| Auth / session / logout | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md) | [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [05](05-effective-api-contract.md); [06](06-auth-session-lifecycle.md); [09](09-error-retry-recovery.md) |
| API used / unused | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md) | [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A16](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [02](02-component-responsibilities.md); [05](05-effective-api-contract.md); [13](13-system-capability-matrix.md) |
| Media discovery / metadata | [S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A14](../../../../PhotoCloudClient/docs/spec/android-as-is/14-security-privacy.md) | [03](03-system-domain-model.md); [07](07-media-sync-lifecycle.md); [11](11-delete-change-reconciliation.md) |
| Folder / CAMERA | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md) | [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [04](04-identity-ownership-and-scope.md); [05](05-effective-api-contract.md); [07](07-media-sync-lifecycle.md) |
| Checksum pre-check | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [05](05-effective-api-contract.md); [07](07-media-sync-lifecycle.md); [08](08-system-state-model.md) |
| Upload | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md) | [A06](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [05](05-effective-api-contract.md); [07](07-media-sync-lifecycle.md); [09](09-error-retry-recovery.md); [12](12-data-consistency-and-durability.md) |
| Duplicate / lost ACK | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [05](05-effective-api-contract.md); [07](07-media-sync-lifecycle.md); [09](09-error-retry-recovery.md); [12](12-data-consistency-and-durability.md) |
| State / SYNCED | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md) | [08](08-system-state-model.md); [12](12-data-consistency-and-durability.md) |
| Retry / failure recovery | [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) | [09](09-error-retry-recovery.md); [18](18-system-flow-index.md) |
| Background / scheduling | [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A13](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md) | [09](09-error-retry-recovery.md); [10](10-startup-restart-reboot.md) |
| Process restart / reboot | [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A13](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md) | [10](10-startup-restart-reboot.md); [18](18-system-flow-index.md) |
| Deletion / change / remote mutations | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A09](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [11](11-delete-change-reconciliation.md); [12](12-data-consistency-and-durability.md) |
| Storage / physical durability | [S01](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md); [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S09](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md); [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md) | [A03](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md); [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md) | [03](03-system-domain-model.md); [12](12-data-consistency-and-durability.md) |
| Capabilities | [S15](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md) | [A16](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [13](13-system-capability-matrix.md) |
| Mismatch | [S03](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md); [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md) | [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [14](14-system-mismatches.md) |
| Risk / security / runtime evidence | [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S13](../../../../PhotoCloudServer/docs/spec/server-as-is/13-observability.md); [S14](../../../../PhotoCloudServer/docs/spec/server-as-is/14-testing-current-state.md); [S16](../../../../PhotoCloudServer/docs/spec/server-as-is/16-risks.md) | [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A14](../../../../PhotoCloudClient/docs/spec/android-as-is/14-security-privacy.md); [A15](../../../../PhotoCloudClient/docs/spec/android-as-is/15-testing-current-state.md); [A18](../../../../PhotoCloudClient/docs/spec/android-as-is/18-risks.md) | [15](15-system-risks.md) |
| Open questions / decisions / unknown | [S17](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md) | [A19](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md) | [16](16-system-open-questions.md) |
| Flows | [S05](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md); [S06](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md); [S07](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md); [S10](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md); [S12](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | [A05](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md); [A07](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md); [A08](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md); [A10](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md); [A11](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md); [A17](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | [18](18-system-flow-index.md) |

В 00 приведена системная сводка профильных разделов; в 01–12 источники сгруппированы по теме. Registry rows/entries 13–16 и flows18 содержат собственные ссылки. Sentence-level references не требуются; существенные claims прослеживаются через профильный раздел и эту таблицу.

## Registry reconciliation

| Registry | Definitions | Source boundary |
| --- | --- | --- |
| SYS-CAP | 26, 001…026 | Новые system capabilities; statuses не скопированы механически из SRV/AND |
| SYS-FLOW | 12, 001…012 | System entry→result/recovery, без нового runtime layer |
| SYS-MISMATCH | 4, 001…004 | One-to-one Android INT-AND-001…004 + relevant Server authority |
| SYS-RISK | 19, 001…019 | Возможные system последствия, включая credential/account control boundary; не полный перенос component risks |
| SYS-OPEN | 21, 001…021 | Консолидированные решения/runtime unknown, исходные OPEN IDs сохранены |

Дополнительные cross-component mismatches: 0. Противоречия frozen specifications: 0 обнаруженных при сопоставлении. Нормальный CAMERA bootstrap, existing без ID и Server-only unused capabilities отдельно не объявлены defects.

## Completeness self-check

| Review question | Coverage |
| --- | --- |
| Что происходит после новой фотографии? | 07; SYS-FLOW-003/004/005 |
| Как определяется наличие на Server? | 05/07; logical user/folder/hash; SYS-FLOW-006 |
| Что означает SYNCED? | 08/12; local conclusion, ID optional, no bytes proof |
| Где authoritative identity? | 03/04; local MediaStore ID отдельно от server FileItem/StoredObject |
| Как выбирается folder? | 05/07; ROOT/direct CAMERA, repeat lookup и detected-type fallback |
| Что происходит при duplicate? | 05/07; SYS-FLOW-012; 200 previous DTO, no repair |
| Что происходит при network failure? | 09; pending/stop/retry, auth/logout branch-specific |
| Что происходит при access expiry? | 06; SYS-FLOW-002; refresh/plain/replay и outcomes |
| Что после process death? | 10; persisted rows/pair, lost URL/observer, stage recovery |
| Что после reboot? | 10; cold gate известен, OS timing неизвестен |
| Что при local deletion? | 11; SYS-FLOW-008; local row delete, remote copy остаётся |
| Что при server copy change? | 11; reverse reconciliation отсутствует |
| Что гарантирует Server DB? | 04/12; logical identity/tuple/transaction, не physical existence |
| Что гарантирует physical storage? | 12; separate bytes evidence, crash durability deployment не установлена |
| Где возможна рассинхронизация? | 12 и SYS-MISMATCH-001…004, SYS-RISK registry |
| Какие Server capabilities Android не использует? | 02/05/13; 26 mappings grouped, не mismatch сами по себе |

## Authority, scope и abstraction self-check

- Состав на freeze: README и 20 документов 00…19; Status FROZEN. [19-freeze-record.md](19-freeze-record.md) фиксирует закрытие четырёх MINOR findings; To-Be SPEC-* items не создавались.
- Изменения ограничены выходным `system-spec/docs/spec/system-as-is/**`; component specifications не изменены. Проверка SHA-256 всех 46 входных Markdown-файлов до/после составления не выявила изменений.
- Production Java/Kotlin, tests, manifest/Gradle/configuration/SQL и live server filesystem не читались. Для closure использован только [independent System review](../../review/system-as-is-review/00-review-summary.md); старые audits/reviews/handoff не открывались; build/tests/live HTTP не запускались.
- Чтение frozen разделов configuration/testing/security описывает их утверждения; это не чтение соответствующих production/config/test файлов.
- Новых Git repositories/коммитов не создано. Source repositories и существующие документы вне output не редактировались.
- В описании сохранены только system boundaries, wire details, lifecycle/recovery и значимые последствия. Class/DAO/controller catalogs и SQL internals не перенесены.
- Logical vs physical objects, client belief vs server record, persisted local markers vs in-memory/framework state разделены. Все derived phases явно описательные.
- Mismatch, risk и OPEN имеют разные роли. Нет fixes, roadmap, автоматически назначенных решений или будущих runtime guarantees.
- Проверены состав, Markdown links, source/system ID references и допустимые registry statuses/types. Self-check не является independent review и не заявляет runtime correctness.

## Input content fingerprints

SHA-256 прочитанных authoritative документов и статусных записей фиксирует конкретный input content. Полные 46 файлов обоих каталогов отдельно сравнивались до/после для проверки неизменности; неиспользованные internal/verification документы не становятся источниками claims от самого факта хеширования.

| Source file | SHA-256 |
| --- | --- |
| [Server/00-server-as-is-summary.md](../../../../PhotoCloudServer/docs/spec/server-as-is/00-server-as-is-summary.md) | `70a270af4a82ac54eb423cd7d1187878efc7dfc9790796de71d55263545c9bc4` |
| [Server/01-system-overview.md](../../../../PhotoCloudServer/docs/spec/server-as-is/01-system-overview.md) | `29402b503f7972294c5c4389b5e2cfc8480b78f618dec1dd0cc84deaeb432627` |
| [Server/03-data-model.md](../../../../PhotoCloudServer/docs/spec/server-as-is/03-data-model.md) | `5a3db31c66568b880389c30be92203a89d61c205528bcb0a5584f4b24962a439` |
| [Server/05-api.md](../../../../PhotoCloudServer/docs/spec/server-as-is/05-api.md) | `167f8db7d1ff62ed2cd4ad92025d09b423ae7322d506551efbb2e79a2be609b8` |
| [Server/06-auth-security.md](../../../../PhotoCloudServer/docs/spec/server-as-is/06-auth-security.md) | `d88f6a5f1aa68dcf025d9a7f0981932e4e42e402f44e6e9760dbd210450b53ca` |
| [Server/07-file-storage.md](../../../../PhotoCloudServer/docs/spec/server-as-is/07-file-storage.md) | `3d0c83eb9b708a09193dea91ee7a5572a440a27804c601b0696947b3952cb993` |
| [Server/09-state-model.md](../../../../PhotoCloudServer/docs/spec/server-as-is/09-state-model.md) | `0875fba085ef064cd27bebe96ad21bbb50d09bcd962f5a9af15807ce89f14ad6` |
| [Server/10-errors-and-recovery.md](../../../../PhotoCloudServer/docs/spec/server-as-is/10-errors-and-recovery.md) | `94849020a23a05bb8dfe48540b9ae2213be7397e3ccfa32d5c935fa88b625534` |
| [Server/12-background-processing.md](../../../../PhotoCloudServer/docs/spec/server-as-is/12-background-processing.md) | `a4243b7d5943611f3a104d2a4ff12940f624a444543e29f3cdd6158b13534466` |
| [Server/13-observability.md](../../../../PhotoCloudServer/docs/spec/server-as-is/13-observability.md) | `8c8128aa996b47654a3227605c052386743dc7b4923421ab70545082076869a8` |
| [Server/14-testing-current-state.md](../../../../PhotoCloudServer/docs/spec/server-as-is/14-testing-current-state.md) | `37f8142bb956e186325f1792381ea08d9492ecc71d870a64d66b489e119cc0bf` |
| [Server/15-feature-matrix.md](../../../../PhotoCloudServer/docs/spec/server-as-is/15-feature-matrix.md) | `9ef2dd1f5237f538588d52e45eff31e947efb59e1ff34d65cec36190187b48c8` |
| [Server/16-risks.md](../../../../PhotoCloudServer/docs/spec/server-as-is/16-risks.md) | `89232d46d1a53e4dc18ee6085889551bd0559ff50b6d1dba924178ecd220f579` |
| [Server/17-open-questions.md](../../../../PhotoCloudServer/docs/spec/server-as-is/17-open-questions.md) | `0a625a8e51ce69c14e76b7a3ade5bb739d4693eb662a812b1c71fb4445442679` |
| [Server/20-freeze-record.md](../../../../PhotoCloudServer/docs/spec/server-as-is/20-freeze-record.md) | `012550fc5a22bcb5dd2e54f839ddd74f90f7197233789c8402a45f696bbc8407` |
| [Server/README.md](../../../../PhotoCloudServer/docs/spec/server-as-is/README.md) | `ba6190cc44893d5c33438defe3fbb85d0ee2649b09af8d7488c53aa6576d79c8` |
| [Android/01-system-boundary.md](../../../../PhotoCloudClient/docs/spec/android-as-is/01-system-boundary.md) | `e68f7dfdb22865db3cc98a5d45a4c911edf68d5c0c22ed2a9aaa4cb3dd498137` |
| [Android/03-local-data-model.md](../../../../PhotoCloudClient/docs/spec/android-as-is/03-local-data-model.md) | `a4abb8fca40cab50e735c932d2c5351e480fbd078db428686cdb7d5a87fbc4f3` |
| [Android/05-media-discovery.md](../../../../PhotoCloudClient/docs/spec/android-as-is/05-media-discovery.md) | `7c0fc2790c4698a7ef5bb674d18fc439dd90e6666176c26dbab54bc6877cdb57` |
| [Android/06-network-api.md](../../../../PhotoCloudClient/docs/spec/android-as-is/06-network-api.md) | `2b4a957e81715801d9c4213eef17e70774830398df666100510145a3eeee3a47` |
| [Android/07-auth-session.md](../../../../PhotoCloudClient/docs/spec/android-as-is/07-auth-session.md) | `2cf7ee097c529cb9a6637a87e3664d48560affe3b3efaf8fa4b522980e23c0ed` |
| [Android/08-sync-pipeline.md](../../../../PhotoCloudClient/docs/spec/android-as-is/08-sync-pipeline.md) | `7f9f503cc91e2fa8a419c27ab9145fe0a032e79afeaababc91c9e2c2cde5d240` |
| [Android/09-state-model.md](../../../../PhotoCloudClient/docs/spec/android-as-is/09-state-model.md) | `6bd22383493c3fc3ca08e9c1fb359a953b9b600207bf385f5ef3a0072ab4d60f` |
| [Android/10-background-execution.md](../../../../PhotoCloudClient/docs/spec/android-as-is/10-background-execution.md) | `494ac7f2e362e63e01028793eb731c2c17441a9e170704f4f585f34182581cbb` |
| [Android/11-errors-retry-recovery.md](../../../../PhotoCloudClient/docs/spec/android-as-is/11-errors-retry-recovery.md) | `8cec197c36365955ab4426d29b76310275f9ac6075e445ff5c09eae50a3fb36a` |
| [Android/13-configuration.md](../../../../PhotoCloudClient/docs/spec/android-as-is/13-configuration.md) | `929aec7fb2db3efba918d607700ede27f8440cb0cc7de256c16b19fc892a939e` |
| [Android/14-security-privacy.md](../../../../PhotoCloudClient/docs/spec/android-as-is/14-security-privacy.md) | `7fa9192995c11c5547267508fc615f0952957957bc632a17804f923179645d56` |
| [Android/15-testing-current-state.md](../../../../PhotoCloudClient/docs/spec/android-as-is/15-testing-current-state.md) | `557bf7860af2e651d00505491e8747323d9a841ffdb2a885721d76ba9133c301` |
| [Android/16-feature-matrix.md](../../../../PhotoCloudClient/docs/spec/android-as-is/16-feature-matrix.md) | `16e774779fe8e5eb62de69f77bf46ad3546a0cfb5f4cdeaaf430f25605adf79c` |
| [Android/17-server-client-consistency.md](../../../../PhotoCloudClient/docs/spec/android-as-is/17-server-client-consistency.md) | `d3fdf4c99f96ac95d708e51a2479fc9e2c9255fbd0a20d1d6f9422211842b8ec` |
| [Android/18-risks.md](../../../../PhotoCloudClient/docs/spec/android-as-is/18-risks.md) | `e119b52453699587a5d6044d3be8a31f921b8085f27b488ee0e439162343bcc7` |
| [Android/19-open-questions.md](../../../../PhotoCloudClient/docs/spec/android-as-is/19-open-questions.md) | `b1ce8a9f41d5f9cb0c64311c557f0e62de5577865b35b62e0ebdddc8552bc468` |
| [Android/22-freeze-record.md](../../../../PhotoCloudClient/docs/spec/android-as-is/22-freeze-record.md) | `45bcf4b7c55a72503496d6aa44731c5bd7d88a9fe8c1689b36337b33fab01018` |
| [Android/README.md](../../../../PhotoCloudClient/docs/spec/android-as-is/README.md) | `6b67052b2f75bdbc632a71ceb94744e89718f9e786311867bd3445a82d97004e` |

## Limited consistency / review closure check — 2026-09-12

Basis: [independent review](../../review/system-as-is-review/00-review-summary.md), verdict `PASS_WITH_MINOR_FIXES`, BLOCKER 0 / MAJOR 0 / MINOR 4; [findings](../../review/system-as-is-review/01-findings.md). Выполнено только закрытие REV-SYS-001…004, без нового review, reconciliation или поиска новых findings.

| Finding | Closure | Limited consistency check |
| --- | --- | --- |
| REV-SYS-001 | CLOSED: SYS-RISK-019, HIGH; RISK-SRV-002/005, S06, A07, AND-AUTH-002 | 05 ↔ 06 ↔ 15 ↔ 16: credential/account consequence согласован с фактическим auth path, без утверждения о произошедшем инциденте |
| REV-SYS-002 | CLOSED: OPEN-SRV-012 → SYS-OPEN-005; OPEN-SRV-017 → SYS-OPEN-008 | 05 ↔ 09 ↔ 16: filename conflict ограничен non-CAMERA path; 05 ↔ 06 ↔ 15 ↔ 16: credential/session boundary остаётся вопросом, без решения |
| REV-SYS-003 | CLOSED: Sources всех 19 risks проверены по поддерживающим component sources; SYS-CAP-012 сохраняет SRV-API-004 | 13 ↔ 15 ↔ frozen registries: RISK-SRV-010 добавлен в risk004; RISK-SRV-012 — в risk005/008 с указанием поддерживаемой части consequence; RISK-SRV-036 — в risk008; общие S16 и Android boilerplate сокращены до релевантных sources |
| REV-SYS-004 | CLOSED: SYS-FLOW-001 начинается с отправки email/password; failure semantics сохранена | 05 ↔ 06 ↔ 18: unknown email, неверный пароль, disabled/banned дают 401 / login failure; остальные 11 flows без изменений |

Registry check README ↔ 13 ↔ 14 ↔ 15 ↔ 16 ↔ 17 ↔ 18 ↔ 19: 26 CAP, 12 FLOW, 4 MISMATCH, 19 RISK, 21 OPEN; IDs непрерывны в объявленных диапазонах. Component source IDs валидны. Все 34 input fingerprints повторно проверены по SHA-256 и сохранены без изменения; System hashes в таблицу не добавлялись. Четыре SYS-MISMATCH и SYS-OPEN-019 не изменены; OPEN не решены, To-Be/change items не добавлены. Component specs и historical review сохранены без изменения.

Итог: **PhotoCloud System As-Is v1 — FROZEN**. Значение freeze и следующий этап зафиксированы в [19-freeze-record.md](19-freeze-record.md).
