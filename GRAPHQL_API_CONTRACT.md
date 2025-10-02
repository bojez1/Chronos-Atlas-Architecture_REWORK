# 4\. 🤝 GRAPHQL API CONTRACT: Frontend Specification

This document defines the strict, non-negotiable contract between the **FastAPI/Strawberry backend** and the **Nextra frontend**. All frontend development should rely exclusively on the queries and types defined here. The backend guarantees stability for this schema.

## 4.1 Core Types (Based on SQLAlchemy Models)

The following GraphQL Types are derived directly from the PostgreSQL models defined in the `ARCHITECTURE_BLUEPRINT.md`.

| GraphQL Type | Description | Key Fields & Types | Relationships (via Resolvers) |
| :--- | :--- | :--- | :--- |
| **`Agent`** | A historical figure or entity. | `id: ID!`, `name: String!`, `birthDate: Date`, `deathDate: Date`, `discipline: String` | `works: [Work]`, `influencedBy: [Agent]` |
| **`Work`** | A published output (book, theory, discovery). | `id: ID!`, `title: String!`, `publicationDate: Date`, `type: String` | `author: Agent`, `builtUpon: [Work]` |
| **`Concept`** | An abstract idea or academic concept. | `id: ID!`, `name: String!`, `definition: String`, `field: String` | `prerequisites: [Concept]`, `associatedAgents: [Agent]` |
| **`Relationship`** | The edge data structure for influence/lineage. | `sourceId: ID!`, `targetId: ID!`, `type: RelationshipType!` | Used internally for graph traversal. |
| **`RoadmapNode`** | **NEW: Composite type for recursive output.** | `concept: Concept!`, **`depth: Int!`** | Combines a Concept with its calculated path distance. |

## 4.2 Enumeration Types

### `RelationshipType`

This enum reflects the filtered types in the PostgreSQL `Influence` table.

```graphql
enum RelationshipType {
    INFLUENCED
    PREREQUISITE_FOR
    BUILT_ON
    ASSOCIATED_WITH
}
```

## 4.3 Root Queries (Frontend Endpoints)

These are the primary queries the Nextra frontend will use. All must be implemented in the FastAPI/Strawberry resolvers.

| Query Name | Arguments | Returns | Purpose |
| :--- | :--- | :--- | :--- |
| `timelineFigures` | `discipline: [String]`, `yearRange: [Int]` | `[Agent!]` | Populates the main **Master Timeline View** with filtered figures. |
| `figureDetail` | `id: ID!` | `Agent` | Fetches a single figure along with their works and direct influence connections for the detail panel. |
| **`knowledgeRoadmap`** | `conceptId: ID!` | **`[RoadmapNode!]`** | **Core Feature.** Traces the prerequisite lineage of a concept using the PostgreSQL **`WITH RECURSIVE`** query. |
| `interdisciplinaryContext` | `year: Int!`, `fields: [String]` | `[Agent!]` | Finds contemporaries across different fields for a specific year. |
