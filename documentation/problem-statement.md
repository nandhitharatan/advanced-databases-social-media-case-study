# Problem Statement

## Case Study Title

**Design and Analysis of Advanced Database Technologies in a Social Media Platform**

---

## Background

The evolution of web applications — particularly social media platforms — has exposed fundamental limitations of traditional relational database management systems (RDBMS). Systems like Facebook, Twitter (X), Instagram, and LinkedIn serve hundreds of millions of users concurrently, managing diverse data types, complex relationship networks, real-time event streams, and intelligent content search at a scale that no single database technology can optimally support.

This diversity of data characteristics has driven the emergence of specialised database paradigms: distributed databases for global consistency, cloud-native document stores for flexible schema and real-time sync, graph databases for relationship-centric queries, vector databases for semantic search, and time-series databases for high-frequency temporal data.

Each paradigm represents a deliberate architectural trade-off — optimising for a specific class of data access pattern at the expense of others. Understanding these trade-offs is a critical skill for any database engineer or backend architect.

---

## Problem Description

A hypothetical social media platform, **SocialGraph**, serves a global user base and must support the following core capabilities:

1. **User management**: Storing and updating user profiles with strong consistency guarantees. Duplicate usernames must be prevented even under concurrent registration.

2. **Content management**: Storing posts, comments, and media references. The schema must be flexible enough to accommodate new post types without full table migrations.

3. **Social network navigation**: Answering graph-structured queries about the follower/following network — including multi-hop traversals, mutual connections, and influence ranking.

4. **Semantic content discovery**: Enabling users to search for posts by meaning rather than keyword — returning relevant content even when the exact search terms do not appear in the posts.

5. **Real-time engagement analytics**: Tracking likes, views, shares, and comments as a continuous time-series stream, enabling trending topic detection, activity heatmaps, and anomaly alerts.

6. **Mobile and offline access**: Synchronising user feeds to mobile devices in real time, with graceful degradation when network connectivity is unavailable.

---

## Research Questions

This case study investigates the following questions:

1. **Data Model Fit**: How does each database technology's data model (relational, document, graph, vector, time-series) map to the social media domain? Where does the mapping feel natural, and where does it require awkward workarounds?

2. **Query Expressiveness**: How does each technology's query language or API express common social media operations (feed generation, friend discovery, trending detection)? What operations are easy, and what is difficult or impossible?

3. **Consistency vs. Availability**: What consistency guarantees does each technology provide, and what are the practical implications for social media operations (e.g., the like_count consistency problem)?

4. **Indexing and Performance**: What indexing strategies does each technology employ, and how do they affect the performance of different query patterns (point lookups, range scans, graph traversals, ANN search, time-window aggregations)?

5. **Scalability Characteristics**: How does each technology scale horizontally or vertically, and what are the practical limits for a social media deployment?

6. **Polyglot Persistence**: Given that no single technology excels at all five capabilities, what is the optimal combination? How should data be partitioned across multiple specialised databases?

---

## Hypothesis

No single database technology is optimal for all data access patterns present in a modern social media platform. A **polyglot persistence architecture** — combining CockroachDB, Firestore, Neo4j, ChromaDB, and InfluxDB for their respective strengths — will provide significantly better performance, expressiveness, and scalability than any single-database approach.

---

## Scope

**Included:**
- Theoretical analysis of five database paradigms
- Schema and data model design for each technology applied to the social media domain
- Practical query demonstrations on a shared synthetic dataset (10 users, 15 posts, 20 follows, 25 likes, 10 shares, 20 comments)
- Comparative analysis of indexing, consistency models, and scalability characteristics
- Conceptual performance discussion based on documented architectural properties

**Excluded:**
- Building a frontend application
- Production-grade security configurations or authentication
- Real user data or live API integrations
- Automated performance benchmarking or profiling
- Cloud infrastructure costs analysis

---

## Significance

This case study develops the following competencies:

- **Database selection skills**: Given a data access pattern, identify the most appropriate database paradigm.
- **Schema design**: Translate a business domain model into database-specific schemas across five different paradigms.
- **Query literacy**: Read and write queries in SQL (CockroachDB), Firestore SDK, Cypher (Neo4j), Python SDK (ChromaDB), and Flux (InfluxDB).
- **Trade-off analysis**: Critically evaluate the CAP theorem implications for each database in a real use case.
- **System thinking**: Understand how multiple specialised databases can be combined into a coherent architecture.

---

*This problem statement defines the academic scope of the case study. All data, scenarios, and platform details are fictional and synthetic.*
