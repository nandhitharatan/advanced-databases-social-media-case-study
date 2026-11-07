# Embedding Generation — ChromaDB

This directory will contain documentation and scripts for the embedding generation pipeline.

## Overview

The embedding generation pipeline converts raw post text from the synthetic dataset into 384-dimensional float32 vectors using the `sentence-transformers/all-MiniLM-L6-v2` model. These vectors are then stored in ChromaDB for approximate nearest-neighbour search.

## Planned Files

| File | Description |
|------|-------------|
| `pipeline-overview.md` | Step-by-step documentation of the full embedding pipeline: text → model → vector → ChromaDB |
| `model-selection.md` | Justification for choosing `all-MiniLM-L6-v2` over alternatives (OpenAI `text-embedding-ada-002`, `all-mpnet-base-v2`). Covers dimension size, model size, accuracy, and local vs. API usage |
| `generate_embeddings.py` | Script to load post data, generate embeddings using SentenceTransformers, and save to `embeddings.npy` |
| `embedding-visualisation.md` | Notes on visualising 384-dim embeddings using dimensionality reduction (UMAP or t-SNE) to verify semantic clustering |

## Embedding Model Details

| Property | Value |
|----------|-------|
| Model name | `sentence-transformers/all-MiniLM-L6-v2` |
| Embedding dimensions | 384 |
| Max input tokens | 256 |
| Similarity metric | Cosine similarity |
| Model size | ~90 MB |
| Runs locally | Yes (no API key required) |

## Status

🔲 To be created during the implementation phase.
