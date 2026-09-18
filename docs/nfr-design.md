# NFR 설계

# **목표**

- 사용자가 입력한 시스템 디자인 주제에서 중요한 비기능적 요구사항을 조사한다.
- 실제 기술 문서를 근거로 NFR Rubric 생성한 후 Harness 검증을 거쳐 Markdown으로 출력한다.

## 전체 흐름

```
TUI 입력
  ↓
1. Research - 어떤 데이터를 바탕으로 해당 주제에서 어떤 NFR이 중요한지 정의
  ↓
2. Crawling - NFR 판단할 근거 수집
  ↓
3. Export - NFR 최종 확정 및 평가 기준 생성
  ↓
4. Make MD - 생성된 NFR이 올바른지 검증하고 문서로 변환
  ↓
nfr_rubric.md
```

# Research

**목표**

- 해당 주제에서 어떤 NFR이 중요한지 탐색한다.
- 이를 검증하기 위해 어떤 기술 자료를 참고할지 결정한다.

**필요성**

- 주제에 따라 요구되는 NFR에는 차이가 있다.
- Rubric의 신뢰도를 위해, 신뢰할 수 있는 자료 기반으로 NFR 정의해야 한다.

## NFR Catalog

**목표**

- 시스템 디자인 면접에서 반복적으로 중요하게 다뤄지는 핵심 NFR을 사전에 정의해 탐색 범위를 좁힌다.
- 주제 특성상 Catalog에 없는 NFR이 더 중요할 수 있으므로, 필요한 경우 검색을 통해 추가 후보를 발견하고 확정한다.

**필요성**

- 주제에 따라 검색 결과에서 NFR이 명확히 드러나지 않을 수 있다. 이 경우 사전에 정의된 핵심 NFR을 활용해 관련 자료를 추가로 탐색할 수 있다.
- 반대로 검색을 통해 너무 많은 NFR이 발견될 수도 있다. 주제마다 서로 다른 NFR을 모두 Rubric에 반영하면 평가 항목이 과도하게 분산되고 문제별 평가 기준의 일관성이 낮아질 수 있다.
- 따라서 NFR Catalog는 검색을 대체하는 정답 목록이 아니라, 검색이 부족할 때 탐색을 보완하고 검색 결과가 과도할 때 평가 범위를 제한하는 기준으로 사용한다.

**구조**

```
CORE NFR CATALOG

1. PERFORMANCE
   - latency
   - throughput

2. SCALABILITY
   - scalability

3. RELIABILITY
   - availability
   - fault_tolerance

4. DATA_GUARANTEES
   - consistency
```

**선정 기준**

- 실제 시스템 디자인 문제 30종을 분석해 반복적으로 등장하는지 파악한다.
- 여러 도메인에 공통적으로 적용되는지 파악한다.
- 해당 요구사항에 따라 아키텍처 선택이 실제로 달라지는지 파악한다.

| NFR | 등장 문제 수 | 비율 |
| --- | --- | --- |
| **Scalability / Scale** | 29 / 30 | 약 97% |
| **Latency / Timeliness** | 27 / 30 | 약 90% |
| **Consistency** | 22 / 30 | 약 73% |
| **Reliability / Fault Tolerance / Recovery** | 21 / 30 | 약 70% |
| **Availability** | 15 / 30 | 약 50% |
| **Throughput** | **12 / 30** | **약 40%** |
| **Freshness** | 8 / 30 | 약 27% |
| **Durability** | 5 / 30 | 약 17% |

### NFR Catalog Entry

**필요성**

- LLM이 각 NFR의 의미와 적용 맥락을 이해해, 주제에 맞는 NFR을 고르고 검색 방향을 잡도록 한다.

```yaml
name: latency

category: performance

meaning:
  요청부터 응답까지 걸리는 시간

important_when:
  - 사용자가 응답을 기다리는 기능
  - 검색, 조회, 피드 같은 interactive request

examples:
  - p95 < 500ms
  - p99 < 2s

common_tradeoffs:
  - throughput
  - consistency
  - cost
```

## Source Policy

**목표**

- NFR을 정의할 때 신뢰할 수 있는 기술 자료를 우선적으로 활용하도록 검색 기준을 정한다.

**필요성**

- 검색 결과에는 공식 문서부터 개인 블로그까지 다양한 품질의 자료가 함께 노출된다.
- 출처 기준 없이 자료를 사용하면 부정확하거나 근거가 약한 정보가 NFR과 Rubric에 반영될 수 있다.

