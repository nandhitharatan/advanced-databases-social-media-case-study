# 02 — Cloud Database: Firebase Firestore

## Introduction

Firebase Firestore is a fully managed, serverless NoSQL document database built on Google's global infrastructure. Unlike traditional databases that require provisioning servers, Firestore automatically scales to handle any volume of traffic, replicates data across multiple regions, and delivers real-time synchronisation to connected clients.

Firestore organises data as **documents** nested inside **collections**, and supports subcollections for hierarchical data — a natural fit for social media objects like posts containing comment threads.

This section studies Firestore from a database design perspective: how the document model differs from relational schemas, how Firestore's indexing strategy affects query design, and what trade-offs arise from its eventual consistency model.

---

## Purpose in the Social Media Domain

Firestore is well-suited for social media for several reasons:

- **Real-time listeners**: A client can subscribe to a document or collection and receive live updates the moment a new post or comment is written — without polling.
- **Document model**: A `post` document can embed its top-level metadata and reference its comments as a subcollection, mirroring how social media UIs display content.
- **Offline support**: Firestore's SDK caches data locally, allowing a mobile app to show the user's feed even without a network connection.
- **Schemaless flexibility**: Posts can have different fields (text-only, image URL, tagged users) without requiring schema migrations.

In this case study, Firestore is used to demonstrate an alternative **cloud-native document model** for the same social media entities, contrasting with CockroachDB's relational approach.

---

## Core DBMS Concepts Involved

| Concept | How It Applies Here |
|---------|-------------------|
| **Document-Collection Model** | Users, posts, and comments are stored as JSON-like documents in collections |
| **Subcollections** | Comments are stored as a subcollection under each post document |
| **Composite Indexes** | Firestore requires explicit index definitions for queries with multiple filter + sort fields |
| **Real-Time Listeners** | `onSnapshot()` listeners push live updates to clients when data changes |
| **Eventual Consistency** | Firestore uses strong consistency for single-document reads, eventual for certain multi-document queries |
| **Security Rules** | Declarative rules control read/write access per document/collection path |
| **Transactions** | Firestore supports optimistic concurrency transactions across multiple documents |
| **Denormalisation** | Embedding counts (like_count, comment_count) in documents avoids expensive aggregation queries |

---

## Proposed Data Model

Firestore uses a document/collection hierarchy. The same social media entities from the shared dataset map to this structure:

```
/users/{user_id}
    username: "alex_nova"
    full_name: "Alex Nova"
    email: "alex@example.com"
    bio: "Coffee enthusiast. Dev."
    location: "Mumbai"
    joined_at: Timestamp
    is_verified: false
    follower_count: 342
    following_count: 128

/posts/{post_id}
    author_id: "u001"
    author_username: "alex_nova"      ← Denormalised for fast reads
    content: "Just deployed CockroachDB..."
    media_type: "text"
    tags: ["databases", "distributed"]
    created_at: Timestamp
    like_count: 47                    ← Denormalised counter
    comment_count: 12

    /comments/{comment_id}            ← Subcollection
        author_id: "u002"
        author_username: "priya_codes"
        content: "Which region did you deploy to?"
        created_at: Timestamp
        parent_comment_id: null

/likes/{like_id}
    user_id: "u003"
    post_id: "p001"
    liked_at: Timestamp

/follows/{follow_id}
    follower_id: "u001"
    followee_id: "u002"
    followed_at: Timestamp
```

### Key Design Decision: Denormalisation

Unlike relational databases, Firestore does not support JOIN operations. To display a post with the author's username, two options exist:

1. **Embed** the username in the post document (denormalisation) — fast reads, but requires updating all posts if username changes.
2. **Reference** the user document and fetch separately — consistent, but requires a second read operation.

This case study demonstrates both patterns and discusses the trade-offs.

---

## Planned Practical Demonstrations

The following demonstrations will be implemented in the `examples/` directory:

1. **Schema setup** — Creating collections and documents using the Firestore Python SDK (`firebase-admin`).
2. **Document read and write** — CRUD operations on `users` and `posts` collections.
3. **Subcollection query** — Fetching all comments for a specific post from its subcollection.
4. **Composite index query** — Querying posts filtered by tag and sorted by `created_at`, demonstrating Firestore's index requirement.
5. **Denormalisation trade-off demo** — Writing a post with embedded author fields vs. reference-only approach.
6. **Firestore transaction** — Atomically incrementing `like_count` on a post and writing a like document.

---

## Expected Queries and Operations

Firestore uses SDK methods rather than a query language like SQL. The following pseudocode illustrates expected operations:

### Fetch a User's Posts
```python
posts_ref = db.collection("posts")
query = posts_ref.where("author_id", "==", "u001").order_by("created_at", direction="DESCENDING").limit(10)
docs = query.stream()
```

### Add a Post (with transaction for counter)
```python
@firestore.transactional
def create_post(transaction, post_ref, user_ref, post_data):
    user = user_ref.get(transaction=transaction)
    transaction.set(post_ref, post_data)
    # (like_count starts at 0 — no counter increment on creation)
```

### Like a Post (Transactional Counter Increment)
```python
@firestore.transactional
def like_post(transaction, post_ref, like_ref, user_id):
    post = post_ref.get(transaction=transaction)
    new_count = post.get("like_count") + 1
    transaction.update(post_ref, {"like_count": new_count})
    transaction.set(like_ref, {"user_id": user_id, "liked_at": firestore.SERVER_TIMESTAMP})
```

### Query Posts by Tag (requires composite index on tags + created_at)
```python
query = db.collection("posts") \
          .where("tags", "array_contains", "databases") \
          .order_by("created_at", direction="DESCENDING") \
          .limit(5)
```

### Real-Time Listener for New Posts
```python
def on_snapshot(doc_snapshot, changes, read_time):
    for change in changes:
        if change.type.name == "ADDED":
            print(f"New post: {change.document.to_dict()}")

posts_ref.on_snapshot(on_snapshot)
```

---

## Advantages and Limitations

### Advantages
- **Zero infrastructure management**: No servers to provision, patch, or scale — fully managed by Google Cloud.
- **Real-time synchronisation**: Built-in `onSnapshot` listeners are ideal for live feeds and notification systems.
- **Flexible schema**: Adding new fields to documents requires no migration — suitable for rapidly evolving social features.
- **Global scalability**: Firestore scales horizontally without developer intervention.
- **Offline-first**: SDK caches data, enabling mobile social apps to function without connectivity.

### Limitations
- **No JOIN support**: Relational queries (e.g., "get posts from all users I follow") require multiple round-trips or denormalisation.
- **Composite index management**: Every `WHERE + ORDER BY` combination on different fields requires a manually defined index.
- **Eventual consistency for some queries**: Collection group queries may not reflect the latest write immediately.
- **Cost at scale**: Firestore charges per read, write, and delete — high engagement events (thousands of likes) can be expensive without careful batching.
- **Limited aggregation**: Firestore has limited native aggregation (only `count()`, `sum()`, `average()` as of recent versions) — complex analytics require exporting to BigQuery.

---

## Directory Contents

| Path | Purpose |
|------|---------|
| `schema/` | Firestore collection structure definitions and index configuration |
| `queries/` | Python SDK query examples for all major operations |
| `examples/` | Complete Python scripts to seed data and demonstrate Firestore behaviour |
| `screenshots/` | Firebase Console screenshots showing collections, documents, and query plans |

---

*Navigate to `schema/` for the index definitions and `examples/` for executable demonstration code.*
