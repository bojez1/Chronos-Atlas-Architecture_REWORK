# 5\. 🗃️ WIKIDATA ETL SPEC: Data Sourcing & Quality

This blueprint documents the rules and specific targets for the ETL process. It ensures the **"Big Bang"** data load is consistent and that all raw Wikidata properties are correctly mapped to our defined Knowledge Graph relationships.

## 5.1 ETL Processing Rules

### A. Idempotency

The ETL process **must** be idempotent. When loading data into PostgreSQL (Step 1.3 in the roadmap), it must use **`INSERT...ON CONFLICT UPDATE`** logic based on the entity's primary key (`id`). This ensures re-running the ETL only updates records, preventing duplicates.

### B. Date Standardization

All dates extracted from Wikidata must be converted to the standard PostgreSQL **`DATE`** format (`YYYY-MM-DD`).

* **Rule:** If Wikidata provides only a year, the date should default to **`YYYY-01-01`** for consistent sorting and timeline rendering.

### C. Relationship Mapping Priority

The raw data extracted from Wikidata contains hundreds of property types. The ETL must filter and map these properties to the four discrete `RelationshipType` values in our model.

| Our `RelationshipType` | Target Wikidata Properties (P-numbers) | Notes on Transformation |
| :--- | :--- | :--- |
| **`INFLUENCED`** | `P737` (influenced by), `P800` (notable work), `P106` (occupation) | Used primarily for `Agent` to `Agent` links. Represents general intellectual debt. |
| **`PREREQUISITE_FOR`** | `P279` (subclass of), `P155` (follows) | **CRITICAL.** Maps Concept prerequisites. If A follows B, B is the prerequisite for A. |
| **`BUILT_ON`** | `P361` (part of), `P629` (edition or translation of), `P144` (based on) | Used for `Work` to `Work` links. Shows explicit dependency or derivation. |
| **`ASSOCIATED_WITH`** | `P1889` (different from), `P31` (instance of) | General association between an `Agent` and a `Concept` they are known for. |

## 5.2 Initial "Big Bang" SPARQL Scope

The first ETL run must target essential, general knowledge (roughly high school level) to establish a dense, well-connected graph.

### A. Agent Filtering

Target figures who satisfy one of the following criteria:

1. **High-Profile Award:** Has property `P166` (award received) with a value like: Nobel Prize (`Q7191`), Pulitzer (`Q218803`), or Fields Medal (`Q190600`).
2. **Core Disciplines:** Has occupation `P106` matching: Philosopher (`Q41508`), Physicist (`Q901`), or Writer (`Q36180`).
3. **Timeframe:** Born after 1500 AD, excluding ancient figures for the MVP scope.

### B. Relationship Deep Dive

The SPARQL query must specifically target and extract the **prerequisite chain** for 10-15 foundational scientific and philosophical concepts (e.g., 'Thermodynamics', 'Calculus', 'Empiricism'). This ensures the **`PREREQUISITE_FOR`** relationships are fully populated for immediate testing.
