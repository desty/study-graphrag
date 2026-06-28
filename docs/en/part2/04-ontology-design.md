# Ch 4. Ontology Design — Concepts, Relations, Constraints, Hierarchy

!!! abstract "What you'll learn"
    - The four things an ontology defines — concepts (classes), relations, constraints, hierarchy
    - How to start design from **competency questions**
    - A practical process for starting small and growing, and a feel for avoiding over-engineering
    - The criteria for deciding relation vs. property, and when to promote something to a class

!!! quote "Assumes"
    The ontology = schema distinction from [Ch 2](../part1/02-terms.md), and triples from [Ch 3](../part1/03-triples.md).

---

## 1. Concept — the four things an ontology defines

An ontology is the domain's "frame." It defines four things.

- **Concepts (classes)** — what kinds of entities exist in the domain. `Person`, `Company`, `Drug`, `Disease`.
- **Relations (properties)** — how entities connect. `employedBy`, `treats`, `cites`.
- **Constraints** — the rules of a relation. `treats` starts at a `Drug` and ends at a `Disease` (domain/range); a person is employed at one place at a time (cardinality).
- **Hierarchy** — inheritance between concepts. `Cardiologist` is a sub-class of `Doctor`, `Doctor` of `Person`.

These four decide "which graphs *make sense*." The knowledge graph (instances) gets filled in only within this frame.

## 2. Why you need this — start from competency questions

The most common failure in ontology design is "trying to model everything in the domain." It's endless, and you spend time on details unrelated to any actual use.

A good starting point is **competency questions** — first write down "the questions this ontology must be able to answer." The ontology only needs to exist to the extent it answers them.

```text
Competency questions (medical domain example)
- What disease does this drug treat?
- Which drugs are contraindicated for this disease?
- Which other drugs interact with this drug?
- Find all drugs containing a given ingredient.
```

In these questions the nouns (drug, disease, ingredient) become **concept candidates**, and the verbs (treats, contraindicated, interacts, contains) become **relation candidates**. The questions drive the design.

## 3. Where to start — start small and grow

The practical process is simple.

1. Write **10–20 competency questions.**
2. Extract the **core concepts** from them (nouns). 5–10 is plenty at first.
3. Define the **relations** the questions follow (verbs). Set direction and both endpoint types (domain/range).
4. Add only the **constraints and hierarchy** you truly need. The rest, later.

A small medical-ontology sketch:

```text
Classes:    Drug, Disease, Ingredient
Relations:  Drug —treats→ Disease
            Drug —contains→ Ingredient
            Drug —interactsWith→ Drug
Hierarchy:  (none — add when needed)
Constraint: treats: domain=Drug, range=Disease
```

Start here and add concepts/relations each time a competency question grows. Not "a perfect ontology first" but "the minimal ontology that answers the questions first."

## 4. Common failure points

**Over-engineering.** Trying to use every OWL feature means you never ship a first graph. Any abstraction the competency questions don't demand is debt.

**Confusing relation vs. property vs. class.** The test: if you need to **traverse and query** the information, it's a relation (edge); if it's a simple value, a property. When a relation accrues its own properties, promote it to a class (e.g., `Prescription` ties together drug, patient, dosage, date).

**Class explosion.** Splitting similar concepts into dozens of classes. First check whether hierarchy (inheritance) can group them, or whether a single property distinguishes them (one `Doctor` with a `specialty` property vs. separate `Cardiologist`/`Neurologist` classes).

## 5. Exercises & next

### Check your understanding

1. Write 10 competency questions for your domain and classify the nouns/verbs as concept/relation candidates.
2. What situation decides whether "prescription" should be a relation vs. promoted to a class?

### Next

Most organizations already have a normalized DB schema. An ontology is hiding in it — on to pulling it out → [Ch 5](05-from-schema.md).

---

## Sources

- Noy & McGuinness (2001). *Ontology Development 101.* Stanford KSL
- Grüninger & Fox (1995). *Methodology for the Design and Evaluation of Ontologies* (competency questions)
- Allemang & Hendler (2011). *Semantic Web for the Working Ontologist.*
