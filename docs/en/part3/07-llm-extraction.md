# Ch 7. Building a KG with an LLM — Entity & Relation Extraction

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part3/ch7_llm_extraction.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "What you'll learn"
    - Extracting **entities and relations from unstructured text with an LLM** into triples
    - Using the ontology as a **guardrail** for the extraction prompt (schema-guided extraction)
    - Getting stable results with structured output (JSON schema)
    - Why entity resolution (merging the same thing) and human review are needed

!!! quote "Assumes"
    Loading from [Ch 6](06-neo4j-cypher.md), the ontology from [Ch 4](../part2/04-ontology-design.md). A feel for LLM structured output helps (see the [RAG Guide](https://desty.github.io/rag-guide/en/)).

---

## 1. Concept — text into triples

If [Ch 5](../part2/05-from-schema.md) built the graph's skeleton from a structured DB, the flesh comes from unstructured text — news, reports, wikis, papers. The core task is one thing: **pull (subject — relation — object) triples out of sentences.** LLMs are quite good at this.

```text
Input:  "Aspirin relieves headaches and contains acetylsalicylic acid as an ingredient."
Output: (Aspirin) —treats→ (headache)
        (Aspirin) —contains→ (acetylsalicylic acid)
```

## 2. Why the ontology is a guardrail

Tell an LLM to just "extract all relations" and it invents a different relation name every time — `treats`, `relieves`, `helps_with`, `alleviates`... and the graph loses consistency. **Bake the ontology from [Ch 4](../part2/04-ontology-design.md) into the prompt as the allowed-type list**, and the LLM picks labels only from within it. Constraining the freedom of extraction is the key.

```python title="extract.py"
# pip install anthropic
import anthropic, json
client = anthropic.Anthropic()

ONTOLOGY = {
    "entities": ["Drug", "Disease", "Ingredient"],
    "relations": ["treats", "contains", "interactsWith"],
}

PROMPT = """Extract triples from the text below and answer with JSON only.
Allowed entity types: {entities}
Allowed relation types: {relations}
Rules: do not invent types outside the allowed lists. If unsure, omit.
Format: {{"triples": [{{"subj": "...", "subj_type": "...", "rel": "...", "obj": "...", "obj_type": "..."}}]}}

Text:
{text}"""

def extract(text):
    msg = client.messages.create(
        model="claude-haiku-4-5",  # a small model is plenty for extraction, lower cost
        max_tokens=1024,
        messages=[{"role": "user", "content": PROMPT.format(
            entities=ONTOLOGY["entities"], relations=ONTOLOGY["relations"], text=text)}],
    )
    return json.loads(msg.content[0].text)

out = extract("Aspirin relieves headaches and contains acetylsalicylic acid as an ingredient.")
print(out["triples"])
```

Load the extracted triples directly with the `MERGE` pattern from [Ch 6](06-neo4j-cypher.md).

```python title="load_triples.py"
def load(triples):
    for t in triples:
        run("""
        MERGE (s {name: $subj}) SET s:`%s`
        MERGE (o {name: $obj}) SET o:`%s`
        MERGE (s)-[:`%s`]->(o)
        """ % (t["subj_type"], t["obj_type"], t["rel"]),
        subj=t["subj"], obj=t["obj"])

load(out["triples"])
```

## 3. Entity resolution — merging the same thing

Different texts call the same entity different things — "Aspirin," "aspirin," "an acetylsalicylic-acid preparation." Load them as-is and the same drug splits into three nodes, breaking the graph. You need **entity resolution.**

The practical approach is staged. ① Normalize (lowercase, whitespace, an alias dictionary) to merge easy duplicates, ② gather candidates by embedding similarity, ③ let an LLM or a human decide the ambiguous ones. Don't try to fully automate it — "auto-merge only the certain ones, send the rest to a review queue" is realistic.

## 4. Common failure points

**Relation-type explosion.** Without a guardrail, synonymous relations proliferate. Group them with the ontology, and if a new relation is truly needed, add it to the ontology first.

**Hallucinated triples.** The LLM plausibly invents relations not in the text. Bake "only what's stated in the text; omit if unsure" into the prompt, and for important graphs keep the source sentence as an edge property for traceability.

**Going to production without review.** Automated extraction is a draft. Entity-resolution errors in particular distort the whole graph's connectivity. Same principle as [Ch 5](../part2/05-from-schema.md) — automated extraction + human review.

## 5. Exercises & next

### Hands-on

1. Pick 3 paragraphs from a domain you care about, run ontology-guardrailed extraction, and query the loaded graph with Cypher from [Ch 6](06-neo4j-cypher.md).
2. Compare how many relation types appear when you drop the guardrail.
3. Modify the load code to keep the source sentence as a relation property `source`.

### Next

The graph so far knows no time. We go to time-aware graphs and Graphiti, which handle changes like "it used to be A, now it's B" → [Ch 8](08-temporal-graphiti.md).

---

## Sources

- Microsoft Research (2024). *GraphRAG* (entity/relation extraction pipeline). arXiv:2404.16130
- Anthropic. *Tool use / structured outputs* docs
- Barlaug & Gulla (2021). *Neural Networks for Entity Matching: A Survey.* ACM TKDD
