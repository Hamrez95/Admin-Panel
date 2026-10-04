# Commerce Admin: product scope and prioritized backlog

This backlog is maintained on `dev`; `main` receives a completed foundation slice or a complete product module. Work in ID order within each priority, respecting dependencies. Keep each ticket small enough to review and ship independently. Update status and acceptance notes in this file as work lands.

## Product goal

Build one reliable operations workspace where a store owner and their team can connect one or more online stores, see the work that needs attention, and manage the day-to-day flow from catalog and order intake through fulfillment, customers, inventory, marketing, and finance.

The interaction direction is a fast, calm, app-like workspace: persistent navigation, clear store context, dense but readable operational tables, helpful empty states, a useful home dashboard, and a calendar for planned work. Use Buffer as inspiration for navigation, calendar clarity, and collaboration patterns; do not copy its branding or screens. Keep the commerce domain visible in the visual language and prioritize everyday tasks over decorative charts.

## Product boundaries

- A “store” is a connected commerce system. A workspace can contain multiple stores, subject to permissions and connector capabilities.
- “Connect any store” means a clear path for supported platforms plus a documented adapter/API path for custom stores. A custom website without an API, webhook, or installable connector cannot be connected automatically.
- The UI depends on a stable commerce contract and advertised connector capabilities. It must not contain platform-specific API assumptions.
- Treat each product area as an independently entitled module with a stable module ID, published boundary, declared dependencies, and its own release acceptance. Keep platform/profile/settings available as the host shell. A user profile can hide or show modules the workspace is entitled to; only a workspace owner can manage module entitlements, and the server must enforce access regardless of navigation visibility.
- Disabling a module hides its entry points and denies its protected operations without deleting its data. Dependent modules cannot be enabled unless required modules are entitled; automations and sync jobs owned by a disabled module must be paused safely.
- Keep Mazeduneh as the first real reference connector and regression target. Keep a deterministic demo store with synthetic fixtures so design and workflows work without credentials or external services. Never put secrets or real customer data in demo fixtures.
- Start with a modular monolith and explicit module boundaries. Add independent services only when measured operational needs justify them.
- Finance and tax screens must clearly distinguish operational summaries from jurisdiction-specific accounting or tax advice. Support exports and integrations before claiming full statutory compliance.

## First usable release

The first milestone should let a small store team connect a demo store and one real connector, customize a dashboard, and handle the core loop: review orders, update products, see stock, fulfill orders, and look up customers. Marketing, advanced logistics, accounting/tax workflows, and automation follow after the core data and permissions are trustworthy.

## Prioritized tasks

### P0 — Product and technical foundation

#### AP-001 — Record the target architecture and migration path
**Priority:** P0 · **Status:** Done on `dev` · **Depends on:** —

Inspect the existing Flutter app and document the smallest architecture that safely supports a reusable client and secure store integrations. Decide and record the client/server boundary, domain modules, API contract strategy, connector execution location, data ownership, module entitlements/profile visibility, and how the current Mazeduneh API fits during migration. Prefer a modular monolith; do not introduce interfaces, layers, queues, or services without a concrete second implementation or operational need.

**Acceptance:** An ADR includes a current-state diagram, target module boundaries, dependency direction, module entitlement rules, migration slices, and explicit trade-offs. It records the current source-integrity issue as a gate before changing entrypoints and preserves Mazeduneh as an optional reference connector.

**Engineering:** Apply Clean Code and Clean Architecture: meaningful names, inward dependencies, explicit boundaries, and no speculative abstractions. Use the clean-architecture skill when deciding module and connector boundaries.

#### AP-002 — Generalize the app identity and configuration
**Priority:** P0 · **Status:** Done on `dev` · **Depends on:** AP-001

Remove Mazeduneh-only identity from the product name, package metadata, app title, and default configuration. Make API/environment configuration explicit and safe; keep store identity and connector configuration out of global constants. Preserve the old environment variable as a compatibility fallback for the Mazeduneh reference connector.

