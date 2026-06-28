# Ch 8. Time-Aware Graphs and Graphiti

!!! abstract "What you'll learn"
    - Knowledge changes — the limit of a graph whose "facts have no timestamp"
    - The **bi-temporal** model — when a fact is valid vs. when we learned it
    - The LPG pattern of pinning time onto relationships, and what Graphiti automates
    - Time-aware graphs as agent memory

!!! quote "Assumes"
    [Ch 6](06-neo4j-cypher.md) and [Ch 7](07-llm-extraction.md). Assumes you know the LPG model of attaching properties to relationships.

---

## 1. Concept — facts have a time

The graph so far knows no time. *Since when* is "Sujin Lee —employedBy→ Apple" true? If she leaves, does this edge become wrong, or does it remain a fact that was once true?

Knowledge changes. CEOs change, prices rise, relationships end. A timeless graph can hold only "now," and an update is the deletion of the past. But many questions care about time — "who was this company's CEO back in 2023?", "since when has this policy applied?".

## 2. Why you need this — bi-temporal

To handle time properly, you distinguish two kinds of timestamp.

- **Valid time** — the period the fact is true in reality. "Sujin Lee employed at Apple 2019–2024."
- **Transaction time** — when our system learned/recorded the fact. "Recorded this employment fact into the graph in January 2026."

Handling both together is **bi-temporal.** With valid time you can query "the truth as of then," and with transaction time you can trace "since when did we know it that way," enabling correction and audit.

## 3. How — pin time onto relationships

In LPG this is natural. Put timestamps in relationship properties (the LPG strength from [Ch 3](../part1/03-triples.md) shines here).

```cypher
// Mark a finished fact with a period instead of deleting it
MATCH (c:Customer {name:'Sujin Lee'})-[r:EMPLOYED]->(co:Company {name:'Apple'})
SET r.valid_from = date('2019-01-01'),
    r.valid_to   = date('2024-06-30')   // null means "until now"

// "Employment relationships as of 2023"
MATCH (p:Person)-[r:EMPLOYED]->(co:Company)
WHERE r.valid_from <= date('2023-06-01')
  AND (r.valid_to IS NULL OR r.valid_to >= date('2023-06-01'))
RETURN p.name, co.name
```

When a new fact arrives, instead of deleting the existing edge you close its `valid_to` and open a new edge. The graph **accumulates history.**

## 4. Graphiti — time-aware graph memory

Writing this pattern all by hand is tedious. **[Graphiti](https://github.com/getzep/graphiti)** (Zep) is an open-source project that manages a time-aware knowledge graph automatically. When new information arrives, it checks whether it contradicts existing facts, and if so closes the old edge with a timestamp and opens a new one — it handles bi-temporal for you.

Where this is especially useful is **agent memory.** [In context engineering](https://desty.github.io/context-engineering-guide/en/) we talked about memory outside the window; a time-aware graph is a powerful form of that memory. Because the graph remembers "the user used to prefer X but now prefers Y" with timestamps, the agent can query both the latest state and the history of change.

```python title="graphiti_sketch.py"
# Conceptual sketch (see the graphiti docs for the real API)
# pip install graphiti-core
from graphiti_core import Graphiti

g = Graphiti(neo4j_uri="bolt://localhost:7687", user="neo4j", password="testpassword")

# Add an episode (a chunk of source text); it extracts entities/relations and loads with timestamps
await g.add_episode(name="profile", episode_body="Sujin Lee now prefers a trackpad over a mouse.")
# If a contradicting fact like 'prefers keyboard' exists, it closes that valid_to and opens the new fact
```

## 5. Common failure points

**Treating update as delete.** Delete the old edge when a new value arrives and you lose the ability to query the past and audit. Close it, don't delete it (set `valid_to`).

**Timestamping everything.** Bi-temporal has a cost (more edges, more query complexity). Apply it only to facts that genuinely change and that you'll query historically. Unchanging facts (date of birth) are just properties.

**Time-zone / date-type confusion.** Insert timestamps as strings and comparisons break. Standardize on `date()`/`datetime()` types.

## 6. Exercises & next

### Hands-on

1. Add `valid_from`/`valid_to` to the employment relationship from [Ch 6](06-neo4j-cypher.md) and query "who was employed as of a given date."
2. Write a load function that, when a contradicting new fact arrives, "closes the old edge and opens a new one."
3. Run the example from the Graphiti README and observe how the graph changes when you inject a contradicting fact.

### Next

The graph is ready. In Part 4 we bolt it onto RAG — starting with the GraphRAG architecture (coming soon).

---

## Sources

- Zep. *Graphiti: Temporal Knowledge Graphs for Agents.* (github.com/getzep/graphiti)
- Snodgrass (1999). *Developing Time-Oriented Database Applications in SQL.* (bi-temporal)
