# ADR 0001: Modular product and integration boundaries

- **Status:** Accepted on `dev`
- **Date:** 2026-10-04
- **Related task:** AP-001

## Context

The repository currently contains a Flutter client copied from `apps/admin` in Mazeduneh. It contains direct Mazeduneh authentication, product, and order API clients. There is no backend implementation in this repository. The intended product must support multiple commerce systems and separately sell business modules, while keeping a demo store usable without credentials.

The selected Mazeduneh `dev` commit (`0c17bf4`) includes commit `96d16a3`, which changed `lib/main.dart`, `lib/secure_main.dart`, `lib/professional_demo_main.dart`, and `lib/demo_main.dart` into files with many non-text control bytes that cannot be read as normal Dart source. Their corresponding files in the source clone and Admin-Panel seed have identical SHA-256 hashes; Git reports binary diffs for some of these paths. The last readable predecessor is Mazeduneh commit `d9cba6a`. The Admin-Panel `dev` branch restores only these four entrypoints from that predecessor and gives them generic product names; Mazeduneh itself remains untouched. This recovers a source-readable baseline, but the app still needs build validation before claiming it runs.

## Decision

1. **Use a modular monolith as the starting product shape.** Organize business capabilities by feature module with a small published API. Do not split deployables or create a plugin runtime without demonstrated operational or customer need.
2. **Keep the Flutter client at the UI edge.** UI reads normalized application contracts and advertises connector capabilities; it does not embed platform-specific API assumptions or make authorization decisions.
3. **Put credentials, webhook verification, entitlements, synchronization, retries, and protected writes behind a trusted server API.** This repository has no such backend today. A later implementation task must select and document its deployment/technology before adding it; do not store store-owner secrets in Flutter preferences or source.
4. **Keep external systems behind connector adapters.** The normalized commerce contract is inside the product boundary. Mazeduneh is an optional first reference adapter; a synthetic deterministic demo adapter is always available. A custom store requires a compatible API, webhook, or installable connector.
5. **Make modules commercial and operational boundaries.** Give each module a stable ID, manifest (label, routes, capabilities, dependencies), public API, ownership of its data and jobs, and independent acceptance/release notes. Initial candidates are Catalog, Orders, Inventory, Fulfillment, CRM, Marketing, Finance, Tax, Purchasing, and Insights. Keep Platform/Profile/Workspace/Connections as host capabilities.
6. **Separate entitlement from profile visibility.** A workspace entitlement records which modules are licensed/enabled for that workspace. A user's profile preference controls which entitled modules appear in that user's navigation. Neither is sufficient for authorization: the server checks workspace entitlement, user role, and resource scope on every protected operation. Turning a module off hides/blocks its operations and pauses its jobs without deleting its data.
7. **Declare dependencies as data and explain conflicts.** For example, Tax can require Finance and Marketing attribution can require Orders. A module cannot be activated when a required module is unavailable. Do not silently switch off dependent modules.
8. **Merge complete slices from `dev` to `main`.** Documentation/architecture is a foundation slice; subsequent modules ship only when their data boundary, permissions, UI entry, integration behavior, and acceptance criteria are complete. Keep the backlog and active task status on `dev`.

## Initial boundary sketch

```text
Flutter app
  Platform shell: profile, workspace/store context, module navigation
  Feature modules: catalog | orders | inventory | fulfillment | CRM | marketing | finance | tax | purchasing | insights
  Application contracts: commerce entities, capability declarations, module manifests
                         |
                   authenticated API
                         |
Trusted server boundary (not present yet)
  workspace roles + module entitlements | secrets | audit | sync/webhooks/jobs
                         |
                 Connector adapters
      Demo | Mazeduneh | WooCommerce | future supported/custom stores
```

Dependencies point toward contracts/domain concepts. Feature internals do not import one another; cross-module actions use published application APIs. The diagram is a target boundary, not a claim that these modules or server components are implemented.

## Consequences

- A modular directory layout and stable IDs can be introduced in small steps without a big-bang rewrite.
- Feature visibility can be personalized while product access remains controlled by workspace entitlements and server authorization.
- Some workflows need multiple entitlements. Their dependency rules must be visible before activation and included in packaging/pricing decisions.
- Supporting arbitrary websites is bounded by the integration surface the store exposes; connector gaps must be visible to users.
- The current binary Dart entrypoints must be resolved before safe app-shell refactoring and build validation.
- This ADR intentionally leaves backend framework/hosting undecided because none exists in the repository and no hosting constraint has been selected.

## Follow-up

- AP-002: generalize application identity and configuration after the source-integrity gate is resolved.
- AP-004: implement workspace roles, module catalog, profile visibility, and server-enforced entitlements when the API boundary exists.
- AP-005–AP-008: introduce commerce contracts, secure connectors, demo fixtures, and reliable synchronization.
