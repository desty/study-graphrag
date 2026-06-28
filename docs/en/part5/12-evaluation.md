# Ch 12. Evaluating GraphRAG — What and How to Measure

!!! abstract "What you'll learn"
    - Evaluating GraphRAG at two layers — **retrieval** and **answer**
    - Graph-specific metrics — multi-hop accuracy, **coverage** for global questions, extraction quality
    - Applying LLM-as-Judge and Ragas to GraphRAG
    - A realistic, semi-automated way to build a gold set for multi-hop and global questions

!!! quote "Assumes"
    The comparison from [Ch 11](../part4/11-vector-vs-graph.md). For evaluation fundamentals, read alongside the [AI Eval Guide](https://desty.github.io/ai-eval-guide/en/).

---

## 1. Concept — split it into two layers

To turn the comparison from [Ch 11](../part4/11-vector-vs-graph.md) into a system rather than a "feel," split evaluation into two layers.

- **Retrieval quality** — did the graph actually fetch the context the answer needed? When an answer is wrong, this separates "retrieval failed to fetch it" from "the LLM failed to use it."
- **Answer quality** — is the final answer accurate, grounded, and complete?

This split matters for debugging. When an answer is bad and you don't know which of the two layers is the culprit, you can't find what to fix.

## 2. Graph-specific metrics

On top of vector-RAG evaluation, GraphRAG has its own measurement points.

**① Multi-hop accuracy.** On questions that require following relationships, did the graph hit the right path? Don't just look at exact-match; break accuracy down by hop count, and the weakness shows — "where does accuracy drop, at how many hops."

**② Coverage for global questions.** A "top 5 themes" question has no single answer. More than accuracy, the keys are **completeness (coverage)** — did it miss an important theme — and **no duplication**. Single-answer matching can't evaluate this.

**③ Extraction quality.** GraphRAG's quality ceiling is the graph's quality. If the indexing-stage entity/relation extraction is inaccurate ([Ch 7](../part3/07-llm-extraction.md)), there's a limit no matter how good retrieval and answering are. Measure extraction precision/recall separately on a sample of gold triples.

## 3. How — tools and methods

**Retrieval evaluation.** Use context precision/recall from tools like [Ragas](https://docs.ragas.io), but reflect that in GraphRAG the "fetched context" is a **subgraph + chunks**, not just chunks. For multi-hop questions, frame it as "did it fetch the nodes for all the needed hops."

**Answer evaluation.** Exactly the three tiers from the [AI Eval Guide](https://desty.github.io/ai-eval-guide/en/) — auto-match for clear factual questions, LLM-as-Judge for global/free-form, humans at decisive moments. The same principles apply: judge biases (position, verbosity), and "validate the judge against a gold set first."

```python title="eval_sketch.py"
# Route to a different evaluator by question type (the Eval Guide's pyramid)
def evaluate(q, answer, gold):
    if q.type == "factual":
        return exact_or_f1(answer, gold)          # automatic, cheap
    elif q.type == "global":
        return judge_coverage(q, answer, gold)    # LLM-judge: missing/duplicate themes
    else:  # multihop
        return judge_correctness(q, answer, gold) # LLM-judge: includes the evidence path?
```

## 4. The gold set — multi-hop and global answers are hard to author

What trips up evaluation is the gold set. Factual answers are easy to write down, but for multi-hop and global questions, authoring the answer is itself labor.

The realistic approach is **semi-automated.** Since you have the graph, you can **generate multi-hop questions in reverse** — "pull a path, and have an LLM write the question that path answers." Now the question is created while you already know the answer (the path). Humans only review the generated question/answer. Balanced against [Ch 11](../part4/11-vector-vs-graph.md)'s point about reflecting real traffic distribution, don't rely on synthetic questions alone.

## 5. Common failure points

**Not splitting retrieval and answer.** Scoring only the final answer leaves you blind to "why it was wrong." Measure the retrieval layer separately to tell a graph problem from a generation problem.

**Single-answer matching on global questions.** Comparing "5 themes" to a gold string is meaningless. You need a judge that looks at coverage and duplication.

**Ignoring extraction quality.** If the graph is wrong but you tune only retrieval and answering, you hit a ceiling. Sample-check indexing quality first.

## 6. Exercises & next

### Hands-on

1. Split the comparison from [Ch 11](../part4/11-vector-vs-graph.md) into retrieval and answer layers and score each.
2. Pull 10 two-hop paths from the graph and semi-automatically generate multi-hop questions/answers.
3. Add a coverage judge to one global question and see whether it catches a missing theme.

### Next

With evaluation in place, on to operations — how to handle graph updates, cost, and latency → [Ch 13](13-production.md).

---

## Sources

- Es et al. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation.* arXiv:2309.15217
- Microsoft Research (2024). *GraphRAG* (claim/coverage evaluation). arXiv:2404.16130
