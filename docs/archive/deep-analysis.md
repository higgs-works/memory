# Human-Like Memory System: 심층 분해 분석

> Status: Archived exploration
>
> 이 문서는 초기 문제 분해 메모다. 일부 구현 가정은 현재 기준 문서인 [PRD](../product/PRD.md)와 다르다.

## 개요

본 문서는 [concept](../product/concept.md)에 정의된 Human-Like Memory System 컨셉에 대한 초기 심층 분해 분석이다.

---

## 1. 핵심 문제 분해

### 1.1 "자동 표면화(Spontaneous Recall)"의 정확한 의미

```
AS-IS: user.search("JWT") -> results
TO-BE: system.surface(current_context) -> relevant_memories
```

**자동 표면화의 구성 요소:**

| 구성 요소 | 설명 | 기술적 함의 |
|----------|------|------------|
| **암묵적 트리거** | 사용자가 명시적 검색 없이 | 시스템이 컨텍스트 변화를 지속적으로 모니터링 |
| **컨텍스트 인식** | 현재 상황을 분석하여 | 실시간 컨텍스트 임베딩 생성 및 비교 |
| **관련성 판단** | 관련 기억을 선별하고 | 활성화 전파 + 임계값 기반 필터링 |
| **자동 제시** | 작업 기억에 표면화 | 제한된 슬롯(7±2개)에 우선순위 배치 |

### 1.2 검색 vs 표면화의 기술적 차이

| 차원 | 검색(Search) | 표면화(Surfacing) |
|-----|-------------|------------------|
| **트리거** | 명시적 쿼리 | 암묵적 컨텍스트 변화 |
| **방향** | Pull (사용자가 요청) | Push (시스템이 제안) |
| **시점** | 요청 시점에만 실행 | 지속적/주기적 실행 |
| **입력** | 키워드/문장 | 전체 컨텍스트 상태 |
| **알고리즘** | 벡터 유사도 + 키워드 매칭 | 연상 활성화 + 임계값 기반 |

### 1.3 하위 문제 분해

```
┌─────────────────────────────────────────────────────────────────────┐
│                       자동 표면화 시스템                               │
├─────────────────────────────────────────────────────────────────────┤
│  [P1] 컨텍스트 표현 문제                                              │
│       └─ 현재 대화/코드/환경을 어떻게 수치적으로 표현할 것인가?          │
│                                                                     │
│  [P2] 초기 활성화 문제                                                │
│       └─ 컨텍스트에서 어떤 노드를 초기 활성화할 것인가?                 │
│                                                                     │
│  [P3] 전파 제어 문제                                                  │
│       └─ 활성화가 어디까지 퍼지고 언제 멈출 것인가?                     │
│                                                                     │
│  [P4] 임계값 결정 문제                                                │
│       └─ 어느 정도 활성화된 기억이 "떠오른" 것인가?                    │
│                                                                     │
│  [P5] 작업 기억 관리 문제                                             │
│       └─ 7±2개 슬롯에 어떤 기억을 배치할 것인가?                       │
│                                                                     │
│  [P6] 실시간성 문제                                                   │
│       └─ 대화 흐름을 방해하지 않는 속도로 계산 가능한가?                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Spreading Activation 심층 분석

### 2.1 인지 과학에서의 원래 의미

Collins & Loftus (1975)의 Spreading Activation Theory:

1. 개념들은 **노드**로 표현되고, 의미적 관련성에 따라 **엣지**로 연결
2. 하나의 노드가 활성화되면, 연결된 노드들로 활성화가 **병렬적으로 전파**
3. 전파 강도는 **링크 가중치**와 **거리**에 따라 감쇠
4. 두 개념이 **동시에 활성화**되면 해당 연결이 **강화**

### 2.2 소프트웨어 구현의 과제

| 과제 | 인지 모델 | 소프트웨어 제약 |
|-----|---------|---------------|
| **병렬성** | 수십억 뉴런이 진정한 병렬 처리 | 제한된 스레드, 순차적 시뮬레이션 |
| **연속성** | 아날로그 활성화 값 | 이산적 수치 표현 |
| **규모** | 자동 관계 형성 | 관계를 명시적으로 저장/관리 |
| **실시간성** | 밀리초 단위 반응 | 그래프 탐색 비용, I/O 지연 |

### 2.3 활성화 무한 확산 방지

```python
def propagate(node, activation, depth, fired_set):
    if depth > MAX_HOPS:           # 1. 홉 제한
        return
    if activation < THRESHOLD:      # 2. 발화 임계값
        return
    if node in fired_set:           # 3. 단일 발화 규칙
        return

    fired_set.add(node)
    node.activation += activation

    for neighbor, weight in node.neighbors[:MAX_FANOUT]:  # 4. 팬아웃 제한
        new_activation = activation * weight * DECAY       # 5. 감쇠 계수
        propagate(neighbor, new_activation, depth + 1, fired_set)