```
1순위
- 공식 Engineering / Technical Blog
- 공식 Technical Documentation

2순위
- 논문
- 기술 Conference 발표

3순위
- 신뢰할 수 있는 제3자 기술 블로그
- 시스템 디자인 면접 자료
```

## Research Planner

**목표**

- 주제를 이해하고 중요한 NFR 후보를 선정한다.
- NFR 후보를 검증할 검색 Query를 생성한다.
- Source Policy를 반영해 신뢰도 높은 자료를 우선 탐색한다.

**필요성**

- NFR을 바로 확정하면 LLM의 기존 지식에 의존할 수 있다.
- 먼저 탐색 범위를 정해 필요한 자료만 검색하도록 한다.

### 제약

```yaml
llm1_policy:
  max_queries: 6
  # NFR별 검색 Query 1개 + 주제 전체 이해용 Query 1~2개

  nfr_candidate_count:
    min: 3
    max: 5

  output_format:
    type: json_schema
    strict: true

  # max_output_tokens, temperature는
  # 구현 후 실험을 통해 결정
```

### 입력 프롬프트

```
LLM1 Prompt Spec

역할
- Research Planner

입력
- target_level
- domain
- subject
- NFR Catalog
- Source Policy
- max_queries

해야 할 일
- 주제 이해
- 중요한 NFR 후보 선정 (3~5개)
- 선정 이유 작성
- 검증용 Search Query 생성

제약
- 검색 전 구체적 수치 확정 금지
- 확인되지 않은 사실 단정 금지
- 모든 NFR을 기계적으로 선택하지 않음

출력
- ResearchPlan
```

### 출력 스키마

```yaml
ResearchPlan:
  topic_summary: string

  nfr_candidates:
    - kind: string
      reason: string

  search_queries:
    - query: string
      purpose: string
      related_nfrs:
        - string
```

## Search API

```
Search API

입력
- ResearchPlan.search_queries

처리
- 각 Query 실제 검색

출력
- query_id
- url
- title
- snippet
- rank

정책
- results_per_query
	- Query 하나당 가져올 결과 수
	- 초기값만 두고 이후 테스트로 조정

- failure_policy
	- 특정 Query 검색 실패 시 해당 Query만 실패 처리
	- 나머지 Query는 계속 진행
```

### 출력 스키마

```yaml
SearchResult:
  query_id: string
  url: string
  title: string
  snippet: string
  rank: integer
```

# Crawling

```
SearchResult[]
  ↓
Candidate Filtering
- 중복 제거
- 검색 rank 우선
- max_docs 제한 (10개)
  ↓
SelectedURL[]
  ↓
Crawler
```

## Crawler

```
입력
- SelectedURL[]

출력
- Document[]
  - id
  - url
  - title
  - content

정책
- max_docs = 10
- 접근 실패/timeout 시 해당 URL만 1회 재시도
```

# Export

## Golden Exapmle

**목표**

- 크롤링한 문서에서 어떤 정보를 핵심 NFR로 선택해야 하는지 기준을 제공한다.
- 선택한 NFR을 0~3 단계의 평가 기준으로 어떻게 변환해야 하는지 예시를 제공한다.

**필요성**

- 스키마만으로는 어떤 NFR을 선택해야 하는지와 각 점수 단계의 깊이를 일관되게 만들기 어렵다.
- 잘 만든 예시를 제공해 NFR 선정 기준과 0~3 점수 간 수준 차이를 안정화한다.

```yaml
Criterion:
  id: NFR-C1
  title: 검색 응답 지연시간
  description: 검색 요청이 요구되는 latency 수준을 만족하도록 설계했는가

  weight: 40

  related_nfr_ids:
    - NFR-1

  levels:
    - score: 0
      descriptor: latency 요구사항을 고려하지 않는다.

    - score: 1
      descriptor: 낮은 latency가 필요하다고 인식하지만 설계에 구체적으로 반영하지 못한다.

    - score: 2
      descriptor: 요구되는 latency를 만족할 수 있는 설계를 제시한다.

    - score: 3
      descriptor: 요구 latency를 만족하면서 병목과 tail latency, 관련 trade-off까지 고려한다.
```

## NFR Agent

