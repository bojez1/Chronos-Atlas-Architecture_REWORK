# 🧠 Chronos Atlas: The Knowledge Navigator & Curriculum Roadmap

**A modern, high-performance, data-driven visualization of the lineage of ideas, powered by FastAPI, GraphQL, and PostgreSQL.**

This project transforms disconnected historical and scientific facts into a visual, relational **Knowledge Graph**. By linking figures, works, and influences across time, Chronos Atlas allows users to map the **lineage of ideas** and build personalized educational roadmaps across disciplines.

## 🎯 Core Value Proposition: Contextualized Learning at Scale

The primary goal is to launch a scalable application that solves the problem of disciplinary isolation in learning.

| Feature | Description |
| :--- | :--- |
| **Knowledge Roadmap Builder** | The core feature. Trace any advanced **Concept** (e.g., 'Relativity,' 'Calculus') backward to its prerequisite works and foundational concepts, providing a clear, visual curriculum path. |
| **Interdisciplinary Context** | Allows filtering by time, field, and influence to quickly visualize the **simultaneity** of events and contemporaries across different disciplines (e.g., philosophy's influence on science). |
| **Relational Knowledge Graph** | Built on specific relationship types (`BUILT_ON`, `PREREQUISITE_FOR`, `INFLUENCED`) that enable the deep, graph-like queries essential for the Roadmap logic. |
| **Interactive Timeline** | Visually displays the lifespan of **Agents** and the chronological sequencing of their **Works** on a scrollable, zoomable axis. |

## 🛠️ Technology Stack

Chronos Atlas is built on a high-performance, asynchronous foundation, chosen specifically for its power in handling complex relational queries at scale.

| Layer | Primary Technology | Rationale |
| :--- | :--- | :--- |
| **Backend/API** | **Python (FastAPI)** + **GraphQL (Strawberry/Ariadne)** | **Asynchronous and high-performance** for handling concurrent queries and minimal overhead. The ideal choice for a pure API service. |
| **Database** | **PostgreSQL** 🐘 (with Async SQLAlchemy) | **Core Logic Engine.** Chosen for its superior capabilities with complex relational data and performing **recursive queries** (using `WITH RECURSIVE`), which are essential for the Roadmap feature. |
| **Frontend** | **Nextra (via Vercel)** | Optimal for rapid development of a **decoupled frontend** and content-heavy, structured knowledge sites. |
| **Performance** | **Redis** + **Scheduled Cloud Jobs** | Caching the results of heavy, repeated queries (Redis) and automating the background data ingestion/maintenance (ETL). |
| **Data Source** | **Wikidata (SPARQL)** | Structured, open source for reliable, comprehensive historical data across all domains. |

## 🧭 Project Blueprint Navigation

This project is fully documented across **five detailed blueprint files**, ensuring every layer of the decoupled architecture is understood. Start with the Architecture Blueprint to understand the new technical foundation and the **Knowledge Graph Model**.

| Document | Focus | Key Topics |
| :--- | :--- | :--- |
| **1. `ARCHITECTURE_BLUEPRINT.md`** | **Knowledge Graph Model** | Stack justification, **Four-Table Schema (Agent, Work, Concept, Influence)**, and indexing strategy. |
| **2. `DATA_FLOW_AND_SCALING.md`** | **ETL & Operational Stability** | Data Flow, Database Connection Pooling, and performance requirements. |
| **3. `EXECUTION_ROADMAP.md`** | **Implementation Plan** | Detailed breakdown of all execution phases, including the **Big Bang ETL** and **Roadmap Logic Implementation**. |
| **4. `GRAPHQL_API_CONTRACT.md`** | **Frontend Specification** | **Specific GraphQL Queries,** Mutations, all Field Names, and required Input Variables. |
| **5. `WIKIDATA_ETL_SPEC.md`** | **Data Sourcing & Quality** | Detailed **SPARQL Queries**, Wikidata P-Numbers targeted, and **Data Normalization Rules** (e.g., date formatting). |

### [🔗 **Read the Architecture Blueprint**](https://www.google.com/search?q=./ARCHITECTURE_BLUEPRINT.md)

### [🔗 **View the Execution Roadmap**](https://www.google.com/search?q=./EXECUTION_ROADMAP.md)

### [🔗 **Explore Data Flow and Scaling**](https://www.google.com/search?q=./DATA_FLOW_AND_SCALING.md)

### [🔗 **Define the API Contract**](https://www.google.com/search?q=./GRAPHQL_API_CONTRACT.md)

### [🔗 **Document the ETL Specification**](https://www.google.com/search?q=./WIKIDATA_ETL_SPEC.md)

## 📈 Current Status: Execution Plan & Next Steps

Since the technology stack and architecture have been completely overhauled, we are resetting Phase 0 to **Pending**. Your immediate task is setting up the new local environment.

| Phase | Status | Next Major Milestone |
| :--- | :--- | :--- |
| **Phase 0: Foundation** | ⚪️ **Pending** | **Establish the Local FastAPI/PostgreSQL Environment.** |
| **Phase 1: MVP Timeline & Data Load** | ⚪️ Pending | Complete the **"Big Bang" ETL** and launch the basic Master Timeline View. |
| **Phase 2: Knowledge Roadmap Logic** | ⚪️ Pending | Implement the **PostgreSQL Recursive Query** and the core **`knowledgeRoadmap`** GraphQL resolver. |
| **Phase 3: Performance & Production** | ⚪️ Pending | Implement Redis Caching and finalize the cloud deployment. |
