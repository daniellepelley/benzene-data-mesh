# Data Mesh Market Landscape — What Exists, What Works, What to Steal

**Document:** `docs/research/02-data-mesh-market-landscape.md`
**Part 2 of 2** (Part 1 covers data mesh principles, patterns and design principles — not re-derived here.)
**Date of research:** August 2026
**Status:** Research input to the design of `benzene-data-mesh` (greenfield).

---

## Research aim

To map the *existing* market for data mesh and adjacent data-product/data-platform solutions, so that the design of a new greenfield implementation is grounded in what has actually been built, sold, adopted and — frequently — abandoned. Specifically: which capabilities are now commodity and should be integrated rather than rebuilt; which are genuinely contested; which problems nobody has solved (the opportunity space); and which standards and tools a new implementation must interoperate with on day one to be credible inside a real organisation.

This is deliberately a *skeptical* survey. A large fraction of "data mesh" product positioning is marketing veneer over pre-existing catalogue, warehouse or virtualisation products. Where that is the case it is said plainly.

### Method and confidence flags

Research was conducted via ~40 web searches and direct fetches of primary sources. A material constraint: a network egress policy blocked direct fetching of several vendor domains (nextdata.com, thoughtworks.com, starburst.io, learn.microsoft.com, docs.aws.amazon.com, docs.getdbt.com, datacontract.com, arxiv.org and others). GitHub, Google Cloud and a number of other primary sources were directly readable. Consequently:

- **[High confidence]** — read from primary source (repo, spec, docs) or corroborated across ≥2 independent sources.
- **[Medium confidence]** — from secondary reporting or search-engine summarisation of a primary page I could not open directly.
- **[Vendor claim — unverified]** — the vendor asserts it; I could not confirm it independently. Treat as marketing until proven.

Every non-trivial claim below carries one of these where it matters. The Sources section lists what was actually consulted.

---

## Executive summary — what the market implies for our design

1. **Data mesh as a product category has not consolidated; data *products* as a unit of work have.** Thoughtworks' own 2026 retrospective concedes the hype cycle is over and describes "data products and self-serve data platforms" as having become *foundational commodities*, while domain ownership remains the thing organisations fake rather than do. Gartner has placed data mesh at "obsolete before plateau" and pushes data fabric instead. **The word is toxic; the substance is mainstream.** Design for the substance, market it as data products and governed sharing. [High confidence]
2. **The four principles are not equally served by anything on the market.** Every surveyed product covers discovery + governance well, contracts moderately, self-serve provisioning poorly, and *cost attribution and domain-level economics* almost not at all. That asymmetry is the map of the opportunity.
3. **Nextdata OS — Zhamak Dehghani's own company — is the only vendor attempting the full original vision** (autonomous "data product containers" with an embedded kernel rather than a central orchestrator, pluggable compute drivers, no data migration). It is also small, pre-revenue on public information, $12M seed, and its technical claims are almost entirely unverifiable from public material. Architecturally it is the most interesting thing in the market and the most commercially fragile. [Medium/vendor claim]
4. **The cloud vendors have all shipped mesh-shaped primitives, and all of them stop at their own perimeter.** AWS (domains/projects in SageMaker Unified Studio over Lake Formation), Microsoft (Fabric domains + OneLake shortcuts + Purview governance domains), Google (Dataplex Universal Catalog + BigQuery Sharing), Snowflake (Horizon Catalog + Internal Marketplace), Databricks (Unity Catalog + Delta Sharing). Each is genuinely good *inside* its walls and each makes cross-vendor federation someone else's problem. **A credible new implementation must be the layer that spans them, not another walled garden.**
5. **The metadata/catalogue layer has consolidated violently and is now a commodity feature of larger platforms.** In 2025 alone: ServiceNow → data.world, Snowflake → Select Star, Atlassian → Secoda, Salesforce → Informatica (~$8bn), Coalesce → CastorDoc. **Do not build a catalogue.** Integrate with DataHub/OpenMetadata/Atlan/Purview and treat catalogue as a projection target. [High confidence]
6. **Contracts have a winner: the Open Data Contract Standard (ODCS, Bitol, LF AI & Data, v3.1.0).** The competing Data Contract Specification (v1.2.1, MIT) has formally deprecated itself in favour of ODCS with migration tooling and support only through end of 2026. This is a rare, clean standards convergence — take it. [High confidence]
7. **The data *product* descriptor layer has NOT converged.** At least four live candidates: Bitol's Open Data Product Standard (ODPS v1.0.0), the Open Data Product Initiative's Open Data Product Specification (confusingly also "ODPS", v3.1/v4.x), OMG/EKGF's DPROD (a DCAT profile, v1.0 beta Feb 2025), and Quantyca's Data Product Descriptor Specification (DPDS). Same acronym, different standards. **Pick one internal canonical model and emit the others.** [High confidence]
8. **OpenLineage is the safe lineage bet.** First-party emitters in Airflow, Spark, Flink and dbt; consumed by Marquez, DataHub, Atlan, OpenMetadata and Microsoft Fabric. Known gaps: BI-tool lineage is essentially absent, and column-level coverage is emitter-dependent. Emit OpenLineage, pick the backend later. [High confidence]
9. **The Iceberg REST catalogue is the emerging interop substrate for storage** — one client per engine, one server per catalogue, plain HTTP, with credential vending. Apache Polaris became a top-level Apache project in Feb 2026. But the spec standardises the wire protocol only — *not* RBAC, masking, lineage or federation — which is exactly where vendors re-fragment (Unity Catalog notably does not expose standard Iceberg REST credential vending). [Medium/High]
10. **Self-serve provisioning is where the market is thinnest and the pattern is clearest.** The credible answer is the internal developer platform stack (Backstage or Port + Crossplane/Terraform + Argo CD) with data-specific templates. Witboost, DataOS and Open Data Mesh Platform are the only products that genuinely take this seriously; Witboost's "tech adapter" microservice-per-technology pattern is the single most stealable architecture in this survey.
11. **Federated computational governance is real and has a proven shape: policy-as-code with OPA/Rego, driven from data product metadata.** UBS's enterprise data mesh maps ODRL constraints to Rego rules evaluated by OPA against user attributes plus data product metadata. This is the pattern to copy. [Medium confidence]
12. **Nobody has solved runtime contract enforcement.** The market is saturated with tools that *describe* contracts and SLAs and starved of tools that continuously enforce them against live data and *block* violations. This is a genuine gap and a differentiation opportunity.
13. **Nobody has solved cost attribution per data product.** FinOps practice exists (tag warehouses, showback then chargeback) but no mesh product makes cost a first-class property of a data product. Given that domain ownership without domain economics is why mesh initiatives stall politically, this is the highest-leverage under-served capability.
14. **The dominant failure mode is social, and tooling amplifies it.** Recurring across every source: domain teams are given data-product responsibility without compensation, staffing or incentive; catalogues become shelfware; duplicated pipelines proliferate; "domains" become renaming exercises. **Design for a low floor of obligation on domain teams** — the platform should make the default path cheap and the governance automatic, not hand domains a second job.
15. **AI agents are the new consumption port and the new justification.** MCP has become the standard way agents discover and invoke data products (Entropy Data ships a `dataproduct-mcp` server; Snowflake, Atlan and others are converging on agent-facing semantic context). Building agent-facing access as a *first-class output port with contract enforcement in the path* is both current and defensible. [Medium/High]

---

# 1. Purpose-built data mesh / data product platforms

## 1.1 Nextdata OS (Nextdata Technologies)

**What it is.** The company Zhamak Dehghani founded in 2022; first product **Nextdata OS** launched **8 April 2025**, positioned as a unified platform for building, governing and discovering "autonomous data products".

**Architecture (as publicly described).** The unit is the **data product container** — a long-running, environment-aware component encapsulating ingestion, transformation, quality enforcement and policy management in one deployable unit. The distinctive claim is the rejection of central orchestration: instead of a control plane driving execution, a **"poly-compute kernel"** is loaded *into every data product at runtime, wherever it is hosted*, managing dependencies peer-to-peer. A **pluggable driver architecture** keeps existing storage/compute/security/quality systems in place — explicitly no migration. Authoring via Python DSL, YAML, SQL, a Studio UI, or a conversational agent. [Vendor claim — unverified: nextdata.com was unreachable and there is no public repo or technical documentation I could locate.]

**Principles.** Uniquely attacks all four by design. Whether it delivers is unverifiable.

**Licensing / cost.** Proprietary, no public pricing. $12M seed (Greycroft, Acrew); described in mid-2026 secondary sources as pre- or early-revenue, single product. [Medium confidence]

**Pros worth stealing.** (a) The **container as the unit of ownership and deployment** — a data product as one versioned artefact carrying transformation logic, contract, quality assertions and policy, rather than a catalogue row pointing at a table someone else manages. This is the strongest idea in the market. (b) **Kernel-in-the-product instead of central orchestrator** — central orchestration is exactly the recentralisation that kills meshes. (c) **Bring-your-own-compute via drivers** — no-migration adoption is the only realistic enterprise entry path.

**Cons.** Severe vendor risk (seed-stage, single-product company owning your data product runtime). "Autonomous" is contested enough that the company published a public correction to sympathetic analyst coverage. Zero public technical surface — no open spec, no OSS core, no reachable API docs — which for the author of an open paradigm is notable and prevents ecosystem formation.

**→ Design implication:** Adopt the *data product container* model and the *thin embedded runtime + registry, not central orchestrator* split — but build it on open primitives (OCI, Kubernetes, Terraform) so the pattern survives the vendor.

## 1.2 Entropy Data (formerly Data Mesh Manager / Data Contract Manager)

**What it is.** A lightweight SaaS "marketplace for data products, enforced by data contracts", from Simon Harrer and Jochen Christ, spun out of INNOQ in 2025 and **rebranded from Data Mesh Manager to Entropy Data on 6 October 2025** (same product, three names). Listed on the Thoughtworks Technology Radar. [High confidence]

**Architecture.** Metadata plane only — stores and processes no data. Entities: domains, data products, output ports, data contracts, global policies, teams. Contracts are modelled as **bilateral agreements** created through a request/approve access workflow — a meaningfully better semantic than ODCS v3's producer-published dataset spec. Natively supports ODCS and the Open Data Product Standard. Integration via connectors (Snowflake, Databricks, GCP, Hive, MariaDB, Snowflake usage), a Java SDK, a Python CLI with Claude Code/Codex integration, dbt and Databricks "data product builder" plugins, and a **`dataproduct-mcp` MCP server** so agents discover and query products under governance. ~24 GitHub repos; connectors and SDK MIT-licensed; a Community Edition (`entropy-data-ce`, Bicep-deployed) exists; commercial product closed. Active through Aug 2026. [High confidence — read from GitHub]

**Principles.** Strong on data-as-a-product and federated governance; explicitly no self-serve provisioning; domain ownership modelled but not enforced.

**Licensing / cost.** SaaS, **per user with read-only users free and unlimited data products/assets**; transparent published pricing, negotiated fixed price for large customers. [Medium confidence] Notably healthy: it does not tax you for creating data products.

**Pros.** Small, opinionated, standards-native, no data plane to operate. The bilateral-contract-via-access-request workflow is exactly right. The MCP server is ahead of the market.

**Cons.** A registry and workflow tool — all hard execution (provisioning, enforcement, quality runs, lineage capture) is delegated to your existing stack. Tiny vendor. Bicep-first CE suggests an Azure lean. Rebranding away from "data mesh" within two years is itself a signal.

