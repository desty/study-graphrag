# Ch 11. 벡터 RAG vs GraphRAG 직접 비교

<a class="colab-badge" href="https://colab.research.google.com/github/desty/study-graphrag/blob/main/notebooks/part4/ch11_vector_vs_graph.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab">
</a>

!!! abstract "이 챕터에서 배우는 것"
    - 같은 코퍼스 · 같은 질문 세트로 **벡터 RAG와 GraphRAG를 나란히** 돌리기
    - 질문 유형(단순 사실 · 멀티홉 · 전역)별로 어디서 갈리는지 눈으로 확인
    - 정확도만이 아니라 **비용·지연**까지 함께 보는 균형 잡힌 결론
    - "둘 중 하나"가 아니라 "질문에 맞게 라우팅"이라는 실전 답

!!! quote "전제"
    [Ch 9](09-what-is-graphrag.md)·[Ch 10](10-search-strategies.md). 벡터 RAG 베이스라인을 만들 수 있다고 가정한다.

---

## 1. 개념 — 말 말고 숫자로

지금까지 "멀티홉·전역은 GraphRAG가 강하다"고 말로 주장했다. 이 챕터는 그걸 **직접 돌려 확인**한다. 공정한 비교의 핵심은 셋이다: 같은 코퍼스, 같은 질문 세트, 같은 생성 모델. 다른 건 검색 방식뿐이어야 한다.

## 2. 실험 설계 — 질문을 세 갈래로

질문 세트를 [Ch 1](../part1/01-why-graph.md)의 분류대로 셋으로 나눠 던진다.

```python title="question_set.py"
QUESTIONS = {
    "factual":   ["아스피린의 주성분은?",            # 단순 사실 — 한 청크에 답
                  "이 회사의 설립 연도는?"],
    "multihop":  ["아스피린과 같은 성분을 쓰는 다른 약은?",  # 관계 추적
                  "A가 인용한 논문들이 쓴 데이터셋은?"],
    "global":    ["이 문서 묶음의 핵심 주제 5개는?",        # 전역 종합
                  "전체에서 가장 많이 언급된 위험은?"],
}
```

각 질문을 두 시스템에 던진다.

```python title="compare.py"
def vector_rag(q):
    chunks = vector_store.search(q, k=5)        # 임베딩 top-k
    return llm_answer(q, context=chunks)

def graph_rag(q, mode):                          # mode: local | global | hybrid
    ctx = graph_search(q, mode)                  # Ch 10의 전략
    return llm_answer(q, context=ctx)

for cat, qs in QUESTIONS.items():
    for q in qs:
        v = vector_rag(q)
        g = graph_rag(q, mode=route(cat))        # factual→hybrid, multihop→local, global→global
        record(cat, q, v, g)
```

## 3. 전형적인 결과 — 어디서 갈리나

실제로 돌리면 대체로 이런 그림이 나온다(코퍼스·구현마다 다르지만 경향은 일관적이다).

| 질문 유형 | 벡터 RAG | GraphRAG | 비고 |
|---|---|---|---|
| 단순 사실 | ◎ 충분 | ◎ 비등 | 한 청크에 답이 있으면 벡터로 족하다 |
| 멀티홉 | △ 자주 누락 | ◎ 강함 | 관계를 따라가야 하는 질문에서 격차가 벌어짐 |
| 전역 종합 | ✗ 구조적 한계 | ◎ 강함 | 커뮤니티 요약이 결정적 |
| **인덱싱 비용** | ◎ 낮음(임베딩만) | ✗ 높음(LLM 추출+요약) | [Ch 9](09-what-is-graphrag.md)에서 본 그 비용 |
| **쿼리 지연** | ◎ 빠름 | △ 글로벌은 느림 | map-reduce 호출 누적 |

요점은 분명하다. **단순 사실 질문에선 GraphRAG가 더 나을 게 없다** — 오히려 비싸다. GraphRAG의 값은 멀티홉과 전역에서 나온다. 그러니 "전부 GraphRAG로"는 틀린 결론이다.

## 4. 그래서 답은 라우팅이다

[Ch 10](10-search-strategies.md)의 라우팅이 여기서 결론이 된다. 질문을 분류해 — 단순 사실은 싼 벡터 RAG로, 멀티홉·전역은 GraphRAG로 — 보내는 하이브리드 시스템이 정확도와 비용을 동시에 잡는다.

```python title="router.py"
def answer(q):
    kind = classify(q)                  # LLM 또는 규칙 기반 분류
    if kind == "factual":
        return vector_rag(q)            # 싸고 충분
    elif kind == "multihop":
        return graph_rag(q, "local")
    else:  # global
        return graph_rag(q, "global")
```

이건 [AI Eval 가이드](https://desty.github.io/ai-eval-guide/)의 "싼 것부터, 비싼 건 필요할 때만" 피라미드와 같은 정신이다 — 검색에도 계층이 있다.

## 5. 자주 깨지는 포인트

**정확도만 보고 비용을 무시한다.** GraphRAG가 "더 정확"해도 모든 질문에 쓰면 비용·지연이 망가진다. 비교표에 반드시 비용·지연 열을 넣어라.

**불공정한 비교.** 벡터 RAG의 k를 너무 작게 잡거나 청킹을 대충 하면 GraphRAG가 부당하게 이긴다. 베이스라인을 제대로 튜닝한 뒤 비교해야 의미가 있다.

**질문 세트가 한쪽에 치우침.** 전역 질문만 모아 놓으면 당연히 GraphRAG가 압승한다. 실제 트래픽의 질문 분포를 반영해야 결론이 현실적이다.

## 6. 연습 & 다음 챕터

### 실습 과제

1. 작은 코퍼스(위키 문서 20개 등)로 두 시스템을 만들고, 세 유형 질문 6개로 비교표를 채워라.
2. 각 답에 정확도뿐 아니라 **토큰 비용·지연**을 기록하라.
3. 라우터를 붙여 "단순 사실은 벡터, 나머지는 그래프"로 보낼 때 전체 비용이 얼마나 주는지 측정하라.

### 다음

Part 5로 넘어가, 이 비교를 체계적인 **평가**로 만든다 — GraphRAG를 무엇으로, 어떻게 측정하나(곧 공개).

---

## 원전

- Microsoft Research (2024). *From Local to Global: A Graph RAG Approach.* arXiv:2404.16130
- Han et al. (2024). *Retrieval-Augmented Generation with Graphs (GraphRAG): A Survey.* arXiv:2501.00309
