# Database Comparison Matrix

This document compares the five database technologies used in this case study across key dimensions relevant to a social media platform. All assessments are qualitative and based on the documented architectural characteristics of each system.

---

## 1. Quick Reference Summary

| Dimension | CockroachDB | Firebase Firestore | Neo4j | ChromaDB | InfluxDB |
|-----------|-------------|-------------------|-------|----------|----------|
| **Category** | Distributed SQL | Cloud NoSQL Document | Graph | Vector | Time-Series |
| **Data Model** | Relational (tables, rows) | Document (JSON-like) | Property Graph | Embedding vectors | Measurements, tags, fields |
| **Query Language** | SQL (PostgreSQL dialect) | SDK / Collection API | Cypher | Python SDK | Flux |
| **Consistency Model** | SERIALIZABLE (strong) | Strong (single doc) / Eventual (multi-doc) | ACID (single node) | None (append store) | Eventual (TSM append) |
| **Primary Index** | B-tree + distributed range | Automatic + composite | Index-free adjacency | HNSW (ANN) | Time-series columnar |
| **Transactions** | Full ACID (distributed) | Optimistic (limited scope) | ACID | Not supported | Not supported |
| **Scaling Model** | Horizontal (Raft sharding) | Automatic (serverless) | Vertical (Enterprise for horizontal) | Vertical (local) | Vertical / Cloud |
| **Best Social Media Use Case** | User profiles, posts, counters | Mobile real-time feeds | Follow network, influence | Semantic search, recommendations | Engagement analytics, trending |
| **Operational Complexity** | High (cluster management) | Low (fully managed) | Medium (single-node setup) | Low (embedded library) | Medium (self-hosted) |
| **Open Source** | Yes (BSL licence) | No (Google Cloud product) | Yes (Community Edition) | Yes (Apache 2.0) | Yes (MIT / BSL) |

---

## 2. Data Model Comparison

### How Social Media Entities Are Represented

| Entity | CockroachDB | Firestore | Neo4j | ChromaDB | InfluxDB |
|--------|-------------|-----------|-------|----------|----------|
| **User** | `users` table row | `/users/{id}` document | `(:User)` node | Not primary | Tag value |
| **Post** | `posts` table row | `/posts/{id}` document | `(:Post)` node | Embedding document | Not primary |
| **Comment** | `comments` table row | `/posts/{id}/comments/{id}` subcollection | `[:COMMENTED]` relationship | Not stored | Event tag |
| **Follow** | `follows` table row (FK join) | `/follows/{id}` document | `[:FOLLOWS]` relationship (native) | Not stored | Event type tag |
| **Like** | `likes` table row (FK join) | `/likes/{id}` document | `[:LIKED]` relationship | Not stored | Event data point |
| **Engagement Event** | Could be a table row (expensive) | Could be a subcollection | Not optimal | Not stored | Native measurement |

### Key Insight: Fit Between Model and Entity

- **Follow relationships** are most naturally expressed as Neo4j relationships — a SQL `follows` table requires self-joins to traverse, while a Firestore approach requires multiple reads.
- **Post content for search** is most naturally expressed as ChromaDB embeddings — no other database in this study supports semantic similarity.
- **Engagement events** are most naturally expressed as InfluxDB measurements — inserting millions of timestamped events into a SQL table causes index fragmentation.
- **User profiles and posts** are equally well-expressed in CockroachDB (relational) and Firestore (document) — the choice depends on consistency requirements and deployment model.

---

## 3. Query Model Comparison

### Feed Generation: "Show posts from users I follow, newest first"

| Database | Approach | Complexity |
|----------|----------|------------|
| **CockroachDB** | `JOIN follows + posts ORDER BY created_at DESC LIMIT 20` | Medium — standard SQL, well-optimised with indexes |
| **Firestore** | Multiple reads: fetch followees → query posts per followee | High — no JOIN; requires client-side merging |
| **Neo4j** | `MATCH (me)-[:FOLLOWS]->(f)-[:POSTED]->(p) RETURN p ORDER BY p.created_at DESC` | Low — single Cypher traversal |
| **ChromaDB** | Not applicable — ChromaDB is for semantic search, not sorted feeds | N/A |
| **InfluxDB** | Not applicable — InfluxDB stores events, not post content | N/A |

### Social Graph: "Find friends-of-friends"

| Database | Approach | Complexity |
|----------|----------|------------|
| **CockroachDB** | `JOIN follows f1 ON f1.followee_id = f2.follower_id` (self-join) | High — exponential cost with depth |
| **Firestore** | Multiple document reads and client-side logic | Very High — impractical for depth > 1 |
| **Neo4j** | `MATCH (me)-[:FOLLOWS*2]->(fof)` | Low — native graph traversal |
| **ChromaDB** | Not applicable | N/A |
| **InfluxDB** | Not applicable | N/A |

### Semantic Search: "Find posts similar in meaning to this query"

| Database | Approach | Complexity |
|----------|----------|------------|
| **CockroachDB** | `LIKE '%keyword%'` or full-text search (limited) | Low accuracy — keyword only |
| **Firestore** | Array-contains for tags only | Low accuracy — no semantic understanding |
| **Neo4j** | Keyword match on node properties | Low accuracy — not designed for this |
| **ChromaDB** | ANN search on embedding vectors | High accuracy — native semantic search |
| **InfluxDB** | Not applicable | N/A |

