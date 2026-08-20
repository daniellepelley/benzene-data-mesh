# Data Mesh Platform Landscape: Architecture Research Report

**Prepared for:** Software architecture team designing a corporate data mesh platform
**Date:** 2026-08-20

---

## 1. Scope and Method

This report surveys how commercial and open-source products implement the core capabilities of a data mesh — source connectivity, data product definition, catalog/discovery, fine-grained authorization, consumption channels, and usage metering — and distills the design decisions most relevant to building a custom .NET-based data mesh with Databricks/SQL Server/Excel connectors, a JSON API + BI + Excel consumption layer, federated authorization with an external IdP, an ownership-based catalog, and chargeback-oriented consumption monitoring.

---

## 2. Cloud-Native Platform Offerings

### 2.1 Databricks — Unity Catalog + Delta Sharing

Unity Catalog is a **centralized governance control plane over a distributed compute/data plane**: a metastore governs a three-level namespace (catalog → schema → table/view/volume/model), with storage credentials and external locations held outside the namespace so that governance owns credentials, not users. Data mesh is mapped organizationally: domains become workspaces/catalogs, each with local ownership, while the metastore provides one federated governance layer. Fine-grained access has evolved from grant-based RBAC to **ABAC**: governed tags plus policies attached at catalog/schema level automatically apply row filters and column masks to any matching table — a single tag-driven policy replaces per-table configuration, and table owners cannot override centrally imposed policies.

**Delta Sharing** is the most instructive consumption design in the market: an **open REST protocol** in which the sharing server authenticates a recipient (bearer token or OIDC federation), evaluates entitlements, and returns **short-lived pre-signed cloud-storage URLs** to Parquet/Delta files. The client (pandas, Spark, Power BI, Excel via connector) fetches data directly from storage. This cleanly separates the *authorization plane* (small, cheap, centralized) from the *data transfer plane* (scales with cloud storage), and short-lived URLs eliminate the need for revocation bookkeeping. System billing tables (`system.access.audit`, usage tables) provide per-query, per-principal usage for chargeback.

### 2.2 Snowflake — Secure Data Sharing, Internal Marketplace, Horizon

Snowflake's signature decision is **zero-copy sharing**: a producer account exposes objects to consumer accounts as *listings* with no data movement; consumers pay for their own compute against shared data. The **Internal Marketplace** turns this into a data mesh storefront — domains publish data products as listings with metadata, terms, and usage visibility; **Horizon Catalog** is the governance layer spanning discovery, classification, row-access policies, dynamic data masking, and cross-cloud sharing. For chargeback, Snowflake is best-in-class among warehouses: `QUERY_ATTRIBUTION_HISTORY` provides per-query compute cost, and object tagging + `TAG_REFERENCES`/`ACCOUNT_USAGE` views let finance attribute spend to domains — the standard pattern is tag warehouses and databases by cost center, then join usage telemetry to tags.

### 2.3 Microsoft Fabric / Azure — Domains, OneLake, Purview

Fabric implements data mesh as a first-class construct: **domains** group workspaces by business function, with **delegated (federated) governance** — tenant-level settings can be delegated to domain admins. **OneLake** is a single logical lake ("OneDrive for data") where **shortcuts** provide zero-copy sharing of data products across domains. **Purview** supplies the enterprise catalog: classifications, sensitivity labels, and access policies travel with data as it is shared, even cross-tenant. The notable design tension (relevant to any builder): Fabric has *two* catalogs — the OneLake catalog (technical, in-platform) and Purview Unified Catalog (business/governance, cross-platform) — illustrating that a technical catalog and a governance catalog are distinct concerns that must be deliberately linked.

### 2.4 AWS — DataZone / SageMaker Catalog + Lake Formation

DataZone (now folded into SageMaker Unified Studio/Catalog) is a **pure control plane layered over Lake Formation, Glue Data Catalog, Athena, and Redshift**. Its object model is the clearest producer/consumer workflow in the market: **Domain → Project → Environment**; producers publish assets with metadata, glossary terms, quality and lineage; consumers file a **subscription request**; the data owner approves; then an automated **fulfillment workflow** creates the actual grants (Lake Formation permissions, Redshift datashares, S3 policies). For "unmanaged" assets, fulfillment is delegated: DataZone emits an EventBridge event and lets custom code make the grant — an extensibility escape hatch worth copying. Fine-grained control comes from Lake Formation **LF-Tags** (tag-based access control: grant on `department=sales` rather than on tables) plus row/column-level filters.

