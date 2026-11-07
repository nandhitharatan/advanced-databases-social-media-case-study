# Examples — CockroachDB

This directory will contain Python scripts demonstrating CockroachDB operations using the `psycopg2` PostgreSQL driver.

## Planned Example Scripts

| File | Description |
|------|-------------|
| `00_connection.py` | Establish a connection to a local CockroachDB cluster |
| `01_create_schema.py` | Execute `CREATE TABLE` statements programmatically |
| `02_seed_data.py` | Insert all synthetic dataset records |
| `03_feed_query.py` | Generate a user's feed — posts from followed users |
| `04_like_transaction.py` | Demonstrate serializable transaction for like increment |
| `05_trending.py` | Query and display trending posts |
| `06_explain_index.py` | Run `EXPLAIN` and display the query plan |

## Prerequisites (Not Yet Installed)

- CockroachDB (local single-node: `cockroach start-single-node --insecure`)
- Python package: `psycopg2-binary`

## Status

🔲 To be created during the implementation phase.
