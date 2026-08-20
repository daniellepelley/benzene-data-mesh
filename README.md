# Benzene Data Mesh

A corporate data mesh built on the [Benzene](https://www.nuget.org/packages?q=Benzene.Core)
hexagonal (ports-and-adapters) framework for .NET.

Data owners register their data sources (Databricks, SQL databases, Excel
spreadsheets, …) once; the mesh catalogues them, controls who can consume them at a
fine grain, delivers the data through whichever channel the consumer prefers (JSON
API, BI/Business Objects, Excel, …), and meters every consumption event for
chargeback and audit.

## Documentation

| Document | Purpose |
| --- | --- |
| [Top-level design](docs/design/00-top-level-design.md) | System overview, components, domain model, work split |
| [Data mesh principles research](docs/research/data-mesh-principles.md) | General design and principles of a data mesh |
| [Market landscape research](docs/research/market-landscape.md) | Designs of existing data mesh solutions in the market |
| `docs/design/components/` | Detailed component designs (produced in separate design sessions) |

## Status

Top-level design phase. Component designs are scoped in
[section 8 of the top-level design](docs/design/00-top-level-design.md#8-component-design-sessions-work-split)
and will be produced next.
