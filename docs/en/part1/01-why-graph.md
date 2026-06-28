# Ch 1. The Limits of Vector RAG and Where Graphs Come In

!!! abstract "What you'll learn"
    - What vector RAG **does well** and what it **structurally can't do**
    - Why vector search breaks down on multi-hop, relational, and global-aggregation questions
    - The gap knowledge graphs fill, and a preview of GraphRAG
    - Correcting the common myth that "graphs replace vectors"

!!! quote "Assumes"
    A basic feel for embeddings and vector search (find "similar chunks" by cosine similarity) is enough. If not, skim the [RAG Guide](https://desty.github.io/rag-guide/en/) first.

---

![Vector RAG vs knowledge graph — same question, where they diverge](../assets/diagrams/ch1-vector-vs-graph.svg#only-light)
![Vector RAG vs knowledge graph — same question, where they diverge](../assets/diagrams/ch1-vector-vs-graph-dark.svg#only-dark)

## 1. Concept — vector search only knows "similar"

Vector RAG is simple and powerful. Chunk documents and embed them, embed the question too, and pull the nearest chunks in vector space. For "find text that means something similar to this question," nothing beats it.

The problem is that "similarity" is all vector RAG knows. Vector space has **no relationships.** "Steve Jobs" and "Apple" may sit close together, but whether the link between them is *founder*, *employee*, or *competitor* is not captured in the vectors. It knows closeness, not how things connect.

## 2. Why you need this — three questions where vectors break

Vector RAG is structurally weak on three kinds of question.

**① Multi-hop — questions that follow relationships.**
"What datasets did the studies citing this paper use?" has to cross two or three steps: *paper → citation → dataset*. Vector search only hands back "similar chunks" in one shot; it can't chain the steps. Unless the answer happens to be spelled out in a single chunk, you're stuck.

**② Synthesis — questions that gather scattered facts.**
"Who are the CEOs of all this company's subsidiaries?" The answer is scattered across a different document per subsidiary. Pull top-k chunks and you catch some while the rest get cut. Raise k and [the longer context drops accuracy](https://desty.github.io/context-engineering-guide/en/).

**③ Global — questions that need the whole corpus.**
"What are the five core themes across this set of reports?" No single chunk holds this answer. You have to summarize across the whole thing, and vector search is, by design, a tool for "pulling out a part."

## 3. Where it's used — the gap knowledge graphs fill

A knowledge graph makes information explicit as **nodes (entities) and edges (relations).** Jobs —[founded]→ Apple, Apple —[owns]→ subsidiary. Relationships are baked right into the data, so the three questions above become tractable.

- Multi-hop → traverse the edges.
- Synthesis → gather "all the X connected to this node" in one pass.
- Global → cluster the graph into communities and pre-summarize each (→ Part 4, retrieval strategies).

| Question type | Vector RAG | Knowledge graph |
|---|---|---|
| "Find similar content" | ◎ strong | △ overkill |
| Multi-hop (follow relations) | ✗ weak | ◎ strong |
| Synthesis (scattered facts) | △ frequent misses | ◎ strong |
| Global summary | ✗ structural limit | ◎ (community summaries) |

## 4. GraphRAG preview

So is it a choose-one between the two? No. **GraphRAG bolts the graph onto RAG to make "connections" retrievable.** A question comes in, you find the relevant nodes, gather context by following the surrounding relationships, then hand that to the LLM. A hybrid — vector search to find an entry point, graph to expand the neighborhood — is common too. We build the architecture in Part 4.

## 5. Common failure points

**"Graphs replace vectors"** — the most common myth. Vectors are still the best at "find similar things," and graphs are strong at "follow connections." They're complementary, not competitors. Real systems usually mix both.

**"Let's build the graph first"** — graph construction isn't free. It costs entity/relation extraction, schema design, and update overhead. If most of your questions are "find similar documents," vector RAG is enough. The graph earns its keep when multi-hop, synthesis, and global questions are central.

## 6. Exercises & next

### Check your understanding

1. Write down 5 real questions from your domain and classify each as solvable by "similarity" or as needing "relationships."
2. What do the questions that vector RAG can't solve — no matter how large you make top-k — have in common?

### Next

Let's sort out the terms. "Ontology," "knowledge graph," and "taxonomy" get conflated, and separating them makes everything else easier → [Ch 2](02-terms.md).

---

## Sources

- Microsoft Research (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization.* arXiv:2404.16130
- Lewis et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* arXiv:2005.11401
- Liu et al. (2024). *Lost in the Middle: How Language Models Use Long Contexts.* TACL
