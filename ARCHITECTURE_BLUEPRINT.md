# 1. 🏗️ ARCHITECTURE BLUEPRINT: Knowledge Graph Model & Stack

The Chronos Atlas is built upon a fully decoupled, asynchronous architecture optimized for high-performance graph traversal and complex relational queries. This document details the technical stack choices and, critically, the **Knowledge Graph Schema** that underpins the entire application logic.

## 1.1 Stack Justification

The choice of **FastAPI, Strawberry, and PostgreSQL** is non-negotiable for this project due to the core feature requirements (the Knowledge Roadmap Builder).

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **API Framework** | **FastAPI (Python)** | **Asynchronous Performance.** Provides high throughput necessary for concurrent frontend requests and handles concurrent database I/O efficiently, crucial for minimizing latency on complex queries. |
| **GraphQL Layer** | **Strawberry** | **Pythonic & Async-Native.** Utilizes Python type hints for schema definition, offering a clean, type-safe development experience that integrates seamlessly with FastAPI's asynchronous nature. |
| **Database** | **PostgreSQL** | **Relational Power for Graph Logic.** Essential for its built-in features: 1) **JSONB** support for flexible metadata, and 2) **Common Table Expressions (CTEs)**, specifically the **`WITH RECURSIVE`** clause, which is necessary to implement the Knowledge Roadmap's graph-traversal logic. |
| **ORM/Async Driver** | **SQLAlchemy Core/AsyncPG** | Provides a robust, Pythonic interface to define and interact with the PostgreSQL schema asynchronously. |

## 1.2 The Knowledge Graph Data Model

The data is structured into four primary tables that form an interconnected relational graph. The `Influence` table is the most critical component, as it defines the entire lineage of knowledge.

### 1. Agents (Historical Figures)

* **Purpose:** The subjects of the timeline.
* **Represents:** People, groups, or organizations.

| Field | Type (PostgreSQL) | Notes |
| :--- | :--- | :--- |
| `id` | `VARCHAR(30)` | Primary Key (e.g., Wikidata ID: `Q42`). |
| `name` | `VARCHAR(255)` | Full common name (e.g., 'Isaac Newton'). |
| `birth_date` | `DATE` | For timeline rendering. |
| `death_date` | `DATE` | For timeline rendering. |
| `discipline` | `VARCHAR(100)` | Primary field (e.g., 'Physics', 'Philosophy'). |
| `metadata` | `JSONB` | Storage for flexible data like alternate names, source URL, citation count, etc. |

### 2. Works (Outputs)

* **Purpose:** Specific tangible outputs created by Agents.
* **Represents:** Books, treatises, discoveries, symphonies, paintings.

| Field | Type (PostgreSQL) | Notes |
| :--- | :--- | :--- |
| `id` | `VARCHAR(30)` | Primary Key. |
| `title` | `VARCHAR(255)` | Full title of the work. |
| `agent_id` | `VARCHAR(30)` | Foreign Key to the `Agents` table. |
| `publication_date` | `DATE` | The date the work was published or created. |
| `type` | `VARCHAR(50)` | Categorization (e.g., 'Book', 'Theory', 'Experiment'). |

### 3. Concepts (Ideas)

* **Purpose:** Abstract ideas or academic concepts that form the nodes of the Knowledge Roadmap.
* **Represents:** 'Relativity,' 'Calculus,' 'Monarchy,' 'Natural Selection.'

| Field | Type (PostgreSQL) | Notes |
| :--- | :--- | :--- |
| `id` | `VARCHAR(30)` | Primary Key (Wikidata ID for the concept). |
| `name` | `VARCHAR(255)` | Common name of the concept. |
| `definition` | `TEXT` | Brief, academic definition for display. |
| `field` | `VARCHAR(100)` | Discipline it belongs to. |

### 4. Influence (The Relational Edge)

* **Purpose:** The core relationship table that links all entities together and drives the graph logic.
* **Structure:** A many-to-many relationship linking a **source** entity to a **target** entity.

| Field | Type (PostgreSQL) | Notes |
| :--- | :--- | :--- |
| `source_id` | `VARCHAR(30)` | ID of the source entity (Agent, Work, or Concept). |
| `target_id` | `VARCHAR(30)` | ID of the target entity (Agent, Work, or Concept). |
| `relationship_type` | `VARCHAR(50)` | **CRITICAL FIELD.** Must use one of the defined types below. |
| `context` | `TEXT` | A brief description of the nature of the influence. |

### Defined Relationship Types

The ETL process **must** map Wikidata properties to these discrete, powerful relationship types:

1. **`INFLUENCED`**: (Agent -> Agent or Work -> Work) A broad historical/intellectual influence.
2. **`PREREQUISITE_FOR`**: (Concept -> Concept) **Crucial for the Roadmap.** Concept A is required knowledge before Concept B can be understood.
3. **`BUILT_ON`**: (Work -> Work) Work A cites or explicitly refines the ideas in Work B.
4. **`ASSOCIATED_WITH`**: (Agent -> Concept) Figure is known for developing or popularizing the concept.

### 5. RoadmapNode (Composite Type for Recursive Output)

* **Purpose:** A composite type returned by the `knowledgeRoadmap` query, combining the Concept data with the path-dependent metadata (`depth`) calculated during the recursive traversal.

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
