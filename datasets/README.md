# Synthetic Dataset — Social Media Case Study

This directory documents the **shared synthetic dataset** used across all five database technology demonstrations in this case study. Every database section (CockroachDB, Firestore, Neo4j, ChromaDB, InfluxDB) draws from the same logical data model so that comparisons remain meaningful.

> **Important:** All data is entirely fictional. No real personal information is used. Names, usernames, emails, and content are generated for academic demonstration purposes only.

---

## Data Entities Overview

The dataset models a minimal but representative social media platform with the following core entities:

| Entity | Description |
|--------|-------------|
| **User** | A registered account on the platform |
| **Post** | A piece of content published by a user |
| **Comment** | A reply attached to a post |
| **Follow** | A directional relationship: User A follows User B |
| **Like** | A user expressing appreciation for a post |
| **Share** | A user redistributing a post to their own feed |
| **EngagementEvent** | A timestamped record of any user interaction |

---

## Entity Field Definitions

### 1. User

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `user_id` | UUID / String | Unique identifier | `u001` |
| `username` | String | Public display handle | `alex_nova` |
| `email` | String | Account email (fictional) | `alex@example.com` |
| `full_name` | String | Display name | `Alex Nova` |
| `bio` | String | Short profile description | `"Coffee enthusiast. Dev."` |
| `location` | String | City / region | `Mumbai` |
| `joined_at` | Timestamp | Account creation time | `2023-01-15T10:00:00Z` |
| `is_verified` | Boolean | Verified account flag | `false` |
| `follower_count` | Integer | Denormalised count (cached) | `342` |
| `following_count` | Integer | Denormalised count (cached) | `128` |

**Sample Users (10 fictional accounts):**

| user_id | username | full_name | location | joined_at |
|---------|----------|-----------|----------|-----------|
| u001 | alex_nova | Alex Nova | Mumbai | 2023-01-15 |
| u002 | priya_codes | Priya Sharma | Bangalore | 2023-02-20 |
| u003 | ravi_design | Ravi Kumar | Chennai | 2023-03-05 |
| u004 | meera_ml | Meera Nair | Hyderabad | 2023-04-10 |
| u005 | dev_sanjay | Sanjay Dev | Pune | 2023-05-18 |
| u006 | tech_tara | Tara Reddy | Delhi | 2023-06-22 |
| u007 | kiran_data | Kiran Patel | Ahmedabad | 2023-07-30 |
| u008 | ananya_art | Ananya Singh | Kolkata | 2023-08-14 |
| u009 | rohan_cloud | Rohan Verma | Jaipur | 2023-09-01 |
| u010 | ishaan_sec | Ishaan Mehta | Chandigarh | 2023-10-25 |

---

### 2. Post

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `post_id` | UUID / String | Unique identifier | `p001` |
| `author_id` | Reference → User | Author's user_id | `u001` |
| `content` | String | Text body of the post | `"Just deployed my first distributed DB!"` |
| `media_type` | Enum | `text`, `image`, `video` | `text` |
| `tags` | Array[String] | Hashtag labels | `["databases", "tech"]` |
| `created_at` | Timestamp | Post creation time | `2024-01-10T14:30:00Z` |
| `like_count` | Integer | Denormalised like total | `47` |
| `comment_count` | Integer | Denormalised comment total | `12` |
| `share_count` | Integer | Denormalised share total | `5` |

**Sample Posts (15 fictional posts):**

