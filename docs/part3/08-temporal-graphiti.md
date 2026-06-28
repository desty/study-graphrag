# Ch 8. 시간 인식 그래프와 Graphiti

!!! abstract "이 챕터에서 배우는 것"
    - 지식은 변한다 — "사실에 시점이 없는" 그래프의 한계
    - **bi-temporal** 모델 — 사실이 유효한 시점 vs 우리가 안 시점
    - 관계에 시간을 매다는 LPG 패턴, 그리고 Graphiti가 자동화하는 것
    - 에이전트 메모리로서의 시간 인식 그래프

!!! quote "전제"
    [Ch 6](06-neo4j-cypher.md)·[Ch 7](07-llm-extraction.md). 관계에 속성을 붙이는 LPG 모델을 안다고 가정한다.

---

## 1. 개념 — 사실에는 시점이 있다

지금까지의 그래프는 시간을 모른다. "이수진 —employedBy→ 애플"은 *언제부터* 참인가? 그가 퇴사하면 이 엣지는 틀린 게 되는가, 아니면 과거엔 참이었던 사실로 남는가?

지식은 변한다. CEO가 바뀌고, 가격이 오르고, 관계가 끝난다. 시점 없는 그래프는 "지금"만 담을 수 있고, 갱신은 곧 과거의 삭제다. 하지만 많은 질문이 시간을 따진다 — "2023년 당시 이 회사 CEO는?", "이 정책은 언제부터 적용됐나?".

## 2. 왜 필요한가 — bi-temporal

시간을 제대로 다루려면 두 종류의 시점을 구분해야 한다.

- **유효 시간(valid time)** — 그 사실이 현실에서 참인 기간. "이수진은 2019~2024년 애플에 고용".
- **기록 시간(transaction time)** — 우리 시스템이 그 사실을 안/기록한 시점. "이 고용 사실을 2026년 1월에 그래프에 넣음".

둘을 함께 다루는 걸 **bi-temporal**이라 한다. 유효 시간이 있으면 "그때의 진실"을 질의할 수 있고, 기록 시간이 있으면 "우리가 언제부터 그렇게 알았나"를 추적해 정정·감사가 가능하다.

## 3. 어떻게 — 관계에 시간을 매단다

LPG에선 자연스럽다. 관계 속성에 시점을 넣는다([Ch 3](../part1/03-triples.md)에서 본 LPG의 강점이 여기서 빛난다).

```cypher
// 끝난 사실을 지우지 않고, 기간으로 표시
MATCH (c:Customer {name:'이수진'})-[r:EMPLOYED]->(co:Company {name:'애플'})
SET r.valid_from = date('2019-01-01'),
    r.valid_to   = date('2024-06-30')   // null이면 '현재까지'

// "2023년 시점의 고용 관계"
MATCH (p:Person)-[r:EMPLOYED]->(co:Company)
WHERE r.valid_from <= date('2023-06-01')
  AND (r.valid_to IS NULL OR r.valid_to >= date('2023-06-01'))
RETURN p.name, co.name
```

새 사실이 들어오면 기존 엣지를 지우는 대신 `valid_to`를 닫고 새 엣지를 연다. 그래프가 **역사를 누적**한다.

## 4. Graphiti — 시간 인식 그래프 메모리

이 패턴을 손으로 다 짜는 건 번거롭다. **[Graphiti](https://github.com/getzep/graphiti)**(Zep)는 시간 인식 지식 그래프를 자동으로 관리하는 오픈소스다. 새 정보가 들어오면 기존 사실과 모순되는지 보고, 모순되면 옛 엣지를 시점으로 닫고 새 엣지를 연다 — bi-temporal을 알아서 처리한다.

이게 특히 쓸모 있는 자리가 **에이전트 메모리**다. [컨텍스트 엔지니어링에서](https://desty.github.io/context-engineering-guide/) 윈도우 밖의 메모리 이야기를 했는데, 시간 인식 그래프는 그 메모리의 강력한 형태다. "사용자가 예전엔 X를 선호했지만 지금은 Y"를 그래프가 시점과 함께 기억하니, 에이전트가 최신 상태와 변화 이력을 둘 다 질의할 수 있다.

```python title="graphiti_sketch.py"
# 개념 스케치 (실제 API는 graphiti 문서 참고)
# pip install graphiti-core
from graphiti_core import Graphiti

g = Graphiti(neo4j_uri="bolt://localhost:7687", user="neo4j", password="testpassword")

# 에피소드(원문 조각)를 넣으면 엔티티·관계를 추출하고 시점과 함께 적재
await g.add_episode(name="profile", episode_body="이수진은 이제 마우스보다 트랙패드를 선호한다.")
# 기존에 '키보드 선호' 같은 모순 사실이 있으면 그쪽 valid_to를 닫고 새 사실을 연다
```

## 5. 자주 깨지는 포인트

**갱신 = 삭제로 처리.** 새 값이 오면 옛 엣지를 지우면, 과거 질의와 감사가 불가능해진다. 닫되 지우지 않는다(`valid_to` 설정).

**모든 것에 시간을 단다.** bi-temporal은 비용이다(엣지·질의 복잡도↑). 정말 변하고, 과거를 질의할 사실에만 적용한다. 안 변하는 사실(생년월일)은 그냥 속성으로.

**시간대·날짜 타입 혼선.** 시점을 문자열로 넣으면 비교가 깨진다. `date()`/`datetime()` 타입으로 통일한다.

## 6. 연습 & 다음 챕터

### 실습 과제

1. [Ch 6](06-neo4j-cypher.md)의 고용 관계에 `valid_from`/`valid_to`를 넣고, "특정 날짜 시점의 재직자"를 조회하라.
2. 모순되는 새 사실을 넣을 때 "옛 엣지를 닫고 새 엣지를 여는" 적재 함수를 작성하라.
3. Graphiti README의 예제를 돌려, 모순 사실 주입 시 그래프가 어떻게 바뀌는지 관찰하라.

### 다음

이제 그래프가 준비됐다. Part 4에서 이 그래프를 RAG에 붙인다 — GraphRAG의 아키텍처부터 다룬다(곧 공개).

---

## 원전

- Zep. *Graphiti: Temporal Knowledge Graphs for Agents.* (github.com/getzep/graphiti)
- Snodgrass (1999). *Developing Time-Oriented Database Applications in SQL.* (bi-temporal)
