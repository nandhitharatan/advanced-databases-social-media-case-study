# 04 — Vector Database: ChromaDB

## Introduction

ChromaDB is an open-source **vector database** (also called an embedding database) designed for storing, indexing, and querying high-dimensional vector embeddings. A vector embedding is a numeric representation of content — text, images, or audio — in a high-dimensional space where semantic similarity corresponds to geometric proximity.

Traditional databases match data by exact field values or keyword patterns (`WHERE content LIKE '%database%'`). ChromaDB matches data by **semantic meaning**: a query for "distributed systems fault tolerance" will return posts about CockroachDB, Raft consensus, and node failures — even if none of those exact words appear in the query.

This section studies ChromaDB through the lens of social media content search and recommendation, exploring the vector embedding pipeline, approximate nearest-neighbour (ANN) indexing, and similarity metrics.

---

## Purpose in the Social Media Domain

Social media platforms generate vast quantities of text — posts, comments, bios, hashtags. Users expect intelligent search and recommendation that goes beyond keyword matching:

- "Find posts similar to this one I just read"
- "Recommend content based on my recent engagement"
- "Surface posts about machine learning even if they use the word 'AI' instead"

These capabilities require **semantic understanding** of content, which is not possible with SQL `LIKE` queries or inverted text indexes. Vector databases solve this by:

1. **Generating embeddings**: Each post is converted into a dense vector (e.g., 384 or 768 dimensions) using a pre-trained language model such as `sentence-transformers/all-MiniLM-L6-v2`.
2. **Storing embeddings**: The vector, along with its metadata (post_id, author, tags), is stored in ChromaDB.
3. **Querying by similarity**: A user's query is embedded into the same vector space, and ChromaDB retrieves the K nearest neighbours by cosine similarity.

In this case study, ChromaDB is used to demonstrate **semantic post search** and **content-based recommendation** on the shared synthetic dataset's post content.

---

## Core DBMS Concepts Involved

| Concept | How It Applies Here |
|---------|-------------------|
| **Vector Embeddings** | Each post's text content is encoded as a floating-point vector of fixed dimension |
| **Approximate Nearest Neighbour (ANN)** | HNSW algorithm finds the most similar posts without scanning every vector |
| **Cosine Similarity** | The angle between two vectors measures semantic similarity (1 = identical, 0 = unrelated) |
| **Collection** | ChromaDB's equivalent of a table — a named store of embeddings with metadata |
| **Metadata Filtering** | Pre-filter by structured fields (e.g., tag, author) before ANN search |
| **Embedding Model** | The external ML model that transforms text into vectors (not stored in ChromaDB itself) |
| **Dimensionality** | The length of the embedding vector — determines trade-off between accuracy and storage |
| **HNSW Index** | Hierarchical Navigable Small World graph index — the data structure behind ANN search |

---

## Proposed Data Model

ChromaDB organises data into **collections**. Each item in a collection has:

- **id**: Unique identifier (matches `post_id` from the shared dataset)
- **embedding**: The vector representation of the post content
- **document**: The original text content (optional, for reference)
- **metadata**: Structured fields for filtering

```python
# ChromaDB collection structure for posts
collection = chroma_client.create_collection(
    name="social_media_posts",
    metadata={"hnsw:space": "cosine"}   # Use cosine similarity
)

# Example document added to collection
collection.add(
    ids=["p001"],
    documents=["Just deployed CockroachDB on three nodes. The Raft consensus is incredible for ensuring consistency across regions."],
    embeddings=[[0.023, -0.145, 0.312, ...]],   # 384-dim vector from MiniLM
    metadatas=[{
        "author_id": "u001",
        "author_username": "alex_nova",
        "tags": "databases,distributed",
        "created_at": "2024-01-10",
        "media_type": "text"
    }]
)
```

### Embedding Generation Pipeline

```
Raw post text
    │
    ▼
sentence-transformers model (all-MiniLM-L6-v2)
    │
    ▼
384-dimensional float32 vector
    │
    ▼
ChromaDB collection (stored + indexed via HNSW)
```

---

## Planned Practical Demonstrations

The following demonstrations will be implemented in the `examples/` directory:

