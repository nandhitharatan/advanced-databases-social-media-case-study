# Performance Analysis — Conceptual Study

> **Important:** This document presents a **conceptual performance analysis** based on the documented architectural properties of each database system. No live benchmarks, load tests, or profiling experiments have been conducted. Performance characteristics described here reflect publicly documented behaviour and theoretical analysis. Actual numbers will vary significantly based on hardware, dataset size, query patterns, and configuration.

---

## 1. Performance Dimensions in a Social Media Platform

Performance in a social media context is not a single metric. The following dimensions matter:

| Dimension | Definition | Social Media Example |
|-----------|------------|---------------------|
| **Write Throughput** | How many write operations per second the database can sustain | Ingesting 50,000 like events per minute during a viral post |
| **Read Latency** | Time to retrieve a single record or query result | Loading a user's profile in under 100ms |
| **Query Complexity** | How well the database handles complex, multi-entity queries | Generating a personalised feed from 500 followed accounts |
| **Scalability** | How performance changes as data volume or concurrent users grow | Maintaining p99 latency as the user base grows from 1K to 1M |
| **Consistency Overhead** | Cost of enforcing strong consistency guarantees | SERIALIZABLE isolation adds latency vs. eventual consistency |

---

## 2. Write Throughput Analysis

### Social Media Write Patterns

Social media platforms have **asymmetric write patterns**:
- **Low frequency, high value**: User profile updates, post creation (once per session, not per second)
- **Medium frequency**: Comments, follows, shares (dozens per minute platform-wide)
- **High frequency**: Likes, views, real-time engagement events (thousands per second during peak)

| Database | Write Model | Expected Write Throughput | Social Media Fit |
|----------|-------------|--------------------------|-----------------|
| **CockroachDB** | Raft consensus (quorum write) | Thousands of TPS per node; latency higher due to Raft round-trip | Good for user/post writes; overkill for event streams |
| **Firestore** | Serverless auto-scale | Very high (Google manages scaling); but 1 write/second per document limit | Good for post writes; problematic for rapid like_count increments |
| **Neo4j** | ACID commit to WAL | Hundreds to thousands per second (single node) | Good for follow/unfollow events; not for event streams |
| **ChromaDB** | Append-only embedding store | Moderate; batch inserts preferred | Suitable for embedding ingestion (batch, not real-time) |
| **InfluxDB** | TSM append-only | Millions of data points per second | Excellent for all engagement event streams |

### Key Insight: The like_count Problem

Consider a viral post receiving 10,000 likes per minute. Each like must increment a counter on the post record:

- **CockroachDB**: Uses a serializable transaction — only one transaction can hold the write lock. At 10,000 likes/min (~167/sec), contention on a single row causes serialization retries. Solution: use `crdb_internal.increment` or a separate counter table.
- **Firestore**: Has a documented **1 write per second per document** limit for single-field updates. Solution: use **distributed counters** (sharded sub-documents summed at read time).
- **Neo4j**: Node property updates require an exclusive lock — similar contention issue to CockroachDB at high rates.
- **InfluxDB**: Each like is an independent append — no contention. The count is derived at query time via aggregation. This is the architecturally correct approach for high-frequency counters.

---

## 3. Read Latency Analysis

### Single Record Reads

For retrieving a single user profile or post:

| Database | Read Mechanism | Expected Latency |
|----------|---------------|-----------------|
| **CockroachDB** | B-tree index lookup on primary key | 1–10ms (single node); 50–150ms (cross-region) |
| **Firestore** | Document lookup by path | 50–200ms (varies by region and cold start) |
| **Neo4j** | Node lookup by ID index | 1–5ms (in-memory cache hit) |
| **ChromaDB** | Direct ID lookup in collection | < 5ms (local, in-memory) |
| **InfluxDB** | Tag-indexed time-range scan | Depends on time range; 5–50ms for recent data |

### Complex Query Reads

The gap between databases widens significantly for complex queries:

**Feed generation** (posts from 500 followed accounts, last 20 posts):

| Database | Approach | Relative Performance |
|----------|----------|---------------------|
| **CockroachDB** | Distributed JOIN with index scan | Moderate — well-suited with covering indexes |
| **Firestore** | 500 separate queries + client merge | Slow — Firestore cannot JOIN across documents |
| **Neo4j** | Single graph traversal | Fast — native index-free adjacency |
| **ChromaDB** | Not applicable | — |
| **InfluxDB** | Not applicable | — |

**Friend-of-friend discovery** (depth-2 follow traversal):

| Database | Expected Performance | Why |
|----------|---------------------|-----|
| **CockroachDB** | Degrades with depth | Self-JOIN on `follows` grows quadratically |
| **Firestore** | Impractical | Requires N+1 reads for each degree |
| **Neo4j** | Constant time per hop | Index-free adjacency — traversal is O(degree) not O(table size) |

---

## 4. Scalability Characteristics

### Horizontal vs. Vertical Scaling

| Database | Scaling Model | Practical Implication |
|----------|-------------|----------------------|
| **CockroachDB** | Horizontal (add nodes; Raft automatically rebalances ranges) | True horizontal scalability — add nodes to increase throughput and storage |
| **Firestore** | Automatic serverless scaling (Google manages) | No manual scaling; but costs scale linearly with operations |
| **Neo4j (Community)** | Vertical only | Constrained by single machine's RAM and CPU |
| **ChromaDB** | Vertical (local library) | Not designed for distributed deployment in this form |
| **InfluxDB (OSS)** | Vertical only (Enterprise/Cloud for horizontal) | High-performance single node; horizontal requires paid tier |

