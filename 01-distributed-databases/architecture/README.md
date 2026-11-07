# Architecture — CockroachDB

This directory will contain:

- **raft-replication.md** — Explanation of the Raft consensus protocol as implemented in CockroachDB, with a diagram showing how a 3-node cluster achieves quorum for write operations.
- **range-sharding.md** — How CockroachDB splits table data into 64 MB ranges and distributes them across nodes. Includes the range lease and leaseholder concept.
- **geo-partitioning.md** — How `PARTITION BY LIST` pins user data to geographic regions, reducing cross-region read latency.
- **cluster-diagram.png** — Diagram of a 3-node CockroachDB cluster with range distribution for the social media tables.

## Status

🔲 To be created during the implementation phase.
