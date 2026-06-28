# Ch 7. LLM으로 KG 구축 — 엔티티·관계 추출

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part3/ch7_llm_extraction.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "이 챕터에서 배우는 것"
    - 비정형 텍스트에서 **엔티티와 관계를 LLM으로 추출**해 트리플로 만드는 법
    - 온톨로지를 추출 프롬프트의 **가드레일**로 쓰는 법(스키마 안내 추출)
    - 구조화 출력(JSON 스키마)으로 결과를 안정적으로 받기
    - 엔티티 해소(같은 것 합치기)와 사람 검수가 필요한 이유

!!! quote "전제"
    [Ch 6](06-neo4j-cypher.md)의 적재, [Ch 4](../part2/04-ontology-design.md)의 온톨로지. LLM 구조화 출력 감이 있으면 좋다([RAG 가이드](https://desty.github.io/rag-guide/) 참고).

---

## 1. 개념 — 텍스트를 트리플로

[Ch 5](../part2/05-from-schema.md)에서 정형 DB로 그래프 뼈대를 세웠다면, 살은 비정형 텍스트에서 온다 — 뉴스, 보고서, 위키, 논문. 핵심 작업은 하나다: **문장에서 (주어 — 관계 — 목적어) 트리플을 뽑는 것.** LLM이 이걸 꽤 잘한다.

```text
입력:  "아스피린은 두통을 완화하며, 성분으로 아세틸살리실산을 포함한다."
출력:  (아스피린) —treats→ (두통)
       (아스피린) —contains→ (아세틸살리실산)
```

## 2. 왜 온톨로지가 가드레일인가

LLM에게 그냥 "관계를 다 뽑아" 하면, 매번 다른 관계 이름을 만든다 — `treats`, `relieves`, `helps_with`, `완화한다`... 그래프가 일관성을 잃는다. [Ch 4](../part2/04-ontology-design.md)의 온톨로지를 **허용 타입 목록으로 프롬프트에 박으면**, LLM이 그 안에서만 라벨을 고른다. 추출의 자유도를 묶는 게 핵심이다.

```python title="extract.py"
# pip install anthropic
import anthropic, json
client = anthropic.Anthropic()

ONTOLOGY = {
    "entities": ["Drug", "Disease", "Ingredient"],
    "relations": ["treats", "contains", "interactsWith"],
}

PROMPT = """다음 텍스트에서 트리플을 추출해 JSON으로만 답하라.
허용 엔티티 타입: {entities}
허용 관계 타입: {relations}
규칙: 허용 목록에 없는 타입은 만들지 말 것. 불확실하면 빼라.
형식: {{"triples": [{{"subj": "...", "subj_type": "...", "rel": "...", "obj": "...", "obj_type": "..."}}]}}

텍스트:
{text}"""

def extract(text):
    msg = client.messages.create(
        model="claude-haiku-4-5",  # 추출은 작은 모델로 충분, 비용↓
        max_tokens=1024,
        messages=[{"role": "user", "content": PROMPT.format(
            entities=ONTOLOGY["entities"], relations=ONTOLOGY["relations"], text=text)}],
    )
    return json.loads(msg.content[0].text)

out = extract("아스피린은 두통을 완화하며, 성분으로 아세틸살리실산을 포함한다.")
print(out["triples"])
```

추출한 트리플은 [Ch 6](06-neo4j-cypher.md)의 `MERGE` 패턴으로 그대로 적재한다.

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

## 3. 엔티티 해소 — 같은 것을 합치기

텍스트마다 같은 개체를 다르게 부른다 — "아스피린", "Aspirin", "아세틸살리실산 제제". 그대로 적재하면 같은 약이 노드 셋으로 쪼개져 그래프가 끊긴다. **엔티티 해소(entity resolution)**가 필요하다.

실전 접근은 단계적이다. ① 정규화(소문자·공백·별칭 사전)로 쉬운 중복을 합치고, ② 임베딩 유사도로 후보를 모으고, ③ 애매한 건 LLM이나 사람이 판정한다. 완벽히 자동화하려 들지 말고, "확실한 것만 자동 병합, 나머지는 검수 큐로"가 현실적이다.

## 4. 자주 깨지는 포인트

**관계 타입 폭발.** 가드레일 없이 추출하면 동의 관계가 난립한다. 온톨로지로 묶고, 새 관계가 정말 필요하면 온톨로지에 먼저 추가한다.

**환각 트리플.** LLM이 텍스트에 없는 관계를 그럴듯하게 지어낸다. "텍스트에 명시된 것만, 불확실하면 빼라"를 프롬프트에 박고, 중요한 그래프는 출처 문장을 엣지 속성으로 남겨 추적 가능하게 한다.

**검수 없이 프로덕션 직행.** 자동 추출은 초안이다. 특히 엔티티 해소 오류는 그래프 전체의 연결성을 왜곡한다. [Ch 5](../part2/05-from-schema.md)와 같은 원칙 — 자동 추출 + 사람 검수.

## 5. 연습 & 다음 챕터

### 실습 과제

1. 관심 도메인 문단 3개를 골라 온톨로지 가드레일 추출을 돌리고, 적재된 그래프를 [Ch 6](06-neo4j-cypher.md)의 Cypher로 조회하라.
2. 가드레일을 뺐을 때 관계 타입이 얼마나 늘어나는지 비교하라.
3. 출처 문장을 관계 속성 `source`로 남기도록 적재 코드를 고쳐라.

### 다음

지금까지의 그래프는 시간을 모른다. "예전엔 A였는데 지금은 B" 같은 변화를 다루는 시간 인식 그래프와 Graphiti로 간다 → [Ch 8](08-temporal-graphiti.md).

---

## 원전

- Microsoft Research (2024). *GraphRAG* (entity/relation extraction pipeline). arXiv:2404.16130
- Anthropic. *Tool use / structured outputs* docs
- Barlaug & Gulla (2021). *Neural Networks for Entity Matching: A Survey.* ACM TKDD