### 2.5 Google Cloud — Dataplex Universal Catalog

Dataplex organizes data into **lakes/zones (domains)** logically without moving it, auto-harvests metadata from BigQuery, GCS, Cloud SQL, and Vertex AI, and layers on data quality scans, lineage, and profiling. Access control delegates to **IAM at project/entry-group/entry level**; BigQuery supplies row-level security and policy-tag-based column masking. Its notable decision is **metadata automation**: discovery, classification, and quality signals are generated by managed scanners rather than manual curation.

---

## 3. Catalog / Governance Products

| Product | Architecture | Ingestion model | Distinctives |
|---|---|---|---|
| **Collibra** | Workflow-centric governance suite | Connector harvesting | BPMN-style stewardship workflows, approval chains, policy management for regulated industries; governance-as-business-process |
| **Alation** | Catalog + behavioral intelligence | Connectors + **query log parsing** | Mines query logs to rank popularity, infer usage, and drive search relevance; strong steward curation |
| **Atlan** | SaaS on open components (Postgres, Kafka, Elasticsearch; Atlas-derived metastore) | 120+ connectors, dbt-native | "Active metadata," embedded collaboration, automatic column-level lineage, fast deployment |
| **DataHub** (OSS, LinkedIn) | **Event-sourced, stream-first**: GMS service, MySQL (source of truth), Elasticsearch (search), graph index; Kafka for Metadata Change Proposals/Events | Push *and* pull; async via Kafka | Extensible **aspect-based metadata model**; consumers subscribe to metadata change streams to build reactive governance |
| **OpenMetadata** | Unified schema-first metadata standard; MySQL/Postgres + Elasticsearch; **deliberately no graph DB** | Pull-based via Airflow-run connectors | All entities defined as JSON Schema; everything through one REST API; simplicity as a design goal |
| **Amundsen** (Lyft) | Microservices: Neo4j (graph), Elasticsearch (search), Flask front/metadata services | Pull via databuilder library | Search-first discovery, page-rank-style popularity; lighter governance features |

Two architectural camps are visible: **event-sourced/streaming metadata** (DataHub — best when you need near-real-time reactions to metadata change) versus **CRUD + standardized schemas** (OpenMetadata — best for simplicity and a clean API surface). Both expose the pattern most worth copying: *metadata as a typed, versioned entity model behind a single API*, with search and lineage as derived indexes.

---

## 4. Data-Mesh-Native and Data Product Platforms

### 4.1 Nextdata OS (Zhamak Dehghani)

Nextdata OS (launched April 2025) operationalizes the original data mesh vision as **autonomous data product containers**: long-running, environment-aware units that package data references, transformation code, quality guarantees, **embedded computational policies**, and APIs together. Governance is not a separate catalog pass — policies are packaged *inside* the product and enforced continuously. Products expose standard APIs for discovery and interoperability, and discovery is dynamic (products register themselves). The design thesis: the data product, not the table or pipeline, is the unit of architecture, deployment, and governance — a "contract-first + policy-in-the-artifact" model.

### 4.2 Starburst (Trino / Galaxy)

Starburst approaches the mesh through **federated query**: Trino queries data where it lives (lakehouse, RDBMS, NoSQL) with no mandatory movement. **Data products** are a first-class construct: a curated schema of views/materialized views over federated sources, with ownership, documentation, and access policies, published through a catalog UI and versioned/reviewed like software. Fine-grained control is notable for being **pluggable**: built-in RBAC, or externalized to **Open Policy Agent** (policy-as-code authorization for catalogs/schemas/tables/columns), Apache Ranger, or Immuta. This "authorization as a replaceable policy engine consulted per query" is the pattern most transferable to a custom platform.

### 4.3 Denodo (data virtualization) — closest analog to the target build

Denodo builds a **logical/semantic layer** over heterogeneous sources: source connectors → normalized base views → layered derived views → published data products. Consumption is multi-modal from a single model with **zero code**: JDBC/ODBC/ADO.NET, REST, **OData 4.0**, and GraphQL endpoints — with OAuth2/Kerberos on all interfaces and **fine-grained row/column privileges enforced once, in the virtual layer, regardless of channel**. This "define once, expose everywhere, authorize centrally" design maps almost one-to-one to the target requirements (JSON API + Business Objects + Excel), since Excel and BI tools consume OData/ODBC while apps consume REST/GraphQL. Caveat learned from virtualization deployments: query pushdown and caching are essential or the virtual layer becomes the bottleneck.

