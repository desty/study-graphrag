# Ch 9. What GraphRAG Is — Architecture and Indexing

!!! abstract "What you'll learn"
    - GraphRAG's two stages — **indexing (offline)** and **query (online)**
    - What indexing does: build the graph → detect communities → summarize communities
    - Why indexing is expensive, and the question types that justify the cost
    - The Microsoft GraphRAG pipeline in broad strokes

!!! quote "Assumes"
    Part 3 (extraction from [Ch 7](../part3/07-llm-extraction.md)). A feel for the vector-RAG pipeline ([RAG Guide](https://desty.github.io/rag-guide/en/)).

---

## 1. Concept — GraphRAG has two stages

![GraphRAG architecture — indexing (offline) and query (online)](../assets/diagrams/ch9-graphrag-arch.svg#only-light)
![GraphRAG architecture — indexing (offline) and query (online)](../assets/diagrams/ch9-graphrag-arch-dark.svg#only-dark)

In [Ch 1](../part1/01-why-graph.md) we called GraphRAG "bolting the graph onto RAG to make connections retrievable." Like vector RAG, its structure splits into **offline indexing** and **online query.** The difference is that indexing builds not just embeddings but **the graph and its summaries.**

```text
[ Indexing (offline, once · expensive) ]
  docs → chunks → (LLM) entity/relation extraction → knowledge graph
       → community detection → (LLM) per-community summaries

[ Query (online, per question) ]
  question → find relevant entities/communities → gather context via the graph → LLM answer
```

## 2. What indexing does

Three steps are key.

**① Build the graph.** Exactly as in [Part 3](../part3/07-llm-extraction.md) — extract entities and relations from chunks to build a knowledge graph. This is GraphRAG's skeleton.

**② Detect communities.** Split the graph into **densely connected clusters (communities).** Algorithms like Leiden find "nodes that connect heavily to each other" as a group. One community roughly corresponds to one topic/cluster (e.g., a "drugs, ingredients, diseases related to aspirin" cluster).

**③ Summarize communities.** An LLM **pre-summarizes** each community. This is the secret to the "global questions" from [Ch 1](../part1/01-why-graph.md) — when a question like "what are the overall themes?" arrives, instead of re-reading all documents you synthesize the pre-built community summaries.

## 3. Why it's expensive, and when it's worth it

Indexing is expensive. You sweep the whole corpus with an LLM to extract entities and relations, then generate a summary per community, so **LLM calls accumulate in proportion to document volume.** Far heavier than vector RAG's indexing (embeddings only).

So GraphRAG is no free lunch. If most questions are "find similar documents," this cost is waste. What justifies the indexing cost is the questions from [Ch 1](../part1/01-why-graph.md) — multi-hop, synthesis, and especially **global summary.** We compare this value/cost directly in [Ch 11](11-vector-vs-graph.md), and cost management is covered in Part 5's production chapter.

!!! note "Microsoft GraphRAG"
    What popularized this indexing→community→summary pipeline is Microsoft Research's GraphRAG (2024). There's an open-source implementation, and many follow-on tools follow this structure. Details vary by implementation, but the "graph + community summaries" skeleton is common.

## 4. Common failure points

**Re-running indexing every time.** Indexing is an offline batch. Re-indexing the whole corpus every time a few documents change makes cost explode. You need an incremental update strategy (→ Part 5, production).

**Skipping community summaries.** Building only the graph without community summaries throws away GraphRAG's biggest strength, global questions. Fine if you only need multi-hop, but if you have "overall theme" questions, summaries are central.

## 5. Exercises & next

### Check your understanding

1. Split your domain's questions into "local (around a specific entity)" and "global (whole synthesis)." Which is more common?
2. If indexing cost scales with document volume, at what corpus size / update frequency does GraphRAG become burdensome?

### Next

With the index ready, how do you search the graph when a question arrives? On to local, global, and community strategies → [Ch 10](10-search-strategies.md).

---

## Sources

- Microsoft Research (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization.* arXiv:2404.16130
- Traag et al. (2019). *From Louvain to Leiden: guaranteeing well-connected communities.* (community detection)
