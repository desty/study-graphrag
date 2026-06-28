# Learning System

## How chapters are structured

Every chapter follows the same skeleton. Get the concept first, name why it's needed, move your hands with a minimal example, then collect the spots where things break.

- **What you'll learn** — the summary at the top of the chapter. When lost, come back here.
- **Concept** — terms and intuition. Build the mental model before the code.
- **Why you need this** — what problem the technique solves; what you miss without it.
- **Minimal example / hands-on** — the smallest code that runs. Copy-paste and try it.
- **Common failure points** — where people actually trip in the field.
- **Exercises & next** — what to try, and the link to the next chapter.

## What "hands-on" means

The hands-on chapters (Parts 3 and 4) are written to follow along in Colab or local Docker. The graph DB is Neo4j (free tier / Docker), and GraphRAG uses an open-source implementation. The code aims for "the minimum that shows the concept," not a verbatim copy of a production library.

!!! tip "Code blocks are there to be copied"
    Grab them with the copy button at the top right. Just read them once before pasting — run code you understand.

## What you walk away with

By the end of the course you can build the following yourself.

- Design a domain's concepts and relationships as an ontology, and map it to a graph schema
- Use an LLM to extract entities and relations from unstructured text and populate a knowledge graph
- Stand up a GraphRAG pipeline and compare/evaluate it against vector RAG on the same questions
- Draw the path to production accounting for graph updates, cost, and latency
