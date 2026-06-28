# 지식 그래프와 GraphRAG

**온톨로지부터 지식 그래프 구축, GraphRAG, 평가·프로덕션까지 손으로 만들어 보는 코스.** 벡터 RAG가 어디서 막히는지 보고, 트리플로 지식을 모델링하고, Neo4j와 LLM으로 그래프를 짓고, GraphRAG로 검색해 벡터 RAG와 정면 비교한다. 마지막엔 도메인 지식 그래프 + GraphRAG 파이프라인 하나를 끝까지 굴려본다.

## 이 코스가 다루는 것 / 다루지 않는 것

<div class="infocards">
  <div class="card">
    <h4>다룬다</h4>
    <p>온톨로지 설계 · RDF/LPG · 트리플 · Neo4j/Cypher · LLM 엔티티·관계 추출 · GraphRAG(로컬/글로벌/커뮤니티) · 벡터 vs 그래프 비교 · 평가 · 프로덕션</p>
  </div>
  <div class="card">
    <h4>가볍게만 언급</h4>
    <p>SPARQL · OWL 추론 · 그래프 임베딩(GNN) · Graphiti 같은 시간 인식 그래프</p>
  </div>
  <div class="card">
    <h4>다루지 않는다</h4>
    <p>그래프 DB 내부 구현 · 대규모 분산 그래프 처리 · 시맨틱 웹 표준 전반</p>
  </div>
  <div class="card">
    <h4>전제</h4>
    <p>Python 입문 · LLM/RAG 기본 감(임베딩·검색) · Colab 또는 로컬 도커</p>
  </div>
</div>

## 왜 이 코스인가

벡터 RAG는 "비슷한 문서"는 잘 찾지만, "A와 B가 어떻게 연결되는가"는 못 찾는다. 여러 사실을 가로질러 종합해야 하는 질문, 관계를 따라가야 하는 질문에서 벡터 검색은 무너진다. 지식 그래프는 그 빈자리를 메우고, GraphRAG는 그래프를 RAG에 붙여 "연결"을 검색 가능하게 만든다.

이 코스는 개념만 훑지 않는다. 실제로 그래프를 짓고, GraphRAG를 돌리고, 같은 질문을 벡터 RAG와 GraphRAG에 던져 **어디서 갈리는지**를 눈으로 본다.

## 어디로 갈까

- [학습 시스템](about/system.md) — 챕터는 어떻게 구성되는가
- [학습 내용](about/curriculum.md) — 전체 14 챕터 + 캡스톤
- [Part 1 시작하기](part1/01-why-graph.md) — 벡터 RAG는 어디서 막히나

## 관련 자료

- 가이드: [온톨로지 & 지식 그래프](https://desty.github.io/ontology-knowledge-graph-guide/) — 개념을 한 장으로
- 가이드: [RAG 완전 가이드](https://desty.github.io/rag-guide/) · [AI Eval](https://desty.github.io/ai-eval-guide/)
- 블로그: [desty.github.io/blog](https://desty.github.io/blog/)