### Time-Series Analytics: "Likes per minute for the last 24 hours"

| Database | Approach | Complexity |
|----------|----------|------------|
| **CockroachDB** | `GROUP BY date_trunc('minute', liked_at)` | Medium — SQL works but index-heavy |
| **Firestore** | Not supported natively — requires BigQuery export | Very High |
| **Neo4j** | Relationship property aggregation — not optimal | High |
| **ChromaDB** | Not applicable | N/A |
| **InfluxDB** | `aggregateWindow(every: 1m, fn: count)` | Low — native, optimised for this |

---

## 4. Consistency and Availability

| Database | CAP Classification | Consistency Guarantee | Availability Approach |
|----------|-------------------|-----------------------|----------------------|
| **CockroachDB** | CP (with high availability) | SERIALIZABLE — no dirty reads, phantoms, or write skew | Multi-active; Raft quorum required for writes |
| **Firestore** | CP (single document) / AP (some queries) | Strong for single-document reads; eventual for collection group queries | Globally replicated; available during regional failures |
| **Neo4j** | CA (single node) | ACID on single node; weaker with Fabric clusters | Single point of failure without clustering |
| **ChromaDB** | Not applicable (no replication) | No consistency guarantees — append store | Local only; no HA |
| **InfluxDB** | AP (TSM append-only) | Eventual — high write throughput prioritised over immediate read consistency | High write availability; queries may lag slightly |

---

## 5. Indexing Strategy Comparison

| Database | Primary Index | Secondary Index | Full-Text / Semantic Index |
|----------|--------------|-----------------|---------------------------|
| **CockroachDB** | Clustered B-tree on primary key | Explicit `CREATE INDEX` on any column | Limited (trigram index with extension) |
| **Firestore** | Automatic single-field indexes | Composite indexes (manually configured in console) | Not supported |
| **Neo4j** | Index-free adjacency (relationship pointers) | B-tree and full-text indexes on node properties | Full-text search via `CREATE FULLTEXT INDEX` |
| **ChromaDB** | HNSW graph index on embedding vectors | Metadata filtering (exact match) | Entire purpose — ANN index IS the primary index |
| **InfluxDB** | Time-ordered TSM columnar index | Tag-based inverted index (tags are always indexed) | Not supported |

---

## 6. Use Case Fit Summary

The following matrix rates each database's suitability for key social media operations (1 = poor, 5 = excellent):

| Operation | CockroachDB | Firestore | Neo4j | ChromaDB | InfluxDB |
|-----------|:-----------:|:---------:|:-----:|:--------:|:--------:|
| User profile CRUD | 5 | 5 | 3 | 1 | 1 |
| Post creation and feed | 5 | 4 | 4 | 1 | 1 |
| Comment threads | 5 | 5 | 3 | 1 | 1 |
| Follow/unfollow (write) | 4 | 4 | 5 | 1 | 1 |
| Friend-of-friend discovery | 2 | 1 | 5 | 1 | 1 |
| Influence / PageRank | 2 | 1 | 5 | 1 | 1 |
| Semantic post search | 1 | 1 | 2 | 5 | 1 |
| Content recommendation | 1 | 1 | 3 | 5 | 1 |
| Real-time feed sync | 2 | 5 | 1 | 1 | 1 |
| Engagement event ingestion | 3 | 2 | 1 | 1 | 5 |
| Trending topic detection | 3 | 2 | 2 | 1 | 5 |
| Activity heatmaps | 3 | 1 | 1 | 1 | 5 |
| Offline mobile support | 1 | 5 | 1 | 1 | 1 |

---

## 7. When to Use Each Technology

| Scenario | Recommended Database | Reason |
|----------|---------------------|--------|
| Storing user accounts and ensuring no duplicate usernames | **CockroachDB** | UNIQUE constraint + ACID transactions |
| Mobile app that must work offline and sync when online | **Firestore** | Built-in offline SDK + real-time listeners |
| "Who should I follow?" friend suggestion feature | **Neo4j** | Friends-of-friends traversal in a single Cypher query |
| "Find posts similar to ones I liked" recommendation | **ChromaDB** | Vector similarity search on post embeddings |
| "What posts are trending right now?" real-time analytics | **InfluxDB** | Native time-window aggregation on engagement event stream |
| Processing a "like" action that must be counted accurately | **CockroachDB** | Serializable transaction prevents double-counting |
| Storing a post that might have different fields in future | **Firestore** | Schemaless document model — no migration needed |

---

## 8. Architectural Decision Summary

For a production-grade social media platform, the optimal architecture would combine **all five** technologies in a polyglot persistence strategy:

```
                    ┌─────────────────────────────────────────┐
                    │         Social Media Platform           │
                    └────────────────┬────────────────────────┘
                                     │
        ┌────────────┬───────────────┼───────────────┬──────────────┐
        │            │               │               │              │
   CockroachDB   Firestore        Neo4j          ChromaDB       InfluxDB
   (Source of    (Real-time      (Social         (Semantic      (Engagement
   truth for     mobile sync     graph &         search &       analytics &
   core data)    & offline)      influence)      recommend.)    trending)
```

This polyglot approach is the standard architecture at companies like Twitter (X), LinkedIn, and Instagram — each of which uses multiple specialised databases rather than a single general-purpose one.

---

*This comparison is based on documented architectural properties. No live benchmarks were run.*
