# Ch 10. Graph Retrieval Strategies — Local, Global, Community Summaries

!!! abstract "What you'll learn"
    - Three retrieval modes that split by question type — **local, global, hybrid**
    - Local search: gathering the neighborhood of a specific entity
    - Global search: synthesizing community summaries with map-reduce
    - The hybrid that finds an entry point with vectors and expands with the graph

!!! quote "Assumes"
    Indexing (graph + community summaries) from [Ch 9](09-what-is-graphrag.md).

---

## 1. Concept — the question decides the retrieval

With the index ready (graph + community summaries), what remains is "what to pull when a question arrives." GraphRAG retrieval splits broadly in two by question type.

- **Local** — "tell me about this entity/topic." Explore around a specific node.
- **Global** — "what stands out across the whole." Synthesize community summaries.

Vector RAG doesn't distinguish these; it always returns "top-k similar chunks." GraphRAG picks the retrieval that fits the question.

## 2. Local search — gathering an entity's neighborhood

Use it when a specific entity is the center of the question. The procedure is intuitive.

1. Find the **entry entity** from the question (name match, or the nearest node by embedding).
2. Expand that node's **neighborhood** via the graph (1–2 hop relations, connected entities, related chunks).
3. Hand the gathered context to the LLM to answer.

```python title="local_search.py"
def local_search(question, entry_entity):
    ctx = run("""
    MATCH (e {name: $name})-[r]-(neighbor)
    OPTIONAL MATCH (e)-[:MENTIONED_IN]->(chunk:Chunk)
    RETURN e, collect(DISTINCT type(r) + ': ' + neighbor.name) AS facts,
           collect(DISTINCT chunk.text)[..5] AS snippets
    """, name=entry_entity)
    # combine facts (graph) + snippets (source text) into the prompt context
    return build_prompt(question, ctx)
```

The key is giving **graph facts + source chunks together.** The graph says "how things connect," the chunks are "the evidence from the source." You need both for multi-hop answers to be accurate and grounded.

## 3. Global search — map-reduce over community summaries

A global question like "what are this corpus's core themes?" has no entry entity. Instead you use the **community summaries** pre-built in [Ch 9](09-what-is-graphrag.md). A classic map-reduce.

1. **Map** — for each community summary, the LLM produces a partial answer of "how this summary contributes to the question."
2. **Reduce** — synthesize the partial answers into a final answer.

You synthesize only the summaries instead of re-reading all documents, so a "global" question becomes tractable at a realistic cost. Exactly the question vector RAG structurally couldn't do.

## 4. Hybrid — enter with vectors, expand with the graph

The most common real-world pattern mixes the two. **Find an entry point with vector search, expand the neighborhood with the graph.**

```text
question → (vector) find top-k most relevant chunks/entities   ← entry
         → (graph) expand via that entity's neighbors/relations  ← multi-hop boost
         → answer with the combined context
```

It combines the vector's "flexible entry" with the graph's "precise expansion." This is the concrete form of "they're complementary, not competitors" from [Ch 1](../part1/01-why-graph.md).

| Question | Strategy |
|---|---|
| "Explain X" | Local |
| "How are X and Y connected" | Local (multi-hop) or hybrid |
| "What are the core themes overall" | Global |
| "Tell me everything about this keyword" | Hybrid |

## 5. Common failure points

**Using global for everything.** Global sweeps all community summaries, so it's expensive. Using global on an entity question (where local suffices) is just slow and costly. Classify the question first (routing).

**Giving only graph facts, no source text.** Graph triples alone leave the LLM short on context or unable to cite evidence. In local search, include the connected source chunks too.

**Entry-entity match failure.** If the question's wording differs from the graph node name, entry fails. Reinforce with embedding-based entity matching or an alias dictionary (connects to entity resolution in [Ch 7](../part3/07-llm-extraction.md)).

## 6. Exercises & next

### Check your understanding

1. Write a rule for routing your question set into local/global/hybrid.
2. If the global search's map step has 100 communities, how many LLM calls is that? How would you cut the cost?

### Next

We've said GraphRAG is better in words. Now we compare it **head-to-head with vector RAG** on the same corpus and questions, and see where they diverge → [Ch 11](11-vector-vs-graph.md).

---

## Sources

- Microsoft Research (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization.* arXiv:2404.16130
- Microsoft. *GraphRAG: local & global search* docs