| post_id | author_id | content (abbreviated) | tags | created_at |
|---------|-----------|----------------------|------|------------|
| p001 | u001 | "Just deployed CockroachDB on three nodes..." | [databases, distributed] | 2024-01-10 |
| p002 | u002 | "Graph databases changed how I think about relationships..." | [neo4j, graphs] | 2024-01-12 |
| p003 | u003 | "My UI design process for data dashboards..." | [design, ui] | 2024-01-14 |
| p004 | u004 | "Machine learning pipeline update — vector search is incredible..." | [ml, vectordb] | 2024-01-16 |
| p005 | u005 | "Time-series data: why InfluxDB beats PostgreSQL for metrics..." | [influxdb, timeseries] | 2024-01-18 |
| p006 | u001 | "Firebase Firestore real-time sync demo..." | [firebase, cloud] | 2024-01-20 |
| p007 | u006 | "Cypher query to find friends-of-friends..." | [neo4j, graphs] | 2024-01-22 |
| p008 | u007 | "ChromaDB embeddings for semantic post search..." | [chromadb, ai] | 2024-01-24 |
| p009 | u008 | "Sketching a distributed system architecture..." | [architecture, design] | 2024-01-26 |
| p010 | u002 | "CockroachDB SERIALIZABLE isolation explained..." | [databases, consistency] | 2024-01-28 |
| p011 | u009 | "Cloud database cost comparison: Firestore vs DynamoDB..." | [cloud, databases] | 2024-02-01 |
| p012 | u003 | "Visual guide to graph traversal algorithms..." | [graphs, algorithms] | 2024-02-03 |
| p013 | u010 | "Security considerations when storing embeddings..." | [security, vectordb] | 2024-02-05 |
| p014 | u004 | "Retention policies in InfluxDB for social analytics..." | [influxdb, analytics] | 2024-02-07 |
| p015 | u005 | "Comparing CAP theorem implications across our five DBs..." | [cap, databases] | 2024-02-10 |

---

### 3. Comment

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `comment_id` | UUID / String | Unique identifier | `c001` |
| `post_id` | Reference → Post | Parent post | `p001` |
| `author_id` | Reference → User | Commenter's user_id | `u002` |
| `content` | String | Comment text | `"Great post! Which region did you deploy to?"` |
| `created_at` | Timestamp | Comment creation time | `2024-01-10T15:00:00Z` |
| `parent_comment_id` | String / null | For nested replies | `null` |

**Sample Comments (20 fictional comments):**

| comment_id | post_id | author_id | content (abbreviated) | created_at |
|------------|---------|-----------|----------------------|------------|
| c001 | p001 | u002 | "Which region did you deploy to?" | 2024-01-10T15:00 |
| c002 | p001 | u004 | "CockroachDB's Raft implementation is impressive." | 2024-01-10T15:30 |
| c003 | p001 | u006 | "Did you test geo-partitioning?" | 2024-01-10T16:00 |
| c004 | p002 | u001 | "Agreed! Cypher makes traversal so readable." | 2024-01-12T10:00 |
| c005 | p002 | u007 | "Neo4j vs ArangoDB — have you tried both?" | 2024-01-12T11:00 |
| c006 | p004 | u002 | "Which embedding model are you using?" | 2024-01-16T09:00 |
| c007 | p004 | u008 | "ChromaDB + sentence-transformers is my go-to." | 2024-01-16T09:30 |
| c008 | p005 | u001 | "The write throughput difference is massive." | 2024-01-18T14:00 |
| c009 | p005 | u009 | "Flux queries took time to learn but are powerful." | 2024-01-18T14:45 |
| c010 | p006 | u003 | "Real-time listeners are a game changer for mobile." | 2024-01-20T08:00 |
| c011 | p007 | u005 | "MATCH (a)-[:FOLLOWS*2]->(b) is so elegant." | 2024-01-22T17:00 |
| c012 | p008 | u010 | "What distance metric are you using — cosine or L2?" | 2024-01-24T12:00 |
| c013 | p010 | u003 | "SERIALIZABLE vs SNAPSHOT — always a tricky choice." | 2024-01-28T11:00 |
| c014 | p011 | u006 | "Firestore pricing can get unpredictable at scale." | 2024-02-01T10:00 |
| c015 | p012 | u004 | "BFS vs DFS in graph DBs matters a lot for social nets." | 2024-02-03T16:00 |
| c016 | p013 | u002 | "Embedding privacy is an underrated concern." | 2024-02-05T09:00 |
| c017 | p014 | u007 | "Downsampling long-retention data is key." | 2024-02-07T13:00 |
| c018 | p015 | u001 | "CAP trade-offs are most visible in CockroachDB vs Firestore." | 2024-02-10T10:00 |
| c019 | p015 | u008 | "Great summary! Would add PACELC as an extension." | 2024-02-10T10:30 |
| c020 | p001 | u005 | "Three-node setup is a great starting point." | 2024-01-11T08:00 |

