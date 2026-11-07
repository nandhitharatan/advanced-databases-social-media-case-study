# Objectives

## Primary Objectives

The following objectives define the scope of work for this case study. Each objective corresponds to a deliverable that will be produced and documented in the repository.

---

### Objective 1: Understand the Theoretical Foundations

**Goal**: Develop a clear understanding of the architectural principles underlying each of the five database paradigms.

**Specific outcomes:**
- Explain the Raft consensus algorithm and how CockroachDB achieves distributed ACID transactions.
- Describe the Firestore document/collection model and how composite indexes work.
- Explain the property graph model, index-free adjacency, and the Cypher query language.
- Define vector embeddings, explain the HNSW index, and describe approximate nearest-neighbour search.
- Explain InfluxDB's TSM storage engine, measurement/tag/field model, and retention policies.

**Deliverable:** README.md files in each database directory (already created).

---

### Objective 2: Design Data Models for the Social Media Domain

**Goal**: Translate the shared synthetic social media dataset into five distinct database-specific schemas.

**Specific outcomes:**
- CockroachDB: Relational schema with appropriate primary keys, foreign keys, indexes, and constraints.
- Firestore: Document/collection hierarchy with denormalised fields and subcollection design.
- Neo4j: Property graph schema with node labels, relationship types, and property definitions.
- ChromaDB: Collection structure with embedding dimensions, metadata fields, and similarity metric.
- InfluxDB: Measurement definition with tag/field separation, bucket configuration, and retention policy.

**Deliverable:** Schema files in each database's `schema/` or `architecture/` directory.

---

### Objective 3: Implement Practical Query Demonstrations

**Goal**: Write and execute queries demonstrating each database's key capabilities applied to the social media dataset.

**Specific outcomes:**
- CockroachDB: Feed generation, transactional like increment, trending posts, follower count aggregation.
- Firestore: Document CRUD, subcollection queries, composite index query, transactional counter.
- Neo4j: Direct follower query, friends-of-friends traversal, shortest path, PageRank.
- ChromaDB: Embedding generation, semantic search, metadata-filtered search, content recommendation.
- InfluxDB: Time-window aggregation, trending detection, activity heatmap, downsampling task.

**Deliverable:** Query files in each database's `queries/` directory; executable scripts in `examples/`.

---

### Objective 4: Analyse Consistency and CAP Theorem Implications

**Goal**: Apply CAP theorem and PACELC theorem reasoning to each database's design, specifically in social media scenarios.

**Specific outcomes:**
- Classify each database as CP, AP, or CA with justification.
- Identify at least one specific social media scenario where the consistency model creates a practical problem (e.g., like_count contention in Firestore, Raft latency in CockroachDB).
- Describe the solution or workaround for each identified consistency problem.

**Deliverable:** [`comparison/database-comparison.md`](../comparison/database-comparison.md) — Consistency section.

---

### Objective 5: Compare Indexing Strategies

**Goal**: Understand and compare how each database indexes data to accelerate queries.

**Specific outcomes:**
- Explain and demonstrate the B-tree index in CockroachDB for feed generation queries.
- Explain and demonstrate Firestore's composite index requirement for multi-field queries.
- Explain index-free adjacency in Neo4j and contrast with SQL JOIN performance.
- Explain the HNSW index in ChromaDB and the concept of approximate nearest-neighbour search.
- Explain InfluxDB's tag-based inverted index and time-ordered TSM columnar storage.

**Deliverable:** [`comparison/performance-analysis.md`](../comparison/performance-analysis.md) — Indexing section.

---

### Objective 6: Evaluate Scalability Characteristics

**Goal**: Assess how each database scales horizontally or vertically and identify the practical limits relevant to a social media deployment.

**Specific outcomes:**
- Explain CockroachDB's range-based automatic sharding and how adding nodes increases throughput.
- Explain Firestore's serverless auto-scaling model and its per-document write rate limits.
- Identify the single-node limitation of Neo4j Community Edition and the Enterprise/Aura alternative.
- Identify ChromaDB's local-library nature and its implications for production deployment.
- Explain InfluxDB OSS's vertical scaling limit and when Cloud/Enterprise is needed.

**Deliverable:** [`comparison/performance-analysis.md`](../comparison/performance-analysis.md) — Scalability section.

---

### Objective 7: Produce a Cross-Technology Comparison

**Goal**: Synthesise findings into a structured comparison that answers the question: "Which database is best suited for which social media operation, and why?"

**Specific outcomes:**
- Complete comparison matrix across all five databases.
- Use case fit table with qualitative ratings for 13 social media operations.
- Architectural recommendation for a polyglot persistence strategy.

**Deliverable:** [`comparison/database-comparison.md`](../comparison/database-comparison.md).

---

## Progress Tracking

| Objective | Status |
|-----------|--------|
| 1. Theoretical foundations documented | ✅ Complete (READMEs) |
| 2. Data model design | 🔲 Pending (schema files) |
| 3. Query demonstrations | 🔲 Pending (queries + examples) |
| 4. CAP theorem analysis | ✅ Complete (comparison doc) |
| 5. Indexing strategy comparison | ✅ Complete (performance doc) |
| 6. Scalability evaluation | ✅ Complete (performance doc) |
| 7. Cross-technology comparison | ✅ Complete (comparison doc) |

---

*Objectives will be updated as implementation progresses.*
