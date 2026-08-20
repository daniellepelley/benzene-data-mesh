# Benzene Data Products — Design

**Status:** Draft v0.1 — decisions marked *proposed* are open to challenge; everything here is grounded in the two research documents and cites them rather than re-arguing them.
**Date:** 2026-08-20.
**Inputs:** [`docs/research/01-data-mesh-principles.md`](../research/01-data-mesh-principles.md) (Part 1 — principles, requirements) and [`docs/research/02-data-mesh-market-landscape.md`](../research/02-data-mesh-market-landscape.md) (Part 2 — market, build/buy, gaps).

---

## 1. What Benzene Data Products is

**A provisioning-backed data product platform: the operating-model layer over an organisation's existing data estate.** It defines, provisions, contracts, governs, prices and serves *data products* across whatever storage and compute the organisation already runs — Snowflake, Databricks, BigQuery, Fabric, Postgres, Kafka — without owning any of it.

It is deliberately **not** marketed as a data mesh, and the repository rename from `benzene-data-mesh` to `benzene-data-products` reflects that. The market evidence is unambiguous: the term has been declared "obsolete before plateau" by Gartner, abandoned by its own vendors (Data Mesh Manager → Entropy Data), and retired even by Thoughtworks in favour of "data products and self-serve platforms as commodities" — while the underlying practices went mainstream (Part 2 §Exec-1). Benzene implements the substance: domain ownership, data as a product, self-serve platform, federated computational governance. A full mesh operating model is a configuration it supports, not a prerequisite it imposes — "data products without full mesh" is the modal successful adoption and is a first-class mode here (Part 1 §12, requirement MUST-context).

### The one-sentence bet

> **If the only supported path to a production data product runs through Benzene's declarative spec and reconciler, then Benzene's metadata is correct by construction — which is the one claim no catalogue, and almost no platform, can make.**