**Acceptance:** Flutter package and app titles use generic Commerce Admin identity; the API base URL can be configured at build time using `COMMERCE_ADMIN_API_BASE_URL`; the previous `MAZEDUNEH_API_BASE_URL` remains a fallback; no default live endpoint or credential is embedded in the app.

**Engineering:** Apply Clean Code and Clean Architecture, preserve behavior with focused characterization coverage, and avoid broad renames that obscure the diff.

#### AP-003 — Define the interface system and responsive application shell
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-001

Create the product design direction, design tokens, shared navigation, store switcher, page header, responsive sidebar/drawer, loading/error/empty states, and accessible interaction rules. Use a Buffer-inspired sense of space and workflow clarity while making Commerce Admin visually distinct. Support Persian RTL and English LTR from the start.

**Acceptance:** A documented shell covers desktop, tablet, and mobile layouts; navigation labels map to the roadmap domains; keyboard focus, text scaling, RTL, and reduced motion are specified; shared components have usage examples.

**Engineering:** Apply Clean Code and Clean Architecture; use Frontend Design and Accessibility Audit skills for UI decisions. Keep presentation reusable and keep business rules out of widgets.

#### AP-004 — Add workspace, store, role, module entitlement, and audit foundations
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-001, AP-003

Define workspace and store membership, invitations, role-based permissions, session lifecycle, and an append-only audit trail for sensitive actions. Add a module catalog with stable IDs, declared dependencies, workspace entitlements, and per-user profile visibility preferences. Keep authentication and authorization decisions server-side; the client only presents permitted actions.

**Acceptance:** A workspace can contain stores and members; roles are least-privilege and checked for read/write actions; workspace owners can enable entitled modules; users can hide/show entitled modules in their profile; profile visibility never grants a license or bypasses server authorization; disabling a module preserves its data and safely pauses its jobs; dependency conflicts are explained; sensitive changes record actor, workspace, module, action, timestamp, and outcome; removing a member revokes access.

**Engineering:** Apply Clean Code and Clean Architecture; define authorization at use-case boundaries, avoid trusting UI state, and add focused tests for denied access and audit behavior.

#### AP-005 — Define a versioned commerce contract and capability model
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-001

Define normalized contracts for products, variants, inventory, orders, payments/refunds, customers, fulfillments, returns, and store metadata. Define connector capabilities and limits so UI actions are enabled only when a connected store supports them. Preserve source IDs and safe raw references for support without leaking secrets.

**Acceptance:** Contract examples cover the demo store and Mazeduneh; unsupported and read-only capabilities are representable; money, currency, timestamps, pagination, and status mappings have explicit semantics; API changes are versioned.

**Engineering:** Apply Clean Code and Clean Architecture; keep domain types independent of Flutter, HTTP, and vendor SDKs; use contract tests and avoid mapping layers that add no decision.

#### AP-006 — Build secure connector setup and connection health
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-004, AP-005

Create a guided flow to select a platform, enter its URL, authenticate through the supported method, request least-privilege access, validate connection, select sync scope, and show connection health. Store credentials only in the trusted server-side secret store; provide rotation, revocation, and disconnect flows.

**Acceptance:** The user sees required permissions before connecting, actionable setup errors, last successful sync, and a safe disconnect/revoke path. Secrets are never returned to the client, logs, analytics, or task fixtures.

**Engineering:** Apply Clean Code and Clean Architecture; use secure defaults, specific error handling, and tests for credential redaction, permission failure, expiry, and revocation.

#### AP-007 — Create a deterministic demo store and isolate Mazeduneh
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-005

Provide a local/demo connector with synthetic products, orders, stock, customers, returns, and edge cases. Move Mazeduneh API behavior behind its own connector boundary and keep its adapter optional. Do not delete the current Mazeduneh integration; make it usable for real integration checks when credentials are supplied.