**→ Design implication:** Steal the **bilateral contract via access request** lifecycle (contracts as negotiated agreements, not just published schemas), the **pricing philosophy** (never charge — in money or ceremony — per data product created), and the **MCP output port**.

## 1.3 Datamesh Architecture (INNOQ open resources)

`datamesh-architecture.com` — a free, vendor-neutral reference architecture with per-stack guides (dbt+Snowflake, BigQuery, others), a Data Product Canvas, Data Mesh Canvas, fitness tests and learnings write-ups; openly maintained on GitHub (~88★, ~796 commits, active). [High confidence]

Its dbt/Snowflake guidance is the best *free* concrete implementation and refreshingly unglamorous: each domain owns its dbt project(s) and CI; models intended as data products are tagged `data_product` and placed in `/models/data_products/`; consumers declare upstream products as dbt `source`s with tests encoding expectations; governance policies (grants, PII hashing by tag) are automated via macros and lifecycle hooks.

**→ Design implication:** This is the baseline we must beat. If our system is not materially better than "tagged dbt models + macros + CI", we have not justified our existence. Also adopt the Data Product Canvas as the human-facing artefact from which the machine-readable descriptor is generated.

## 1.4 Agile Lab Witboost

**What it is.** A commercial enterprise data product management platform (Italy) aimed at regulated industries — the most complete *provisioning-oriented* mesh platform in the market.

**Architecture — three ideas that matter.**
- **Practice Shaper** — models Witboost's own entity types (domains, systems, components, templates) as a **fully configurable property graph**, so an organisation shapes the metamodel to its structure rather than adopting the vendor's ontology.
- **Tech Adapters (specific provisioners)** — microservices, one per target technology, that actually deploy a component. The Apache-2.0 Starter Kit (~27★, ~57 commits) ships 30+ adapters across Snowflake, Databricks, AWS, GCP, Azure, Kafka and dbt, plus templates for data products, storage areas, output ports and workloads. [High confidence — read from GitHub]
- **Computational Governance Platform** — deploy-time policy enforcement as code, plus data contracts with "guardian agents" for runtime monitoring. Catalogue plugins publish *to* Collibra, OpenMetadata and Purview rather than replacing them.

**Licensing / cost.** Proprietary core; the Apache-2.0 Starter Kit's 27 stars indicate a shop window, not a community. No public pricing; expect enterprise licence plus substantial implementation services.

**Pros.** The **provisioner-per-technology microservice pattern** is the most reusable architecture in this survey: a stable internal provisioning API plus an adapter contract means new technologies are added without touching the core. The **configurable metamodel** avoids the classic catalogue failure of imposing a vendor ontology. Publishing to existing catalogues is the right integration posture.

**Cons.** Heavy, enterprise sales motion, consultancy-shaped, and effectively invisible in open practitioner discourse — which makes scale claims unverifiable. Lock-in is real despite "technology-agnostic" positioning: the Practice Shaper graph and adapter contracts are proprietary.

**→ Design implication:** **Copy the tech-adapter architecture** — define a stable `Provisioner` interface (validate → plan → provision → unprovision → reverse-provision access) with adapters as independent plugins. **Copy the configurable metamodel** — do not hardcode domain→product→port. **Be a producer of catalogue entries, not a catalogue.**

## 1.5 Blindata + Open Data Mesh Platform (Quantyca)

**Blindata** is a governance/catalogue product with genuinely mesh-native modelling: data products as first-class catalogue citizens with terms, schema, SLAs and producer↔consumer contracts; a **marketplace** with certification and access request; **computational governance policies** with automated enforcement; automated SQL lineage across product dependencies; and a quality module. [Medium confidence — vendor docs]

**Open Data Mesh Platform (ODM)** is Quantyca's Apache-2.0 lifecycle platform (since early 2023) built on the **Data Product Descriptor Specification (DPDS)** — describing identity, owner, domain, version, *interface components* (ports) and *internal components* — with a microservice **Blueprint Service** for data product bootstrap. [High confidence on shape; Medium on traction, which appears modest.]

**Pros.** DPDS is the most explicitly **port-oriented** descriptor available (input / output / discovery / observability / control ports as distinct interface components) — closest to the original book. Apache 2.0. Blueprint-based bootstrap is the right self-serve primitive.

**Cons.** Low adoption; a fourth competing product descriptor in an already-fragmented space; the OSS platform is under-resourced relative to the standards it defines.

**→ Design implication:** Adopt DPDS's **port taxonomy** as internal modelling vocabulary even where the serialisation is ODCS + ODPS. It is the cleanest decomposition of what a data product exposes.

## 1.6 DataOS (The Modern Data Company)

A "data operating system": distributed microservices over existing infrastructure, **declarative YAML specs** for data products driven via CLI/API, end-to-end lifecycle, semantic layer and governance. On the Thoughtworks Radar. [Medium confidence]

**Pros.** Declarative-spec-first with CLI/API primacy is the right DX; multi-cloud abstraction; composability over an existing stack. **Cons.** "The only real data operating system on the market" is unfalsifiable marketing; the drift of its messaging toward AI/semantic layer suggests the mesh framing did not sell; proprietary and opaque.

**→ Design implication:** Declarative YAML + CLI/API as the primary interface, UI as a secondary projection — not the reverse. Every product surveyed that leads with a UI has weaker developer adoption.

## 1.7 Adjacent and mis-shelved players

- **Starburst** (commercial Trino) and **Denodo** (data virtualisation) both market "data mesh". Both are federated query engines with a data-products veneer — Starburst's "Data Products" is a curated-dataset abstraction over Trino federation. **Marketing veneer over an older product.** Federation is a useful *output port implementation*, not a mesh. Integrate as a query substrate; ignore the positioning.
- **Confluent Stream Governance** is more substantive: Schema Registry data contracts with metadata, quality rules and migration rules; **AsyncAPI** export/import describing topics and schemas as an interface spec; stream lineage and catalogue. For event-shaped data products this is the strongest **write-time** contract enforcement anywhere. [Medium/High]
- **Synthesized** (named in the brief) is a test-data-generation / environment-provisioning vendor, not a mesh platform. No further coverage warranted.
- "10 best data mesh companies" listicles dominate search results and are content marketing. Treated as noise.

---

# 2. Cloud-vendor offerings

**Shared verdict:** all five implement domain-oriented decentralisation *within their own control plane* and treat everything outside it as the customer's federation problem. They are excellent domain-scoping and access-control substrates and poor cross-organisational mesh substrates.

## 2.1 AWS — Lake Formation + Glue + DataZone → SageMaker Unified Studio

**What it is.** Two layers. Underneath, **Lake Formation** provides fine-grained cross-account permissions over the **Glue Data Catalog**, enabling the canonical AWS mesh reference architecture: producer accounts, consumer accounts, and a central governance account holding the catalogue, with permissions granted cross-account rather than data copied. On top, **Amazon DataZone** (business catalogue, domains, projects, publish/subscribe) was folded into **Amazon SageMaker Unified Studio** (GA **March 2025**), with SageMaker Catalog as DataZone's evolution; DataZone domains became upgradeable to SageMaker unified domains in **June 2025**. Hierarchy: Account → Domain → Domain Unit → Project → Member. [High confidence]

**Principles.** Domain ownership strong (accounts + domains + units). Data-as-a-product moderate — a "data product" is a curated asset bundle published with business metadata, not a deployable artefact. Self-serve partial — blueprints provision *environments*, not data products. Federated governance strong within AWS.

**Cost.** SageMaker Unified Studio carries **no direct charge**; you pay underlying services. DataZone/SageMaker Catalog **removed its per-user subscription fee on 1 November 2024** for pay-as-you-go, with metadata storage ~**$0.40/GB**. [High confidence] A genuinely mesh-friendly model — no per-seat tax on discovery.

**Pros.** Account-per-domain is a hard isolation boundary *with its own bill* — the best **cost-attribution primitive of any cloud**, essentially by accident. Cross-account sharing without copying is mature.

**Cons and reported problems.** Cross-account Lake Formation grants are a documented pain source: authorisation failures on assets in associated accounts, "insufficient Glue permissions" errors, and the requirement to explicitly grant DataZone access to external Glue databases; cross-region access has documented limits; per-account user-profile limits exist in DataZone domains. And the product has been renamed/absorbed twice in three years — a real planning risk. [Medium/High]

**→ Design implication:** Use **account/project-per-domain** as the isolation and cost-attribution primitive on AWS and emit Lake Formation grants rather than inventing a policy engine there. Do **not** bind our data product model to DataZone/SageMaker Catalog entities — that surface has been renamed twice.

## 2.2 Google Cloud — Dataplex Universal Catalog + BigQuery Sharing

**What it is.** Dataplex organises data into **Lakes** (≈ domains), **Zones** (≈ groupings by readiness/workload/data product) and **Assets** (references to GCS buckets or BigQuery datasets, possibly in other projects) — a logical overlay with no data movement, plus IAM data policies, automatic metadata extraction, data quality rules and audit logging, with metadata surfaced as BigQuery external tables for federated query. Google's canonical mapping: **GCP project = domain environment, BigQuery dataset = data product boundary**. Since rebranded **Dataplex Universal Catalog** with a first-class Data Products concept. **BigQuery Sharing (ex-Analytics Hub)** provides cross-organisational exchanges. [High on the 2022 reference model, read directly; Medium on current Data Products.]

**Pros.** The cleanest conceptual mapping of mesh concepts to platform primitives of any cloud. Zero-copy by construction (BigQuery separates storage and compute). Analytics Hub/BigQuery Sharing is a mature listing and exchange mechanism.

**Cons.** Lake/zone/asset is *purely logical governance metadata* — it does not provision, build, or enforce contracts. Google's own launch material states no limitations, which is itself a tell. Heavy BigQuery gravity. Repeated rebranding (data fabric → universal catalog) makes it a moving target.

**→ Design implication:** Project our model onto Dataplex entries and BigQuery Sharing listings; use BigQuery dataset boundaries as the product storage boundary on GCP. Do not adopt lake/zone/asset internally — it is a GCP-specific governance overlay.

## 2.3 Microsoft — Fabric domains + OneLake shortcuts + Purview Unified Catalog

**What it is.** The most explicitly mesh-shaped hyperscaler answer. **Fabric domains** group data by business area; **workspaces** isolate workloads and permissions and belong to exactly one capacity and one domain, with items inheriting a domain attribute. **OneLake shortcuts** let a workspace reference data stored elsewhere with no copy — the mesh-shaped primitive. Every Fabric item is auto-catalogued in **Purview**, whose **Unified Catalog** adds **governance domains** as boundaries for ownership and discovery of **data products** (packaged sets of tables, files and Power BI reports), plus glossary terms, OKRs and critical data elements. Sensitivity labels and policies follow data shared cross-tenant. [High confidence]

**Cost.** Fabric is capacity-based (CU). Purview Data Governance moved to pay-as-you-go **effective 6 January 2025**: **$0.0165 per governed asset per day (~$0.50/month)** plus **Data Governance Processing Units at $15 / $60 / $240** (basic/standard/advanced) for quality scanning and health controls. Scanned Data Map assets are free; you pay when an asset is linked to a governance concept such as a data product. [High confidence]

**Pros.** Domain → workspace → item with automatic metadata inheritance is good modelling. **OneLake shortcuts are the best zero-copy cross-domain sharing primitive in any cloud.** Pay-per-*governed* asset rather than per scanned asset is well judged.