### 4.4 K2view

K2view inverts the modeling grain: data products are **business entities** (a customer, an order), each materialized into its own encrypted "Micro-Database" assembled from all source systems. It excels at operational/real-time use (customer 360, test data) rather than analytical mesh — a reminder that "data product" can mean entity-grain APIs, not just dataset-grain tables. (Agile Data Engine, by contrast, is a warehouse-automation/DataOps tool — adjacent, but not a mesh platform.)

---

## 5. Cross-Cutting Patterns

1. **Control plane / data plane separation.** Every successful design (DataZone over Lake Formation, Unity Catalog over cloud storage, Delta Sharing's token service over pre-signed URLs) keeps the governance/catalog/authorization plane small and centralized while data flows through a separate, horizontally scalable path.
2. **Publish → discover → subscribe → approve → fulfill.** DataZone, Snowflake's internal marketplace, and Starburst all converge on the same marketplace workflow, with automated *grant fulfillment* after human (or policy-based) approval, plus an event-based escape hatch for assets the platform can't grant itself.
3. **Tag/attribute-based authorization over per-object grants.** Unity Catalog ABAC, LF-Tags, Snowflake tags + policies, Purview sensitivity labels: classification-driven policies scale where per-table ACLs collapse. Policies attach high in the hierarchy and cannot be overridden by asset owners.
4. **Policy-as-code / externalized decision points.** Trino+OPA is the purest form; Nextdata embeds policies in the product artifact. Authorization is a policy engine consulted at query time, not logic scattered through services.
5. **Contract-first data products.** Emerging standards — **ODCS** (Open Data Contract Standard, Linux Foundation/Bitol, machine-readable YAML: schema, SLOs, quality rules, ownership) and **ODPS** (Open Data Product Specification) — decouple the product's promise from its implementation.
6. **Zero-copy sharing where possible; open protocols where not.** Snowflake shares, OneLake shortcuts, and Delta Sharing all avoid ETL replication; Delta Sharing shows how to do it across trust boundaries with an open REST protocol.
7. **Metadata as an event stream + derived indexes.** DataHub's model — a typed entity/aspect store as source of truth, Kafka change events, Elasticsearch/graph as projections — is the reference architecture for reactive catalogs and automated lineage.
8. **Usage telemetry as a first-class governance output.** Per-query attribution joined to tags/cost centers (Snowflake), system billing tables (Databricks), and query-log mining for popularity (Alation) all treat consumption logs as the substrate for both chargeback and discovery ranking.

---

## 6. Recommendations for the .NET Data Mesh Build

**Architecture shape.** Build a thin **control plane** (catalog, contracts, authorization, subscription workflow, metering) that is strictly separated from the **data plane** (connector execution and result delivery). Never let raw data transit the control plane; where feasible, hand consumers a governed, short-lived path to data (the Delta Sharing pre-signed-URL idea generalizes: issue short-lived, scoped result tokens/links rather than long-lived credentials).

**Data product definition.** Adopt **contract-first products** using ODCS-style YAML/JSON: schema, owner, domain, SLOs, quality rules, classification tags, and allowed consumption channels — versioned in git, validated in CI, and registered in the catalog on deploy. The product (not the connector or the table) is the unit of ownership, discovery, authorization, and billing. Follow Starburst's practice of treating product changes like software releases (review + versioning + deprecation policy).

**Connectors (Databricks, SQL Server, Excel).** Follow OpenMetadata/DataHub: a connector SDK with a standard interface (capabilities, schema extraction, incremental read, health check), configuration as declarative recipes, and connectors that emit both *data* and *metadata/lineage events* to the catalog. For Databricks, prefer consuming via Delta Sharing or SQL warehouse endpoints so Unity Catalog's own governance stays authoritative upstream. Treat Excel sources as first-class but low-trust: enforce contract validation (schema + quality rules) at ingestion, since they lack a governing system of record.

**Catalog.** Model it as a typed entity store (products, domains, owners, contracts, sources, subscriptions) behind one API — Postgres/SQL Server as source of truth, a search index as a projection, and change events on a bus so lineage, notifications, and metering can react (DataHub's pattern, simplified per OpenMetadata's "no graph DB unless needed" lesson). Assign every product an accountable owner and steward from day one; ownership is the field that makes federation work.

**Authorization.** Externalize decisions into a **central PDP (policy decision point)** — OPA or a .NET equivalent — consulted by every consumption channel with `(principal, product, action, context)`. Use **attribute/tag-based policies** (domain, sensitivity, purpose) rather than per-object ACLs, with row filters and column masks compiled into queries at the data plane. Authenticate exclusively via the **external IdP (OIDC/OAuth2)**; map IdP groups/claims to platform roles, and use on-behalf-of token flows so BI and Excel access carries end-user identity, not a service account — otherwise fine-grained policy and chargeback both break. Copy DataZone's **subscription → owner approval → automated grant fulfillment** workflow (owners approve; the platform, not humans, writes the grants), with an event hook for grants the platform can't automate.

**Consumption layer.** Follow Denodo: one semantic definition per product, many protocol facades — JSON REST for applications, **OData** (which Excel and many BI tools, including SAP Business Objects via generic connectors, consume natively) for self-service, plus ODBC/JDBC or a query-passthrough path for heavy BI. Enforce authorization in one shared enforcement layer beneath all facades so every channel gets identical row/column-level results. Plan pushdown/caching early — a naive virtualization layer becomes the bottleneck.

**Metering and chargeback.** Emit a **usage event per request** at the enforcement layer (principal, product, channel, rows/bytes, duration, upstream compute cost where retrievable — e.g., Databricks system tables), tagged with the consumer's cost center from the IdP/catalog. Start with **showback** dashboards per domain and product before enforcing chargeback (the consistent industry lesson: attribution granularity and trust must precede billing). Usage data doubles as discovery input — popularity ranking à la Alation.

**Adoption.** Start with one pilot domain and one high-value product end-to-end (Zalando/Adevinta lesson); federated governance succeeds only when the paved-road platform is easier than going around it.

---

## Key Sources

- Databricks Unity Catalog ABAC: https://www.databricks.com/blog/abac-row-filtering-and-column-masking-policies-governed-tags-and-data-classification-are-now
- Delta Sharing protocol spec: https://github.com/delta-io/delta-sharing/blob/main/PROTOCOL.md ; VLDB paper: https://www.vldb.org/pvldb/vol18/p5197-puttaswamy.pdf
- Snowflake data mesh / internal marketplace: https://www.snowflake.com/en/blog/data-mesh-internal-marketplace-demo-recap/ ; cost attribution: https://docs.snowflake.com/en/user-guide/cost-attributing
- Microsoft Fabric domains: https://learn.microsoft.com/en-us/fabric/governance/domains ; Purview + Fabric: https://learn.microsoft.com/en-us/fabric/governance/microsoft-purview-fabric
- Amazon DataZone projects/environments & subscriptions: https://docs.aws.amazon.com/datazone/latest/userguide/working-with-projects.html ; custom subscription fulfillment: https://aws.amazon.com/blogs/big-data/implement-a-custom-subscription-workflow-for-unmanaged-amazon-s3-assets-published-with-amazon-datazone/
- Google Dataplex data mesh guide: https://cloud.google.com/dataplex/docs/build-a-data-mesh
- DataHub architecture: https://docs.datahub.com/docs/architecture/architecture ; OpenMetadata vs Amundsen: https://atlan.com/openmetadata-vs-amundsen/
- Nextdata OS launch: https://www.nextdata.com/our-pov/introducing-nextdata-os-autonomous-data-products-for-the-autonomous-era ; https://siliconangle.com/2025/04/08/exclusive-data-mesh-creator-brings-first-product-market/
- Starburst data products: https://www.starburst.io/solutions/data-products/ ; Trino OPA access control: https://trino.io/docs/current/security/opa-access-control.html
- Denodo consumption endpoints: https://community.denodo.com/tutorials/browse/dataservices/6generic_endpoints ; data mesh design guidelines: https://community.denodo.com/kb/en/view/document/Denodo%20Design%20Guidelines%20for%20Data%20Mesh%20and%20Decentralized%20Data%20Organizations
- K2view data mesh architecture: https://www.k2view.com/platform/data-mesh-architecture/
- Open Data Contract Standard (ODCS): https://bitol-io.github.io/open-data-contract-standard/v3.1.0/ ; https://datacontract-specification.com/
- Adevinta data mesh journey: https://adevinta.com/techblog/from-lakehouse-architecture-to-data-mesh/ ; Zalando case study: https://datameshlearning.com/case-study/zalando/