```

### 2.4 활성화 강도 계산 공식

```
A_j = Σ(A_i × W_ij × D^d)

Where:
- A_j: 노드 j의 활성화 수준
- A_i: 소스 노드 i의 활성화 수준
- W_ij: i와 j 사이 링크 가중치 (0 < W ≤ 1)
- D: 감쇠 계수 (0 < D < 1, typically 0.85)
- d: 소스로부터의 거리 (홉 수)
```

---

## 3. 에피소드 vs 청크 분석

### 3.1 에피소드 경계 정의

| 경계 유형 | 트리거 | AI 대화 맥락 예시 |
|----------|-------|------------------|
| **시간적 불연속** | 시간 간격 > 임계값 | 30분 이상 대화 중단 |
| **공간적 불연속** | 맥락/환경 변화 | 다른 프로젝트로 전환 |
| **목표 변화** | 수행 중인 목표 전환 | "인증 구현" → "UI 수정" |
| **주제 전환** | 대화 주제 변경 | "JWT" → "DB 설계" |

### 3.2 대화와 에피소드의 관계

**하나의 대화 ≠ 하나의 에피소드**

```
┌─────────────────── 하나의 대화 세션 ───────────────────┐
│  [에피소드 1]        [에피소드 2]        [에피소드 3]  │
│  "인증 시스템 논의"   "점심 잡담"         "DB 설계"    │
│  ~~~~~~~~~~~~~~~    ~~~~~~~~~~        ~~~~~~~~~~~~   │
│                                                       │
│  경계: 주제 전환     경계: 주제 전환                   │
└───────────────────────────────────────────────────────┘
```

### 3.3 에피소드 내부 구조

```typescript
Episode {
  id: UUID
  created_at: Timestamp
  context: {
    project: string
    participants: string[]
    environment: object
  }

  segments: [
    { type: "problem_statement", content: string, timestamp: T }
    { type: "discussion", content: string, timestamp: T }
    { type: "decision", content: string, timestamp: T, salience: HIGH }
    { type: "outcome", content: string, timestamp: T }
  ]

  extracted_entities: string[]
  connections: [{ target: UUID, type: string, weight: number }]

  strength: number
  last_accessed: Timestamp
  access_count: number
}
```

---

## 4. 동적 강도의 수학적 모델

### 4.1 망각 곡선 (Forgetting Curve)

Ebbinghaus (1885) 기반:

```
R(t) = e^(-t/S)

Where:
- R: Retrievability (회수 가능성, 0~1)
- t: 마지막 접근 이후 경과 시간
- S: Stability (기억 안정성)
```

### 4.2 강화 메커니즘 (Strengthening)

```
S_new = S_old × (1 + α × spacing_factor × salience)

Where:
- α: 기본 강화 계수 (0.1~0.5)
- spacing_factor: 간격 효과 (1 - e^(-gap/τ))
- salience: 중요도 배율
```

### 4.3 통합 강도 함수

```python
def calculate_strength(episode, current_time):
    S = episode.base_stability

    # 접근 이력 기반 안정성 증가
    for i, access in enumerate(episode.access_history):
        if i == 0: continue
        gap = access.time - episode.access_history[i-1].time
        spacing_factor = 1 - exp(-gap / TAU)
        salience = access.salience or 1.0
        S = S * (1 + ALPHA * spacing_factor * salience)

    # 경과 시간
    t = current_time - episode.last_accessed

    # 망각 곡선 적용
    R = exp(-t / S)

    return R * episode.base_salience
