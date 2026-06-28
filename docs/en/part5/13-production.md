# Ch 13. Production — Graph Updates, Cost, Latency

!!! abstract "What you'll learn"
    - An **incremental update** strategy that avoids full re-indexing
    - Levers to cut GraphRAG's big cost (indexing LLM calls)
    - Handling global-search latency — summary caching and routing
    - Where to monitor graph quality in production

!!! quote "Assumes"
    Indexing cost from [Ch 9](../part4/09-what-is-graphrag.md), evaluation from [Ch 12](12-evaluation.md).

---

## 1. Concept — GraphRAG's operational load concentrates in indexing

Operating vector RAG is relatively simple — make embeddings, run a vector DB, retrieve. GraphRAG, as we saw in [Ch 9](../part4/09-what-is-graphrag.md), has heavy indexing (LLM extraction + community summaries). So the three production concerns — updates, cost, latency — mostly pile up at the **indexing stage.**

## 2. Updates — incremental indexing

The most common mistake is re-indexing the whole corpus every time a document changes. With a large corpus that's an immediate cost explosion. The skeleton of an incremental strategy:

1. **Extract only new/changed documents** → `MERGE` the new triples into the existing graph (the idempotent loading from [Ch 6](../part3/06-neo4j-cypher.md) pays off here).
2. **Re-summarize only affected communities.** Re-summarize only the communities that gained new nodes, and leave unchanged community summaries alone. Don't rebuild all summaries.
3. **If time-awareness is needed,** use [Ch 8](../part3/08-temporal-graphiti.md)'s bi-temporal to close old facts and open new ones (not delete).

```text
new docs → extract → MERGE (merge into graph)
         → identify changed communities → re-summarize only those
         (leave the rest of the graph and summaries untouched)
```

## 3. Cost — cut indexing LLM calls

Most of the indexing cost is LLM calls. A few levers:

- **Extract with a small model.** Entity/relation extraction doesn't need heavy reasoning. As in [Ch 7](../part3/07-llm-extraction.md), a small/fast model is enough and cuts cost a lot.
- **Cache and batch.** Hash-cache so you don't re-extract the same chunk, and bundle extraction/summarization through a batch API (same spirit as the cost levers in [AI Assistant Engineering Ch30](https://desty.github.io/study-ai-assistant-engineering/en/)).
- **Selective graphing.** Don't graph every document. Graph only the corpus that multi-hop/global questions reach, and leave the rest on vector RAG.

## 4. Latency — global search is slow

The main culprit in query latency is the global search's map-reduce ([Ch 10](../part4/10-search-strategies.md)). The more communities, the more map calls accumulate. Responses:

- **Summaries pre-built and hierarchical.** Community summaries are built at indexing time, so at query time you only read them. Putting communities into a hierarchy (summaries of parent communities) can narrow the map scope.
- **Save global with routing.** Exactly [Ch 11](../part4/11-vector-vs-graph.md)'s conclusion — don't use global on factual/local questions. Global only for genuinely global questions.

## 5. Monitoring — graph quality rots quietly

Easy to miss in production is drift in graph quality.

- **Entity-resolution drift.** As new documents arrive, the same entity accumulates under different names and nodes split ([Ch 7](../part3/07-llm-extraction.md)). Periodically monitor the count of suspected-duplicate nodes.
- **Orphan nodes / a fragmented graph.** A rising number of nodes floating without relations signals that extraction is missing relationships.
- **Evaluation regression.** Run [Ch 12](12-evaluation.md)'s eval set regularly and gate against indexing/model changes that drop retrieval quality.

## 6. Common failure points

**The full-reindex habit.** Re-running everything for a small change becomes unaffordable. Incremental should be the default.

**Extracting everything with a big model.** Using a frontier model for extraction multiplies indexing cost. A small model + review is the balance.

**Build the graph and forget it.** A graph is a living asset; without updates and monitoring it quietly ages. Same as the [AI Eval Guide](https://desty.github.io/ai-eval-guide/en/)'s "an eval is a living asset."

## 7. Exercises & next

### Hands-on

1. Compare the LLM-call count of "full re-index" vs. "incremental (MERGE + re-summarize only affected communities)" when adding 5 documents.
2. When you switch the extraction model from big to small, measure cost and extraction quality with [Ch 12](12-evaluation.md).
3. Write a monitoring query that counts suspected-duplicate nodes and orphan nodes.

### Next

The last one. We tie all the pieces so far into a single pipeline in the capstone → [Ch 14](../capstone/14-capstone.md).

---

## Sources

- Microsoft. *GraphRAG: incremental indexing* docs
- Google SRE. *The Site Reliability Workbook* (capacity · cost)