**Acceptance:** Demo mode runs without network access or secrets and can reset to a known fixture state; Mazeduneh data never appears in demo mode; fixtures include duplicate events, missing optional fields, failed fulfillment, and refund cases.

**Engineering:** Apply Clean Code and Clean Architecture; keep fixtures deterministic and domain-shaped; test connectors through the same public contract used by the app.

#### AP-008 — Add safe synchronization, webhooks, retries, and conflict visibility
**Priority:** P0 · **Status:** Backlog · **Depends on:** AP-005, AP-006, AP-007

Implement the chosen server-side sync path with webhook verification where available, polling fallback, idempotency, retry/backoff, replay protection, pagination, rate-limit handling, and per-store sync history. Make conflict resolution and stale data visible instead of silently overwriting store data.

**Acceptance:** Replayed events do not duplicate orders or stock changes; failed syncs can be retried safely; webhook signatures are verified; the UI shows sync time, error, affected records, and a recovery action.

**Engineering:** Apply Clean Code and Clean Architecture; isolate I/O at adapter boundaries, use explicit retry policies and error types, and test duplicate, out-of-order, partial, and failed events.

### P1 — Core daily operations

#### AP-009 — Build a customizable operations dashboard
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-003, AP-004, AP-005, AP-007

Create a widget registry for operational metrics, order queues, low-stock alerts, sync health, and recent activity. Let each user add/remove, reorder, resize where supported, filter by store/date, and restore defaults. Save layouts per user and workspace, with role-aware data.

**Acceptance:** Layout changes persist across sessions and devices; widgets have loading, empty, error, and permission states; charts and numbers disclose range, currency, and source; dashboard remains useful on mobile and with one widget.

**Engineering:** Apply Clean Code and Clean Architecture; use Frontend Design and Accessibility Audit skills; keep widgets composable, typed, and independent from connector-specific response shapes.

#### AP-010 — Deliver a unified order workbench
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-005, AP-008, AP-009

Provide searchable/filterable orders across connected stores, a normalized order detail, line items, payment and fulfillment status, timeline, internal notes, tags, and permitted state transitions. Support safe bulk actions with a preview and confirmation.

**Acceptance:** Operators can find an order by ID/customer/status/date/store; unsupported actions are explained; every state change is permission-checked and audited; duplicate requests do not apply an action twice.

**Engineering:** Apply Clean Code and Clean Architecture; model lifecycle transitions explicitly, keep UI independent of source status strings, and cover boundary/error paths.

#### AP-011 — Build catalog and product publishing
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-005, AP-008

Manage products and variants across stores with search, filters, bulk edits, media, descriptions, SKU/barcode, price, publication state, and a clear source-of-truth indicator. Show field-level differences when platforms cannot represent the same product data.

**Acceptance:** Users can view and edit fields supported by the connector; bulk changes preview affected records; validation errors identify the field and store; unsupported fields never silently disappear.

**Engineering:** Apply Clean Code and Clean Architecture; use domain commands for product changes, avoid giant page widgets, and cover normalization and connector capability cases.

#### AP-012 — Add inventory, locations, and replenishment
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-005, AP-008, AP-011

Show available, reserved, and on-hand quantities by store/location; provide stock adjustment history, low-stock thresholds, CSV import/export, and replenishment suggestions. Add purchase orders and supplier management only after quantity semantics and location support are stable.

**Acceptance:** Quantity changes are audited with reason and actor; reserved stock is not presented as available; connector sync conflicts are visible; bulk imports validate before applying.

**Engineering:** Apply Clean Code and Clean Architecture; keep inventory invariants in the domain layer and test concurrency, negative/zero quantities, and partial sync.

#### AP-013 — Fulfillment, shipping, returns, and refunds
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-010, AP-012

Create a fulfillment queue, packing status, shipment/tracking details, carrier/service selection through connectors, partial shipments, cancellation, return requests, exchanges, and refund visibility. Start with connector-supported workflows; do not imply a carrier action succeeded before confirmation.

