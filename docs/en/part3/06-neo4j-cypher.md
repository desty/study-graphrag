# Ch 6. Neo4j and Cypher Basics

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part3/ch6_neo4j_cypher.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "What you'll learn"
    - Stand up the graph DB **Neo4j** via Docker / free tier and connect from Python
    - **Cypher** basics — create nodes/relationships (`CREATE`/`MERGE`), query (`MATCH`), multi-hop traversal
    - Load the e-commerce mapping from [Ch 5](../part2/05-from-schema.md) as a real graph
    - Solve, in one line of Cypher, the multi-hop question vector RAG couldn't

!!! quote "Assumes"
    Through [Part 2](../part2/04-ontology-design.md). Basic Python and Docker (or a free Neo4j Aura account) are enough.

---

## 1. Concept — Cypher is "a query language you draw"

Cypher's core intuition is one thing — **you draw the graph in ASCII.** Nodes are parentheses `()`, relationships are arrows `-[]->`. Write the graph in your head and that's the query.

```cypher
(customer)-[:PLACES]->(order)-[:CONTAINS]->(product)
```

That one line is the pattern "a customer places an order, an order contains a product." A query is asking the DB to **find the parts of the graph that match this pattern.**

## 2. Environment — standing up Neo4j

The fastest path is Docker.

```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/testpassword \
  neo4j:5
```

You can open the browser console at `http://localhost:7474`. To skip installation, use a free instance on [Neo4j Aura](https://neo4j.com/cloud/aura/). Connecting from Python:

```python title="connect.py"
# pip install neo4j
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "testpassword"))

def run(cypher, **params):
    with driver.session() as s:
        return list(s.run(cypher, **params))

print(run("RETURN 'connected' AS status")[0]["status"])
```

## 3. Creating — CREATE and MERGE

Make nodes and relationships. Let's load the e-commerce mapping from [Ch 5](../part2/05-from-schema.md) directly.

```python title="load.py"
run("""
MERGE (c:Customer {id: 1, name: 'Sujin Lee', tier: 'gold'})
MERGE (o:Order {id: 101, total: 52000})
MERGE (p1:Product {id: 'A', name: 'Keyboard'})
MERGE (p2:Product {id: 'B', name: 'Mouse'})
MERGE (c)-[:PLACES]->(o)
MERGE (o)-[:CONTAINS {qty: 1, price: 40000}]->(p1)
MERGE (o)-[:CONTAINS {qty: 1, price: 12000}]->(p2)
""")
```

!!! tip "Default to MERGE, not CREATE"
    `CREATE` makes a fresh copy every time, breeding duplicates. `MERGE` "finds it if it exists, creates it if not" — safe to re-run a load (idempotent). Real-world loads are almost all `MERGE`.

## 4. Querying — MATCH and multi-hop

Now let's solve the multi-hop question [Ch 1](../part1/01-why-graph.md) said vector RAG can't. "What products did Sujin Lee buy?" — two hops from customer through order to product.

```python title="query.py"
rows = run("""
MATCH (c:Customer {name: 'Sujin Lee'})-[:PLACES]->(:Order)-[:CONTAINS]->(p:Product)
RETURN p.name AS product
""")
print([r["product"] for r in rows])   # ['Keyboard', 'Mouse']
```

With vector search you'd have to hope "Sujin Lee" and "Keyboard" were written in the same document. In a graph you just **follow the relationships**. Add more hops and the pattern just gets longer.

```cypher
// "Other customers who bought the same product" — the seed of a 3-hop recommendation
MATCH (c:Customer {name: 'Sujin Lee'})-[:PLACES]->(:Order)-[:CONTAINS]->(p:Product)
      <-[:CONTAINS]-(:Order)<-[:PLACES]-(other:Customer)
WHERE other <> c
RETURN DISTINCT other.name
```

## 5. Common failure points

**Using one big MERGE and getting duplicate matches.** In `MERGE (a)-[:R]->(b)`, if `a` or `b` doesn't exist yet, the whole pattern is created anew. Safer to `MERGE` each node first, then `MERGE` the relationship.

**Mass matching without an index.** If `MATCH (c:Customer {id: ...})` is slow, add an index on the label+property with `CREATE INDEX`. Index your key lookup properties before loading.

**Obsessing over direction, or ignoring it.** Cypher's `-[:R]->` respects direction. For direction-agnostic queries use `-[:R]-` (no arrow). Decide first whether the question cares about direction.

## 6. Exercises & next

### Hands-on

1. Load the medical ontology from [Ch 4](../part2/04-ontology-design.md) (Drug/Disease/Ingredient) as 5 nodes and 5 relations.
2. Find "drugs that treat headache and contain acetylsalicylic acid" in a single Cypher query.
3. Adapt the recommendation query to get "top 3 customers whose taste overlaps with Sujin Lee," sorted by number of shared products.

### Next

So far we entered structured data by hand. Next we **extract entities and relations from unstructured text with an LLM** to populate the graph automatically → [Ch 7](07-llm-extraction.md).

---

## Sources

- Neo4j. *Cypher Manual* (v5).
- Robinson, Webber & Eifrem (2015). *Graph Databases* (2nd ed.), O'Reilly
