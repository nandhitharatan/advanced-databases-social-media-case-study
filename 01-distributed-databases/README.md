# 01 — Distributed Database: CockroachDB

## Introduction

CockroachDB is a distributed relational database built on top of a Raft consensus protocol. It presents a standard PostgreSQL-compatible SQL interface while transparently replicating data across multiple nodes. The name reflects its design goal: surviving failures the way a cockroach survives catastrophe — by distributing itself so that no single point of failure brings the system down.

This section studies CockroachDB through the lens of the social media domain. A globally distributed social platform must ensure that user accounts, posts, and engagement records are consistent across data centres in different geographic regions, even when individual nodes or regions fail.

---

## Purpose in the Social Media Domain

Social media platforms operate globally. When a user in Mumbai publishes a post, a follower in London must see that post immediately and consistently — not an outdated or partial version. Traditional single-node relational databases cannot guarantee this at global scale.

CockroachDB addresses this by:

- **Replicating every row** across a minimum of three nodes (Raft replication factor = 3).
- **Providing SERIALIZABLE isolation** so concurrent writes (e.g., two users liking the same post simultaneously) do not produce inconsistent results.
- **Geo-partitioning tables** so user data is pinned close to the users' actual geographic region, reducing read latency.

In this case study, CockroachDB is responsible for the **core transactional data layer**: Users, Posts, Comments, Likes, and Shares — the entities that require strong consistency and cannot tolerate stale reads.

---

## Core DBMS Concepts Involved

| Concept | How It Applies Here |
|---------|-------------------|
| **Distributed Transactions** | A single "post with initial like" operation spanning multiple table rows must be atomic |
| **Raft Consensus** | Each range of data achieves consensus across replicas before a write is acknowledged |
| **Range-Based Sharding** | Rows are split into ranges (~64 MB) and distributed automatically |
| **SERIALIZABLE Isolation** | The highest isolation level — prevents phantom reads, dirty reads, and write skew |
| **Geo-Partitioning** | Pin user data to the region closest to that user (e.g., India, US, Europe) |
| **Multi-Active Availability** | All nodes accept reads and writes (unlike traditional primary-replica setups) |
| **SQL Compatibility** | Uses PostgreSQL wire protocol — standard JOINs, indexes, and constraints apply |

---

## Proposed Data Model

CockroachDB uses a relational schema. The following tables represent the social media entities:

```sql
-- Users table
CREATE TABLE users (
    user_id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username    STRING NOT NULL UNIQUE,
    email       STRING NOT NULL UNIQUE,
    full_name   STRING NOT NULL,
    bio         STRING,
    location    STRING,
    joined_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    is_verified BOOL NOT NULL DEFAULT FALSE
);

-- Posts table
CREATE TABLE posts (
    post_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    author_id     UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    content       STRING NOT NULL,
    media_type    STRING CHECK (media_type IN ('text', 'image', 'video')) DEFAULT 'text',
    tags          STRING[],
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    like_count    INT NOT NULL DEFAULT 0,
    comment_count INT NOT NULL DEFAULT 0,
    share_count   INT NOT NULL DEFAULT 0,
    INDEX idx_posts_author (author_id),
    INDEX idx_posts_created (created_at DESC)
);

-- Comments table
CREATE TABLE comments (
    comment_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id           UUID NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
    author_id         UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    content           STRING NOT NULL,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    parent_comment_id UUID REFERENCES comments(comment_id),
    INDEX idx_comments_post (post_id),
    INDEX idx_comments_created (created_at)
);

-- Likes table
CREATE TABLE likes (
    like_id  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id  UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    post_id  UUID NOT NULL REFERENCES posts(post_id) ON DELETE CASCADE,
    liked_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE INDEX unique_user_post_like (user_id, post_id)
);

-- Follows table
CREATE TABLE follows (
    follow_id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    follower_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    followee_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    followed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE INDEX unique_follow_pair (follower_id, followee_id),
    INDEX idx_follows_followee (followee_id)
);
```

---

## Planned Practical Demonstrations

The following demonstrations will be implemented in the `examples/` directory:

1. **Schema creation and seeding** — Creating the schema and inserting synthetic dataset records.
2. **Distributed transaction demo** — Atomically creating a post and incrementing a counter using `BEGIN` / `COMMIT`.
3. **Geo-partitioned query** — Demonstrating how `PARTITION BY` pins rows to a specific region.
4. **Conflict resolution** — Simulating two concurrent `UPDATE likes SET like_count = like_count + 1` on the same row under SERIALIZABLE isolation.
5. **Index utilisation** — Running `EXPLAIN (DISTSQL)` to show how secondary indexes speed up feed generation queries.

---

## Expected Queries and Operations

### Feed Generation
```sql
-- Retrieve posts from users that u001 follows, ordered by recency
SELECT p.post_id, p.content, p.created_at, u.username
FROM   posts p
JOIN   users u ON p.author_id = u.user_id
WHERE  p.author_id IN (
    SELECT followee_id FROM follows WHERE follower_id = 'u001'
)
ORDER BY p.created_at DESC
LIMIT  20;
```

### Like Consistency (Transactional Update)
```sql
BEGIN;
INSERT INTO likes (user_id, post_id) VALUES ('u003', 'p001');
UPDATE posts SET like_count = like_count + 1 WHERE post_id = 'p001';
COMMIT;
```

### Trending Posts (Last 24 Hours)
```sql
SELECT p.post_id, p.content, p.like_count
FROM   posts p
WHERE  p.created_at >= now() - INTERVAL '24 hours'
ORDER  BY p.like_count DESC
LIMIT  10;
```

### Follower Count Aggregation
```sql
SELECT followee_id AS user_id, COUNT(*) AS follower_count
FROM   follows
GROUP  BY followee_id
ORDER  BY follower_count DESC
LIMIT  5;
```

---

## Advantages and Limitations

### Advantages
- **Strong consistency**: SERIALIZABLE isolation prevents data anomalies — critical for financial-grade operations like counters and unique constraints.
- **Fault tolerance**: Raft replication means the platform survives node failures without downtime.
- **SQL familiarity**: Developers use standard SQL — no new query language to learn.
- **Automatic sharding**: The database handles data distribution without manual shard management.
- **Geo-partitioning**: Reduces cross-region latency by keeping data close to users.

### Limitations
- **Higher latency for writes**: Raft consensus requires a quorum acknowledgement — write latency is higher than a single-node database.
- **Operational complexity**: Running a distributed cluster requires infrastructure management (even on a local test cluster).
- **Not optimised for graph queries**: Complex follow/friend-of-friend queries become expensive JOINs — a graph database is better suited for these.
- **Not optimised for time-series**: High-frequency event ingestion (thousands of likes per second) is better handled by InfluxDB.
- **Cost**: Cloud-hosted CockroachDB (Cockroach Serverless/Dedicated) has pricing implications at scale.

---

## Directory Contents

| Path | Purpose |
|------|---------|
| `architecture/` | Diagrams and notes on CockroachDB's Raft replication and range distribution |
| `queries/` | SQL query files demonstrating feed, trending, and transactional operations |
| `examples/` | Python scripts to connect, seed data, and run demonstration queries |
| `screenshots/` | CockroachDB Admin UI screenshots captured during demonstrations |

---

*Navigate to `queries/` and `examples/` as they are implemented for executable code.*
