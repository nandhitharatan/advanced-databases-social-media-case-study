# Examples — Firebase Firestore

This directory will contain complete Python scripts for Firestore demonstrations using the `firebase-admin` SDK.

## Planned Example Scripts

| File | Description |
|------|-------------|
| `00_connection.py` | Initialise Firebase Admin SDK with a service account |
| `01_seed_users.py` | Write all 10 synthetic user documents to Firestore |
| `02_seed_posts.py` | Write all 15 synthetic post documents with subcollections |
| `03_seed_likes_follows.py` | Write like and follow documents |
| `04_like_transaction.py` | Transactional like increment with distributed counter pattern |
| `05_realtime_listener.py` | Demonstrate `on_snapshot()` real-time listener |
| `06_denorm_vs_reference.py` | Compare denormalised vs. reference-based post reads |

## Prerequisites (Not Yet Installed)

- Firebase project (with Firestore enabled)
- Service account key JSON file (not stored in this repository)
- Python package: `firebase-admin`

## Status

🔲 To be created during the implementation phase.
