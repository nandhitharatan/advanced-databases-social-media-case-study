# References

This document lists all academic papers, official documentation, technical books, and online resources referenced in this case study. References are grouped by database technology and then by general DBMS theory.

---

## General DBMS Theory

### CAP Theorem and Consistency

1. **Brewer, E. A.** (2000). *Towards Robust Distributed Systems*. Keynote address at PODC 2000. ACM Symposium on Principles of Distributed Computing.
   - Original presentation of the CAP conjecture.

2. **Gilbert, S., & Lynch, N.** (2002). *Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services*. ACM SIGACT News, 33(2), 51–59.
   - Formal proof of the CAP theorem.

3. **Abadi, D. J.** (2012). *Consistency Tradeoffs in Modern Distributed Database System Design: CAP is Only Part of the Story*. IEEE Computer, 45(2), 37–42.
   - Introduces the PACELC model extending CAP with latency/consistency trade-off.

### Foundational Database Textbooks

4. **Ramakrishnan, R., & Gehrke, J.** (2003). *Database Management Systems* (3rd ed.). McGraw-Hill Education.
   - Standard DBMS textbook covering relational algebra, SQL, indexing, and transactions.

5. **Date, C. J.** (2003). *An Introduction to Database Systems* (8th ed.). Addison-Wesley.
   - Foundational text on relational model, normalisation, and ACID properties.

6. **Kleppmann, M.** (2017). *Designing Data-Intensive Applications*. O'Reilly Media.
   - Comprehensive treatment of modern database architectures including distributed systems, consistency models, stream processing, and storage engines. Directly relevant to this case study.

### NoSQL and Polyglot Persistence

7. **Fowler, M., & Sadalage, P. J.** (2012). *NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot Persistence*. Addison-Wesley.
   - Covers document, graph, key-value, and column-family databases. Introduces the concept of polyglot persistence.

8. **Stonebraker, M., & Cetintemel, U.** (2005). *'One Size Fits All': An Idea Whose Time Has Come and Gone*. Proceedings of ICDE 2005.
   - Argues against the universal relational model and motivates specialised database engines.

---

## CockroachDB — Distributed Database

9. **CockroachDB Architecture Documentation**. Cockroach Labs.
   URL: https://www.cockroachlabs.com/docs/stable/architecture/overview.html

10. **Taft, R., Sharif, I., Matei, A., VanBenschoten, N., Lewis, A., Grieger, T., ... & Kimball, A.** (2020). *CockroachDB: The Resilient Geo-Distributed SQL Database*. Proceedings of ACM SIGMOD 2020. doi:10.1145/3318464.3386134
    - The primary academic paper on CockroachDB's design — Raft replication, distributed transactions, and geo-partitioning.

11. **Ongaro, D., & Ousterhout, J.** (2014). *In Search of an Understandable Consensus Algorithm*. Proceedings of USENIX ATC 2014.
    - Original Raft consensus algorithm paper — the foundation of CockroachDB's replication layer.

12. **Cockroach Labs Blog**. *Serializable, Lockless, Distributed: Isolation in CockroachDB*.
    URL: https://www.cockroachlabs.com/blog/serializable-lockless-distributed-isolation-cockroachdb/

---

## Firebase Firestore — Cloud Database

13. **Firebase Firestore Documentation**. Google LLC.
    URL: https://firebase.google.com/docs/firestore

14. **Firestore Data Model**. Google LLC.
    URL: https://firebase.google.com/docs/firestore/data-model

15. **Firestore Index Management**. Google LLC.
    URL: https://firebase.google.com/docs/firestore/query-data/indexing

16. **Firestore Distributed Counters**. Google LLC.
    URL: https://firebase.google.com/docs/firestore/solutions/counters

17. **Chang, F., Dean, J., Ghemawat, S., Hsieh, W. C., Wallach, D. A., Burrows, M., ... & Gruber, R. E.** (2008). *Bigtable: A Distributed Storage System for Structured Data*. ACM Transactions on Computer Systems, 26(2), 1–26.
    - Firestore's underlying storage engine is Bigtable. This paper is the primary reference for Bigtable's architecture.

---

## Neo4j — Graph Database

18. **Neo4j Documentation**. Neo4j, Inc.
    URL: https://neo4j.com/docs/

19. **Cypher Query Language Reference**. Neo4j, Inc.
    URL: https://neo4j.com/docs/cypher-manual/current/

20. **Robinson, I., Webber, J., & Eifrem, E.** (2015). *Graph Databases: New Opportunities for Connected Data* (2nd ed.). O'Reilly Media.
    - Comprehensive treatment of graph database concepts, the property graph model, and Cypher.

21. **Neo4j Graph Data Science Library Documentation**. Neo4j, Inc.
    URL: https://neo4j.com/docs/graph-data-science/current/

22. **Page, L., Brin, S., Motwani, R., & Winograd, T.** (1998). *The PageRank Citation Ranking: Bringing Order to the Web*. Technical Report, Stanford University.
    - Original PageRank algorithm paper — the basis for the GDS PageRank algorithm used in the follower influence demonstration.

---

## ChromaDB — Vector Database

23. **ChromaDB Documentation**. Chroma Core, Inc.
    URL: https://docs.trychroma.com/

24. **Malkov, Y. A., & Yashunin, D. A.** (2020). *Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs*. IEEE Transactions on Pattern Analysis and Machine Intelligence, 42(4), 824–836.
    - Original HNSW paper — the index algorithm powering ChromaDB's ANN search.

25. **Reimers, N., & Gurevych, I.** (2019). *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*. Proceedings of EMNLP 2019. arXiv:1908.10084.
    - The paper introducing sentence-transformers — the embedding model used to generate post embeddings in this case study.

26. **Johnson, J., Douze, M., & Jégou, H.** (2019). *Billion-Scale Similarity Search with GPUs*. IEEE Transactions on Big Data, 7(3), 535–547.
    - Background on large-scale vector similarity search — motivates the HNSW and IVF indexes used in production vector databases.

---

## InfluxDB — Time-Series Database

27. **InfluxDB 2.x Documentation**. InfluxData, Inc.
    URL: https://docs.influxdata.com/influxdb/v2/

28. **Flux Query Language Documentation**. InfluxData, Inc.
    URL: https://docs.influxdata.com/flux/v0/

29. **InfluxDB Storage Engine (TSM)**. InfluxData, Inc.
    URL: https://docs.influxdata.com/influxdb/v2/reference/internals/storage-engine/

30. **Jensen, S. K., Pedersen, T. B., & Thomsen, C.** (2017). *Time Series Management Systems: A Survey*. IEEE Transactions on Knowledge and Data Engineering, 29(11), 2581–2600.
    - Academic survey of time-series database design, covering storage engines, compression, and query optimisation.

---

## Online Courses and Supplementary Learning

31. **CS186: Introduction to Database Systems**. UC Berkeley (available on YouTube).
    - Covers B-trees, query optimisation, transactions, and distributed databases.

32. **CMU 15-445: Database Systems**. Carnegie Mellon University.
    URL: https://15445.courses.cs.cmu.edu/

33. **Neo4j Graph Academy**. Free online graph database courses.
    URL: https://graphacademy.neo4j.com/

34. **InfluxDB University**. Free online time-series database courses.
    URL: https://university.influxdata.com/

---

*References will be updated as additional sources are consulted during the implementation phase. All URLs were verified accessible as of the project start date.*