```yaml
NFR Agent

목적
- Planner가 만든 NFR 후보를 실제 Document로 검증한다.
- Planner가 놓친 NFR도 추가로 발견한다.
- 해당 NFR과 관련된 Trade-off도 발견한다.
- 최종 NFR을 확정하고 0~3 Rubric Criterion으로 변환한다.

1. NFR 후보군 구성

입력
- Planner의 nfr_candidates
  - kind
  - reason
- Document[]

처리
- Planner가 선택한 NFR과 reason을 Document와 대조한다.
- reason을 뒷받침하는 내용이 실제 문서에 있는지 의미적으로 확인한다.
- Document 전체에서도 추가적인 NFR 관련 내용을 탐색한다.
- Planner 후보에 없던 중요한 NFR이 발견되면 신규 후보로 추가한다.

후보가 하나도 없는 경우
- Core NFR Catalog를 기준으로 Document를 다시 확인한다.
- 그래도 근거가 없다면 억지로 NFR을 생성하지 않는다.
- candidate_not_found

2. 핵심 NFR 선정

각 후보에 대해 다음을 확인한다.

- subject의 핵심 기능과 직접 관련 있는가
- 실제 문서 근거가 존재하는가
- 해당 NFR의 요구 수준이 달라지면 주요 설계 선택이 달라지는가

판단
- 조건을 충분히 만족하면 최종 NFR로 확정한다.
- 단순히 문서에서 한 번 언급됐다는 이유만으로 선정하지 않는다.
- 근거가 약하면 제외하거나 보류한다.

3. NFR 목표 수준 결정

처리
- Document에 수치, 단위, percentile, consistency 수준 등이 명시되어 있으면 그대로 추출한다.
- 여러 문서에 비슷한 수치가 있으면 공통 범위로 정리할 수 있다.
- 수치 없이 `low latency`, `high availability`처럼 정성적 표현만 있으면 정성적 요구사항으로 유지한다.

수치가 없는 경우
- LLM이 임의의 숫자를 생성하지 않는다.
- value_not_found

4. Trade-off 확인

처리
- Planner가 예상한 trade-off가 있다면 Document에서 근거를 먼저 찾는다.
- Planner가 놓쳤더라도 Document에 다른 핵심 trade-off가 나타나면 추가한다.
- 단순히 Catalog에 common_tradeoffs가 있다는 이유만으로 추가하지 않는다.

근거가 없는 경우
- 해당 trade-off를 최종 결과에 포함하지 않는다.

5. Rubric Criterion 생성

입력
- 최종 확정 NFR
- 관련 evidence
- trade-off
- target_level
- Golden Examples

처리
- NFR마다 평가 Criterion을 생성한다.
- Golden Example을 참고해 0~3 단계의 수준 차이를 만든다.
- target_level이 높을수록 높은 점수에서 요구하는 설계 깊이를 높인다.

기본 의미
0
- NFR을 고려하지 못함

1
- NFR을 인식했지만 설계 반영이 부족함

2
- NFR을 만족하는 설계를 제시함

3
- NFR을 만족하고 병목, 예외 상황, trade-off까지 설명함

최종 출력
- confirmed_nfrs
- 각 NFR의 rationale
- target 또는 qualitative target
- evidence_refs
- tradeoffs
- rubric criteria
- level 0~3 descriptors
```

### 출력 스키마

```yaml
NFRExport:

  confirmed_nfrs:
    # 문서 근거를 통해 최종적으로 확정된 NFR 목록

    - id: string
      # NFR 고유 ID

      kind: string
      # NFR 종류
      # 예: latency, scalability, availability, consistency

      statement: string
      # 이번 주제에서 실제 요구되는 NFR 문장
      # 예: "검색 응답은 낮은 tail latency를 유지해야 한다."

      rationale: string
      # 왜 이 NFR이 중요한지에 대한 설명
      # Planner의 reason + Document 근거를 바탕으로 정리

      target:
        # 문서에 구체적인 목표 수준이 있는 경우 저장
        # 없으면 null

        value: number | null
        # 수치
        # 예: 2

        unit: string | null
        # 단위
        # 예: second, %, requests/sec

        condition: string | null
        # 측정 조건
        # 예: p99, peak traffic

      qualitative_target: string | null
      # 수치가 없고 정성적인 수준만 확인된 경우 사용
      # 예: "low latency", "high availability"

      evidence_refs:
        - string
      # 이 NFR을 뒷받침하는 Document ID 목록

  tradeoffs:
    # 확정된 NFR과 관련된 핵심 trade-off

    - id: string

      related_nfr_ids:
        - string
      # 어떤 NFR과 관련된 trade-off인지 연결

      description: string
      # 어떤 선택 사이에 충돌이 있는지 설명
      # 예: "강한 consistency를 높이면 latency가 증가할 수 있다."

      evidence_refs:
        - string
      # 해당 trade-off의 근거 문서

  rubric:
    # 확정된 NFR을 실제 평가 기준으로 변환한 결과

    criteria:
      - id: string
        # Criterion 고유 ID

        title: string
        # 평가 항목 이름

        description: string
        # 무엇을 평가하는 Criterion인지 설명

        related_nfr_ids:
          - string
        # 어떤 NFR을 평가하는지 연결

        weight: number
        # 해당 Criterion의 중요도

        levels:
          - score: 0
            descriptor: string
            # NFR을 고려하지 못한 수준

          - score: 1
            descriptor: string
            # NFR을 인식했지만 설계 반영이 부족한 수준

          - score: 2
            descriptor: string
            # NFR 요구사항을 만족하는 설계를 제시한 수준

          - score: 3
            descriptor: string
            # 병목, 예외 상황, trade-off까지 고려한 수준
```