**Cons and reported problems.** Fabric's capacity model generates persistent, first-party-documented pain: throttling under concurrent load, `CapacityLimitExceeded`, capacity metrics app breakages, phantom background processes reported at 330% of capacity, and a bug showing 100% capacity while paused, firing false alerts. **A shared capacity is an anti-mesh construct** — it couples domains through one noisy-neighbour resource and defeats per-domain attribution and isolation, exactly inverting AWS's account-per-domain advantage. Purview's per-governed-asset pricing means governance cost scales with mesh size. [Medium/High]

**→ Design implication:** Steal the **shortcut** concept — cross-domain consumption should create a *reference with policy inheritance*, never a copy or a new pipeline. Learn the negative lesson: **never let domains share a throttled resource pool without per-domain quota and attribution.**

## 2.4 Snowflake — Horizon Catalog + Internal Marketplace

**What it is.** **Horizon Catalog** (governance, discovery, policies, lineage) plus the **Internal Marketplace** — a built-in self-service hub for publishing and discovering data products *within* an organisation over Secure Data Sharing, so no data moves. A Snowflake data product is a collection of related objects plus metadata (description, ownership, contacts, SLOs, data dictionary) governed by RBAC with fine-grained control over which consumers see which parts. From Summit 2026: Horizon Context consolidates business logic and pulls context from schemas, BI models, pipelines and query logs **via metadata connectors and the OpenLineage API**; Semantic View Autopilot GA Feb 2026; Openflow GA; "Intent-Driven Governance" translating natural-language rules into policies. Snowflake acquired **Select Star** (Nov 2025) to extend Horizon's metadata reach. [High on announcements; Medium on maturity]

**Cost.** Credit-based, bundling compute and infrastructure into one price. Cross-cloud/internet egress ~**$90–$155/TB**, mitigated by an Egress Cost Optimizer claiming up to 96% savings. [Medium confidence]

**Pros.** The **internal marketplace with request/approve access over zero-copy sharing** is the most polished cross-domain consumer experience anywhere and the model most worth emulating for UX. Listings carry SLOs and ownership as first-class metadata. Horizon Context consuming the **OpenLineage API** confirms OpenLineage as the safe standard.

**Cons.** All of it is Snowflake-perimeter. "Data mesh" here repackages Secure Data Sharing + catalogue + RBAC; Snowflake's own material frames mesh as a use case for existing features. Compute is centralised, so domain compute autonomy is nominal. Egress economics penalise genuinely cross-cloud meshes. LLM-generated governance policy carries obvious, unaddressed correctness risk.

**→ Design implication:** Copy the **marketplace flow**: browse → understand (contract, SLOs, owner, sample) → request access → approved → immediately usable, with zero pipeline work by the consumer. That flow, not the catalogue, drives adoption.

## 2.5 Databricks — Unity Catalog + Delta Sharing + Lakehouse Federation

**What it is.** **Unity Catalog** is the governance/metadata plane (catalog → schema → table/volume/function/model, with lineage, audit and fine-grained access control); the common mesh pattern is catalog-per-domain within a metastore, shared across workspaces. **Delta Sharing** handles sharing beyond the metastore boundary; **Lakehouse Federation** provides query-time access to external systems.

**Delta Sharing** [High confidence — read from the repo]: open REST protocol, Apache 2.0, ~956★. A sharing server exposes Delta/Parquet data on object storage; recipients authenticate with a profile file (endpoint + bearer token); the server issues **pre-signed URLs** so clients read directly from storage without the provider exposing storage credentials or proxying bytes. Clients for Python/pandas, Spark, Power BI, Tableau, Java, Rust, Go, Node.js, C++, R. The repo itself warns the **reference server is not a complete secure web server** and needs a secure proxy; Structured Streaming has trigger constraints.

**Unity Catalog OSS** [High confidence — read from the repo]: open-sourced June 2024, Apache 2.0, LF AI & Data sandbox, ~3.5k★, ~294 open issues. OpenAPI REST API; compatible with the Hive metastore API and the **Iceberg REST catalog API**; supports tables (Delta/Iceberg/Hudi via UniForm, plus Parquet/JSON/CSV), volumes, functions and AI models; works with DuckDB, Spark and others.