### Social Media Scale Projections (Conceptual)

The following represents expected relative performance as data volume scales (not measured values):

| Data Size | CockroachDB | Firestore | Neo4j | ChromaDB | InfluxDB |
|-----------|:-----------:|:---------:|:-----:|:--------:|:--------:|
| 1K users, 10K posts | Excellent | Excellent | Excellent | Excellent | Excellent |
| 100K users, 1M posts | Excellent | Excellent | Good | Good | Excellent |
| 1M users, 100M posts | Excellent (with sharding) | Excellent (serverless) | Degraded (single node) | Degraded (local) | Good (single node limits) |
| 100M users, 10B posts | Excellent | Excellent | Requires Enterprise | Requires migration | Requires Cloud/Enterprise |

---

## 5. Index Performance Analysis

### Indexing Strategy Impact

| Scenario | Best Database + Index | Why |
|----------|----------------------|-----|
| Find posts by author | CockroachDB: `INDEX ON posts(author_id)` | B-tree secondary index — O(log n) lookup |
| Find posts by tag | Firestore: `ARRAY_CONTAINS` composite index | Automatic inverted index on array fields |
| Find second-degree followers | Neo4j: index-free adjacency | No index needed — pointers stored in relationship structure |
| Find semantically similar posts | ChromaDB: HNSW index | ANN graph index — sub-linear query time |
| Find events in last 5 minutes | InfluxDB: TSM time index | Columnar time-ordered storage — range scans are sequential reads |

### The HNSW Index (ChromaDB)

The **Hierarchical Navigable Small World (HNSW)** index used by ChromaDB deserves special explanation:

- Traditional exact nearest-neighbour search is O(n × d) where n = number of vectors and d = dimensions.
- HNSW builds a layered graph structure that enables approximate search in O(log n) time.
- The trade-off is **recall** — HNSW may miss some true nearest neighbours for the sake of speed.
- For 15 posts (this case study's dataset), brute force and HNSW are equivalent; the index advantage becomes apparent at millions of vectors.

---

## 6. Consistency vs. Performance Trade-off

This is the fundamental trade-off articulated by the **CAP Theorem** and extended by the **PACELC Theorem**.

### PACELC Analysis for Each Database

PACELC states: "In case of Partition (P), choose Availability (A) or Consistency (C); Else (E), choose Latency (L) or Consistency (C)."

| Database | P → A or C? | E → L or C? | Practical Implication |
|----------|------------|------------|----------------------|
| **CockroachDB** | C (Raft quorum required) | C (SERIALIZABLE adds latency) | Writes block until quorum — higher latency, guaranteed correctness |
| **Firestore** | A (available during regional issues) | L (eventual for some reads) | Lower latency single-doc reads; eventual for collection queries |
| **Neo4j** | C (single node ACID) | C (WAL commit required) | Strong consistency; no partition tolerance without clustering |
| **ChromaDB** | N/A (no replication) | L (no consistency overhead) | Lowest latency; no durability guarantees without disk persistence |
| **InfluxDB** | A (TSM appends succeed) | L (eventual consistency) | Highest write throughput; reads may lag slightly behind latest writes |

---

## 7. Performance Recommendations for This Case Study

Based on the analysis above, the following recommendations apply to the social media scenario studied here:

| Recommendation | Rationale |
|---------------|-----------|
| Use **InfluxDB** for all like, view, and engagement event ingestion | Avoids write contention and counter hotspot problems |
| Use **Neo4j** for all follower graph queries deeper than 1 hop | SQL JOINs become exponentially expensive; Neo4j traversal remains constant |
| Use **CockroachDB** for user profile and post storage | ACID transactions prevent data anomalies for high-value writes |
| Use **Firestore** for mobile client synchronisation | Real-time listeners eliminate polling; offline support is native |
| Use **ChromaDB** for content search and recommendation | No other database in this study supports semantic similarity |
| Avoid storing high-cardinality content (post text) in InfluxDB tags | Tag cardinality blows up the TSM index — use fields or external metadata |
| Implement distributed counters in Firestore for like_count | Avoids the 1 write/second per document limitation |

---

## 8. Summary Table

| Performance Aspect | CockroachDB | Firestore | Neo4j | ChromaDB | InfluxDB |
|-------------------|:-----------:|:---------:|:-----:|:--------:|:--------:|
| Write throughput (events) | Medium | Medium | Medium | Low | Very High |
| Read latency (single record) | Low | Medium | Very Low | Very Low | Low |
| Complex join queries | Good | Poor | Excellent | N/A | N/A |
| Graph traversal | Poor | Very Poor | Excellent | N/A | N/A |
| Semantic search | Poor | Poor | Poor | Excellent | N/A |
| Time-series aggregation | Medium | Poor | Poor | N/A | Excellent |
| Horizontal scalability | Excellent | Excellent | Poor (OSS) | Poor | Poor (OSS) |
| Consistency guarantee | Strong | Mixed | Strong | None | Eventual |

---

*All performance assessments in this document are conceptual and based on published architecture documentation, not measured experiments. Actual performance depends heavily on deployment configuration, hardware, and workload characteristics.*
