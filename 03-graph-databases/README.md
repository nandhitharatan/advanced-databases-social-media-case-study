# 03 — Graph Database: Neo4j

## Introduction

Neo4j is a native graph database that stores data as **nodes**, **relationships**, and **properties**. Unlike relational databases that represent connections as foreign keys and JOINs, Neo4j stores relationships as first-class data structures with direct physical pointers between nodes. This means traversing a relationship (e.g., "find everyone Alex follows") is a constant-time operation regardless of the total number of nodes in the database.

Neo4j uses **Cypher**, a declarative pattern-matching query language designed specifically for graph queries. Cypher's ASCII-art syntax (`(a)-[:FOLLOWS]->(b)`) makes complex graph patterns readable and expressive.

This section demonstrates why graph databases are architecturally superior to relational databases for social network relationship queries.

---

## Purpose in the Social Media Domain

The defining feature of a social media platform is its **social graph** — the network of users connected by follow, friend, and interaction relationships. These relationships are what differentiate social media from a simple content repository.

Relational databases struggle with graph queries because:
- Traversing a follower network of depth 3 requires three nested JOINs.
- The result set grows exponentially with depth.
- Query planner hints are needed to avoid full table scans on large `follows` tables.

Neo4j excels at exactly these queries:
- **Who are friends-of-friends?** Traverse 2 hops from a node.
- **What is the shortest path between two users?** Use Dijkstra or BFS algorithms.
- **Who are the most influential users?** Calculate PageRank or Betweenness Centrality.
- **What posts are popular in my network?** Traverse follows → posts → likes in a single Cypher query.

In this case study, Neo4j is responsible for modelling and querying the **social relationship layer**: follow networks, influence detection, and graph-based content recommendation.

---

## Core DBMS Concepts Involved

| Concept | How It Applies Here |
|---------|-------------------|
| **Property Graph Model** | Nodes (User, Post) have properties; Relationships (FOLLOWS, LIKES) have properties and direction |
| **Index-Free Adjacency** | Each node directly stores pointers to its connected nodes — traversal does not require index lookups |
| **Cypher Query Language** | Declarative pattern-matching language for expressing graph queries |
| **Graph Traversal** | BFS and DFS used to explore follower networks and content reach |
| **Graph Algorithms** | PageRank for influence scoring, Betweenness Centrality for bridge-user detection |
| **Directed Relationships** | FOLLOWS is directional — `(u1)-[:FOLLOWS]->(u2)` ≠ `(u2)-[:FOLLOWS]->(u1)` |
| **Labels** | Nodes are labelled `User` or `Post` to partition the graph by entity type |

---

## Proposed Data Model

Neo4j uses a property graph model. The social media entities map to the following graph schema:

### Node Labels and Properties

```cypher
// User node
(:User {
    user_id:        "u001",
    username:       "alex_nova",
    full_name:      "Alex Nova",
    location:       "Mumbai",
    joined_at:      datetime("2023-01-15T10:00:00Z"),
    is_verified:    false
})

// Post node
(:Post {
    post_id:    "p001",
    content:    "Just deployed CockroachDB on three nodes...",
    media_type: "text",
    tags:       ["databases", "distributed"],
    created_at: datetime("2024-01-10T14:30:00Z")
})
```

### Relationship Types and Properties

```cypher
// User follows User (directed)
(u1:User)-[:FOLLOWS { followed_at: datetime("2024-01-05T09:00:00Z") }]->(u2:User)

// User authored a Post
(u:User)-[:POSTED { created_at: datetime("2024-01-10T14:30:00Z") }]->(p:Post)

// User liked a Post
(u:User)-[:LIKED { liked_at: datetime("2024-01-10T16:00:00Z") }]->(p:Post)

// User commented on a Post
(u:User)-[:COMMENTED { comment_id: "c001", created_at: datetime("2024-01-10T15:00:00Z") }]->(p:Post)

// User shared a Post
(u:User)-[:SHARED { shared_at: datetime("2024-01-13T09:00:00Z"), note: "Graph DBs!" }]->(p:Post)
```

### Graph Schema Diagram

```
(User) -[:FOLLOWS]──────────► (User)
(User) -[:POSTED]───────────► (Post)
(User) -[:LIKED]────────────► (Post)
(User) -[:COMMENTED]────────► (Post)
(User) -[:SHARED]───────────► (Post)
```