# Make

**목표 채점 모델**: criterion 마다 0–3 레벨. 가중 합산 후 100점 정규화한다.

```
criterion_pct = level / 3
section_pct   = Σ(w × criterion_pct) / Σw
total         = Σ(section.weight × section_pct)      # section weight 합 = 100
```

## Pre-Render

### core.weight-sum

**목표**

- 모든 Criterion에 weight가 존재하는지 검사한다.
- NFR Rubric의 총 배점이 미리 정해져 있다면 weight 합이 그 값을 만족하는지 검사한다.

**검사**

- weight 필드가 존재하는지 확인한다.
    - WEIGHT_MISSING
- 0 ≤ weight ≤ NFR 배점인지 확인한다.
    - WEIGHT_OUT_OF_RANGE
- sum(weight) == NFR 배점인지 확인한다.
    - WEIGHT_SUM_INVALID

### core.levels

**목표**

- 각 Criterion이 0~3 단계로 채점 가능한지 검사한다.

**검사**

- score 0, 1, 2, 3이 모두 존재하는지 확인한다.
    - LEVEL_MISSING
- score 중복이 없는지 확인한다.
    - LEVEL_DUPLICATED
- descriptor에 중복이나 누락이 없는지 확인한다.
    - LEVEL_DESCRIPTOR_DUPLICATED
    - LEVEL_DESCRIPTOR_MISSING

### core.nfr-measurable

**목표**

- NFR이 실제로 평가 가능한 형태인지 검사한다.

**검사**

- NFR 종류가 명확한지 확인한다.
    - NFR_KIND_MISSING
- 정량형 NFR은 숫자, 단위, 비교 기준이 있는지 확인한다.
    - NFR_VALUE_NOT_FOUND
    - NFR_UNIT_MISSING
    - NFR_COMPARATOR_MISSING
- 보장형 NFR은 consistency 수준이나 장애 허용 조건처럼 명확한 기준이 있는지 확인한다.
    - NFR_GUARANTEE_UNCLEAR

### core.cross-ref

**목표**

- 확정된 NFR과 Rubric Criterion의 연결이 정상인지 검사한다.

**검사**

- Criterion이 존재하는 NFR ID만 참조하는지 확인한다.
    - NFR_REFERENCE_INVALID
- 모든 확정 NFR이 최소 하나의 Criterion에서 평가되는지 확인한다.
    - NFR_NOT_COVERED
- Criterion이 최소 하나의 NFR과 연결되어 있는지 확인한다.
    - CRITERION_NFR_REFERENCE_MISSING

### 실패

- NFR Agent를 재실행한다.
- 최대 2회 정도 실행한다.

## Render

**목표**

- Jinja2로 렌더링해 섹션 순서·헤딩 레벨·표 구조를 코드로 고정한다.

```
# NFR

{% for criterion in rubric.criteria %}

## {{ criterion.id }}. {{ criterion.title }}

### 평가 항목
{{ criterion.description }}

### 관련 NFR
{% for nfr in criterion.related_nfrs %}
- {{ nfr.kind }}: {{ nfr.statement }}
{% endfor %}

### 배점
{{ criterion.weight }}

### 평가 기준

| Level | 기준 |
|---|---|
{% for level in criterion.levels %}
| {{ level.score }} | {{ level.descriptor }} |
{% endfor %}

{% endfor %}
```

## Post-Render

**목표**

- 최종 Markdown이 NFRExport 내용을 빠짐없이 반영했는지 확인한다.

**실패**

- Finding을 생성한다.

## LLM Judge