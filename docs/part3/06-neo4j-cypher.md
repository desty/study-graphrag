# Ch 6. Neo4j와 Cypher 입문

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part3/ch6_neo4j_cypher.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "이 챕터에서 배우는 것"
    - 그래프 DB **Neo4j**를 도커/무료 티어로 띄우고 파이썬에서 붙기
    - **Cypher** 기본 — 노드·관계 만들기(`CREATE`/`MERGE`), 조회(`MATCH`), 멀티홉 순회
    - [Ch 5](../part2/05-from-schema.md)의 전자상거래 매핑을 실제 그래프로 적재
    - 벡터 RAG가 못 풀던 멀티홉 질문을 Cypher 한 줄로 푸는 경험

!!! quote "전제"
    [Part 2](../part2/04-ontology-design.md)까지. 파이썬 기본과 도커(또는 Neo4j Aura 무료 계정)면 된다.

---

## 1. 개념 — Cypher는 "그림을 그리는 질의어"

Cypher의 핵심 직관은 하나다 — **ASCII로 그래프를 그린다.** 노드는 괄호 `()`, 관계는 화살표 `-[]->`. 머릿속 그래프를 그대로 적으면 그게 질의가 된다.

```cypher
(고객)-[:PLACES]->(주문)-[:CONTAINS]->(상품)
```

이 한 줄이 "고객이 주문을 하고, 주문이 상품을 담는다"는 패턴이다. 조회는 이 패턴에 **맞는 부분을 그래프에서 찾아 달라**는 것이다.

## 2. 환경 — Neo4j 띄우기

가장 빠른 길은 도커다.

```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/testpassword \
  neo4j:5
```

`http://localhost:7474`로 브라우저 콘솔에 접속할 수 있다. 설치 없이 가려면 [Neo4j Aura](https://neo4j.com/cloud/aura/)의 무료 인스턴스를 써도 된다. 파이썬에서 붙는 코드:

```python title="connect.py"
# pip install neo4j
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "testpassword"))

def run(cypher, **params):
    with driver.session() as s:
        return list(s.run(cypher, **params))

print(run("RETURN 'connected' AS status")[0]["status"])
```

## 3. 만들기 — CREATE와 MERGE

노드와 관계를 만든다. [Ch 5](../part2/05-from-schema.md)의 전자상거래 매핑을 그대로 적재해 보자.

```python title="load.py"
run("""
MERGE (c:Customer {id: 1, name: '이수진', tier: 'gold'})
MERGE (o:Order {id: 101, total: 52000})
MERGE (p1:Product {id: 'A', name: '키보드'})
MERGE (p2:Product {id: 'B', name: '마우스'})
MERGE (c)-[:PLACES]->(o)
MERGE (o)-[:CONTAINS {qty: 1, price: 40000}]->(p1)
MERGE (o)-[:CONTAINS {qty: 1, price: 12000}]->(p2)
""")
```

!!! tip "CREATE 말고 MERGE를 기본으로"
    `CREATE`는 매번 새로 만들어 중복을 낳는다. `MERGE`는 "있으면 찾고 없으면 만든다" — 적재를 여러 번 돌려도 안전하다(멱등). 실전 적재는 거의 `MERGE`다.

## 4. 조회 — MATCH와 멀티홉

이제 [Ch 1](../part1/01-why-graph.md)에서 벡터 RAG가 못 푼다고 한 멀티홉 질문을 풀어 보자. "이수진이 산 상품은?" — 고객에서 주문을 거쳐 상품까지 두 홉이다.

```python title="query.py"
rows = run("""
MATCH (c:Customer {name: '이수진'})-[:PLACES]->(:Order)-[:CONTAINS]->(p:Product)
RETURN p.name AS product
""")
print([r["product"] for r in rows])   # ['키보드', '마우스']
```

벡터 검색이라면 "이수진"과 "키보드"가 한 문서에 같이 적혀 있길 기대해야 했다. 그래프에선 **관계를 따라가면** 끝이다. 홉이 더 늘어도 패턴만 길어질 뿐이다.

```cypher
// "같은 상품을 산 다른 고객" — 3홉 추천의 씨앗
MATCH (c:Customer {name: '이수진'})-[:PLACES]->(:Order)-[:CONTAINS]->(p:Product)
      <-[:CONTAINS]-(:Order)<-[:PLACES]-(other:Customer)
WHERE other <> c
RETURN DISTINCT other.name
```

## 5. 자주 깨지는 포인트

**MERGE를 통째로 쓰다 중복 매칭.** `MERGE (a)-[:R]->(b)`에서 `a`나 `b`가 아직 없으면 통째로 새로 만든다. 노드를 먼저 각각 `MERGE`하고, 그 다음 관계를 `MERGE`하는 게 안전하다.

**인덱스 없이 대량 매칭.** `MATCH (c:Customer {id: ...})`가 느리면 `CREATE INDEX`로 라벨+속성에 인덱스를 건다. 적재 전에 핵심 조회 키에 인덱스부터.

**방향에 집착하거나 무시하거나.** Cypher의 `-[:R]->`는 방향을 지킨다. 방향 무관 조회는 `-[:R]-`(화살표 없이). 질문이 방향을 따지는지 먼저 정하라.

## 6. 연습 & 다음 챕터

### 실습 과제

1. [Ch 4](../part2/04-ontology-design.md)의 의료 온톨로지(Drug/Disease/Ingredient)를 노드 5개·관계 5개로 적재하라.
2. "두통을 치료하는 약 중 아세틸살리실산을 포함한 것"을 Cypher 한 질의로 찾아라.
3. 위 추천 쿼리를 변형해 "이수진과 취향이 겹치는 고객 top 3"를 공통 상품 수로 정렬해 보라.

### 다음

지금까지는 정형 데이터를 손으로 넣었다. 다음은 **비정형 텍스트에서 LLM으로 엔티티·관계를 뽑아** 그래프를 자동으로 채운다 → [Ch 7](07-llm-extraction.md).

---

## 원전

- Neo4j. *Cypher Manual* (v5).
- Robinson, Webber & Eifrem (2015). *Graph Databases* (2nd ed.), O'Reilly
