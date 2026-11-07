# Queries — CockroachDB

This directory will contain SQL query files for all CockroachDB demonstrations.

## Planned Query Files

| File | Description |
|------|-------------|
| `01_schema_create.sql` | `CREATE TABLE` statements for all social media entities |
| `02_seed_data.sql` | `INSERT` statements for the synthetic dataset |
| `03_feed_generation.sql` | Query to retrieve posts from followed users |
| `04_like_transaction.sql` | `BEGIN/COMMIT` transaction for atomic like increment |
| `05_trending_posts.sql` | Query for most liked posts in the last 24 hours |
| `06_follower_aggregation.sql` | Follower count aggregation with `GROUP BY` |
| `07_explain_index.sql` | `EXPLAIN (DISTSQL)` to demonstrate index usage |
| `08_geo_partition.sql` | Geo-partitioning setup and query demonstration |

## Status

🔲 To be created during the implementation phase.
