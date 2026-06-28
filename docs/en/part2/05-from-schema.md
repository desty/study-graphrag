# Ch 5. Extracting an Ontology from a DB Schema

!!! abstract "What you'll learn"
    - The view that a normalized relational schema **already has an ontology hiding in it**
    - Rules for mapping tables, columns, foreign keys, and join tables to concepts, properties, and relations
    - Why extracting instead of designing from scratch is cheap and easy to maintain
    - The trap of mechanical 1:1 conversion, and how to assist with an LLM

!!! quote "Assumes"
    Concepts/relations/constraints from [Ch 4](04-ontology-design.md). Assumes relational-DB basics (tables, foreign keys).

---

## 1. Concept — a schema is already half an ontology

Designing a new ontology from a blank page is expensive. But most organizations already have a well-normalized relational DB, and **the act of normalization itself broke the domain into concepts and relations.** In other words, the skeleton of an ontology is already in the schema.

The mapping is surprisingly direct.

| Relational schema | → | Ontology / graph |
|---|---|---|
| Table | → | Concept (class) / node label |
| Row | → | Instance (node) |
| Plain column | → | Property (literal) |
| Foreign key (FK) | → | Relation (edge) |
| Join table (M:N) | → | Relation (edge), sometimes a node |

The `customers`, `orders`, and `products` tables and the FKs between them become, directly, the graph `Customer —places→ Order —contains→ Product`.

## 2. Why you need this — cheap, live, and easy to maintain

Extracting from the schema gets you three things. It's **cheap** — you don't re-analyze the domain from scratch. It's **live** — the operational DB is the source of truth, so the data is already there. It's **easy to maintain** — when the schema changes, you just update the mapping.

So a common starting point for real-world GraphRAG isn't "start from unstructured documents" but "start from the structured DB you already have." Build the graph's skeleton from structured data, then flesh it out with unstructured text (→ Part 3, LLM extraction).

## 3. Where to start — the mapping decisions

Before moving things over mechanically, you have to make a few judgments.

**Not every table is a concept.** An M:N join table (`order_items`) is usually a **relation**, not a concept. But if it carries properties (quantity, unit price), keep it as a relation and hang the properties on the edge (LPG), or promote it to its own node (`OrderItem`) — that decision from [Ch 3](../part1/03-triples.md).

**FK direction and relation names.** An FK has direction but no meaning. Is `orders.customer_id` `Order —placedBy→ Customer` or `Customer —places→ Order`? Name it in the direction the competency questions follow.

**What to drop.** You don't have to move every column and table. Leave audit logs, cache tables, and columns you'll never query out of the graph. [The same principle as context](https://desty.github.io/context-engineering-guide/en/) — only what the questions need.

```text
Mapping example (e-commerce)
TABLE customers(id, name, email, tier)
  → (:Customer {name, email, tier})
TABLE orders(id, customer_id→FK, total, created_at)
  → (:Order {total, created_at})
  → (:Customer)-[:PLACES]->(:Order)
TABLE order_items(order_id→FK, product_id→FK, qty, price)
  → (:Order)-[:CONTAINS {qty, price}]->(:Product)   # join table → relation with properties
```

## 4. Assisting with an LLM

When the schema is large you can hand the mapping to an LLM. Give it the DDL (`CREATE TABLE ...`) and ask it to "map tables→classes and FKs→relations, and propose join-table handling and relation names." But the LLM's proposal is a **draft** — the meaning of FK direction and which join tables to promote to nodes need review with domain knowledge. Same principle as the next chapter's unstructured extraction: automated extraction + human review.

## 5. Common failure points

**The trap of mechanical 1:1 conversion.** Move every table to a node and every FK to an edge verbatim, and the graph is just a copy of the operational DB — not a graph that's good to query. Collapse join tables into relations and drop what you don't use.

**Meaningless relation names.** A literal FK translation like `HAS_CUSTOMER_ID` makes the graph hard to read. Use domain verbs (`PLACES`, `BELONGS_TO`).

## 6. Exercises & next

### Check your understanding

1. Pick 5 tables and the FKs from a DB you know and map them to a graph schema. What's a node, what's a relation?
2. For an M:N join table with properties, what question decides between "relation with properties" and "separate node"?

### Next

On to Part 3, where we actually build the graph. First, making and querying nodes/relations with Neo4j and Cypher (coming soon).

---

## Sources

- Sequeda & Lassila (2021). *Designing and Building Enterprise Knowledge Graphs.* Morgan & Claypool
- W3C. *R2RML: RDB to RDF Mapping Language.*
- Hogan et al. (2021). *Knowledge Graphs.* ACM Computing Surveys (§schema)
