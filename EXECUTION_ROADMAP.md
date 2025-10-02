# 3. 🗺️ EXECUTION ROADMAP: Implementation Plan (FastAPI/PostgreSQL)

This roadmap outlines the detailed, phase-by-phase implementation plan for the Chronos Atlas project. It follows a decoupled, data-first approach, prioritizing the establishment of the data foundation (ETL) and the core features (Knowledge Roadmap) before focusing on optimization.

---

## Phase 0: Foundation & Environment Setup (Status: Pending)

**Goal:** Establish a portable, reproducible development environment and define the core data models. This phase must be completed before any ETL or API code is written.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **0.1** | **Create Docker Environment** | `docker-compose.yml` (for PostgreSQL and Python service) | Running `docker-compose up` brings up both containers, linked via an internal Docker network. |
| **0.2** | **Project Structure & Dependencies** | `requirements.txt` (FastAPI, Uvicorn, SQLAlchemy, AsyncPG, Strawberry, SPARQLWrapper) | All Python libraries are installed and a clean project structure (`/api`, `/etl`, `/config`) is established. |
| **0.3** | **Define SQLAlchemy Models** | `models.py` (Define Agent, Work, Concept, Influence, and the composite **RoadmapNode** type) | The four primary tables and the composite output type are defined, precisely matching the schema. |
| **0.4** | **Database Migration & Initialization** | SQLAlchemy `MetaData.create_all()` or a basic **Alembic** setup. | The PostgreSQL database instance contains the four empty tables ready to receive data. |
| **0.5** | **Basic FastAPI App** | `main.py`, Uvicorn | The FastAPI server runs successfully on the configured port (e.g., `http://localhost:8000`). |

---

## Phase 1: MVP Timeline & Data Load (Status: Pending)

**Goal:** Execute the "Big Bang" data load to fill the database and launch the initial, read-only API endpoints required for the Master Timeline View.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **1.1** | **Write SPARQL Extraction Script** | `SPARQLWrapper`, complex Wikidata queries (as detailed in `WIKIDATA_ETL_SPEC.md`). | The script successfully fetches a large raw JSON/dictionary dataset of Agents, Works, and their influence links. |
| **1.2** | **Implement Transformation Logic** | Python, `pandas` (optional, for processing), date/string standardization functions. | Raw Wikidata data is converted into clean Python object lists that perfectly match the fields and types defined in `models.py`. |
| **1.3** | **Execute "Big Bang" Data Load** | Async SQLAlchemy Sessions, `ON CONFLICT DO UPDATE` or `merge()` logic. | The PostgreSQL tables are populated with thousands of records, and the ETL script can be run multiple times without duplicating data. **Data Foundation is set.** |
| **1.4** | **Implement Basic GraphQL Setup** | **Strawberry** schema definition (Types and Inputs) | A valid GraphQL schema is exposed at `/graphql` and shows the basic `Agent` and `Work` types. |
| **1.5** | **Implement `timelineFigures` Resolver** | Async SQLAlchemy, `async def` function. | The frontend can successfully query `query { timelineFigures { id, name, birthDate } }` and receive all data loaded in 1.3. |
| **1.6** | **Implement `figureDetail` Resolver** | Async SQLAlchemy, `relationship()` mapping. | The frontend can fetch a single figure and their associated works and influence links in one query. |

---

## Phase 2: Knowledge Roadmap Logic (Status: Pending)

**Goal:** Implement the complex graph-traversal logic that powers the core value proposition of the Chronos Atlas.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **2.1** | **Targeted Concept ETL** | **ETL script modification** to focus on `PREREQUISITE_FOR` relationships between Concepts. | The `Concepts` and related `Influence` table records are fully populated with the relationships needed to map knowledge lineage. |
| **2.2** | **Design Recursive SQL Query** | **PostgreSQL `WITH RECURSIVE` CTE** including a path counter. | A native SQL query successfully traces the prerequisite path and **calculates the `depth` (distance) for every concept** in the chain. |
| **2.3** | **Implement `knowledgeRoadmap` Resolver** | Async SQLAlchemy, mapping the recursive query result to the **`RoadmapNode`** type. | The frontend can query `query { knowledgeRoadmap(conceptId: "Q123") { concept { name }, depth } }` and receive the full, ordered, and structured curriculum path. |
| **2.4** | **Interdisciplinary Context Resolver** | Implement a resolver that takes time and discipline filters (e.g., `contemporaries(year: 1700, field: "Science")`). | The API can serve complex queries showing figures and events active at a specific time, filtered by field. |

---

## Phase 3: Performance & Production (Status: Pending)

**Goal:** Optimize the application for speed, reliability, and deployment to a production environment.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **3.1** | **Implement Redis Caching** | **Redis** container, FastAPI dependencies/middleware. | The results of the most complex, repeated queries (e.g., `knowledgeRoadmap` for popular concepts) are served instantly from the cache, bypassing the database. |
| **3.2** | **Implement ETL Scheduling** | Cloud Scheduler (e.g., AWS Lambda, Google Cloud Functions) running the Dockerized ETL script. | The ETL process runs autonomously (e.g., daily) to update the PostgreSQL database without manual intervention. |
| **3.3** | **Final Cloud Deployment** | Kubernetes/Docker Swarm or equivalent PaaS (e.g., Heroku, Render) for FastAPI, Managed Service for PostgreSQL. | The API is live and accessible via a public URL with HTTPS configured, ready for the frontend deployment on Vercel. |
| **3.4** | **API Contract Finalization** | Final review of `GRAPHQL_API_CONTRACT.md`. | The API is stable, the GraphQL schema is locked, and your friend can begin full frontend development knowing the contract will not change. |
