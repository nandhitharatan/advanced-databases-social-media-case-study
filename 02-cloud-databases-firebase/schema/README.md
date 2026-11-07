# Schema — Firebase Firestore

This directory will contain Firestore schema definitions and index configurations.

## Planned Files

| File | Description |
|------|-------------|
| `collection-structure.md` | Full document structure for all Firestore collections (`/users`, `/posts`, `/likes`, `/follows`) including subcollections |
| `index-definitions.json` | Composite index definitions (equivalent to `firestore.indexes.json`) for multi-field queries |
| `security-rules-example.txt` | Example Firestore security rules (read-only, for reference — no credentials) |
| `denormalisation-decisions.md` | Documentation of denormalisation choices (e.g., embedding `author_username` in post documents) and their trade-offs |

## Status

🔲 To be created during the implementation phase.
