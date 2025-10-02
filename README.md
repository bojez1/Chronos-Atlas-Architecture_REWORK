# 🧠 Chronos Atlas: The Knowledge Navigator & Curriculum Roadmap

**A modern, high-performance, data-driven visualization of the lineage of ideas, powered by FastAPI, GraphQL (Strawberry), SQLAlchemy, and PostgreSQL.**

This project transforms disconnected historical and scientific facts into a visual, relational **Knowledge Graph**. By linking figures, works, and influences across time, Chronos Atlas allows users to map the **lineage of ideas** and build personalized educational roadmaps across disciplines.

## 🚀 Quick Start: Development Environment

All development and administrative tasks are **Docker-first** and executed via `make`.

1. **Start Services:** Build and start the FastAPI (`api`) and PostgreSQL (`db`) containers.

    ```bash
    make up
    ```

2. **Run Migrations:** Apply the database schema from the models.

    ```bash
    make alembic-upgrade
    ```

3. **Load Initial Data:** Execute the "Big Bang" ETL to seed the database with core data.

    ```bash
    make data-load
    ```

4. **Run Tests:** Execute the full test suite.

    ```bash
    make pytest
    ```

The FastAPI application will be accessible at `http://localhost:8000`.

---

## 🎯 Core Value Proposition: Contextualized Learning at Scale

| Feature | Description |
| :--- | :--- |
| **Knowledge Roadmap Builder** | The core feature. Trace any advanced **Concept** (e.g., 'Relativity,' 'Calculus') backward to its prerequisite works and foundational concepts, providing a clear, visual curriculum path. |
| **Interdisciplinary Context** | Allows filtering by time, field, and influence to quickly visualize the **simultaneity** of events and contemporaries across different disciplines. |
| **Relational Knowledge Graph** | Built on specific relationship types (`BUILT_ON`, `PREREQUISITE_FOR`, `INFLUENCED`) that enable the deep, graph-like queries essential for the Roadmap logic using PostgreSQL's **`WITH RECURSIVE`** capability. |
| **Interactive Timeline** | Visually displays the lifespan of **Agents** and the chronological sequencing of their **Works** on a scrollable, zoomable interface. |

---

## 📚 Project Documentation Hub

All detailed plans, architecture, and contracts are documented below.

| Document | Purpose | Key Content |
| :--- | :--- | :--- |
| **1. `ARCHITECTURE_BLUEPRINT.md`** | **Technical Stack & Schema** | Justifies FastAPI/PostgreSQL, defines the **Knowledge Graph Models**, and specifies the **`WITH RECURSIVE`** query for the Roadmap. |
| **2. `DATA_FLOW_AND_SCALING.md`** | **Operational Logic** | Details the **Asynchronous ETL** pipeline, the **Indexing Strategy**, and the plan for scaling via Redis and Read Replicas. |
| **3. `EXECUTION_ROADMAP.md`** | **Implementation Plan** | Phase-by-phase plan (Foundation, MVP, Production), prioritizing the core **Roadmap** feature. |
| **4. `GRAPHQL_API_CONTRACT.md`** | **Frontend API Contract** | Defines the strict GraphQL types and queries (e.g., `knowledgeRoadmap`) for the Nextra frontend. |
| **5. `WIKIDATA_ETL_SPEC.md`** | **Data Sourcing & Quality** | Detailed **SPARQL Queries**, Wikidata P-Numbers targeted, and **Data Normalization Rules** (e.g., date formatting). |

### [🔗 **Read the Architecture Blueprint**](docs/Chronos-Atlas/ARCHITECTURE_BLUEPRINT.md)

### [🔗 **View the Execution Roadmap**](docs/Chronos-Atlas/EXECUTION_ROADMAP.md)

### [🔗 **Explore Data Flow and Scaling**](docs/Chronos-Atlas/DATA_FLOW_AND_SCALING.md)

### [🔗 **Define the API Contract**](docs/Chronos-Atlas/GRAPHQL_API_CONTRACT.md)

### [🔗 **Document the ETL Specification**](docs/Chronos-Atlas/WIKIDATA_ETL_SPEC.md)

---

## 📈 Current Status: Execution Plan & Next Steps

| Phase | Status | Next Major Milestone |
| :--- | :--- | :--- |
| **Phase 0: Foundation** | ✅ **Complete** | Local FastAPI/PostgreSQL Environment, Alembic, CLI Scripts, and Docker Automation are set up. |
| **Phase 1: MVP Timeline & Data Load** | ⚪️ Pending | Complete the **"Big Bang" ETL** using `make data-load` and launch the basic Master Timeline View. |
| **Phase 2: Knowledge Roadmap Logic** | ⚪️ Pending | Implement the **PostgreSQL `WITH RECURSIVE`** query and the `knowledgeRoadmap` GraphQL resolver. |
