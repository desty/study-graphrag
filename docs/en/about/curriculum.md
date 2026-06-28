# Curriculum

14 chapters, 5 parts + a capstone. It moves from concepts (Parts 1–2) through building by hand (Parts 3–4) to evaluation and operations (Part 5).

## Part 1. Why Knowledge Graphs

Start from the limits of vector RAG, sort out the terms ontology / knowledge graph / taxonomy, and pin down the graph data model (triples).

1. **The Limits of Vector RAG and Where Graphs Come In** — where it breaks, what graphs fill
2. **Ontology, Knowledge Graph, Taxonomy — Terms Sorted Out** — separating words that get conflated
3. **Triples and Graph Data Models (RDF vs LPG)** — the minimal unit for representing knowledge

## Part 2. Ontology Modeling

Design a domain as concepts, relations, and constraints, and see the practical path of pulling an ontology out of an existing DB schema.

4. **Ontology Design — Concepts, Relations, Constraints, Hierarchy**
5. **Extracting an Ontology from a DB Schema**

## Part 3. Building a Knowledge Graph (hands-on)

Put data into a graph DB directly, and build a graph from unstructured text with an LLM.

6. **Neo4j and Cypher Basics** — nodes, relationships, queries
7. **Building a KG with an LLM — Entity & Relation Extraction**
8. **Time-Aware Graphs and Graphiti**

## Part 4. GraphRAG (hands-on)

Bolt the graph onto RAG. Indexing, retrieval strategies, and a head-to-head with vector RAG.

9. **What GraphRAG Is — Architecture and Indexing**
10. **Graph Retrieval Strategies — Local, Global, Community Summaries**
11. **Vector RAG vs GraphRAG, Head to Head**

## Part 5. Evaluation and Production

What to evaluate GraphRAG on, and how to handle graph updates, cost, and latency.

12. **Evaluating GraphRAG — What and How to Measure**
13. **Production — Graph Updates, Cost, Latency**

## Capstone

14. **A Domain Knowledge Graph + GraphRAG Pipeline** — Parts 1–5 as one

!!! note "Progress"
    This course is released part by part. Start with Part 1.
