# 3\. 🗺️ EXECUTION ROADMAP: Implementation Plan (FastAPI/PostgreSQL)

This roadmap outlines the detailed, phase-by-phase implementation plan for the Chronos Atlas project. It follows a decoupled, data-first approach, prioritizing the establishment of the data foundation (ETL) and the core features (Knowledge Roadmap) before focusing on optimization.

-----

## Phase 0: Foundation & Environment Setup (Status: ✅ Complete)

**Goal:** Establish a portable, reproducible development environment and define the core data models. This phase is complete and the local setup is ready for data loading.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **0.1** | **Create Docker Environment** | `docker-compose.yml` (for PostgreSQL and Python service) | Running `make up` brings up both containers, linked via an internal Docker network. |
| **0.2** | **Project Structure & Dependencies** | `requirements.txt` (FastAPI, Uvicorn, SQLAlchemy, AsyncPG, Strawberry, SPARQLWrapper) | All Python libraries are installed and a clean project structure is established. |
| **0.3** | **Define SQLAlchemy Models** | `models.py` (Define Agent, Work, Concept, Influence, and the composite **RoadmapNode** type) | The four primary tables and the composite output type are defined, precisely matching the schema. |
| **0.4** | **Database Migration & Initialization** | **Alembic** setup and initial migration. | Running `make alembic-upgrade` successfully applies the schema to the database. |

-----

## Phase 1: MVP Timeline & Data Load (Status: ⚪️ Pending)

**Goal:** Complete the "Big Bang" ETL process and launch the first API endpoints to populate the basic timeline view.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **1.1** | **Implement ETL Logic** | Python scripts (using `SPARQLWrapper`), Idempotency logic (`ON CONFLICT`). | The ETL script successfully fetches, transforms, and loads 1,000+ records into the `Agent`, `Work`, and `Concept` tables. |
| **1.2** | **Execute Initial Data Load** | **`make data-load`** target. | The `data-load` script runs end-to-end without errors and populates all tables. |
| **1.3** | **Implement Timeline Resolver** | FastAPI/Strawberry GraphQL resolver for `timelineFigures`. | A query can fetch a list of figures, ordered by birth date, filtered by discipline. |
| **1.4** | **Implement Detail Resolver** | FastAPI/Strawberry GraphQL resolver for `figureDetail`. | A query can fetch a single figure and its directly related Works and Influence connections. |

-----

## Phase 2: Knowledge Roadmap Logic (Status: ⚪️ Pending)

**Goal:** Implement the core graph traversal logic that provides the project's unique value.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **2.1** | **Develop Recursive SQL** | **PostgreSQL `WITH RECURSIVE`** on the `Influence` table, calculating the `depth` field. | The raw SQL query can trace the prerequisite chain for a concept and return a flat list of `Concept` IDs with their path depth. |
| **2.2** | **Implement Roadmap Resolver** | FastAPI/Strawberry GraphQL resolver for **`knowledgeRoadmap`**. | The query successfully executes the recursive SQL and maps the result to the `RoadmapNode` GraphQL type. |
| **2.3** | **Implement Context Resolver** | Implement a resolver that takes time and discipline filters (e.g., `contemporaries(year: 1700, field: "Science")`). | The API can serve complex queries showing figures and events active at a specific time, filtered by field. |

-----

## Phase 3: Performance & Production (Status: Pending)

**Goal:** Optimize the application for speed, reliability, and deployment to a production environment.

| Step | Task | Tools & Concepts | Success Criteria |
| :--- | :--- | :--- | :--- |
| **3.1** | **Implement Redis Caching** | **Redis** container, FastAPI dependencies/middleware. | The results of the most complex, repeated queries (e.g., `knowledgeRoadmap` for popular concepts) are served instantly from the cache, bypassing the database. |
| **3.2** | **Implement ETL Scheduling** | Cloud Scheduler (e.g., AWS Lambda, Google Cloud Functions) running the Dockerized ETL script. | The ETL process runs autonomously (e.g., daily) to update the PostgreSQL database without manual intervention. |
| **3.3** | **Final Cloud Deployment** | Kubernetes/Docker Swarm or equivalent PaaS (e.g., Heroku, Render) for FastAPI, Managed Service for PostgreSQL. | The API is live and accessible via a public URL with HTTPS configured, ready for the frontend deployment on Vercel. |
| **3.4** | **API Contract Finalization** | Final review of `GRAPHQL_API_CONTRACT.md`. | The API is stable, the GraphQL schema is locked, and your friend can begin full frontend development knowing the contract will not change. |