---

### 4. Follow (Relationship)

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `follow_id` | UUID / String | Unique identifier | `f001` |
| `follower_id` | Reference → User | The user who follows | `u001` |
| `followee_id` | Reference → User | The user being followed | `u002` |
| `followed_at` | Timestamp | Relationship creation time | `2024-01-05T09:00:00Z` |

**Sample Follow Relationships (20 directed edges):**

| follow_id | follower_id | followee_id | followed_at |
|-----------|-------------|-------------|-------------|
| f001 | u001 | u002 | 2024-01-05 |
| f002 | u001 | u004 | 2024-01-06 |
| f003 | u002 | u001 | 2024-01-07 |
| f004 | u002 | u003 | 2024-01-08 |
| f005 | u003 | u002 | 2024-01-09 |
| f006 | u003 | u006 | 2024-01-10 |
| f007 | u004 | u001 | 2024-01-11 |
| f008 | u004 | u005 | 2024-01-12 |
| f009 | u005 | u004 | 2024-01-13 |
| f010 | u005 | u007 | 2024-01-14 |
| f011 | u006 | u001 | 2024-01-15 |
| f012 | u006 | u008 | 2024-01-16 |
| f013 | u007 | u005 | 2024-01-17 |
| f014 | u007 | u009 | 2024-01-18 |
| f015 | u008 | u006 | 2024-01-19 |
| f016 | u008 | u010 | 2024-01-20 |
| f017 | u009 | u007 | 2024-01-21 |
| f018 | u009 | u002 | 2024-01-22 |
| f019 | u010 | u008 | 2024-01-23 |
| f020 | u010 | u003 | 2024-01-24 |

---

### 5. Like

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `like_id` | UUID / String | Unique identifier | `l001` |
| `user_id` | Reference → User | User who liked | `u003` |
| `post_id` | Reference → Post | Post that was liked | `p001` |
| `liked_at` | Timestamp | Timestamp of the like | `2024-01-10T16:30:00Z` |

**Sample Likes (25 records):**

| like_id | user_id | post_id | liked_at |
|---------|---------|---------|----------|
| l001 | u002 | p001 | 2024-01-10T16:00 |
| l002 | u003 | p001 | 2024-01-10T16:10 |
| l003 | u004 | p001 | 2024-01-10T16:20 |
| l004 | u005 | p001 | 2024-01-10T16:30 |
| l005 | u001 | p002 | 2024-01-12T12:00 |
| l006 | u007 | p002 | 2024-01-12T12:30 |
| l007 | u008 | p002 | 2024-01-12T13:00 |
| l008 | u001 | p004 | 2024-01-16T10:00 |
| l009 | u003 | p004 | 2024-01-16T10:30 |
| l010 | u006 | p004 | 2024-01-16T11:00 |
| l011 | u002 | p005 | 2024-01-18T15:00 |
| l012 | u004 | p005 | 2024-01-18T15:30 |
| l013 | u009 | p005 | 2024-01-18T16:00 |
| l014 | u001 | p007 | 2024-01-22T18:00 |
| l015 | u004 | p007 | 2024-01-22T18:30 |
| l016 | u010 | p008 | 2024-01-24T13:00 |
| l017 | u001 | p010 | 2024-01-28T12:00 |
| l018 | u006 | p010 | 2024-01-28T12:30 |
| l019 | u003 | p011 | 2024-02-01T11:00 |
| l020 | u005 | p012 | 2024-02-03T17:00 |
| l021 | u002 | p013 | 2024-02-05T10:00 |
| l022 | u007 | p014 | 2024-02-07T14:00 |
| l023 | u003 | p015 | 2024-02-10T11:00 |
| l024 | u006 | p015 | 2024-02-10T11:30 |
| l025 | u009 | p015 | 2024-02-10T12:00 |