**Acceptance:** Operators can follow an order from paid to delivered/returned; partial fulfillment and partial refund are represented; labels/tracking are linked to the correct order; errors offer safe retry or escalation.

**Engineering:** Apply Clean Code and Clean Architecture; model order, fulfillment, and refund lifecycles separately and keep external carrier adapters replaceable.

#### AP-014 — Add customer CRM and service history
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-005, AP-008, AP-010

Unify customer profiles across stores with consent-aware contact details, order history, lifetime value definition, notes, tags, segments, and support context. Detect likely duplicates for review; do not merge profiles automatically.

**Acceptance:** Profiles show source stores and last sync; access respects roles; consent and deletion/export requests are visible; duplicate suggestions include reasons and reversible review.

**Engineering:** Apply Clean Code and Clean Architecture; minimize personal data, separate identity matching from UI, and test access and deletion boundaries.

#### AP-015 — Marketing calendar, promotions, and campaign measurement
**Priority:** P1 · **Status:** Backlog · **Depends on:** AP-003, AP-009, AP-011, AP-014

Provide a calendar/list for campaigns, discounts, coupons, and planned content; filter by store, channel, campaign, and status. Start with commerce-native promotions and UTM attribution; add social/email publishing only through explicit provider integrations and permissions.

**Acceptance:** Campaign schedule gaps and status are visible; coupon rules are validated by the connected store; sales can be attributed when source data exists; integrations show their publication limits and failures.

**Engineering:** Apply Clean Code and Clean Architecture; use Frontend Design and Accessibility Audit skills; keep scheduling separate from execution adapters and do not claim attribution when data is missing.

### P2 — Business management depth

#### AP-016 — Finance workspace and accounting exports
**Priority:** P2 · **Status:** Backlog · **Depends on:** AP-010, AP-013

Show gross/net sales, discounts, refunds, fees, payout status, cash flow indicators, invoices, and expenses only where source data supports them. Provide configurable mappings and exports to established accounting systems before building a full general ledger.

**Acceptance:** Every total has a documented formula, date basis, currency, and source; refunds and fees reconcile to orders/payouts; exports are repeatable and idempotent; estimated values are labeled.

**Engineering:** Apply Clean Code and Clean Architecture; use decimal-safe money types, explicit currency conversion policy, and tests for rounding and reconciliation.

#### AP-017 — Tax configuration and jurisdiction-aware documents
**Priority:** P2 · **Status:** Backlog · **Depends on:** AP-016, AP-006

Represent tax collected by the store, exemptions, tax-inclusive/exclusive prices, invoice details, and exportable tax reports. Use provider or jurisdiction-specific rules through integrations; keep tax calculation/version/source explicit and avoid hardcoded universal rates.

**Acceptance:** Tax totals can be traced to source transactions and rule version; unsupported jurisdictions are clearly marked; invoice templates and exports identify required store settings; the product does not label generic estimates as compliant filings.

**Engineering:** Apply Clean Code and Clean Architecture; isolate jurisdiction rules behind explicit policies and test boundaries, effective dates, and rounding.

#### AP-018 — Reports, saved views, alerts, and guided insights
**Priority:** P2 · **Status:** Backlog · **Depends on:** AP-009, AP-010, AP-012, AP-014, AP-016

Add cross-domain reports, saved filters/views, CSV export, scheduled summaries, and rule-based alerts for stale sync, overdue orders, low stock, refund spikes, and margin changes where costs are known. Keep recommendations explainable and distinguish observation from action.

**Acceptance:** Users can reproduce a report with its filters and time zone; exports respect permissions; each alert links to underlying records and states its rule; insight data shows freshness and limitations.

**Engineering:** Apply Clean Code and Clean Architecture; keep metric formulas named and centralized, avoid hidden dashboard calculations, and test date/currency/filter boundaries.

