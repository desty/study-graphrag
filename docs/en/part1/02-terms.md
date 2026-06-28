# Ch 2. Ontology, Knowledge Graph, Taxonomy — Terms Sorted Out

!!! abstract "What you'll learn"
    - Four often-conflated terms — glossary, taxonomy, thesaurus, ontology — laid out as a spectrum
    - The decisive difference between an **ontology (schema)** and a **knowledge graph (instance data)**
    - Pinning it down with a "class vs. instance" analogy
    - The terminology gap between the RDF/OWL camp and the property-graph camp

!!! quote "Assumes"
    [Ch 1](01-why-graph.md). You've got a feel for why graphs matter; now we settle the vocabulary.

---

## 1. Concept — a spectrum of structuring

Ways to structure knowledge form a spectrum from simple to complex. People lump them together, but their expressive power differs.

| Level | What | Expresses | Example |
|---|---|---|---|
| **Glossary** | Term–definition list | Word meanings | Company acronym list |
| **Taxonomy** | Hierarchical classification | "is-a / parent–child" | Biological taxonomy, category tree |
| **Thesaurus** | Classification + synonyms/related | Hierarchy + similar/related | Keyword thesaurus |
| **Ontology** | Concepts + arbitrary relations + constraints | Rich domain meaning | "a drug treats a disease," "dosage constraint" |

The higher you go, the richer the relationships you can express. A taxonomy handles only one kind ("parent–child"), but an ontology captures arbitrary relations like *treats · owns · cites* and even constraints (a person is employed at one place at a time).

## 2. Why you need this — ontology and knowledge graph are different layers

This is the crux, and the most commonly confused point.

- **Ontology = schema.** It defines what *kinds* of entities exist in the domain (classes) and what *kinds* of relationships are possible between them. "A person can be employed by a company," "a drug treats a disease" — the **rules and frame.**
- **Knowledge graph = instance data.** The frame filled with actual data. "Sujin Lee —[employed]→ Apple," "Aspirin —[treats]→ headache" — a graph of **concrete facts.**

!!! tip "Remember it as class vs. instance"
    The ontology is the **class definition** in programming; the knowledge graph is the **objects (instances)** stamped out from it. If the ontology is `class Person { employedBy: Company }`, the knowledge graph is `Person("Sujin Lee").employedBy = Company("Apple")`.

You can build a knowledge graph without an ontology (a loosely-schema'd graph). But having one helps a lot with consistency checking, inference, and guiding an LLM to "fill the graph correctly" → Part 2.

## 3. Where it's used — the two camps' terminology

There are two big technical camps for graphs, and they call the same things by different names. Ch 3 covers the data models in detail; here we just pin the vocabulary.

- **RDF / semantic-web camp** — uses "ontology" strictly. Define classes, properties, and constraints with OWL; store data as triples; query with SPARQL. Strong in academia, open government data, life sciences.
- **Property-graph camp** — Neo4j and friends. Prefers "graph schema" / "data model" over "ontology," and attaches properties to nodes and relationships. Query with Cypher. Popular in industry and applications.

Both share the essence: structuring knowledge as concepts and relations. This course centers on the more hands-on **property graph (Neo4j)** while explaining concepts camp-neutrally.

## 4. Common failure points

**Lumping "ontology = knowledge graph."** Mix the schema (the frame) with the instances (the data) and your design tangles. Think separately about "what relationships are *possible*" (ontology) and "what facts actually *exist*" (KG).

**Starting from an over-engineered ontology.** Trying to write a perfect OWL ontology first means you never ship a first graph. In practice you start with a light schema and grow it as you fill in data.

## 5. Exercises & next

### Check your understanding

1. Write your domain up one level at a time: glossary → taxonomy → ontology. Where do you start needing "relations other than parent–child"?
2. Is "a customer places an order" an ontology or a knowledge graph? What about "Younghee Kim placed order #3"?

### Next

We drop down to the minimal unit for representing knowledge — the triple — and see how RDF and property graphs hold the same fact differently → [Ch 3](03-triples.md).

---

## Sources

- Noy & McGuinness (2001). *Ontology Development 101.* Stanford KSL
- W3C. *OWL 2 Web Ontology Language Primer.*
- Hogan et al. (2021). *Knowledge Graphs.* ACM Computing Surveys
