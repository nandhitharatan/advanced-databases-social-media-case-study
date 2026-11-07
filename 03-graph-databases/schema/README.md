# Schema — Neo4j

This directory will contain Neo4j graph schema definitions and constraint/index setup scripts.

## Planned Files

| File | Description |
|------|-------------|
| `01_constraints.cypher` | `CREATE CONSTRAINT` statements for `User.user_id` and `Post.post_id` uniqueness |
| `02_indexes.cypher` | `CREATE INDEX` statements for commonly queried properties |
| `03_node-label-definitions.md` | Documentation of all node labels and their properties |
| `04_relationship-type-definitions.md` | Documentation of all relationship types, directionality, and properties |
| `05_graph-schema-diagram.md` | ASCII or Mermaid diagram of the full property graph schema |

## Status

🔲 To be created during the implementation phase.