Part 1 arrived at this from theory (the manifest must be executable, or we've built a catalogue — §2.3) and Part 2 from the market's failures (catalogue shelfware is the near-universal failure mode — §9.3, §11.1). Both halves of the research converged on it independently; it is the design's centre of gravity.

### What it must fix

The three 2019 failure modes are the acceptance criteria (Part 1 §1). Every design decision below is checked against them:

1. **No global serialisation point** in the critical path of publishing a data product — no human platform gate, no central pipeline team, no shared throttled capacity.
2. **Decomposition by domain**, with independently deployable products and no cross-product coordinated release.
3. **Accountability for meaning co-located with the producer**, expressed as an enforced contract in the producer's own repository.

---

## 2. Goals and non-goals

### Goals

- **G1.** A domain team ships a compliant, contracted, discoverable, governed data product from a template in **under one working day**, knowing nothing platform-specific (TTFDP is a CI-benchmarked platform SLO).
- **G2.** Publishing a data product costs a domain team **one YAML file and a CI run**; catalogue entry, lineage, access workflow, policy enforcement, quality gates, agent port and cost report happen automatically (Part 2 §11.9 — the obligation floor).
- **G3.** Contracts are **fail-closed**: a violated contract blocks the new data version from becoming visible to consumers, at publish time, by default.
- **G4.** One policy model, authored once, enforced across heterogeneous engines — **cross-vendor neutrality** is the structural differentiation (Part 2 §11.2).
- **G5.** Every data product carries **computed trust** and **attributed cost** as first-class, visible properties.
- **G6.** Radically low operational footprint: the control plane is small stateless services + Postgres. No Kafka, no Elasticsearch, no Airflow of our own (Part 2 §11.8).

### Non-goals (the platform must never…)

Lifted from Part 1 §6.4 as binding review criteria — every platform feature proposal answers "does this move judgement about domain data into the platform team?":

- Write or review domain transformation logic; model domain data; own domain schemas.
- Be a mandatory human approval gate for creating or changing a data product.
- Hold the only credentials to publish.
- Define what "good data" means per domain (we define the vocabulary and mechanism; domains set thresholds).
- Force a single storage/compute engine where a conforming port is satisfiable otherwise (strong *defaults*, never *rules*).
- Build: a catalogue UI, a lineage format, a query engine, a table format, a contract schema, an orchestrator, a developer portal, a quality-check DSL, an ML anomaly detector (Part 2 §10).

---

## 3. Key decisions

Each decision records the fork, the position, and the primary rationale. Alternatives are in the research docs; they are not re-argued here.

| # | Decision | Position (proposed) |
|---|---|---|
| D1 | Artefact vs catalogue entry | **Deployable composite artefact.** The manifest is executable; applying it provisions, registers, attaches policy, grants access. Brownfield *wrapping* of existing assets is the supported entry path, not a second architecture. |
| D2 | Control plane vs embedded runtime | **External control plane** (thin, stateless, Postgres-backed) with a *logical* control port per product. No in-product kernel at v1; sidecar model stays on the roadmap (Part 1 COULD-41). |
| D3 | Do we run a data plane? | **No.** Never proxy bytes. Credential vending + zero-copy references; compute stays in the org's engines. Fail-closed read-side guarantees come from publish-time gating (write–audit–publish), not an inline proxy. |
| D4 | Policy enforcement locus | **Hybrid compile-down.** Policy intent (ODRL-flavoured) → Rego → compiled to each engine's native constructs (LF-Tags, UC governed tags/ABAC, Snowflake tag policies). Continuous **drift detection** between intended and effective policy is a required capability, not an afterthought. |
| D5 | Contract format | **ODCS v3.1, unmodified**, extended only via `customProperties` under an `x-benzene` namespace. Data Contract CLI as the lint/test engine. |
| D6 | Descriptor / internal model | **Own canonical model, structured on DPDS's five-port taxonomy**, with mechanical exporters: Bitol ODPS (product metadata), DCAT/DPROD (catalogue interop), OpenAPI/AsyncAPI (API/event ports). No bet on any descriptor winning (Part 2 §4.4). |
| D7 | Contract semantics | **Producer-published spec + bilateral agreement layer.** ODCS describes the port; an *access agreement* (consumer, purpose, relied-upon SLOs, notice terms) is created per consumer through the request/approve flow (Entropy Data / EDC pattern, Part 2 §13). |
| D8 | Change impact | **Computed, not asserted.** Column-level lineage (SQLGlot-style parsing) diffed against active consumer agreements classifies changes breaking/non-breaking pre-merge; declared `grain`/`semanticVersionOf` fields cover what parsing cannot see (Part 1 §4.3, Part 2 §11.5). |
| D9 | Access model | **ABAC over governed tags**, default-deny, classification declared in the contract, applied at provisioning, with future-grants semantics. Per-object ACLs are not the primary mechanism (Part 1 §7.3). |
| D10 | Lake substrate default | **Iceberg + Iceberg REST catalog** as the default technical substrate for lake-resident products; credential vending as the standard output-port access mechanism; Delta Sharing as a supported port type. A default, not a rule (per non-goals). |
| D11 | Isolation boundary | **Per-domain, adapter-native**: AWS account/project, Databricks catalog, BigQuery project, Fabric domain+dedicated capacity. Hard rule: **no shared throttled resource pool across domains without per-domain quota and attribution** (the Fabric lesson, Part 2 §2.3). |
| D12 | Cost attribution | **First-class product property.** Storage + compute + egress per product, consumption attributed to consuming domains, surfaced in the marketplace next to SLOs. Honest about shared-cost allocation (showback before chargeback). |
| D13 | Trust | **Computed only.** Derived from measured SLIs vs declared SLOs, contract test outcomes and policy evaluations; never manually settable; evidence always exposed. |
| D14 | Polysemes | **Registry + mappings, not forced agreement.** Global polyseme IDs for shared entities; each domain registers its *mapping* to the polyseme (definition deltas explicit). Disagreement is surfaced and versioned, not resolved by fiat; resolution is the federated body's job and we ship the process artefacts. |
| D15 | Interface primacy | **Declarative spec + CLI/API first; UI is a projection.** We ship a Backstage plugin and Port blueprint, not a portal (Part 2 §6.1). |

**Standards conformance statement** (divergence requires an ADR): ODCS v3.1 (contracts) · DPDS port taxonomy (internal model) · Bitol ODPS + DCAT/DPROD (export) · OpenLineage (lineage, emit + reconcile) · OpenTelemetry (SLI metrics) · Iceberg REST (technical catalogue) · Delta Sharing (cross-boundary port) · OpenAPI/AsyncAPI (API/event ports) · JSON Schema/Avro/Protobuf + schema registry compatibility modes (streaming) · ODRL vocabulary → OPA/Rego (policy) · OIDC/workload identity (authn) · MCP (agent port) · OCI + Kubernetes CRDs (packaging/reconciliation).

---

## 4. Architecture

Three planes (Part 1 §6), with Benzene's own effort deliberately weighted to planes 2 and 3; plane 1 is adopted, not built.

```
┌─────────────────────────────────────────────────────────────────────┐
│ MESH EXPERIENCE PLANE                                               │
│ marketplace · discovery/addressing · lineage graph & impact         │
│ scorecard & trust · access workflow · policy registry & waivers     │
│ polyseme registry · cost & adoption analytics · catalogue exporters │
├─────────────────────────────────────────────────────────────────────┤
│ DATA PRODUCT EXPERIENCE PLANE (the paved road)                      │
│ benzene CLI · templates/scaffolding · manifest & contract authoring │
│ local validate loop · CI checks (lint/compat/policy) · reconciler   │
│ + provisioner adapters · port generation · publish gate (WAP)       │
├─────────────────────────────────────────────────────────────────────┤
│ INFRASTRUCTURE / UTILITY PLANE (adopted, not built)                 │
│ Snowflake · Databricks/UC · BigQuery · Fabric/OneLake · AWS         │
│ Glue/LF/S3 · Postgres · Kafka · Iceberg REST/Polaris · OPA ·        │
│ Terraform/Crossplane · dbt/SQLMesh · Soda/GX · OIDC/cloud IAM       │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.1 Core primitives

**The manifest** (`dataproduct.yaml`) — the single source of truth, in the domain's repository. Contents per Part 1 §3.3: identity & ownership, purpose & archetype, the five port groups, embedded/referenced ODCS contracts per output port, policy declarations & classifications, operations (infra requirements, schedule, cost centre, on-call), provenance. Hard schema-design rule: **every field is enforced, surfaced to consumers, or used to provision — fields that are none of those three are rejected at design time** (they rot). The manifest references upstream *contracts with version constraints*, never upstream tables — this is what makes the estate a dependency graph with compatibility semantics.

**The contract** (ODCS v3.1 per output port) — schema, per-field semantics and classification, quality rules in a bounded DAMA-based vocabulary, SLOs (freshness, completeness, availability, schema stability, support), terms, versioning policy. No output port without a contract (MUST-5).

**The registry** — versioned, machine-readable system of record for products, contracts, ports, agreements, policies, owners, domains, polysemes. Authoritative because it is the thing that provisions; projections flow *outward* to whatever catalogue the org runs (DataHub, OpenMetadata, Purview, Atlan, Horizon…). Availability: the registry is a hard dependency for *deploys*, never for *serving* — data continues to flow when the registry is down; deploys queue; the reconciler holds last-known-good desired state (degraded mode, Part 2 open-Q9).

**The address.** Global scheme `dp://<domain>/<product>/<version>/<port>`, resolving via the mesh plane to physical endpoints. Physical location never leaks into the address; addresses are immutable public API (MUST-4).

### 4.2 The reconciler and adapters

The data product is a **composite resource**: one declarative spec fanning out into storage, compute bindings, schedules, grants, catalogue entries, quality checks, lineage emitters and ports — reconciled continuously, not provisioned one-shot (drift between declared and actual is guaranteed; the reconciliation loop is what makes the registry authoritative).

Provisioning uses the **adapter pattern** (Witboost's most-stealable idea, on open primitives): a stable interface —

```
Provisioner: validate → plan → provision → unprovision → reverse-provision-access
```

— with one independent adapter per target technology. v1 adapter set (Part 2 §12): Snowflake, Databricks/Unity Catalog, BigQuery, AWS (Glue/Lake Formation/S3), Postgres, Kafka/Confluent, plus dbt Core and SQLMesh as peer transformation integrations. Adapters compose Terraform/Crossplane under the hood rather than reimplementing providers. Fabric/OneLake follows (capacity-isolation caveats documented per D11).

### 4.3 Ports

All five DPDS port types, with the platform generating what domain teams shouldn't hand-write:

- **Input ports** — declared upstream contract references with version constraints. Continuously **reconciled against observed OpenLineage**; an undeclared observed dependency is a defect (the distributed-monolith early warning, MUST-14).
- **Output ports** — multiple per product, generated from the contract: table/Iceberg (credential-vended), Delta Sharing, file, event (AsyncAPI + schema registry), REST (OpenAPI), and **MCP** — every product gets a governed agent interface derived from schema + semantics + policy, for free. Policy sits in the path of every port.
- **Discovery & observability ports** — platform-generated from manifest + telemetry by default, overridable.
- **Control port** — logical at v1: the interface through which the control plane applies policy, rotates credentials, and drives lifecycle transitions.

### 4.4 Contract enforcement — five points, fail-closed where it counts

Per Part 1 §4.4; the publish-time gate is the one most implementations skip and the one that determines trust:

| Point | Runs | On failure |
|---|---|---|
| Pre-merge (CI) | ODCS lint + Benzene profile; **computed compatibility diff** vs previous version and vs *active consumer agreements*; policy-as-code on the manifest | Block merge |
| Pre-deploy | Contract-vs-implementation schema check; provisioning plan validation | Block deploy |
| **Publish** | **Write–audit–publish**: produce to staging, run data-vs-contract quality tests (executed via Data Contract CLI → Soda/GX/dbt tests), then atomically promote the version | **Block promotion** — consumers never see the bad version; explicit break-glass waiver object required to override |
| Runtime | SLI vs SLO, drift/freshness monitors, policy re-evaluation | Alert, degrade trust signal, page owner |
| Consume | ABAC decision (classification × purpose × principal — including per-agent identity), version pinning, masking/filtering in the data path via native engine enforcement | Deny / mask |

Breaking changes (structural, semantic, key/cardinality, grain, time-logic, scope, policy — Part 1 §4.3) require the derived major bump, a declared deprecation window with parallel serving, and notification of all consumers — who are **derivable** from agreements, grants and lineage, never voluntarily registered (MUST-17). Deprecation has teeth: window expiry escalates, then cuts off, per policy, with waivers as first-class expiring objects.

### 4.5 Governance

- **Policy registry**: `id, rationale, scope (global|domain|product), enforcement point(s), mode (blocking|advisory), implementation ref, owner, version, waiver policy`. A policy without an implementation is explicitly `aspirational`. Initial policy set mined from datamesh-governance.com, not written from scratch.
- **Compilation pipeline**: declarative intent (ODRL vocabulary) → Rego → native engine constructs, with continuous **drift detection** between intent and effective state. Domain teams never write Rego.
- **The federated body ships standards and code, never per-product approvals.** Benzene ships the operating-model artefacts alongside the software: governance charter template, policy template, role definitions, RACI, funding-model guidance, fitness self-assessment (Part 1 §11.12 — the tool cannot succeed if the operating model is absent).
- **Small global, large local**: identity, addressing, contract format/profile, serialisations, lineage format, quality vocabulary, classification taxonomy, versioning rules, access protocol. Nothing else is standardised; everything inside a product is local choice, with escape hatches everywhere and conformance only at the boundary.

### 4.6 Observability, trust, cost

- **Lineage**: OpenLineage emitted from every platform-managed execution; column-level where the emitter supports it; Marquez as optional default backend. Known market gap acknowledged: BI-layer lineage is dark everywhere (Part 2 §9.7) — treated as roadmap, not pretended away.
- **Telemetry**: data SLIs (freshness, volume, quality outcomes) as OpenTelemetry metrics into the org's existing observability stack.
- **Trust signal**: computed per product from SLI attainment, contract test history and policy evaluation results; recomputed continuously; the composing formula is versioned and public (anti-gaming, Part 1 open-Q8).
- **Mesh scorecard**: per product and per domain — discoverability, contract conformance, SLO attainment, lineage completeness, policy coverage, adoption, cost. The governance conversation runs on measurement, not opinion.
- **Cost**: per-product attribution assembled per adapter (Snowflake tags/credits, DBU tags + cloud bill, BigQuery project, AWS account), consumption attributed to consuming domains via agreements + query attribution. Products with zero consumers over a window surface as retirement candidates. Where allocation of shared cost is estimated, it is labelled as estimated.

### 4.7 The platform's own SLOs

Published, regression-tested, treated as defects when missed (Part 1 §2.3, §5):

- **TTFDP < 1 working day** (synthetic CI benchmark, not a survey).
- **Paved-road adoption > 50%** — below that, the paved road is fixed, not mandated.
- **Policy-as-code coverage**: % of global policies with executable implementations; % of products evaluated per policy in the last N days.
- **Lineage completeness**; **registry availability** (with the serving-independence guarantee above).

---

## 5. Core flows

**Producer (paved road):**
`benzene init` (template, canvas-derived prompts) → edit `dataproduct.yaml` + contract → `benzene validate` (local: lint, compat, policy, quality dry-run) → PR → CI runs the same checks + computed change impact → merge → reconciler provisions (storage, compute binding, grants, catalogue entry, lineage registration, ports incl. MCP) → first publish passes the WAP gate → product is live, discoverable, governed. No human platform gate anywhere in the path.

**Brownfield wrapping (the adoption path):**
`benzene wrap <connection/table|topic>` → introspect → draft contract inferred → owner refines semantics/classifications/SLOs → registered as a source-aligned product with observability attached, **without rebuilding anything**. The lake/warehouse stays; the operating model changes on top of it (Part 1 §10). This is how the first ten products arrive.

**Consumer (marketplace flow, the Snowflake-grade UX):**
browse → understand (contract, semantics, SLOs, owner, trust, cost, sample) → request access (purpose, relied-upon SLOs) → producer approves (or auto-approval policy fires) → **access agreement** created → grants compiled to the native engine → immediately usable via the port of choice, zero consumer pipeline work. The agreement is what makes the consumer known for change impact, notification and cost attribution.

**Change:**
producer edits contract → pre-merge computed impact: "breaking for consumers X, Y (fields a, b; agreement clauses c)" → if breaking: major bump derived, deprecation window scheduled, consumers notified, parallel serving planned → merge only when the plan is valid.

---

## 6. Adoption modes and applicability

Three supported configurations, all first-class (Part 1 §9.2, §12):

1. **Contracts + registry only** ("minimum viable") — contracts, CI enforcement, discovery projection, quality/SLO monitoring over existing tables. No reconciler. Valuable standalone; no reorganisation required.
2. **Data products without full mesh** — the above plus provisioning, marketplace, policy compilation; a central team may still own cross-cutting products (the Adevinta hybrid is legitimate).
3. **Full operating model** — per-domain isolation, federated governance body, domain-owned products end-to-end.

Benzene ships the **fitness self-assessment** (domain count, bottleneck evidence, team capability, governance maturity, funding model) that tells an honest adopter when *not* to use mode 3 — this is a differentiator and it prevents failures that would be blamed on the tool.

---

## 7. Phasing

- **Phase 0 — Contracts & registry (minimum viable, coherent alone).** Manifest + ODCS profile schemas; registry + addresses; `benzene init/validate/wrap`; CI checks (lint, declared-field compat, manifest policy); catalogue projection (OpenMetadata first — the most standards-aligned target); OpenLineage emission from dbt/SQLMesh integrations. *Beats the datamesh-architecture.com dbt baseline or we have no product.*
- **Phase 1 — Provisioning & access.** Reconciler + first three adapters (proposed: Snowflake, Databricks/UC, Postgres); WAP publish gate with Data Contract CLI execution; marketplace + access agreements; policy compilation (tags/ABAC) + drift detection; credential-vended Iceberg port; Backstage plugin.
- **Phase 2 — The differentiators.** Computed change impact from column-level lineage; cost attribution + scorecard + trust signal; MCP port generation; BigQuery + AWS + Kafka adapters; DCAT/DPROD + ODPS exporters.
- **Roadmap.** Sidecar/portable enforcement; EDC/IDS cross-organisational port; Gravitino federation for brownfield multi-catalogue estates; Delta Sharing server hosting; BI-lineage emitters.

Sequencing rule from the research: **never platform-first**. Each phase lands with real data products built on it; patterns are extracted into templates only after they've worked twice.

---

## 8. Requirements traceability

Part 1's checklist is the contract for this design: all 21 **MUST** requirements are satisfied by Phases 0–1 as specified above (manifest/registry/addresses: MUST 1–4; contracts & enforcement: 5–9; policy & access: 10–13; lineage & discovery: 14–15; trust: 16; consumers & deployability: 17–18; governance shape: 19–20; retirement: 21). **SHOULD** 22–37 map to Phases 1–2; **COULD** 38–43 to Phase 2 and roadmap. Deviations require an ADR against the checklist item.

## 9. Resolved vs remaining questions

**Resolved by this draft (D1–D15):** artefact vs entry; control plane vs kernel; no data plane; hybrid policy enforcement; standards stack; contract semantics; computed change impact; ABAC; Iceberg default; isolation; cost; trust; polysemes; CLI-first.

**Remaining — need input before or during Phase 1:**

1. **Target adopter** — internal platform for one org, or a product for many? Changes multi-tenancy, packaging (SaaS vs self-host) and the adapter roadmap priority.
2. **Implementation stack** — control-plane language/runtime, and Terraform vs Crossplane as the reconciler substrate (the repo family suggests candidates; not a research question, a preference).
3. **First real domain/products** — Phase 0 must land on real data; which?
4. **Chargeback posture default** — showback-only vs chargeback-enabled (research says showback first; the default matters for adoption).
5. **dbt concentration-risk hedge** — how much Phase 0 leans on dbt-only ergonomics vs treating SQLMesh as an equal from day one.

---

*Decisions D1–D15 stand unless challenged; challenge them by ADR referencing the research documents.*