1. **Embedding generation** — Using `sentence-transformers` to convert all 15 synthetic posts into 384-dim vectors.
2. **Collection setup and seeding** — Creating a ChromaDB collection and inserting all post embeddings with metadata.
3. **Semantic search** — Querying the collection with a natural language query and retrieving the top-K most semantically similar posts.
4. **Metadata-filtered search** — Restricting semantic search to posts with a specific tag (e.g., `"databases"`) before ANN retrieval.
5. **Similarity comparison** — Demonstrating that two thematically related posts have high cosine similarity despite different vocabulary.
6. **Recommendation stub** — Given a post a user liked, find other similar posts (content-based filtering).

---

## Expected Queries and Operations

### Add All Posts with Embeddings
```python
from sentence_transformers import SentenceTransformer
import chromadb

model = SentenceTransformer("all-MiniLM-L6-v2")
client = chromadb.PersistentClient(path="./chromadb_store")
collection = client.get_or_create_collection("social_media_posts", metadata={"hnsw:space": "cosine"})

posts = [
    {"id": "p001", "content": "Just deployed CockroachDB...", "author_id": "u001", "tags": "databases,distributed"},
    # ... all 15 posts
]

ids       = [p["id"] for p in posts]
documents = [p["content"] for p in posts]
embeddings = model.encode(documents).tolist()
metadatas  = [{"author_id": p["author_id"], "tags": p["tags"]} for p in posts]

collection.add(ids=ids, embeddings=embeddings, documents=documents, metadatas=metadatas)
```

### Semantic Search Query
```python
query_text = "distributed consensus and node fault tolerance"
query_embedding = model.encode([query_text]).tolist()

results = collection.query(
    query_embeddings=query_embedding,
    n_results=3,
    include=["documents", "metadatas", "distances"]
)
# Returns the 3 posts most semantically similar to the query
```

### Metadata-Filtered Semantic Search
```python
results = collection.query(
    query_embeddings=query_embedding,
    n_results=3,
    where={"tags": {"$contains": "databases"}},
    include=["documents", "distances"]
)
```

### Content-Based Recommendation (Post Similarity)
```python
# Given that a user liked p002, find similar posts
target = collection.get(ids=["p002"], include=["embeddings"])
target_embedding = target["embeddings"][0]

similar = collection.query(
    query_embeddings=[target_embedding],
    n_results=4,
    where={"$not": {"id": {"$eq": "p002"}}}   # Exclude the original
)
```

---

## Advantages and Limitations

### Advantages
- **Semantic search**: Finds content by meaning, not keyword — far superior for discovery and recommendation in content-heavy platforms.
- **Simple API**: ChromaDB's Python SDK is straightforward — add, query, filter in a few lines.
- **Persistent storage**: ChromaDB supports both in-memory and disk-persistent modes (`PersistentClient`).
- **Metadata filtering**: Combines structured field filtering with vector similarity — the best of both worlds.
- **Open-source and local**: Runs entirely on a local machine, no cloud account or API key needed.
- **HNSW efficiency**: Approximate nearest-neighbour search is much faster than brute-force cosine calculation at scale.

### Limitations
- **Requires external embedding model**: ChromaDB does not generate embeddings itself — a separate model (e.g., SentenceTransformers, OpenAI) is needed.
- **Not suited for exact lookups**: Retrieving a specific post by ID is possible but not the primary use case — use a relational or document DB for that.
- **No transaction support**: ChromaDB does not support multi-operation transactions or ACID guarantees.
- **Approximate results**: ANN search trades accuracy for speed — results are not guaranteed to be the mathematically closest vectors.
- **Scaling complexity**: For billions of vectors, ChromaDB requires a distributed deployment or replacement with Pinecone, Weaviate, or Milvus.
- **Embedding drift**: If the embedding model is updated, all stored embeddings must be regenerated to remain comparable.

---

## Directory Contents

| Path | Purpose |
|------|---------|
| `embedding-generation/` | Scripts and documentation for the embedding pipeline (SentenceTransformers → ChromaDB) |
| `queries/` | Python query scripts for semantic search, filtered search, and recommendation |
| `examples/` | Complete end-to-end scripts: seed embeddings, run queries, interpret results |
| `screenshots/` | Terminal output and visualisation screenshots from demonstration runs |

---

*Navigate to `embedding-generation/` to understand the embedding pipeline before running queries in `examples/`.*