---

### 6. Share

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `share_id` | UUID / String | Unique identifier | `s001` |
| `user_id` | Reference → User | User who shared | `u004` |
| `post_id` | Reference → Post | Post that was shared | `p002` |
| `shared_at` | Timestamp | Timestamp of the share | `2024-01-13T09:00:00Z` |
| `note` | String / null | Optional reshare comment | `"Worth reading!"` |

**Sample Shares (10 records):**

| share_id | user_id | post_id | shared_at | note |
|----------|---------|---------|-----------|------|
| s001 | u004 | p002 | 2024-01-13T09:00 | "Graph DBs for social networks!" |
| s002 | u006 | p001 | 2024-01-11T10:00 | null |
| s003 | u003 | p005 | 2024-01-19T08:00 | "InfluxDB is underrated." |
| s004 | u007 | p004 | 2024-01-17T14:00 | null |
| s005 | u001 | p015 | 2024-02-11T09:00 | "Great CAP analysis!" |
| s006 | u008 | p007 | 2024-01-23T10:00 | null |
| s007 | u002 | p008 | 2024-01-25T11:00 | "ChromaDB demo is excellent." |
| s008 | u005 | p011 | 2024-02-02T15:00 | null |
| s009 | u009 | p010 | 2024-01-29T13:00 | "Important for consistency study." |
| s010 | u010 | p012 | 2024-02-04T16:00 | null |

---

### 7. EngagementEvent (Time-Series)

This entity is primarily used in the **InfluxDB** demonstration but provides context for the other databases as well.

| Field | Data Type | Description | Example |
|-------|-----------|-------------|---------|
| `event_id` | UUID / String | Unique identifier | `e001` |
| `event_type` | Enum | `like`, `comment`, `share`, `view`, `follow` | `like` |
| `user_id` | String (tag) | Actor performing the event | `u003` |
| `post_id` | String (tag) | Target post (if applicable) | `p001` |
| `timestamp` | Timestamp | Exact event time (ns precision) | `2024-01-10T16:00:00.000Z` |
| `session_duration_s` | Integer / null | Session time in seconds (for views) | `45` |

---

## Entity Relationship Summary

```
User ──(posts)──► Post
User ──(comments)──► Comment ──(on)──► Post
User ──(FOLLOWS)──► User          [directed graph edge]
User ──(likes)──► Post
User ──(shares)──► Post
User ──(generates)──► EngagementEvent
Post ──(embedded as)──► VectorEmbedding   [ChromaDB]
EngagementEvent ──(stored in)──► TimeSeries  [InfluxDB]
```

---

## How Each Database Uses This Dataset

| Database | Entities Used | Primary Focus |
|----------|--------------|---------------|
| **CockroachDB** | Users, Posts, Comments, Likes | ACID transactions, distributed queries, schema design |
| **Firestore** | Users, Posts, Comments, Likes | Document model, real-time sync, nested collections |
| **Neo4j** | Users, Follow relationships, Likes | Graph traversal, friend networks, influence analysis |
| **ChromaDB** | Posts (content as embeddings) | Vector similarity, semantic post search |
| **InfluxDB** | EngagementEvents, Likes, Views | Time-series ingestion, aggregation, downsampling |

---

## Dataset Files (To Be Created)

The following data files will be added during the implementation phase:

| File | Format | Purpose |
|------|--------|---------|
| `users.csv` | CSV | Base user records |
| `posts.csv` | CSV | Post records |
| `comments.csv` | CSV | Comment records |
| `follows.csv` | CSV | Follow relationship edges |
| `likes.csv` | CSV | Like records |
| `shares.csv` | CSV | Share records |
| `engagement_events.csv` | CSV | Time-series engagement records |
| `generate_dataset.py` | Python | Script to regenerate the synthetic dataset |

---

*All names, emails, and content in this dataset are entirely fictional and generated for academic purposes.*
