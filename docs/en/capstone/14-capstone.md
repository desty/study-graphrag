# Ch 14. Capstone — A Domain Knowledge Graph + GraphRAG Pipeline

!!! abstract "What you'll build"
    - An **end-to-end pipeline** tying together every piece from Parts 1–5
    - Ontology design → graph construction → GraphRAG indexing → routed retrieval → evaluation
    - A step-by-step checklist to run one domain of your own all the way through

!!! quote "Assumes"
    All of Parts 1–5. This chapter is **integration**, not new concepts.

---

## 1. Goal — one domain, all the way

We've learned the pieces separately. The capstone ties them to one domain. Small and concrete is better — an internal wiki, a set of product docs, 50 papers in a specific field. Ideally the very corpus where "vector RAG couldn't solve multi-hop/global questions."

## 2. The pipeline — 5 stages

![Capstone — domain KG + GraphRAG pipeline, 5 stages](../assets/diagrams/ch14-pipeline.svg#only-light)
![Capstone — domain KG + GraphRAG pipeline, 5 stages](../assets/diagrams/ch14-pipeline-dark.svg#only-dark)

The whole picture follows this course directly.

```text
① Ontology design      (Part 2)  competency questions → core concepts/relations
② Graph construction   (Part 3)  DB schema + LLM text extraction → Neo4j
③ GraphRAG indexing    (Part 4)  community detection → community summaries
④ Routed retrieval     (Part 4)  classify question → vector / local / global
⑤ Evaluation & ops     (Part 5)  retrieval/answer evaluation, incremental updates
```

## 3. Step-by-step checklist

### ① Ontology ([Ch 4](../part2/04-ontology-design.md))

- [ ] Wrote 10–20 competency questions (reflecting real traffic).
- [ ] Settled on 5–10 core concepts and a relation-type list.
- [ ] Didn't over-engineer — only what the questions demand.

### ② Graph construction ([Ch 5](../part2/05-from-schema.md)–[Ch 8](../part3/08-temporal-graphiti.md))

- [ ] If you have structured data, extracted the skeleton from the DB schema.
- [ ] Extracted unstructured text with the LLM using the ontology as a guardrail.
- [ ] Merged duplicate nodes with entity resolution (auto only the certain ones, review the rest).
- [ ] Pinned timestamps onto changing facts (where needed).

### ③ Indexing ([Ch 9](../part4/09-what-is-graphrag.md))

- [ ] Ran community detection.
- [ ] Generated per-community summaries (essential if you have global questions).

### ④ Retrieval ([Ch 10](../part4/10-search-strategies.md)–[Ch 11](../part4/11-vector-vs-graph.md))

- [ ] Added a router that classifies questions into factual/multihop/global.
- [ ] Local search gives graph facts + source chunks together.
- [ ] Factual questions go to cheap vector RAG.

### ⑤ Evaluation & ops ([Ch 12](../part5/12-evaluation.md)–[Ch 13](../part5/13-production.md))

- [ ] Built an eval set across the retrieval and answer layers.
- [ ] Compared against a vector-RAG baseline to confirm the questions where GraphRAG earns its keep.
- [ ] Set up an incremental update path (not full re-indexing).

## 4. What you walk away with

Run this pipeline through once and you return to the opening question — **"does this domain really need GraphRAG?"** The answer is set by your corpus and question distribution. If it's mostly factual questions, vector RAG is enough; if multi-hop and global are central, the graph earns its keep. The capstone's real deliverable isn't just "a working GraphRAG" but **a firsthand-validated judgment of when to reach for a graph.**

## 5. Where to go next

- **Deeper hybrid** — vector entry + graph expansion, refined within a single question.
- **Agent memory** — [Ch 8](../part3/08-temporal-graphiti.md)'s time-aware graph as an agent's long-term memory.
- **Ontology evolution** — a loop that grows the ontology each time a new question type appears.

---

## Closing

If vector RAG knows only "what's similar," a knowledge graph knows "how things connect." GraphRAG makes that connection retrievable, and its value lives in multi-hop and global questions. But it's no free lunch — routing by question, managing cost, and validating with evaluation are all part of one set.

Related guides: [Ontology & Knowledge Graphs](https://desty.github.io/ontology-knowledge-graph-guide/en/) · [Complete RAG Guide](https://desty.github.io/rag-guide/en/) · [AI Eval](https://desty.github.io/ai-eval-guide/en/)
