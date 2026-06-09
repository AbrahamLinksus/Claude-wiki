# Data Storage Hierarchy

**Summary**: Describes the internal data storage hierarchy from operational databases up to Data Lakes, including key comparisons between Data Warehouses, Data Marts, and Data Lakes, plus open data provider resources.

**Sources**: `Unit I- Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Internal Data Stores (Hierarchy)

```
Databases  →  Data Marts  →  Data Warehouse  →  Data Lake
(operational)   (dept-level)    (enterprise)       (raw everything)
```

### Databases
Operational systems (OLTP) storing day-to-day transactional data. Fast writes, row-oriented, normalized schema.

### Data Mart (DM)
- Subject-specific subset of a DWH
- **Size**: < 100 GB
- **Users**: specific business group or department
- Built from the DWH (dependent) or directly from source systems (independent)

### Data Warehouse (DWH)
- Centralized enterprise repository for analytical queries (OLAP)
- **Size**: 100 GB to 1+ TB
- **Users**: entire organization, all departments
- Uses **ETL** (Extract → Transform → Load) — data is structured *before* loading
- **Schema-on-write**: schema defined when data is loaded

### Data Lake
- Stores raw data in any format (structured, semi-structured, unstructured)
- **Size**: petabyte scale
- Uses **ELT** (Extract → Load → Transform) — data is transformed *after* loading
- **Schema-on-read**: schema is applied when data is queried
- More flexible but requires more governance to avoid becoming a "data swamp"

---

## DWH vs Data Mart

| Feature | Data Warehouse | Data Mart |
|---|---|---|
| Scope | Enterprise-wide | Department-specific |
| Size | 100 GB – 1 TB+ | < 100 GB |
| Users | All departments | Specific team |
| Build time | Longer | Faster |

(source: Unit I- Data Science.pptx.pdf)

---

## DWH vs Data Lake

| Feature | Data Warehouse | Data Lake |
|---|---|---|
| Data type | Structured only | Any (raw) |
| Processing | ETL (before load) | ELT (after load) |
| Schema | Schema-on-write | Schema-on-read |
| Query speed | Faster (pre-processed) | Slower (flexible) |
| Governance | High | Lower (risk of data swamp) |

(source: Unit I- Data Science.pptx.pdf)

---

## Open Data Providers

External sources for retrieving public datasets (Step 2 of the [[data-science-process]]):

- **Data.gov** — US government open data
- **open-data.europa.eu** — EU open data portal
- **Freebase.org** — knowledge graph data
- Web APIs (e.g., COVID-19 API, REST endpoints returning JSON)
- Web scraping using `requests` + `BeautifulSoup`
