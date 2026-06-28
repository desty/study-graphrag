# Ch 5. DB 스키마에서 온톨로지 추출하기

!!! abstract "이 챕터에서 배우는 것"
    - 정규화된 관계형 스키마에 **이미 온톨로지가 숨어 있다**는 관점
    - 테이블 · 컬럼 · 외래키 · 조인 테이블을 개념 · 속성 · 관계로 매핑하는 규칙
    - 처음부터 짜지 않고 추출해서 시작하면 싸고 유지보수가 쉬운 이유
    - 기계적 1:1 변환이 만드는 함정과, LLM으로 보조하는 법

!!! quote "전제"
    [Ch 4](04-ontology-design.md)의 개념·관계·제약. 관계형 DB의 기본(테이블·외래키)을 안다고 가정한다.

---

## 1. 개념 — 스키마는 이미 절반의 온톨로지다

새 온톨로지를 백지에서 짜는 건 비싸다. 그런데 대부분의 조직엔 이미 잘 정규화된 관계형 DB가 있고, **정규화 과정 자체가 도메인을 개념과 관계로 쪼갠 작업**이었다. 즉 스키마 안에 온톨로지의 골격이 이미 들어 있다.

매핑은 놀랍도록 직관적이다.

| 관계형 스키마 | → | 온톨로지 / 그래프 |
|---|---|---|
| 테이블 | → | 개념(클래스) / 노드 라벨 |
| 행(row) | → | 인스턴스(노드) |
| 일반 컬럼 | → | 속성(literal) |
| 외래키(FK) | → | 관계(엣지) |
| 조인 테이블(M:N) | → | 관계(엣지), 때로는 노드 |

`customers`·`orders`·`products` 테이블과 그 사이 FK는 그대로 `Customer —places→ Order —contains→ Product` 그래프가 된다.

## 2. 왜 필요한가 — 싸고, 살아 있고, 유지보수가 쉽다

스키마에서 추출하면 세 가지를 얻는다. **싸다** — 도메인을 처음부터 다시 분석하지 않는다. **살아 있다** — 운영 DB가 진실의 원천이니, 데이터가 이미 거기 있다. **유지보수가 쉽다** — 스키마가 바뀌면 매핑만 갱신하면 된다.

그래서 실전 GraphRAG의 흔한 출발점은 "비정형 문서부터"가 아니라 "이미 있는 정형 DB부터"다. 정형 데이터로 그래프의 뼈대를 세우고, 비정형 텍스트(→ Part 3, LLM 추출)로 살을 붙인다.

## 3. 어디서부터 — 매핑의 결정들

기계적으로 옮기기 전에 몇 가지를 판단해야 한다.

**모든 테이블이 개념은 아니다.** M:N 조인 테이블(`order_items`)은 보통 개념이 아니라 **관계**다. 하지만 거기에 속성(수량·단가)이 붙어 있으면, 관계로 두되 속성을 엣지에 매달거나(LPG), 별도 노드(`OrderItem`)로 승격한다 — [Ch 3](../part1/03-triples.md)의 그 결정이다.

**FK 방향과 관계 이름.** FK는 방향만 있고 의미가 없다. `orders.customer_id`는 `Order —placedBy→ Customer`인가 `Customer —places→ Order`인가? 컴피턴시 질문이 따라가는 방향으로 이름을 짓는다.

**무엇을 버릴까.** 모든 컬럼·테이블을 옮길 필요는 없다. 감사 로그, 캐시 테이블, 질의에 쓸 일 없는 컬럼은 그래프에서 뺀다. [컨텍스트와 같은 원리](https://desty.github.io/context-engineering-guide/) — 질문에 필요한 것만.

```text
매핑 예시 (전자상거래)
TABLE customers(id, name, email, tier)
  → (:Customer {name, email, tier})
TABLE orders(id, customer_id→FK, total, created_at)
  → (:Order {total, created_at})
  → (:Customer)-[:PLACES]->(:Order)
TABLE order_items(order_id→FK, product_id→FK, qty, price)
  → (:Order)-[:CONTAINS {qty, price}]->(:Product)   # 조인테이블 → 속성 있는 관계
```

## 4. LLM으로 보조하기

스키마가 크면 매핑을 LLM에 맡길 수 있다. DDL(`CREATE TABLE ...`)을 주고 "테이블→클래스, FK→관계로 매핑하고, 조인 테이블과 관계 이름을 제안하라"고 시킨다. 단, LLM의 제안은 **초안**이다 — FK 방향의 의미, 어떤 조인 테이블을 노드로 올릴지는 도메인 지식으로 검수해야 한다. 다음 챕터의 비정형 추출과 같은 원칙이다: 자동 추출 + 사람 검수.

## 5. 자주 깨지는 포인트

**1:1 기계 변환의 함정.** 모든 테이블을 노드로, 모든 FK를 엣지로 그대로 옮기면 그래프가 운영 DB의 복사본이 될 뿐, 질의하기 좋은 그래프가 안 된다. 조인 테이블은 관계로 접고, 안 쓰는 건 버린다.

**의미 없는 관계 이름.** `HAS_CUSTOMER_ID` 같은 FK 직역은 그래프를 읽기 어렵게 만든다. 도메인 동사로(`PLACES`, `BELONGS_TO`).

## 6. 연습 & 다음 챕터

### 확인 문제

1. 익숙한 DB의 테이블 5개와 FK를 골라 그래프 스키마로 매핑하라. 어떤 게 노드고 어떤 게 관계인가?
2. 속성이 붙은 M:N 조인 테이블을 "속성 있는 관계"와 "별도 노드" 중 무엇으로 둘지, 어떤 질문이 그 결정을 가르는가?

### 다음

Part 3로 넘어가 실제로 그래프를 짓는다. 먼저 Neo4j와 Cypher로 노드·관계를 만들고 질의하는 법부터 → [Ch 6](../part3/06-neo4j-cypher.md).

---

## 원전

- Sequeda & Lassila (2021). *Designing and Building Enterprise Knowledge Graphs.* Morgan & Claypool
- W3C. *R2RML: RDB to RDF Mapping Language.*
- Hogan et al. (2021). *Knowledge Graphs.* ACM Computing Surveys (§schema)
