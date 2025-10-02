# 2\. 🌊 DATA FLOW AND SCALING: ETL & Operational Stability

This blueprint details the operational logic of the Chronos Atlas: how data flows from the source (Wikidata) through the ETL pipeline into the database, and how the API is optimized to serve complex queries rapidly and reliably.

-----

## 2.1 The Two-Phase Data Flow Cycle

The entire data lifecycle is separated into two distinct, asynchronous pipelines to ensure maximum API uptime and fresh data.

### Phase A: The Offline ETL (Extraction, Transformation, Load) Pipeline

This pipeline runs periodically as a background job and is decoupled from the live API server.

| Stage | Action | Tool / Logic | Notes |
| :--- | :--- | :--- | :--- |
| **1. Extraction** | Fetching raw, unstructured data from Wikidata. | Python (`SPARQLWrapper`) | Queries are targeted and must be executed in bulk to minimize latency (the "Big Bang" approach). |
| **2. Transformation** | Cleaning, validating, and structuring the raw data. | Python ETL Logic | **CRITICAL:** This stage converts Wikidata IDs and properties into the defined `Influence` relationship types (`PREREQUISITE_FOR`, `BUILT_ON`, etc.) and standardizes all dates. |
| **3. Loading** | Inserting transformed data into PostgreSQL. | Async SQLAlchemy Sessions | Data must be loaded using **idempotent** logic (`ON CONFLICT DO UPDATE`) to prevent duplicates and ensure clean updates to existing records. |

### Phase B: The Online API Serving Pipeline

This pipeline runs continuously and handles all live user interactions (queries).

| Strategy | Implementation | Benefit |
| :--- | :--- | :--- |
| **Asynchronous API** | **FastAPI/Uvicorn** | Allows the server to handle **thousands of concurrent connections** without blocking while waiting on the PostgreSQL database. |
| **Connection Pooling** | Handled by **Async SQLAlchemy/AsyncPG**. | Reuses established database connections, significantly reducing connection overhead and latency for frequent queries. |
| **CORS/Load Balancing** | **CORS Middleware** in FastAPI, **Load Balancer** on deployment platform. | Ensures secure communication with the Nextra frontend and distributes traffic across multiple FastAPI worker processes. |

### Database Scaling (PostgreSQL)

| Strategy | Implementation | Benefit |
| :--- | :--- | :--- |
| **Indexing** | Implement all indices defined in `ARCHITECTURE_BLUEPRINT.md`. | **Non-Negotiable.** Speeds up all lookups, particularly the compound index on `Influence` for graph traversal. |
| **Caching (L1)** | **Redis Caching** for query results (Phase 3). | Provides sub-millisecond retrieval for frequently requested data (e.g., the top 20 requested roadmaps), drastically reducing database load. |
| **Read Replicas** | (Future Scaling) Deploy one or more PostgreSQL replicas. | Separates heavy **Read** traffic (the API queries) from the lighter **Write** traffic (the weekly ETL, which includes the `depth` calculation). |

### Query Performance Target

The goal for the complex **`knowledgeRoadmap`** query (which includes recursive traversal and `depth` calculation) is:

* **Uncached:** Under **500 milliseconds**.
* **Cached:** Under **50 milliseconds**.