#### AP-019 — Connector catalog and supported-platform adapters
**Priority:** P2 · **Status:** Backlog · **Depends on:** AP-005, AP-006, AP-008

Ship a documented connector catalog and implementation path. Prioritize WooCommerce and Mazeduneh reference coverage, then select the next platform based on customer demand. Define a generic REST/webhook adapter only for stores that expose compatible APIs; maintain a capability matrix per connector.

**Acceptance:** Each adapter documents auth, supported entities, read/write directions, rate limits, webhooks/polling, and known gaps; contract conformance runs against demo fixtures; customers can request a custom connector with a clear scope.

**Engineering:** Apply Clean Code and Clean Architecture; use ports/adapters at the real external-system boundary and avoid a “universal” abstraction that hides vendor differences.

#### AP-020 — Purchasing, suppliers, and multi-warehouse operations
**Priority:** P2 · **Status:** Backlog · **Depends on:** AP-012

Add supplier records, purchase orders, receipts, stock transfers, and multi-location replenishment. Keep purchasing documents linked to inventory movements and source systems.

**Acceptance:** A purchase order can be drafted, approved, received partially or fully, and reconciled to stock; transfers preserve location-level movement history; permissions cover approval and adjustment.

**Engineering:** Apply Clean Code and Clean Architecture; define state machines and audit events before UI, and keep supplier-specific behavior in adapters.

### P3 — Scale, extensibility, and refinement

#### AP-021 — Add extensibility and documented custom connector onboarding
**Priority:** P3 · **Status:** Backlog · **Depends on:** AP-019

Document the connector contract, capability declarations, authentication options, mapping rules, conformance fixtures, and safe rollout process for custom stores. Introduce a public SDK only if external implementation needs are validated.

**Acceptance:** A developer can implement and validate a connector using public examples without importing application internals; versions and deprecation rules are documented; secrets remain server-side.

**Engineering:** Apply Clean Code and Clean Architecture; document only stable public boundaries and do not build a plugin runtime without a concrete use case.

#### AP-022 — Production readiness, privacy, accessibility, and localization
**Priority:** P3 · **Status:** Backlog · **Depends on:** AP-004 through AP-021 as applicable

Set operational SLOs from observed usage; add monitoring, backups/restore checks, migration strategy, incident diagnostics, data export/deletion, localization review, keyboard/screen-reader support, and performance budgets. Treat these as release gates throughout development, not a final cleanup sprint.

**Acceptance:** Restore and data-export procedures are documented and exercised; sensitive data is redacted from logs; critical flows work with keyboard and assistive technology; RTL/LTR layouts pass review; slow/failed connectors are observable.

**Engineering:** Apply Clean Code and Clean Architecture; use Accessibility Audit skills for the UI and security/privacy review for data flows; keep reliability checks measurable and actionable.

## Research references

- [Shopify admin and Shopify Home](https://help.shopify.com/en/manual/shopify-admin) — central operational hub; customizable metrics and actionable cards.
- [Buffer team features](https://buffer.com/resources/every-team-feature-in-buffer-built-for-agencies/) — calendar organization by brand/client/channel/campaign, collaboration, and report exports.
- [Buffer calendar design](https://buffer.com/resources/new-social-media-calendar/) — global schedule visibility and weekly/monthly planning.
- [Odoo applications](https://www.odoo.com/documentation/19.0/applications.html) and [Shopify connector](https://www.odoo.com/documentation/19.0/applications/sales/sales/shopify_connector.html) — cross-domain coverage and the practical trade-offs of synchronization.
- [WooCommerce REST API](https://woocommerce.com/document/woocommerce-rest-api/) and [webhooks](https://developer.woocommerce.com/docs/apis/rest-api/v3/webhooks/) — credentials, event delivery, logs, and failure handling that inform connector tasks.

These references are feature benchmarks, not a commitment to reproduce another product or to support every platform in the first release.
