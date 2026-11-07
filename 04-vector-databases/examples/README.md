# Examples — ChromaDB

This directory will contain complete Python scripts for the ChromaDB demonstration pipeline.

## Planned Example Scripts

| File | Description |
|------|-------------|
| `00_setup_collection.py` | Create a persistent ChromaDB client and collection |
| `01_seed_embeddings.py` | Generate embeddings for all 15 posts and insert into ChromaDB |
| `02_semantic_search_demo.py` | Interactive semantic search — input query, receive similar posts |
| `03_recommendation_demo.py` | Input a post ID, receive top-K recommended similar posts |
| `04_similarity_matrix.py` | Compute pairwise cosine similarity for all 15 posts |
| `05_embedding_visualisation.py` | Reduce to 2D using UMAP and plot semantic clusters |

## Prerequisites (Not Yet Installed)

- Python packages: `chromadb`, `sentence-transformers`, `numpy`
- Optional (for visualisation): `umap-learn`, `matplotlib`

## Status

🔲 To be created during the implementation phase.
