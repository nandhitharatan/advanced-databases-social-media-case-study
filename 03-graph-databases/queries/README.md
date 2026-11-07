# Queries — Neo4j

This directory will contain Cypher query files for all Neo4j demonstrations.

## Planned Query Files

| File | Description |
|------|-------------|
| `01_create_nodes.cypher` | `CREATE` all User and Post nodes from the synthetic dataset |
| `02_create_relationships.cypher` | `MATCH + CREATE` all FOLLOWS, POSTED, LIKED, COMMENTED, SHARED relationships |
| `03_direct_followers.cypher` | Find all users that a given user follows |
| `04_friends_of_friends.cypher` | 2-hop traversal to find second-degree connections |
| `05_mutual_follows.cypher` | Detect bidirectional follow relationships |
| `06_shortest_path.cypher` | `shortestPath()` between two non-connected users |
| `07_post_reach.cypher` | Find all users who can see a post through the follow graph |
| `08_pagerank.cypher` | Run GDS PageRank on the follow graph for influence scoring |
| `09_community_detection.cypher` | GDS Louvain algorithm for community clusters in the user graph |

## Status

🔲 To be created during the implementation phase.