**Cons and reported problems.**
- **Unity Catalog's governance is tightly coupled to the Databricks perimeter**; Delta-first design makes Iceberg and non-Databricks engines second-class. [Medium]
- **Delta Sharing is read-only** — no bidirectional collaboration. **Lakehouse Federation is read-only, performance-limited and suited to low-volume aggregated data and ad hoc use** — not a production serving path. [Medium]
- **UC OSS open issues cluster exactly where a neutral substrate must be solid**: custom S3 endpoints and bucket configuration failures (#43, #805), inability to write to UC or create tables from local Spark (#966, #560, #143), Structured Streaming unsupported (#715), Trino integration access failures (#740), no official Docker image or Helm chart (#433), CI flakiness (#1133). It is not yet a drop-in multi-engine catalogue. [High confidence — read from GitHub]
- **UC does not expose a standard Iceberg REST credential-vending endpoint**, forcing external engines onto a UC-specific client — re-fragmenting the standard it claims to support. [Medium]
- Pricing: DBU **plus** a separate cloud infrastructure bill (infra typically 50–70% of total); Standard tier retiring in 2026, pushing workloads to Premium rates. [Medium]

**Pros worth stealing.** **Delta Sharing's credential model** — short-lived, scoped, pre-signed access to object storage, so the provider never hands over storage credentials and never proxies data. That is the right shape for any cross-boundary output port. Catalog-per-domain is a clean isolation primitive.

**→ Design implication:** Adopt **credential vending / short-lived scoped pre-signed access** as the standard output-port mechanism over object storage, and support **Delta Sharing as an output port type**. Do not build on Unity Catalog OSS as the neutral substrate yet — its issue tracker says it isn't ready.

---

# 3. Catalogue / governance / metadata layer

**Market-level verdict: this layer is consolidating into larger platforms and should be integrated, not built.** 2025 acquisitions: ServiceNow → data.world (May); Snowflake → Select Star (Nov); Atlassian → Secoda (for Rovo AI); Salesforce → Informatica (~$8bn); Coalesce → CastorDoc. The stated driver in every case is *supplying context to AI agents*. Independent catalogues are being absorbed as context layers for agentic platforms. [High confidence]

## 3.1 Commercial catalogues

| Vendor | Position | Notes |
|---|---|---|
| **Collibra** | Leader, 2025 Gartner MQ for D&A Governance Platforms | Deepest workflow/stewardship; heaviest |
| **Alation** | Visionary in MQ; Leader in 2025 Forrester Wave; "Agentic Data Intelligence Platform" | Better UX than Collibra by review consensus |
| **Atlan** | Visionary → **Leader in 2026 MQ**; active metadata, cloud-native | Markets a "Data Marketplace — replace the catalog nobody uses" |
| **data.world** | Acquired by ServiceNow (2025) | Knowledge-graph/RDF-native; ~$90k/yr Essentials list price reported |
| **Secoda / Select Star** | Acquired (Atlassian / Snowflake, 2025) | Lightweight catalogues, now features of other platforms |

**Reported practitioner problems** [Medium confidence — review aggregation]: Collibra — cumbersome, slows work down, complex setup, unclear roles pulling too many people into reviews, approval bottlenecks, policy reviews as box-ticking, navigation cited as the single biggest satisfaction barrier. Alation — UI slowness on large assets, cost prohibitive for smaller teams. Both typically **6–12 months to deploy**; a ~$1.27M three-year TCO including licences, implementation, training, maintenance and stewardship labour is plausible at enterprise scale.

The deeper universal complaint: **catalogues become shelfware.** Teams document everything without asking what people need; workflows are too complex for non-technical users; the catalogue is bought, shelved and forgotten; the primary failure is failing to change user behaviour. Atlan's own marketing concedes the point.

**→ Design implication:** **Do not build a catalogue.** Build a data product *registry* — the authoritative, versioned, machine-readable source of truth for products, contracts, ports, policies and owners — and publish projections into whatever catalogue the org already owns. And **fight shelfware by making the registry the thing that provisions**: if the only path to a deployed data product runs through the registry, the registry cannot go stale and discovery inherits accuracy for free. This is the most important structural lesson from the catalogue market's failures.

## 3.2 Open source catalogues

**DataHub** (Apache 2.0, ~12.5k★) — streaming-first, API-first; entity/aspect metadata model with relationships, column-level lineage and profiling; push/pull ingestion with 80+ connectors; MySQL + Elasticsearch + Kafka storage; GraphQL/REST via GMS; supports domains, tags, glossary, policies and data products. [High confidence — read from repo]
*Reported problems* [from the issue tracker]: OOM during BigQuery ingestion, connection reliability (retry exhaustion, MSK TLS), broken OIDC logout, role configuration gaps after fresh installs, Airflow 2.7+ plugin incompatibility, Oracle ingestion failures post-upgrade, entity duplication in navigation, search metadata gaps, tag application failures. Pattern: **integration reliability, performance at scale, operational friction.**

**OpenMetadata** (Apache 2.0, ~14.9k★, v1.12.x) — schema-first with 700+ JSON Schemas; 130+ connectors; knowledge-graph storage linking assets, columns, people, teams, quality, lineage, policies and business concepts. The **most standards-aligned product in the survey**: DCAT/DPROD for catalogue representation, PROV-O for provenance, **OpenLineage** for lineage, **ODCS 3.1** for contracts, RDF/OWL + JSON-LD for semantic interop, MCP for agents. Data products modelled DCAT/DPROD-aligned with lifecycle states, ownership, input/output datasets and policies. [High confidence — read from repo]
*Reported problems*: profiler lacks complex-type support (JSON/arrays/geo), connector metadata gaps (Hive partitions, MSSQL/Oracle backends, ClickHouse materialised-view lineage), `MetaData.reflect()` causing excessive catalog queries and long-lived idle transactions, UI context gaps.

**Operational reality for both** [Medium]: DataHub is heavier (Kafka + Elasticsearch + MySQL) with higher infra and monitoring cost; OpenMetadata's unified design is lighter but still needs Kubernetes, Helm, Postgres, Elasticsearch and Airflow expertise. Cited estimate: **a self-hosted catalogue consumes 0.25–1 platform engineer** once upgrades, connector repair and access requests are counted, and most teams eventually find that burden exceeds the licence savings.

**Amundsen** — **effectively abandoned**: last release August 2024, ~0 commits/week, no roadmap. Do not adopt. [High confidence]
**Apache Atlas** — still maintained with an active committer community, but Hadoop-era in design. Relevant only if you already run it. [Medium]
**Marquez / OpenLineage** — OpenLineage is the *standard*, Marquez the reference backend (Postgres lineage graph + UI, table and column granularity). First-party emitters: Airflow, Spark, Flink, dbt, some warehouse producers. Consumers: Marquez, DataHub Cloud, Atlan, OpenMetadata, **Microsoft Fabric (since 2024)** and **Snowflake Horizon Context**. Gaps: **column-level coverage is emitter-dependent** (strongest in Spark) and **BI-tool lineage is essentially absent** — no native Looker/Tableau/Power BI emitters as of 2026. [High confidence]
**Unity Catalog OSS / Polaris / Gravitino** — see §5.2; these are *technical* catalogues (table metadata + credentials), not business catalogues.

**→ Design implications:** Emit **OpenLineage** from everything we run — it is the only lineage format with genuine multi-consumer support, now including Microsoft and Snowflake. Model our data product entity to be losslessly projectable into **OpenMetadata's DCAT/DPROD-aligned representation**. Assume the org already has a catalogue and that it is partly stale; our value is being its authoritative upstream. And budget honestly: anything self-hosted with Kafka/Elasticsearch/Airflow imposes ~0.25–1 FTE of ops on the adopter — **minimise operational footprint aggressively.**

---

# 4. Data product & contract tooling

## 4.1 dbt (Core v2 / Fusion, dbt Mesh, model contracts) — and the Fivetran merger

**What it is.** The de facto transformation layer and, via **model contracts**, **access modifiers** (private/protected/public), **model versions** and **cross-project `ref()`**, the closest thing to a native data-product-interface mechanism in the mainstream stack. **dbt Mesh** is the multi-project pattern: upstream projects publish a manifest, downstream projects reference public models across boundaries with cross-project lineage. Contracts are enforced *before* tests run — a failed contract means the model is not built and tests do not execute. Requires dbt ≥1.6 and, for the managed cross-project experience, **Enterprise/Enterprise+ tiers**. [High confidence]

**2025–26 upheaval** [High confidence]. The Rust **dbt Fusion engine** launched in public beta **28 May 2025 under Elastic License v2**, triggering community concern. **dbt Labs and Fivetran announced a merger 13 October 2025, completed 1 June 2026**, alongside **dbt Core v2.0** bringing Fusion into dbt Core under **Apache 2.0**. Separately, **Fivetran became steward of the GX Core OSS project** after FICO acquired GX Cloud (public availability ended **1 June 2026**). One company now owns dbt, Fivetran and GX Core stewardship.

**Principles.** Data-as-a-product strong at the *interface* level; domain ownership strong (project per domain with own CI); self-serve weak (provisions nothing); federated governance partial (contracts, tests, macros — no policy engine).

**Cost.** Core free (Apache 2.0). dbt Cloud Enterprise reportedly **~$200–400 per developer seat per month** annually billed (~$36k–$84k/yr for 20 seats); Mesh included in Enterprise rather than separately metered. [Medium confidence]

**Pros.** Model contracts + versions + access modifiers are a proven-at-scale data product interface, and **a contract failure blocking the build** is exactly the enforcement semantics contracts need. Ubiquity: most 2026 data engineering roles still list dbt. `dbt-loom` gives cross-project refs on dbt Core without dbt Cloud.

**Cons.** Managed cross-project `ref()` is **paywalled behind Enterprise** — a direct tax on the mesh pattern. Contracts add maintenance overhead for still-evolving models with no external consumers, and teams over-apply them. Persistent community concern that investment shifts to Cloud/Fusion leaving Core on bug fixes, now compounded by the merger. **Concentration risk is material.**

**→ Design implication:** **Integrate deeply with dbt but never assume it.** Treat a contracted dbt model as one *implementation* of an output port; ingest `manifest.json`, contracts and tests; emit OpenLineage from dbt runs. Deliberately support **dbt Core-only** paths (manifest exchange à la `dbt-loom`) so adopters are not forced onto Enterprise to get cross-domain refs. Support SQLMesh as a peer.

## 4.2 SQLMesh / Tobiko Data

The credible architectural alternative to dbt (OSS since 2023). Three differentiators: **virtual data environments** (dev/staging/prod as views over shared physical tables — no duplication, dev compute cost collapses); **automatic column-level lineage** from SQL parsed by SQLGlot, enabling compile-time validation; and **state awareness / change categorisation** — column-level lineage distinguishes breaking from non-breaking changes so only genuinely affected models rebuild. [Medium/High]

**Pros worth stealing.** **Breaking vs non-breaking change classification derived from column-level lineage is the correct foundation for contract evolution.** Nobody else does it automatically. A system that can *compute* whether a change breaks a declared consumer expectation — rather than asking a human to assert a semver bump — is a major DX improvement and a real differentiator.

**Cons.** Much smaller ecosystem and hiring pool; job postings still overwhelmingly list dbt. Migration friction. Small vendor.

**→ Design implication:** Build **automated change-impact classification** into contract versioning: parse transformation SQL, derive column-level lineage, compare against declared consumer expectations from active contracts, classify breaking/non-breaking *before* merge. Borrow the technique (SQLGlot-style parsing) whether or not we adopt SQLMesh.

## 4.3 Data contract standards — the one place the market converged

**Open Data Contract Standard (ODCS)** — Bitol, a **Linux Foundation AI & Data** project (formed 30 Nov 2023 from AIDA User Group + LF AI & Data), Apache 2.0, **v3.1.0**, ~1.1k★, media type `application/odcs+yaml;version=3.1.0`. Sections: fundamentals, schema, references, data quality, support/communication, pricing, team, roles, SLA, infrastructure/servers, custom properties. v3.1.0 added relationships, stricter validation, richer metadata. **Semantic note: ODCS v3 describes a producer-published dataset specification, not a bilateral producer↔consumer agreement.** [High confidence — read from repo]

**Data Contract Specification (datacontract.com)** — **v1.2.1, MIT, ~418★, and formally deprecated**: the repo directs users to ODCS v3.1.0 as "the industry's unified standard" and "the conceptual successor", with support only **through the end of 2026** and migration tooling in the Data Contract CLI. [High confidence — read from repo] *A genuine, clean standards convergence.*

**Data Contract CLI** — MIT, Python 3.10–3.12, ~1.0k★, ~1,870 commits, active. ODCS-native: lints contracts, connects to sources, executes schema and quality tests. Imports/exports **25+ formats** (SQL, Avro, dbt, JSON Schema, Excel, Terraform, RDF); connects to **18+ systems** (Snowflake, BigQuery, Databricks, Postgres, DuckDB, Redshift, Trino, Kafka, S3/GCS/Azure). Runs as CLI, CI step, Python library or web API server. [High confidence — read from repo]

**→ Design implication:** **Adopt ODCS v3.1 as the canonical contract format, unmodified, and use the Data Contract CLI as the enforcement engine rather than writing our own test runner.** Highest-value integrate-don't-build decision available. Extend only via `customProperties`, never by forking the schema. Layer *bilateral agreement* semantics on top — that gap is real and ours to fill.

## 4.4 Data product descriptor standards — where it has NOT converged

| Standard | Owner | Version / date | Basis | Notes |
|---|---|---|---|---|
| **Open Data Product Standard (ODPS)** | Bitol / LF AI & Data | v1.0.0 | YAML, sibling of ODCS | ~115★; media type `application/odps+yaml;version=1.0.0`; pairs naturally with ODCS |
| **Open Data Product Specification (also "ODPS")** | Open Data Product Initiative / Linux Foundation | v3.0, **v3.1**, v4.0, v4.1 | YAML metadata model | v3.1 adds contract support referencing **both** DCS and ODCS; v4.1 links products to business performance; BASF cited as adopter |
| **DPROD (Data Product Ontology)** | OMG, originated at EKGF | **v1.0 beta1, Feb 2025** | **W3C DCAT profile** + RDF/OWL/SHACL/PROV | Decentralised publishing, cross-marketplace metadata aggregation; adopted by OpenMetadata |
| **DPDS (Data Product Descriptor Spec)** | Open Data Mesh Initiative / Quantyca | Apache 2.0, since ~2023 | JSON/YAML | Strongest **port** decomposition (input/output/discovery/observability/control) |

**Two live standards share the acronym ODPS.** That alone tells you this layer has not settled. [High confidence]

**→ Design implication:** Define **one internal canonical data product model** (versioned, stable IDs) and treat all four as **serialisation targets**. Ship exporters: ODCS for contracts (mandatory), Bitol ODPS and/or the ODP Specification for product metadata, DCAT/DPROD for semantic-web and catalogue interop. Structure the internal model on DPDS's port taxonomy. Do not bet on any single descriptor winning.

## 4.5 Quality & observability

- **Great Expectations** — GX Core remains Apache 2.0, but **GX Cloud was acquired by FICO and withdrawn from public availability on 1 June 2026**, with **Fivetran taking stewardship of the OSS project**. Anyone with GX Cloud in a critical path was forced to migrate. [High confidence]
- **Soda** — developer-first checks embedded in pipelines via SodaCL; strong data-contract story; good CI-native fit.
- **Monte Carlo / Anomalo** — managed ML-driven observability with minimal configuration; typical spend **$50k–$200k+/year**. Anomalo specialises in unsupervised anomaly detection across large estates.
- **Elementary** — dbt-native monitoring; cheap default for dbt shops.
- Common guidance: under ~15 engineers, GX Core + Elementary; 20+ engineers or multi-warehouse/compliance, evaluate Monte Carlo or Anomalo. [Medium confidence]

**The gap everyone names.** Governance platforms and contract formats have transformed how teams *define and share expectations*; **what they do not solve is continuous enforcement at runtime.** Few tools continuously validate live data with the power to *block* non-compliant data from moving downstream. [Medium confidence, corroborated across independent sources.]

**→ Design implication:** **Make runtime contract enforcement core, not an integration.** Quality assertions declared in the ODCS contract must execute on every publish, and a failed assertion must be able to *block promotion of the new output-port version* (fail-closed on the write path, like dbt contracts blocking a build) rather than merely alerting after consumers are poisoned. Delegate *check execution* to Soda/GX/dbt tests via the Data Contract CLI; own the *policy* of what happens on failure. That division is the defensible part.

---

# 5. Sharing / interop substrate

## 5.1 Intra-platform sharing

Covered above: **Snowflake Secure Data Sharing + Internal Marketplace** (zero-copy, RBAC-governed, best consumer UX); **Delta Sharing** (open REST, Apache 2.0, pre-signed URL credentials, wide clients, read-only); **OneLake shortcuts** (zero-copy reference with policy inheritance, best-in-class primitive); **BigQuery Sharing** (exchanges and listings); **Lake Formation cross-account grants** (mature, fiddly).

**The consistent pattern:** every mature platform converged on *reference-with-policy* rather than *copy-with-pipeline*. That convergence is strong enough to treat as settled architecture.

## 5.2 The table/catalogue substrate — Iceberg REST, Polaris, Gravitino

The **Iceberg REST Catalog specification** is the genuine interop win: implement the REST client once per engine and the server once per catalogue and everything interoperates over plain HTTP, replacing the previous N×M connector matrix. It includes **credential vending** — the catalogue mints short-lived, prefix-scoped storage credentials at table-load time, for exactly the table and operation the client is authorised to perform. [High confidence]

- **Apache Polaris** — **Apache top-level project since February 2026** (Snowflake + Dremio donation; commercially Snowflake Open Catalog). Full REST implementation with fine-grained RBAC, credential vending, and **Iceberg REST federation** of external catalogues (another Polaris, AWS Glue, custom implementations) — a "catalog of catalogs". [Medium/High]
- **Apache Gravitino** — donated by Datastrato in 2023; **1.2.0 shipped March 2026** as a "federated metadata lake". A **meta-catalogue**: stores no metadata, provides a unified API across Iceberg, Hive, RDBMS, Kafka and file metadata. Crucially, **access control is delegated to underlying catalogues — it adds discoverability, not enforcement** — and adds a network hop, typically 10–50 ms on metadata operations. Justified where business units already sit on different catalogue standards. [Medium]
- **Unity Catalog OSS** — Iceberg-REST-compatible in principle, but no standard credential-vending endpoint and an issue tracker showing it is not yet drop-in multi-engine (§2.5).

**Critical caveat:** the Iceberg REST spec standardises the wire protocol — namespaces, commits, credential vending — **but not RBAC, masking, lineage or federation**. Those are exactly where vendors re-fragment. [Medium/High]

**→ Design implication:** Target **Iceberg REST as the default technical catalogue interface** for lake-resident products, and **credential vending as the access mechanism for output ports** — short-lived, scoped, auditable, with no data proxying by us. But **own the RBAC/masking/lineage layer ourselves**, because the spec deliberately does not, and that is where our policy-as-code layer lives. Gravitino's meta-catalogue is the right model for brownfield estates; note its enforcement gap and fill it.

## 5.3 Cross-organisational: Gaia-X / IDS / Eclipse Dataspace Connector

The European answer to sharing across *organisational* boundaries. The **Eclipse Dataspace Connector (EDC)** implements the **IDS** standard and the **Gaia-X Trust Framework**; **Eclipse Tractus-X** is the connector distribution for Catena-X and Manufacturing-X. It is in production — a November 2025 peer-to-peer dataspace transaction between Korea and Europe ran on the Tractus-X stack with the IDSA protocol and Gaia-X trust framework. [Medium confidence]

**What it gets right that nothing else does:** data **sovereignty** as a first-class concept — usage policies (ODRL) that travel with the data and are enforced at the consumer's connector, verifiable credentials for participant identity, contract negotiation as an explicit protocol, no central authority. It is the only part of the market that takes cross-organisational federation seriously.

**Cons.** Heavy, consortium-driven, EU/manufacturing-centric, steep learning curve, thin adoption outside Catena-X.

**→ Design implication:** Do not build on EDC on day one, but borrow two concepts: **ODRL as the policy vocabulary** for usage constraints (a W3C recommendation, what UBS mapped to Rego, what the dataspace world already speaks), and **contract negotiation as an explicit protocol** with verifiable participant identity. Keep an EDC-shaped output port on the roadmap and design the policy model so adding it later is not a rewrite.

## 5.4 MCP as the emerging agent-facing port

MCP has, within roughly 18 months, become the standard way agents discover and invoke data. Entropy Data ships an MCP server for data product discovery and query with contract enforcement in the path; OpenMetadata exposes MCP; Snowflake and Atlan converge on agent-facing governed semantic context; Gartner predicts 75% of gateway vendors and 10% of iPaaS providers will have MCP features by 2026, and CData reported 28% of the Fortune 500 with production MCP servers in Q1 2025, up from 12% the prior quarter. The emerging mesh pattern: **domain teams expose products as MCP resources with stable capability contracts; agents discover them via catalogues and invoke via tool calls; access control is enforced at the server layer.** [Medium/High]

**→ Design implication:** **Ship an MCP output port as a first-class, platform-generated port type**, derived automatically from the product's contract (schema, semantics, quality guarantees, access policy). Generated-not-hand-written is the point: every data product gets a governed agent interface for free. This is the clearest reason a 2026 organisation buys a data product platform at all.

---

# 6. Self-serve platform / infrastructure

## 6.1 The IDP pattern applied to data

The mainstream platform-engineering stack in 2026 is **Backstage (or Port) + Crossplane + Argo CD**, with Terraform still dominant in many shops.

- **Backstage** — Spotify-origin OSS portal: Software Catalog, **Scaffolder (golden-path templates)**, TechDocs, plugins including Crossplane resource views. Free but heavy to run and customise.
- **Port** — commercial IDP. Consensus heuristic: *Backstage if you need a catalogue plus self-service UI and have a platform team; Port if you need scorecards and governance and want to move faster.*
- **Crossplane** — platform engineers define **Composite Resource Definitions (XRDs)**: abstract resources like `AppDatabase` that compose lower-level provider resources; developers request the abstraction and Crossplane reconciles. Exactly the shape a data product provisioner needs.
- **Terraform** — the pragmatic alternative; the Data Contract CLI already exports Terraform, a useful bridge.

Data-specific applications are rare but real: **Witboost** (tech adapters, templates), **Open Data Mesh Platform** (Blueprint Service, DPDS), **DataOS** (declarative YAML via CLI/API).

**→ Design implications:** **The data product is a composite resource** — one declarative spec fanning out into storage, compute, schedules, access grants, catalogue entries, quality checks, lineage emitters and ports. Whether the reconciler is Crossplane, Terraform or ours, the *abstraction shape* is settled practice. **Golden-path templates are the self-serve primitive**, not a UI form: `scaffold → PR → CI validates contract → merge → reconcile → provisioned`. **Do not build a portal** — ship a Backstage plugin and a Port blueprint. And prefer **reconciliation loops to one-shot provisioning**: drift between declared and actual state is guaranteed, and the reconciler is what makes the registry authoritative rather than aspirational.

## 6.2 Policy-as-code / federated computational governance

The pattern is settled: **OPA + Rego**, decoupling policy decisions from application logic, evaluated against user attributes plus data product metadata. The best-documented enterprise instance is **UBS's Enterprise Data Mesh**, which defines Rego policies from patterns, **maps ODRL constraints to Rego rules**, and has OPA evaluate them against real-time inputs — supporting policy-based and location-aware access control. Academic work exists on generating Rego automatically from OpenAPI-extended data product descriptions. OPA integrates at provisioning, orchestration, API access and service mesh layers. [Medium confidence]

**→ Design implication:** **Policy-as-code with OPA/Rego, generated from declarative ODRL-flavoured policy intent attached to products and domains, evaluated at two points: deploy time (does this product comply?) and access time (may this principal use this port for this purpose?).** Generating Rego from higher-level intent is essential — asking domain teams to write Rego is how federated governance dies. This is the concrete mechanism for federated computational governance, proven in a tier-1 bank.

---

# 7. Comparison matrices

Legend: **●** strong / native · **◐** partial or requires assembly · **○** absent or out of scope · **—** not applicable

## 7.1 Mesh capability coverage

| Solution | Domain ownership modelling | Data product lifecycle | Contracts | Discovery / catalogue | Lineage | Quality / SLOs | Access control & policy-as-code | Self-serve provisioning | Cross-domain sharing | Observability | Cost attribution | Developer experience |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Nextdata OS** | ● | ● *(claimed)* | ● | ◐ | ◐ | ● *(claimed)* | ● *(claimed)* | ● *(claimed)* | ● | ◐ | ○ | ● *(DSL/CLI/agent)* |
| **Entropy Data** | ● | ◐ *(registry only)* | ● *(bilateral)* | ● | ◐ | ◐ | ◐ *(policy defn)* | ○ | ● *(marketplace)* | ◐ | ○ | ● |
| **Witboost** | ● *(configurable graph)* | ● | ● | ◐ *(publishes out)* | ● | ◐ | ● *(CGP)* | ● *(tech adapters)* | ◐ | ◐ | ○ | ◐ |
| **Blindata / ODM** | ● | ◐ / ● | ● | ● | ● *(SQL lineage)* | ● | ● | ◐ *(Blueprint svc)* | ● *(marketplace)* | ◐ | ○ | ◐ |
| **DataOS** | ● | ● | ◐ | ● | ● | ● | ● | ● | ◐ | ◐ | ○ | ● *(YAML/CLI)* |
| **AWS SM Unified Studio** | ● *(domains/units/projects)* | ◐ | ○ | ● | ◐ | ◐ | ● *(Lake Formation)* | ◐ *(blueprints)* | ● *(x-account)* | ◐ | ● *(account-level)* | ◐ |
| **GCP Dataplex UC** | ● *(lakes/zones)* | ◐ | ○ | ● | ◐ | ● *(DQ rules)* | ● *(IAM)* | ○ | ● *(BQ Sharing)* | ◐ | ◐ *(project-level)* | ◐ |
| **MS Fabric + Purview** | ● *(domains/workspaces)* | ◐ | ○ | ● | ● *(OpenLineage in)* | ● | ● *(labels/policies)* | ◐ | ● *(shortcuts)* | ◐ | ◐ *(capacity — poor)* | ◐ |
| **Snowflake Horizon** | ◐ *(via RBAC/db)* | ◐ | ○ | ● *(marketplace)* | ● | ● | ● | ○ | ● *(zero-copy)* | ● | ● *(credits/tags)* | ◐ |
| **Databricks UC** | ● *(catalog/domain)* | ◐ | ○ | ● | ● | ● *(DQ/monitors)* | ● | ○ | ● *(Delta Sharing)* | ● | ● *(DBU tags)* | ◐ |
| **DataHub** | ● *(domains, data products)* | ○ | ◐ *(ODCS ingest)* | ● | ● *(column-level)* | ◐ | ◐ | ○ | ○ | ◐ | ○ | ◐ |
| **OpenMetadata** | ● *(domains, DPROD)* | ◐ | ● *(ODCS 3.1)* | ● | ● | ● | ◐ | ○ | ○ | ◐ | ○ | ◐ |
| **Collibra / Alation / Atlan** | ● | ◐ | ◐ | ● | ● | ◐ | ◐ | ○ | ◐ *(marketplace)* | ◐ | ○ | ◐ / ● (Atlan) |
| **dbt (Core v2 + Mesh)** | ● *(project per domain)* | ◐ | ● *(model contracts)* | ◐ *(docs/Catalog)* | ● | ● *(tests)* | ○ | ○ | ◐ *(x-project refs)* | ◐ | ○ | ● |
| **SQLMesh** | ● | ◐ | ◐ | ○ | ● *(column-level, auto)* | ● *(audits)* | ○ | ○ | ○ | ◐ | ◐ *(virtual envs cut cost)* | ● |
| **ODCS + DC CLI** | ○ | ○ | ● | ○ | ◐ | ● | ○ | ○ | ○ | ○ | ○ | ● |
| **Delta Sharing** | ○ | ○ | ○ | ◐ | ○ | ○ | ◐ *(recipient-scoped)* | ○ | ● | ○ | ◐ *(egress metering)* | ◐ |
| **Iceberg REST / Polaris** | ◐ *(namespaces)* | ○ | ○ | ◐ | ○ | ○ | ◐ *(RBAC in Polaris)* | ○ | ● *(federation)* | ○ | ○ | ◐ |
| **EDC / Gaia-X / IDS** | ◐ | ○ | ● *(negotiated + ODRL)* | ◐ | ○ | ○ | ● *(sovereignty)* | ○ | ● *(cross-org)* | ○ | ○ | ○ |
| **Backstage + Crossplane** | ● *(catalog model)* | ● *(generic)* | ○ | ● *(software)* | ○ | ○ | ◐ *(via OPA)* | ● | ○ | ◐ | ○ | ● |

**What the matrix says.** Two columns are near-empty across the entire market: **cost attribution** (only cloud-native account/credit tagging, never per-data-product) and **data product lifecycle as a deployable artefact** (only the purpose-built platforms attempt it). Two columns are saturated: **discovery/catalogue** and **lineage**. That is the build/buy line, drawn empirically.

## 7.2 Openness, deployment and cost model

| Solution | Licence | Deployment | Standards adopted | Lock-in risk | Public cost model |
|---|---|---|---|---|---|
| Nextdata OS | Proprietary | SaaS + in-situ kernel | None public | **High** (vendor + runtime) | None public |
| Entropy Data | Proprietary core; MIT connectors/SDK; CE available | SaaS / self-host CE | ODCS, ODPS, MCP | Low–Medium | Per user; read-only free; unlimited products |
| Witboost | Proprietary; Apache-2.0 starter kit | Self-hosted / managed | Publishes to Collibra/OM/Purview | Medium–High | Enterprise, not public |
| Blindata | Proprietary | SaaS / self-host | ODM/DPDS ecosystem | Medium | Not public |
| Open Data Mesh Platform | Apache 2.0 | Self-host microservices | DPDS | Low | Free (self-run cost) |
| DataOS | Proprietary | Self-host / marketplace | Declarative YAML | Medium–High | Not public |
| AWS SMUS / DataZone | Proprietary | Managed | Glue/Iceberg | Medium | Studio free; PAYG catalog; ~$0.40/GB metadata |
| GCP Dataplex UC | Proprietary | Managed | DCAT-ish, Iceberg via BigLake | Medium | PAYG |
| MS Fabric + Purview | Proprietary | Managed | Delta/OneLake, OpenLineage | Medium–High | Capacity (CU) + $0.0165/governed asset/day + DGPU $15/$60/$240 |
| Snowflake Horizon | Proprietary | Managed | Iceberg, OpenLineage (consume) | Medium–High | Credits; egress ~$90–155/TB |
| Databricks UC | Proprietary (UC OSS Apache 2.0) | Managed / self-host OSS | Delta, Iceberg REST (partial), Delta Sharing | Medium–High | DBU + separate cloud infra bill |
| DataHub | Apache 2.0 (+ DataHub Cloud) | Self-host (Kafka/ES/MySQL) | OpenLineage, ODCS ingest | Low | Free + ops (~0.25–1 FTE) |
| OpenMetadata | Apache 2.0 (+ Collate) | Self-host (K8s/PG/ES/Airflow) | DCAT, DPROD, PROV-O, OpenLineage, ODCS 3.1, MCP | Low | Free + ops |
| Collibra / Alation | Proprietary | SaaS | Varies | High | Enterprise; ~$1.27M 3-yr TCO cited; 6–12 mo deploy |
| Atlan | Proprietary | SaaS | Broad | Medium | Enterprise |
| dbt Core v2 / Cloud | Apache 2.0 / commercial | Local + SaaS | Own contract format | Low (Core) / Medium (Cloud) | Core free; Enterprise ~$200–400/seat/mo |
| SQLMesh / Tobiko | Apache 2.0 / commercial | Local + SaaS | — | Low | OSS free |
| ODCS / ODPS / Bitol | Apache 2.0 (LF AI & Data) | Spec | — | None | Free |
| Data Contract CLI | MIT | CLI/library/CI/server | ODCS native, 25+ formats, 18+ sources | None | Free |
| Delta Sharing | Apache 2.0 | Self-host or managed | Own open protocol | Low | Free protocol; egress applies |
| Polaris / Gravitino | Apache 2.0 (ASF) | Self-host / managed | Iceberg REST | Low | Free + ops |
| EDC / Tractus-X | Apache 2.0 (Eclipse) | Self-host connector | IDS, Gaia-X, ODRL | Low | Free + ops |
| Backstage / Crossplane / Argo | Apache 2.0 (CNCF/Linux Fdn) | Self-host K8s | OCI, K8s | Low | Free + ops |

---

# 8. Where the market has converged — and where it genuinely has not

## 8.1 Converged (treat as settled; adopt without debate)

1. **Zero-copy, reference-with-policy sharing over copy-and-pipeline.** OneLake shortcuts, Snowflake sharing, Delta Sharing, Lake Formation grants, BigQuery Sharing — all five independently arrived here.
2. **Short-lived, scoped credential vending** rather than long-lived credentials or a data-proxying gateway. Iceberg REST and Delta Sharing both do it.
3. **Domain as a first-class platform entity with a real isolation boundary** (AWS domain/project, Fabric domain/workspace, Dataplex lake, Snowflake/Databricks catalog).
4. **A publish/subscribe access request workflow** as the interaction model for cross-domain consumption. Universal across DataZone, Fabric/Purview, Snowflake Marketplace, Blindata, Entropy Data.
5. **OpenLineage as the lineage wire format.** Now consumed by Microsoft and Snowflake, not just OSS catalogues.
6. **ODCS as the data contract format.** Its competitor formally deprecated itself.
7. **Declarative spec + reconciliation + golden-path templates** as the provisioning model.
8. **OPA/Rego for policy-as-code.**
9. **Table format duality is over**: Iceberg is the interop format; engines converge on Iceberg REST.

## 8.2 Genuinely contested (these are real design decisions, not settled questions)

1. **Central control plane vs embedded runtime.** Nextdata explicitly rejects central orchestration for an in-product kernel; everyone else runs a central control plane. This is the deepest unresolved architectural split, and it maps directly to whether you truly decentralise or merely federate a centre.
2. **Is a data product a *deployable artefact* or a *catalogue entry over existing assets*?** Nextdata, Witboost, ODM and DataOS say artefact. Every cloud vendor and every catalogue says entry. This determines whether your platform provisions or merely describes — and it is the most consequential choice in the whole design.
3. **Contract semantics: producer-published spec (ODCS v3) vs bilateral negotiated agreement (Entropy Data, IDS/EDC).** Different lifecycle, different enforcement, different politics. Both are defensible.
4. **Where policy is enforced**: at the storage/catalogue layer (Lake Formation, Unity Catalog, Polaris RBAC), at a query gateway (Starburst, Denodo, Immuta-style), or at the consumer's connector (EDC/dataspace sovereignty). No convergence.
5. **Descriptor standard** — four competitors, two sharing an acronym.
6. **Domain compute autonomy vs platform-provided compute.** Snowflake/Databricks/Fabric centralise compute (and thus cost and noisy-neighbour effects); the mesh purist position wants domain-owned compute. Nobody has a clean answer that is also economical.
7. **Contract versioning: human-asserted semver vs computed change impact.** SQLMesh computes it from column-level lineage; everyone else asks a human. Computed is clearly better and clearly rarer.
8. **Business catalogue and technical catalogue: one system or two?** OpenMetadata/DataHub say one; the Iceberg REST/Polaris world says the technical catalogue is a separate, lower layer. Both work; the seam is awkward either way.

---

# 9. Recurring pain points nobody has solved well

These are the opportunities. Each is corroborated across multiple independent sources.

1. **Cost attribution per data product.** No surveyed product makes cost a first-class property of a data product. FinOps guidance exists (tag warehouses consistently, showback before chargeback, split shared platform costs proportionally) but it is bolted on after the fact, and shared-cost allocation for centralised platforms is repeatedly named as the top friction point. Meanwhile the mesh literature's whole political argument — domains own their data — collapses without domains owning their data *economics*. **Nobody ships "this data product cost £X last month, £Y of which was consumption by domain Z."**
2. **Runtime contract enforcement.** The market describes contracts beautifully and enforces them barely. Multiple independent sources note that few tools address continuous validation against live data with the power to block non-compliant data from moving downstream.
3. **Catalogue staleness / shelfware.** The near-universal failure mode. Catalogues describe a world they do not control, so they drift, so nobody trusts them, so nobody uses them, so they drift faster. Atlan's own positioning concedes it.
4. **Cross-domain change management.** When a producer changes a data product, no mainstream tool reliably answers "who breaks, and how badly?" SQLMesh comes closest; dbt contracts block a build but do not model downstream consumer expectations across projects with any richness; catalogues show lineage but not *impact*.
5. **Operational burden of the self-hosted OSS stack.** Consistently ~0.25–1 FTE for a catalogue alone, before anything else. Connector repair is the recurring cost, not installation.
6. **Cross-vendor federation.** Every platform is excellent inside its perimeter. Real enterprises have Snowflake *and* Databricks *and* Fabric *and* Postgres. Gravitino addresses discovery but explicitly not enforcement; Polaris federates Iceberg catalogues only.
7. **BI-layer lineage.** OpenLineage has no native Looker/Tableau/Power BI emitters. The last mile to actual business consumption is dark, which is precisely the mile that matters for impact analysis and for proving value.
8. **The incentive problem.** Domain teams are handed end-to-end data product responsibility that is "rarely compensated and usually benefits other domains". Duplicated pipelines and inconsistent domain-level transformations follow. Organisations create "data domains" that are lip service. Tooling cannot fix incentives — but tooling that *lowers the marginal cost of doing it right to near zero* and *makes the value visible* is the only lever available.
9. **Connector/ingestion reliability at scale.** OOMs, upgrade breakage, auth failures, plugin incompatibility — the dominant theme in both DataHub's and OpenMetadata's issue trackers.
10. **Multi-engine catalogue neutrality.** UC OSS can't reliably do local Spark writes or Trino access; UC doesn't do standard Iceberg credential vending; Gravitino adds latency and no enforcement. There is still no boring, neutral, works-everywhere catalogue.

---

# 10. Build vs buy vs assemble

**Adopt as-is (never build):**
- **ODCS v3.1** for contracts, and **Data Contract CLI** as the lint/test engine. Free, MIT, 25+ format converters, 18+ source connectors.
- **OpenLineage** for lineage events. Marquez as an optional default backend.
- **Apache Iceberg + Iceberg REST** as the table/catalogue substrate; **Polaris** where a catalogue is needed.
- **OPA/Rego** as the policy decision engine.
- **Terraform and/or Crossplane** as the reconciliation engine; **Argo CD** for GitOps.
- **Delta Sharing** as an output port type for cross-boundary sharing.
- **Soda / GX Core / dbt tests** as quality check executors.
- **Backstage plugin + Port blueprint** as portal integrations.
- **MCP** as the agent-facing port protocol.
- **DCAT / DPROD** as the semantic-web export format.

**Integrate (buy or connect to the incumbent):**
- Business catalogue: DataHub, OpenMetadata, Atlan, Collibra, Purview, Snowflake Horizon — publish into whichever the org has.
- Warehouse/lakehouse compute: Snowflake, Databricks, BigQuery, Fabric, Trino.
- Observability at scale: Monte Carlo / Anomalo for orgs that need ML anomaly detection.
- Transformation: dbt Core v2 and SQLMesh as peer, first-class supported engines.
- Streaming contracts: Confluent Schema Registry data contracts + AsyncAPI for event-shaped products.

**Build (this is where our value is):**
1. **The data product registry as the authoritative, provisioning-backed source of truth** — versioned, machine-readable, with stable IDs, and *upstream of* every catalogue rather than a competitor to them.
2. **The data product as a deployable composite artefact**, with a provisioner/adapter architecture (Witboost's tech-adapter pattern, on open primitives).
3. **Bilateral contract lifecycle** — request → negotiate → approve → active → deprecate → revoke — layered over ODCS's producer-published semantics.
4. **Computed change-impact classification** (breaking/non-breaking) from column-level lineage against *live consumer contracts*, gating merges.
5. **Fail-closed runtime enforcement** — a contract violation blocks promotion of the output port version, not just an alert.
6. **Policy compilation** — from declarative, ODRL-flavoured policy intent attached to products/domains, down to Rego, evaluated at deploy time and access time.
7. **Per-data-product cost attribution and unit economics**, including consumption attributed to consuming domains.
8. **Generated ports** — SQL/table, Iceberg, Delta Sharing, file, event and **MCP** ports all generated from one contract, with policy in the path.

**Explicitly do not build:** a catalogue UI, a lineage format, a query engine, a table format, a contract schema, an orchestrator, a developer portal, a data quality check DSL, an ML anomaly detector.

---

# 11. Gap analysis — where a new implementation can genuinely be better

1. **Be the *provisioning-backed* registry.** The catalogue market's central failure is describing a world it does not control. If the only supported path to a production data product runs through our declarative spec and our reconciler, our metadata is correct by construction. Nobody in the catalogue market can claim that; only the purpose-built platforms attempt it, and they are either closed (Nextdata, DataOS), enterprise-heavy (Witboost) or under-resourced (ODM).
2. **Own the layer above the perimeter wars.** Every cloud vendor stops at its own boundary. A neutral control plane that models domains and data products across Snowflake *and* Databricks *and* Fabric *and* Postgres, provisioning into each via adapters and enforcing one policy model across all, is a position no incumbent can occupy without abandoning their own lock-in.
3. **Make cost a first-class data product attribute.** This is the widest open gap and the one most tied to whether a mesh survives politically. Storage + compute + egress + consumption-by-consumer, attributed per product, exposed in the marketplace next to the SLOs. It converts "who owns this?" from an org-chart question into an economic one.
4. **Fail-closed contracts.** dbt proved the semantics people want — a contract violation stops the build. Generalise it: a violation stops the *promotion of the port*, so consumers never see bad data, with an explicit break-glass. That is a meaningfully different product from an alerting tool.
5. **Computed, not asserted, change impact.** Merge-time answer to "what breaks and for whom", derived from column-level lineage against active consumer contracts. SQLMesh has the technique; nobody has applied it across *organisational* boundaries with contracts as the consumer registry.
6. **Contract as bilateral agreement with lifecycle.** ODCS gives the schema; the *negotiation, approval, deprecation and revocation lifecycle* is where the governance actually lives, and only Entropy Data and the dataspace world model it.
7. **Generated ports, especially MCP.** Every data product gets SQL, Iceberg, Delta Sharing, event and agent interfaces generated from one contract, with policy enforced in the path. This is the 2026 buying reason.
8. **Radically low operational footprint.** Given that a self-hosted catalogue costs 0.25–1 FTE, a platform that runs as a small stateless control plane plus a Postgres and delegates all heavy lifting is differentiated on TCO alone.
9. **Low obligation floor for domain teams.** Every failure narrative traces to domain teams being handed unfunded work. The design target should be: *publishing a data product costs a domain team one YAML file and a CI run, and everything else — catalogue entry, lineage, access workflow, policy enforcement, quality gates, agent port, cost report — happens automatically.* If it costs more than that, adoption fails regardless of architecture.

---

# 12. Integration surface — day one requirements to be viable

**Standards (must speak natively):**
- ODCS v3.1 (contracts) — import and export, canonical.
- OpenLineage (lineage events) — emit; optionally consume.
- Iceberg REST catalog API — client and, where we act as a catalogue façade, server.
- ODRL (policy vocabulary) → compiled to Rego for OPA.
- DCAT / DPROD — export for catalogue and semantic interop.
- Bitol ODPS and/or the ODP Specification — export for product metadata.
- MCP — server implementation for agent-facing ports.
- AsyncAPI + JSON Schema/Avro/Protobuf — for event-shaped products.
- OCI images + Kubernetes CRDs — packaging and reconciliation.

**Platforms (must have working adapters at v1):**
- Snowflake, Databricks (Unity Catalog), BigQuery, Microsoft Fabric/OneLake, AWS Glue + Lake Formation + S3, Postgres, Kafka/Confluent, Trino/Starburst.

**Tooling (must integrate, not replace):**
- dbt Core v2 (manifest, contracts, tests) and SQLMesh.
- Data Contract CLI (contract lint + test execution).
- Soda Core, GX Core, dbt tests, Elementary (quality executors).
- DataHub, OpenMetadata, Atlan, Collibra, Microsoft Purview, Snowflake Horizon (catalogue publish targets).
- Backstage plugin; Port blueprint.
- Terraform provider and/or Crossplane XRDs; Argo CD-friendly GitOps.
- OPA (policy decision point).
- Airflow / Dagster / Argo Workflows (orchestration hand-off; we do not orchestrate).
- Git + GitHub/GitLab CI as the primary control surface.
- OIDC/SCIM for identity; cloud IAM for enforcement delegation.

**Roadmap-but-not-day-one:** Eclipse Dataspace Connector / IDS output port for cross-organisational sharing; Gravitino federation for brownfield multi-catalogue estates; Delta Sharing server hosting.

---

# 13. Consolidated design implications

**Borrow:**
- Data product as a **versioned, deployable container/artefact** carrying code, contract, policy, quality and observability config (Nextdata).
- **Thin embedded runtime + registry, not central orchestrator** (Nextdata).
- **Tech-adapter / provisioner-per-technology microservice architecture** with a stable provisioning interface (Witboost).
- **Configurable entity metamodel** rather than a hardcoded ontology (Witboost Practice Shaper).
- **Port taxonomy**: input / output / discovery / observability / control (DPDS).
- **Bilateral contract lifecycle via access request** (Entropy Data; IDS/EDC).
- **Marketplace UX**: browse → understand → request → approved → usable with no consumer pipeline work (Snowflake Internal Marketplace).
- **Zero-copy reference with policy inheritance** (OneLake shortcuts).
- **Short-lived scoped credential vending** (Delta Sharing, Iceberg REST).
- **Contract failure blocks the build** (dbt model contracts).
- **Computed breaking/non-breaking change classification from column-level lineage** (SQLMesh).
- **ODRL → Rego → OPA** policy compilation at deploy time and access time (UBS).
- **Golden-path templates + declarative spec + reconciliation** (Crossplane/Backstage).
- **Pricing/ceremony philosophy: never tax the creation of a data product** (Entropy Data; AWS removing DataZone per-user fees).

**Integrate rather than build:** ODCS, Data Contract CLI, OpenLineage, Iceberg REST/Polaris, OPA, Terraform/Crossplane/Argo, Delta Sharing, Soda/GX/dbt tests, dbt and SQLMesh, Backstage/Port, existing catalogues, existing warehouses, MCP.

**Avoid:**
- Building a catalogue, a lineage format, a contract schema, an orchestrator, a query engine or a portal.
- Any architecture that requires a **shared throttled resource pool** across domains without per-domain quota and attribution (Fabric capacity lesson).
- **Read-only-only federation** as the cross-domain answer (Lakehouse Federation lesson) — it is an ad hoc tool, not a serving path.
- Heavy self-hosted dependency chains (Kafka + Elasticsearch + Airflow) that impose 0.25–1 FTE on adopters.
- Coupling our model to a vendor's entity names that have already been renamed twice (DataZone → SageMaker Catalog; Dataplex → Dataplex Universal Catalog).
- Forking or extending ODCS's schema outside `customProperties`.
- Leading with a UI. Declarative spec + CLI/API first; UI as projection.
- Using the phrase "data mesh" as the primary market framing. Gartner has it "obsolete before plateau"; Data Mesh Manager renamed itself away from it inside two years; even Thoughtworks now frames it as maturity rather than movement.

**Differentiate on:** provisioning-backed authoritative registry · cross-vendor neutrality · per-data-product cost attribution · fail-closed runtime contract enforcement · computed change impact · generated ports including MCP · a near-zero obligation floor for domain teams.

---

# 14. Open questions for the design

1. **Artefact or entry?** Do we provision data products (and therefore own a reconciliation loop, adapters and drift management), or do we describe them over assets others provision? This is the fork in the road. The market's failures argue strongly for *provision*, but it is 5–10× the engineering.
2. **Central control plane or embedded runtime?** If we take Nextdata's embedded-kernel position, what exactly runs in-product, and how do we distribute policy and collect telemetry without a control plane becoming one by the back door?
3. **Where does policy get enforced** when the org spans Snowflake + Databricks + Fabric + Postgres? Compile down to each platform's native enforcement (best performance, worst consistency), or interpose a gateway (best consistency, worst performance and worst adoption)? Or both, per port type?
4. **Do we run any data plane at all?** Delta Sharing's model (never proxy bytes; vend credentials) argues strongly for no. But then how do we enforce fail-closed contracts at read time rather than only at publish time?
5. **Which descriptor is canonical internally?** Our own model with exporters (flexible, more work, risk of divergence) vs adopting Bitol ODPS or DPROD directly (interop for free, constrained by someone else's roadmap)?
6. **How do we get cost data?** Snowflake credits and tags, Databricks DBU tags plus a separate cloud bill (50–70% of cost is outside the DBU meter), Fabric CU, AWS account-level, GCP project-level — five incompatible sources, none of which natively knows what a "data product" is. What is the minimum viable attribution model, and how honest can it be about shared costs?
7. **What is the consumer-side obligation?** Do consumers declare their expectations (enabling computed change impact and real bilateral contracts) at the cost of friction, or do we infer usage from query logs (frictionless but weaker guarantees)? Probably both — but which is authoritative?
8. **How much do we bet on dbt** now that dbt, Fivetran and GX Core stewardship sit under one owner? What is the concentration-risk hedge?
9. **Multi-tenancy and blast radius:** if the registry is the only path to production, it is a hard dependency for every domain's deploys. What is the availability target and the degraded mode?
10. **Do we ship an org-design opinion?** Every failure narrative is socio-technical. Do we encode a maturity model, staffing expectations and a "fitness function" set (à la datamesh-architecture.com) as product surface, or stay purely technical and let it fail the same way?
11. **Cross-organisational sharing:** is EDC/IDS a real day-two requirement for our target adopters, or European-manufacturing-specific? The answer changes the policy model materially.
12. **Naming and positioning:** given the market's flight from the term, what do we call this?

---

# 15. Sources

Accessed August 2026. † = read directly as a primary source; unmarked entries were reached via search retrieval/summarisation (see *Method and confidence*).

## Purpose-built data mesh / data product platforms
- **Nextdata, "Introducing Nextdata OS"** — nextdata.com/our-pov — Apr 2025. Founder's own statement of the container + poly-compute kernel model.
- **Nextdata OS launch coverage** — GlobeNewswire, SiliconANGLE, BigDATAwire — 8–16 Apr 2025. Confirms launch date, positioning, $12M seed (Greycroft, Acrew).
- **theCUBE Research, "Nextdata OS and the Promise of Autonomous Data Products"** — 2025. Prompted Dehghani's public "gentle correction"; shows "autonomous" is contested.
- **Nextdata company profiles** — PeerSpot, GoodFirms, 7wData — 2025–26. Only public signal on commercial maturity (pre/early revenue, single product).
- †**Entropy Data GitHub org** — github.com/entropy-data — commits to Aug 2026. What is MIT vs proprietary; connectors, SDK, CLI, `dataproduct-mcp`.
- **"Data Mesh Manager is now Entropy Data"** — entropy-data.com — 6 Oct 2025. The rebrand away from "data mesh" — a market signal in itself.
- **Entropy Data docs** — docs.datamesh-manager.com — 2026. Entity model, bilateral contract workflow, ODCS/ODPS support.
- **Data Mesh Manager pricing** — GetApp/Capterra/Software Advice — 2026. Per-user, read-only free, unlimited data products.
- **Thoughtworks Technology Radar: Data Mesh Manager; DataOS** — independent evidence of real-world use.
- †**datamesh-architecture.com repo** — github.com/datamesh-architecture — ~88★, ~796 commits. Canvases, fitness tests, per-stack guides.
- **datamesh-architecture.com — dbt + Snowflake stack** — the low-tech baseline our design must beat (tagged models, macros for policy).
- †**Witboost Starter Kit** — github.com/agile-lab-dev/witboost-starter-kit — Apache 2.0, 30+ tech adapters. The provisioner-per-technology pattern.
- †**Witboost Practice Shaper presets** — github.com/agile-lab-dev/witboost-practice-shaper-presets. The configurable property-graph metamodel.
- **Blindata product docs** — blindata.io — 2024–26. Data products as catalogue citizens, marketplace, computational governance policies.
- **Open Data Mesh Initiative / Quantyca — ODM Platform & DPDS** — github.com/opendatamesh-initiative, platform.opendatamesh.org — OSS since 2023. Port-oriented descriptor, Blueprint bootstrap.
- **The Modern Data Company — DataOS** — themoderndatacompany.com; Blocks & Files, Jul 2026. Declarative YAML data products.
- **Starburst, "Data Mesh: What Happened?"** and Starburst Data Products — candid retrospective plus an example of federation marketed as mesh.

## Cloud vendor offerings
- **AWS: SageMaker Unified Studio GA (Mar 2025); DataZone→SageMaker domain upgrade (Jun 2025)** — aws.amazon.com/about-aws/whats-new. GA dates and the renaming risk.
- **AWS: "DataZone removes the user-level subscription fee"** — Nov 2024. Shift to pay-as-you-go; mesh-friendly cost model.
- **AWS Well-Architected Analytics Lens — Data mesh; Lake Formation cross-account/cross-region docs; re:Post cross-account consumption failures** — reference architecture plus real operational friction.
- †**Google Cloud, "Build a data mesh on Google Cloud with Dataplex (GA)"** — 23 Feb 2022. Lake/zone/asset → domain/product mapping; states no limitations, which is itself a tell.
- **Dataplex Universal Catalog intro; BigQuery Sharing (ex-Analytics Hub)** — current catalogue and exchange model.
- **Microsoft Learn: Fabric Domains; Fabric governance overview; Purview Unified Catalog governance domains** — hierarchy and Purview data products.
- **Microsoft Purview data governance GA pricing** — effective 6 Jan 2025. $0.0165/governed asset/day; DGPU $15/$60/$240.
- **Microsoft Fabric Community forums — capacity throttling, `CapacityLimitExceeded`, metrics-app breakage, 100%-while-paused bug** — 2025–26. First-party evidence of the shared-capacity anti-pattern.
- **MS Tech Community, "Establishing Data Mesh with Domains and OneLake"** — the shortcut-based zero-copy sharing model.
- **Snowflake: Internal Marketplace; Snowflake for Data Mesh; Horizon Catalog press release; Summit 2026 recaps (Flexera, Select.dev, Atlan, Sanjeev Mohan)** — 2025–26. Data products with SLOs, Horizon Context consuming OpenLineage, Semantic View Autopilot GA Feb 2026.
- **Snowflake Egress Cost Optimizer docs/blog** — cross-cloud egress ~$90–155/TB.
- †**Delta Sharing repo** — github.com/delta-io/delta-sharing — Apache 2.0, ~956★. Pre-signed-URL credential model, client matrix, and the repo's own caveat that the reference server is not a complete secure implementation.
- †**Unity Catalog OSS repo + issues** — github.com/unitycatalog/unitycatalog — ~3.5k★, ~294 open issues (#43, #433, #560, #715, #740, #805, #966). Concrete adoption blockers.
- **Databricks, "Open sourcing Unity Catalog" (Jun 2024); Unity Catalog limitations (Atlan); Delta Sharing / Lakehouse Federation read-only limits** — the perimeter problem.
- **Databricks pricing guides** — Mammoth, CloudForecast, OWOX, 2026. DBU + separate infra bill; Standard tier retirement.

## Catalogue / governance / metadata
- †**DataHub repo** — github.com/datahub-project/datahub — Apache 2.0, ~12.5k★. Streaming-first architecture, GMS, 80+ connectors, domains/data products.
- †**DataHub open issues by comment count** — OOM on ingestion, connection reliability, OIDC logout, Airflow 2.7+ breakage, Oracle post-upgrade failures.
- †**OpenMetadata repo** — github.com/open-metadata/OpenMetadata — Apache 2.0, ~14.9k★, v1.12.x. 700+ JSON Schemas; strongest standards alignment (DCAT/DPROD, PROV-O, OpenLineage, ODCS 3.1, MCP).
- †**OpenMetadata open issues by comment count** — profiler complex-type gaps, connector metadata gaps, `MetaData.reflect()` query storm.
- **Open-source catalogue comparisons and operational burden** — Decube, TheDataGuy, DataHub blog, 2025–26. Source of the 0.25–1 FTE figure.
- **Amundsen repo + status reviews** — last release Aug 2024; evidence the project is dormant.
- **Apache Atlas project pages / alternatives analyses** — still maintained, Hadoop-era design.
- †**OpenLineage repo**; **Marquez column-lineage blog**; **DataHub, "Open Source Data Lineage: Tools and Tradeoffs" (2026)** — emitter/consumer matrix, column-lineage limits, absent BI emitters.
- **Gartner MQ for D&A Governance Platforms 2025 & 2026** (via Collibra and Atlan summaries) — Collibra Leader, Alation Visionary, Atlan Visionary→Leader 2026.
- **G2: Collibra pros/cons; Alation reviews** — 2026. Navigation as top satisfaction barrier, workflow bottlenecks, UI slowness, cost.
- **Castordoc, "Why Most Data Catalogs Fail"; Atlan, "Replace the catalog nobody uses"** — the shelfware failure mode, conceded by a vendor.
- **Consolidation coverage** — TechTarget (Atlassian/Secoda), Snowflake (Select Star), ServiceNow (data.world), Everest Group — 2025. The acquisition wave and its agentic-AI rationale.

## Data product & contract tooling
- †**Bitol ODCS repo** — github.com/bitol-io/open-data-contract-standard — v3.1.0, Apache 2.0, LF AI & Data, ~1.1k★. Section list and media type.
- **Bitol ODCS v3.1.0 announcement; bitol.io** — governance (LF AI & Data, formed Nov 2023) and v3.1 additions.
- †**Data Contract Specification repo** — github.com/datacontract/datacontract-specification — v1.2.1, MIT, ~418★. **Carries the deprecation notice pointing to ODCS**; support to end-2026. The most decision-relevant source here.
- †**Data Contract CLI repo** — github.com/datacontract/datacontract-cli — MIT, ~1.0k★. ODCS-native lint/test, 25+ converters, 18+ source connectors.
- †**Bitol Open Data Product Standard repo** — github.com/bitol-io/open-data-product-standard — v1.0.0, ~115★.
- **opendataproducts.org — ODP Specification v3.0/3.1/4.x** — Linux Foundation. The *other* ODPS; v3.1 references both DCS and ODCS; BASF cited as adopter.
- **OMG DPROD v1.0 beta1 (Feb 2025); EKGF/dprod repo** — a DCAT profile on RDF/OWL/SHACL/PROV.
- **dbt Developer Hub: model governance; About dbt Mesh; Mesh FAQs** — contracts, access modifiers, versions, cross-project refs, Enterprise gating, contract-before-tests enforcement.
- **dbt Core v2 announcement; dbt licensing FAQ; Fivetran/dbt merger (BusinessWire, 13 Oct 2025; completed 1 Jun 2026); Tobiko, "Is dbt Fusion the death of dbt Core?"; Datacoves & Kestra analyses** — the ELv2→Apache 2.0 arc and concentration risk.
- **dbt-loom** — cross-project refs on dbt Core; the OSS escape hatch from Enterprise gating.
- **Tobiko Data: virtual data environments; SQLMesh vs dbt comparisons (2026)** — SQLGlot column-level lineage, breaking/non-breaking classification.
- **Great Expectations community update; DataKitchen, "When the cloud goes dark" (2026)** — FICO acquisition, GX Cloud withdrawn 1 Jun 2026, Fivetran stewardship of GX Core.
- **Data observability comparisons** — Medium/Modern DataTools/Basedash, 2026. Monte Carlo/Anomalo $50k–$200k+/yr; OSS-first guidance by team size.
- **Acceldata, Soda, Tacnode, datadef.io on data contracts** — 2026. The repeated finding that few tools enforce at runtime.
- **Confluent Stream Governance; Schema Registry data contracts; AsyncAPI integration** — write-time contract enforcement for event-shaped products.

## Sharing / interop substrate
- **Apache Polaris Iceberg REST federation docs; Dremio on Polaris; Alex Merced on Iceberg catalogs; AppScale "Iceberg Catalog Wars"; amdatalakehouse "State of Apache Iceberg Catalogs, June 2026"** — REST value, credential vending, Polaris TLP Feb 2026, Gravitino 1.2.0 Mar 2026, and the caveat that the spec omits RBAC/masking/lineage/federation.
- **Datastrato / Apache Gravitino materials** — meta-catalogue model, delegated enforcement, 10–50 ms metadata latency.
- **Eclipse Dataspace Connector project pages; Eclipse Tractus-X Connector Kit; IDSA connector report; Gaia-X Hub Austria whitepaper; doubleSlash Gaia-X vs Catena-X vs IDSA** — sovereignty, ODRL policies, contract negotiation; Nov 2025 Korea–Europe production transaction.
- **arXiv, "Data Product MCP: Chat with your Enterprise Data" (2026); CData on 2026 MCP adoption; Promethium on agentic analytics** — MCP as the agent-facing data product port.

## Self-serve platform, infrastructure, governance-as-code
- **Backstage plugin directory; freeCodeCamp IDP guide (Backstage + Argo CD + Crossplane); ArchitectureDiagram.ai IDP architecture 2026; StackGen Backstage infra plugins** — the settled IDP stack and the Backstage-vs-Port heuristic.
- **Crossplane XRD/composition docs** — the composite-resource abstraction that maps onto a data product.
- **UBS, "Policies as code" (2025)** — enterprise data mesh mapping ODRL constraints to Rego, evaluated by OPA against user attributes + data product metadata.
- **OpenCredo, "Computational Governance in Data Mesh with OPA"; OPA docs; Springer chapter on generating Rego from OpenAPI-driven data product descriptions** — the mechanism and its automation.
- **FinOps Foundation, FinOps for Data Cloud Platforms; Seemore Data on chargeback vs showback; CloudZero; datalakehousehub FinOps for warehouses** — 2026. Tagging discipline, showback-before-chargeback, shared-cost allocation as the named friction point.

## Market-level and critical commentary
- **Thoughtworks, "The state of data mesh in 2026: from hype to hard-won maturity"** — data products and self-serve platforms as commodities; domain ownership as the faked principle.
- **Gartner Hype Cycle ("obsolete before plateau"); Gartner, "How Data Leaders Can Complement Fabric and Mesh"; DataOps.live and DZone rebuttals** — the analyst position and counter-arguments.
- **arXiv 2304.01062, "Data Mesh: A Systematic Gray Literature Review"** — duplication per domain, increased operating cost, cross-domain inconsistency, uncompensated ownership.
- **Jenny Kwan, "Data Mesh: Why It's Not Working"; peterbaumann, "The State of Data Mesh in Practice"** — practitioner critiques.
- **Data Mesh Learning case-study library (Zalando, Saxo Bank, JPMorgan Chase, Adevinta, Roche)** — what large regulated organisations actually built.
- **Data mesh (Wikipedia); O'Reilly *Data Mesh* ch.4** — canonical statement of the undesired consequences to design against.

---

*End of Part 2. Read alongside Part 1 (principles and patterns) before making architectural decisions.*
