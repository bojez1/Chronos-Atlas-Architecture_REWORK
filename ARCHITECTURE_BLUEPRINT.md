# 1\. 🏗️ ARCHITECTURE BLUEPRINT: Knowledge Graph Model & Stack

The Chronos Atlas is built upon a fully decoupled, asynchronous architecture optimized for high-performance graph traversal and complex relational queries. This document details the technical stack choices and, critically, the **Knowledge Graph Schema** that underpins the entire application logic.

## 1.1 Stack Justification

The choice of **FastAPI, Strawberry, and PostgreSQL** is non-negotiable for this project due to the core feature requirements (the Knowledge Roadmap Builder).

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **API Framework** | **FastAPI (Python)** | **Asynchronous Performance.** Provides high throughput necessary for concurrent frontend requests and handles concurrent database I/O efficiently, crucial for minimizing latency on complex queries. |
| **GraphQL Layer** | **Strawberry** | **Pythonic & Async-Native.** Utilizes Python type hints for schema definition, offering a clean, type-safe development experience that integrates seamlessly with FastAPI's asynchronous nature. |
| **Database** | **PostgreSQL** | **Relational Power for Graph Logic.** Essential for its built-in features: 1) **JSONB** support for flexible metadata, and 2) **Common Table Expressions (CTEs)**, specifically the **`WITH RECURSIVE`** clause, which is necessary to implement the Knowledge Roadmap's graph-traversal logic. |
| **ORM/Async Driver** | **SQLAlchemy Core/AsyncPG** | Provides a robust, Pythonic interface to define models and executes database operations efficiently using asynchronous drivers. |

-----

## 1.2 Knowledge Graph Schema (PostgreSQL Models)

The data is structured around four primary entities and one critical relationship table.

### Primary Entities

| Entity | Description | Key Fields | Notes |
| :--- | :--- | :--- | :--- |
| **`Agent`** | A historical figure, group, or institution. | `id` (PK), `name`, `birth_date`, `death_date`, `discipline` (e.g., 'Science', 'Philosophy') | Supports the main Timeline View. |
| **`Work`** | A published output (book, theory, discovery). | `id` (PK), `title`, `publication_date`, `type` (e.g., 'Book', 'Treatise'), `author_id` (FK to Agent) | Links to the original creator. |
| **`Concept`** | An abstract idea or academic concept. | `id` (PK), `name`, `definition`, `field` | Central to the Roadmap feature. |
| **`Influence`** | **The Relationship Edge.** | `source_id` (FK), `target_id` (FK), `relationship_type` (Enum: INFLUENCED, PREREQUISITE\_FOR, BUILT\_ON, ASSOCIATED\_WITH) | This is the core table used for graph traversal. |

### Composite Output Type

The **`RoadmapNode`** is a composite type returned by the `knowledgeRoadmap` query, combining the Concept data with the path-dependent metadata (`depth`) calculated during the recursive traversal.

| Field | Source | Notes |
| :--- | :--- | :--- |
| `concept` | `Concepts` Table | The Concept entity itself. |
| `depth` | Calculated in SQL | **CRITICAL.** The number of steps away from the initial concept. Used for frontend rendering/indentation. |

## 1.3 Key Querying Strategy

The success of the Chronos Atlas rests on the ability of PostgreSQL to execute the recursive query quickly.

### The Knowledge Roadmap Query

The **`knowledgeRoadmap`** GraphQL resolver will translate into a single, complex PostgreSQL query using **`WITH RECURSIVE`**.

1. **Input:** A target concept ID (e.g., 'Relativity').
2. **Logic:** The query starts at the target concept and recursively traverses the `Influence` table, following all records where the `target_id` is the current node and the `relationship_type` is **`PREREQUISITE_FOR`**. The `depth` value is calculated as part of the CTE iteration.
3. **Output:** A list of **`RoadmapNode`** entities ordered by their relationship distance, effectively mapping the curriculum back to foundational knowledge.

### Indexing Strategy

To support these intensive relational lookups, the following indices are mandatory:

* **Compound Index on `Influence`:** `(source_id, target_id, relationship_type)`
* **Simple Indices:** `Agents.discipline`, `Works.publication_date`, `Concepts.field`.