```

**권장 파라미터:**

| 파라미터 | 값 | 설명 |
|---------|---|------|
| `ALPHA` | 0.2 | 기본 강화 계수 |
| `TAU` | 86400 (1일) | 간격 효과 시간 상수 |
| `base_stability` | 3600 (1시간) | 초기 안정성 |
| `salience_decision` | 2.0 | 결정 순간 중요도 배율 |

---

## 5. 기술적 도전과제

### 5.1 실시간 활성화 계산

```
최악: O(f^h)  -- 지수적 증가
제한: O(min(V, f^h)) ≈ O(500) with h=3, f=8
```

**핵심:** 홉/팬아웃 제한으로 탐색 범위를 상수적으로 유지 가능

### 5.2 하이브리드 아키텍처 제안

```
┌─────────────────────────────────────────────────────────┐
│                    Hybrid Memory System                  │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐      ┌─────────────────────────┐  │
│  │  Vector DB      │ ←──→ │   Graph DB               │  │
│  │  (임베딩, ANN)   │      │   (연상 네트워크)         │  │
│  └─────────────────┘      └─────────────────────────┘  │
│           │                          │                  │
│           └──────────┬───────────────┘                  │
│                      ▼                                  │
│              ┌───────────────┐                          │
│              │ Orchestrator  │                          │
│              │ 1. Vector→seed│                          │
│              │ 2. Graph→전파 │                          │
│              │ 3. 결합/순위화│                          │
│              └───────────────┘                          │
└─────────────────────────────────────────────────────────┘
```

---

## 6. 미결 질문들

### 6.1 명확히 정의되지 않은 개념

| 개념 | 필요한 정의 |
|-----|-----------|
| **"현재 컨텍스트"** | 어떤 신호들을 포함? 어떻게 인코딩? |
| **"작업 기억 7±2개"** | 무엇이 하나의 "항목"인가? |
| **"감정 가중치"** | AI 대화에서 어떻게 감지/정량화? |
| **"에피소드 최소 단위"** | 한 문장? 한 턴? 한 주제? |

### 6.2 설계 결정 포인트

| 결정 포인트 | 선택지 | 트레이드오프 |
|------------|-------|------------|
| **엣지 생성** | 명시적 추출 vs 임베딩 유사도 | 정확도 vs 자동화 |
| **활성화 트리거** | 매 메시지 vs 주제 변화 감지 | 반응성 vs 연산 비용 |
| **표면화 UI** | 자동 삽입 vs 사이드바 | 몰입감 vs 통제 |
| **그래프 저장소** | Neo4j vs 관계형+인접리스트 | 성능 vs 복잡도 |

### 6.3 검증이 필요한 가정

| 가정 | 리스크 |
|-----|-------|
| 연상 활성화 > 검색 | 핵심 가치 제안 무효화 |
| 에피소드 > 청크 | 비용 증가 정당화 실패 |
| 자동 표면화가 UX 개선 | 오히려 방해가 될 수 있음 |

---

## 7. 권장 다음 단계

### 우선순위 1: 핵심 정의 명확화
- "컨텍스트"의 구체적 구성 요소와 인코딩 방식
- "에피소드" 경계 판단 알고리즘
- 작업 기억 "항목" 단위 결정

### 우선순위 2: MVP 범위 결정
- 최소 기능 집합 정의
- 검증할 가정 우선순위화
- 성공 지표 정의

### 우선순위 3: 기술 스택 결정
- 그래프 저장소 선택
- 벡터 DB 통합 여부
- 실시간성 요구사항 정량화

---

## 참고 자료

- Collins & Loftus (1975) - Spreading Activation Theory
- Ebbinghaus (1885) - Forgetting Curve
- Baddeley & Hitch (1974) - Working Memory Model