---

## Planned Practical Demonstrations

The following demonstrations will be implemented in the `examples/` directory:

1. **Graph seeding** — Creating all 10 User nodes, 15 Post nodes, and all relationship edges from the shared dataset.
2. **Direct follower query** — Finding all users that a given user follows.
3. **Friends-of-friends** — Traversing 2 hops in the follow graph to find second-degree connections.
4. **Mutual followers** — Finding users that mutually follow each other (bidirectional check).
5. **Shortest path** — Finding the shortest FOLLOWS path between two non-connected users.
6. **Post reach analysis** — Finding all users who can see a post through their follow network.
7. **Influence scoring** — Running PageRank on the follow graph to rank users by influence.

---

## Expected Queries and Operations

### Find All Users That alex_nova Follows
```cypher
MATCH (alex:User {username: "alex_nova"})-[:FOLLOWS]->(followee:User)
RETURN followee.username, followee.full_name, followee.location
ORDER BY followee.username;
```

### Friends-of-Friends (2-Hop Traversal)
```cypher
MATCH (alex:User {username: "alex_nova"})-[:FOLLOWS*2]->(fof:User)
WHERE NOT (alex)-[:FOLLOWS]->(fof) AND fof <> alex
RETURN DISTINCT fof.username AS suggested_user, fof.full_name
LIMIT 10;
```

### Mutual Follow Detection
```cypher
MATCH (a:User {username: "alex_nova"})-[:FOLLOWS]->(b:User)-[:FOLLOWS]->(a)
RETURN b.username AS mutual_follower;
```

### Shortest Path Between Two Users
```cypher
MATCH path = shortestPath(
    (alex:User {username: "alex_nova"})-[:FOLLOWS*]-(tara:User {username: "tech_tara"})
)
RETURN path, length(path) AS degrees_of_separation;
```

### Posts Liked by Followers (Graph-Based Feed)
```cypher
MATCH (alex:User {username: "alex_nova"})-[:FOLLOWS]->(friend:User)-[:LIKED]->(p:Post)
WHERE NOT (alex)-[:LIKED]->(p)
RETURN p.post_id, p.content, COUNT(friend) AS friend_likes
ORDER BY friend_likes DESC
LIMIT 10;
```

### PageRank for User Influence
```cypher
CALL gds.pageRank.stream('social-graph', { maxIterations: 20, dampingFactor: 0.85 })
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).username AS user, score
ORDER BY score DESC
LIMIT 5;
```

---

## Advantages and Limitations

### Advantages
- **Native relationship storage**: Traversing relationships is O(1) per hop — no JOINs, no index lookups, no performance degradation as the graph grows.
- **Expressive query language**: Cypher patterns are human-readable and closely mirror how we naturally think about social networks.
- **Built-in graph algorithms**: The Graph Data Science (GDS) library provides PageRank, community detection, centrality, and pathfinding out of the box.
- **Variable-depth traversal**: `[:FOLLOWS*1..5]` traverses up to 5 hops in a single query — impossible efficiently in SQL.
- **Visualisation**: Neo4j Browser renders graph results visually, making social network analysis intuitive.

### Limitations
- **Not suited for tabular analytics**: Aggregation queries (e.g., "total likes per post over the last week") are less efficient than in SQL or time-series databases.
- **Write throughput**: Graph databases are optimised for read-heavy traversal workloads, not for the high write throughput of engagement event streams.
- **No native sharding**: Neo4j Community Edition is single-node; horizontal scaling requires Neo4j Enterprise (Fabric) or Aura.
- **Learning curve**: Cypher syntax and the property graph mental model are unfamiliar to developers coming from a relational background.
- **Data denormalisation needed for counts**: Storing `follower_count` as a node property requires manual updates — Cypher `COUNT()` on large graphs can be slow.

---

## Directory Contents

| Path | Purpose |
|------|---------|
| `schema/` | Node label definitions, relationship type constraints, and index setup Cypher |
| `queries/` | Cypher query files for all demonstration scenarios |
| `examples/` | Python scripts using the `neo4j` driver to seed data and run queries |
| `screenshots/` | Neo4j Browser screenshots showing graph visualisations and query results |

---

*Navigate to `queries/` for the complete Cypher query collection and `examples/` for runnable Python scripts.*
