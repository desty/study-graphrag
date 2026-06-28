# Knowledge Graphs & GraphRAG

**A hands-on course from ontology to building knowledge graphs, GraphRAG, and evaluation/production.** See where vector RAG hits a wall, model knowledge as triples, build a graph with Neo4j and an LLM, retrieve with GraphRAG and compare it head-to-head with vector RAG. At the end you run one domain knowledge-graph + GraphRAG pipeline all the way through.

## What this course covers / doesn't

<div class="infocards">
  <div class="card">
    <h4>Covers</h4>
    <p>Ontology design · RDF/LPG · triples · Neo4j/Cypher · LLM entity & relation extraction · GraphRAG (local/global/community) · vector vs graph comparison · evaluation · production</p>
  </div>
  <div class="card">
    <h4>Mentions lightly</h4>
    <p>SPARQL · OWL reasoning · graph embeddings (GNNs) · time-aware graphs like Graphiti</p>
  </div>
  <div class="card">
    <h4>Doesn't cover</h4>
    <p>Graph DB internals · large-scale distributed graph processing · the semantic web stack broadly</p>
  </div>
  <div class="card">
    <h4>Assumes</h4>
    <p>Intro Python · a feel for LLM/RAG basics (embeddings, retrieval) · Colab or local Docker</p>
  </div>
</div>

## Why this course

Vector RAG finds "similar documents" well, but it can't find "how A and B connect." On questions that need synthesizing across many facts or following relationships, vector search breaks down. Knowledge graphs fill that gap, and GraphRAG bolts the graph onto RAG to make "connections" retrievable.

This course doesn't just skim concepts. You actually build a graph, run GraphRAG, and throw the same question at vector RAG and GraphRAG to **see where they diverge** with your own eyes.

## Where to go

- [Learning System](about/system.md) — how chapters are structured
- [Curriculum](about/curriculum.md) — all 14 chapters + capstone
- [Start Part 1](part1/01-why-graph.md) — where does vector RAG hit a wall

## Related

- Guide: [Ontology & Knowledge Graphs](https://desty.github.io/ontology-knowledge-graph-guide/en/) — the concepts on one page
- Guide: [Complete RAG Guide](https://desty.github.io/rag-guide/en/) · [AI Eval](https://desty.github.io/ai-eval-guide/en/)
- Blog: [desty.github.io/blog](https://desty.github.io/en/blog/)
