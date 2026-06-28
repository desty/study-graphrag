# Ch 3. Triples and Graph Data Models (RDF vs LPG)

!!! abstract "What you'll learn"
    - The minimal unit of knowledge — the **triple** (subject–predicate–object)
    - Two graph data models — **RDF** (triple store) and **LPG** (labeled property graph)
    - How the two hold the same fact differently, and when to pick which
    - Why this course centers on LPG (Neo4j)

!!! quote "Assumes"
    [Ch 2](02-terms.md). You need the ontology (schema) vs. knowledge graph (instance) distinction in your head.

---

## 1. Concept — all knowledge breaks down into triples

The atom of a knowledge graph is the **triple.** Write a single fact as *subject — predicate — object.*

```text
(Sujin Lee)  —[employed]→  (Apple)
 subject        predicate     object
```

"Sujin Lee is employed at Apple" is one triple. Several sentences also break into a bundle of triples.

```text
(Aspirin) —[treats]→ (headache)
(Aspirin) —[ingredient]→ (acetylsalicylic acid)
(headache) —[location]→ (head)
```

String these triples together and you get a graph. Subjects and objects are nodes, predicates are edges. A knowledge graph is, in the end, **a giant graph where countless triples connect through shared nodes.**

## 2. Why you need this — representation split into two camps

Historically, two ways of storing and querying the same triples diverged. Both are "node–edge–node," but they emphasize different things.

### RDF (Resource Description Framework)

The semantic-web standard. It sees everything as **pure triples.** Nodes and predicates carry global identifiers (URIs), and to attach a property to a relationship itself you have to mint more triples (reification). You query with **SPARQL** and reason with OWL.

```turtle
:SujinLee  :employed  :Apple .
:Apple     :industry  :Technology .
```

Strengths: standardization, interoperability, global identifiers. Strong in open data, life sciences, and knowledge integration.

### LPG (Labeled Property Graph)

The industry standard, exemplified by Neo4j. It attaches properties (key-value) **directly to both nodes and relationships.** "Sujin Lee employed at Apple *since 2019*" — pinning a timestamp onto the relationship is natural. You query with **Cypher.**

```cypher
(SujinLee:Person)-[:EMPLOYED {since: 2019}]->(Apple:Company {industry: "Technology"})
```

Strengths: intuitive modeling, relationship properties, fast traversal. Popular in applications and real-time services.

## 3. Where it's used — same fact, different container

Comparing how the two hold "Sujin Lee has been employed at Apple since 2019" makes the difference sharp.

| | RDF | LPG |
|---|---|---|
| Property on a relation (since=2019) | Can't attach directly → needs reification | Just `{since: 2019}` on the relation |
| Identifiers | URIs (global) | Internal IDs (local) |
| Query language | SPARQL | Cypher |
| Reasoning (OWL) | Strong | Weak (at the app level) |
| Main use | Open / academic / data integration | Apps / real-time / recommendations |

If regular reasoning and global interoperability are central, RDF; if relationships carry many properties and intuitive modeling and fast traversal matter, LPG is more comfortable.

## 4. This course's choice — LPG/Neo4j

This course centers on **LPG (Neo4j)** for three reasons. Attaching properties to relationships fits GraphRAG's real-world patterns (timestamps, weights, source attribution) well; Cypher has a low barrier to entry; and the GraphRAG open-source ecosystem mostly clusters on the property-graph side. We mention RDF/SPARQL in context when relevant.

!!! note "Neither one is 'correct'"
    The concept — representing knowledge as triples — is identical. The data model is a tooling choice, not a creed. The graph thinking you learn here carries straight over to RDF.

## 5. Common failure points

**Unsure whether a relation should be a node or a property.** When "employment" accrues a start date, a title, and a department, it's better to promote it from a relationship to its own node (e.g., `Employment`). A common modeling decision in LPG.

**Splitting triples too finely.** Turn every adjective into a triple and the graph explodes. The balance is to make only "the relationships a question needs to follow" into edges, and leave the rest as properties.

## 6. Exercises & next

### Check your understanding

1. Break 3 sentences from your domain into triples. What becomes a node, what becomes a relation, what becomes a property?
2. If "employment" carries a start date, end date, and title, how would you express it in RDF vs. LPG?

### Next

On to Part 2 — the step before triples: **ontology modeling**, where you design which concepts and relations to have → Part 2.

---

## Sources

- W3C. *RDF 1.1 Primer.*
- Robinson, Webber & Eifrem (2015). *Graph Databases* (2nd ed.), O'Reilly
- Angles et al. (2017). *Foundations of Modern Query Languages for Graph Databases.* ACM Computing Surveys
