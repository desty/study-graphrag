# Ch 11. Vector RAG vs GraphRAG, Head to Head

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part4/ch11_vector_vs_graph.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "What you'll learn"
    - Running **vector RAG and GraphRAG side by side** on the same corpus and question set
    - Seeing where they diverge by question type (factual, multi-hop, global)
    - A balanced conclusion that weighs not just accuracy but **cost and latency**
    - The practical answer — not "one or the other" but "route by question"

!!! quote "Assumes"
    [Ch 9](09-what-is-graphrag.md) and [Ch 10](10-search-strategies.md). Assumes you can build a vector-RAG baseline.

---

## 1. Concept — numbers, not words

So far we've argued in words that "GraphRAG is strong on multi-hop and global." This chapter **runs it and checks.** The keys to a fair comparison are three: same corpus, same question set, same generation model. The only thing that should differ is the retrieval method.

## 2. Experiment design — questions in three buckets

Throw the question set in the three buckets from [Ch 1](../part1/01-why-graph.md).

```python title="question_set.py"
QUESTIONS = {
    "factual":   ["What is aspirin's main ingredient?",       # factual — answer in one chunk
                  "What year was this company founded?"],
    "multihop":  ["What other drugs use the same ingredient as aspirin?",  # relation tracing
                  "What datasets did the papers cited by A use?"],
    "global":    ["What are the 5 core themes of this document set?",       # global synthesis
                  "What risk is mentioned most across the whole corpus?"],
}
```

Send each question to both systems.

```python title="compare.py"
def vector_rag(q):
    chunks = vector_store.search(q, k=5)        # embedding top-k
    return llm_answer(q, context=chunks)

def graph_rag(q, mode):                          # mode: local | global | hybrid
    ctx = graph_search(q, mode)                  # strategies from Ch 10
    return llm_answer(q, context=ctx)

for cat, qs in QUESTIONS.items():
    for q in qs:
        v = vector_rag(q)
        g = graph_rag(q, mode=route(cat))        # factual→hybrid, multihop→local, global→global
        record(cat, q, v, g)
```

## 3. Typical results — where they diverge

Run it for real and you generally get this picture (varies by corpus and implementation, but the trend is consistent).

| Question type | Vector RAG | GraphRAG | Note |
|---|---|---|---|
| Factual | ◎ sufficient | ◎ on par | if the answer's in one chunk, vector is enough |
| Multi-hop | △ frequent misses | ◎ strong | the gap widens on relation-tracing questions |
| Global synthesis | ✗ structural limit | ◎ strong | community summaries are decisive |
| **Indexing cost** | ◎ low (embeddings only) | ✗ high (LLM extraction + summaries) | the cost from [Ch 9](09-what-is-graphrag.md) |
| **Query latency** | ◎ fast | △ global is slow | map-reduce calls accumulate |

The point is clear. **On factual questions GraphRAG offers nothing better** — it's just more expensive. GraphRAG's value comes from multi-hop and global. So "everything through GraphRAG" is the wrong conclusion.

## 4. So the answer is routing

The routing from [Ch 10](10-search-strategies.md) becomes the conclusion here. A hybrid system that classifies the question — factual to cheap vector RAG, multi-hop and global to GraphRAG — captures accuracy and cost at once.

```python title="router.py"
def answer(q):
    kind = classify(q)                  # LLM or rule-based classification
    if kind == "factual":
        return vector_rag(q)            # cheap and sufficient
    elif kind == "multihop":
        return graph_rag(q, "local")
    else:  # global
        return graph_rag(q, "global")
```

This is the same spirit as the [AI Eval Guide](https://desty.github.io/ai-eval-guide/en/)'s "cheapest first, expensive only when needed" pyramid — retrieval has tiers too.

## 5. Common failure points

**Looking only at accuracy and ignoring cost.** Even if GraphRAG is "more accurate," using it on every question wrecks cost and latency. Always put cost and latency columns in the comparison table.

**An unfair comparison.** Set vector RAG's k too small or chunk poorly and GraphRAG wins unfairly. The comparison only means something after you tune the baseline properly.

**A skewed question set.** Gather only global questions and GraphRAG wins by a landslide, of course. The conclusion is realistic only if it reflects your real traffic's question distribution.

## 6. Exercises & next

### Hands-on

1. Build both systems on a small corpus (e.g., 20 wiki articles) and fill the comparison table with 6 questions across the three types.
2. Record not just accuracy but **token cost and latency** for each answer.
3. Add a router that sends "factual to vector, the rest to graph," and measure how much total cost drops.

### Next

On to Part 5, where we turn this comparison into systematic **evaluation** — what and how to measure GraphRAG (coming soon).

---

## Sources

- Microsoft Research (2024). *From Local to Global: A Graph RAG Approach.* arXiv:2404.16130
- Han et al. (2024). *Retrieval-Augmented Generation with Graphs (GraphRAG): A Survey.* arXiv:2501.00309
