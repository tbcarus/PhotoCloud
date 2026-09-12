# PhotoCloud System As-Is v1

Status: `READY_FOR_REVIEW`

Дата составления: 2026-09-12. Версия: v1. Independent System As-Is Review ещё не выполнялся; freeze record не создан.

## Inputs

- **PhotoCloud Server As-Is v1 — FROZEN**: [входной README](../../../../PhotoCloudServer/docs/spec/server-as-is/README.md), [S20](../../../../PhotoCloudServer/docs/spec/server-as-is/20-freeze-record.md).
- **PhotoCloud Android As-Is v1 — FROZEN**: [входной README](../../../../PhotoCloudClient/docs/spec/android-as-is/README.md), [A22](../../../../PhotoCloudClient/docs/spec/android-as-is/22-freeze-record.md).

Фактические имена component repositories в workspace — `PhotoCloudServer` и `PhotoCloudClient`, соответствующие `server` и `android` из задания. Выходной пакет создан по заданному пути `system-spec/docs/spec/system-as-is/`. Отдельного repository `system-spec` на старте не было; Git metadata не создавалась, поскольку разрешены изменения только внутри выходного пакета. Существующий каталог `docs/spec/system-as-is/` вне выходного пути не использован как источник и не изменён.

## Purpose

Единое описание фактической системной логики Android + Server: boundaries, contracts, lifecycle, identity, ownership, state, recovery и ограничения. Это system integration specification. Отдельного System runtime layer в приложениях нет.

## Authority и scope

- Server internals → Server As-Is v1.
- Android internals → Android As-Is v1.
- Cross-component behaviour → этот System As-Is как traceable synthesis двух frozen inputs, пока со статусом READY_FOR_REVIEW.
- Scope: **system-level behaviour only**. Deployment и OS unknown не подменяются предположениями; четыре установленных semantic mismatch не означают противоречия источников.

Использованы только документы двух frozen directories. Production code, tests, конфигурационные файлы приложений, live DB/storage, historical audits/reviews/handoff не читались. Описание конфигурации и тестирования взято из frozen спецификаций. Component specs не изменялись; коммитов нет.

## Composition

- [00-system-as-is-summary.md](00-system-as-is-summary.md)
- [01-system-boundary.md](01-system-boundary.md)
- [02-component-responsibilities.md](02-component-responsibilities.md)
- [03-system-domain-model.md](03-system-domain-model.md)
- [04-identity-ownership-and-scope.md](04-identity-ownership-and-scope.md)
- [05-effective-api-contract.md](05-effective-api-contract.md)
- [06-auth-session-lifecycle.md](06-auth-session-lifecycle.md)
- [07-media-sync-lifecycle.md](07-media-sync-lifecycle.md)
- [08-system-state-model.md](08-system-state-model.md)
- [09-error-retry-recovery.md](09-error-retry-recovery.md)
- [10-startup-restart-reboot.md](10-startup-restart-reboot.md)
- [11-delete-change-reconciliation.md](11-delete-change-reconciliation.md)
- [12-data-consistency-and-durability.md](12-data-consistency-and-durability.md)
- [13-system-capability-matrix.md](13-system-capability-matrix.md)
- [14-system-mismatches.md](14-system-mismatches.md)
- [15-system-risks.md](15-system-risks.md)
- [16-system-open-questions.md](16-system-open-questions.md)
- [17-source-traceability.md](17-source-traceability.md)
- [18-system-flow-index.md](18-system-flow-index.md)

## Чтение и registries

Начало: сводка 00 → границы/identity 01–04 → API/auth/media 05–07 → state/recovery/consistency 08–12. Реестры 13–16, traceability 17 и flow index 18 позволяют проверить выводы.

IDs `SYS-CAP-*`, `SYS-FLOW-*`, `SYS-MISMATCH-*`, `SYS-RISK-*`, `SYS-OPEN-*` принадлежат отдельным реестрам. Исходные `INT-AND-*`, `SRV-*`, `AND-*`, risk/open IDs сохранены в ссылках. После review/freeze системные IDs остаются стабильными; удаление пункта не означает повторное использование его ID.

IMPLEMENTED означает механизм в указанном объёме, PARTIAL — существенные ограничения, STUB — доступную заглушку, NOT PRESENT — отсутствие системной функции в текущем контуре. Это не оценка runtime test pass. Mismatch описывает несовместимость семантик, risk — возможное последствие, OPEN — вопрос без назначенного ответа.

## Проверка пакета

Полнота и self-check отражены в [17](17-source-traceability.md). Следующий этап — независимое review этого пакета. Автоматическая заморозка и To-Be items не выполнялись.
