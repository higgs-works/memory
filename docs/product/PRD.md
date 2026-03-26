# Human-Like Memory System PRD

> 로컬 AI 에이전트를 위한 인간형 기억 시스템

**Version**: 0.1.2
**Last Updated**: 2026-03-26
**Status**: Draft

---

## 1. 개요

### 1.1 문제 정의

기존 AI 메모리 서비스(Supermemory, Mem0 등)는 본질적으로 **검색 엔진**이다.

```
사용자: "그거 뭐였지..."
시스템: "검색어를 입력하세요"
```

**근본적 모순**: 기억이 안 나서 찾는 건데, 찾으려면 이미 알아야 함.

### 1.2 솔루션

인간의 기억 메커니즘을 모방한 **자동 표면화(Spontaneous Recall)** 시스템:

- 명시적 검색 없이 현재 맥락에 관련된 기억이 자동으로 떠오름
- 연상 네트워크(Associative Network)를 통한 기억 간 연결
- 시간에 따른 망각과 사용에 따른 강화

### 1.3 목표

| 목표 | 측정 지표 |
|-----|----------|
| 자동 표면화 | 관련 기억 precision@5 > 70% (오프라인 평가셋 기준) |
| 비침투성 | no-memory-needed 프롬프트에서 표면화율 < 5% |
| 저지연 | 표면화 응답 < 100ms |
| 경량화 | 메모리 사용 < 200MB |
| 로컬 우선 | 표면화/저장 핵심 경로에 외부 API 호출 없음 |

---

## 2. 사용자 시나리오

### 2.1 Target User

로컬 AI 코딩 에이전트를 사용하는 개발자

- 1차 타겟: Claude Code 사용자
- 확장 타겟: MCP를 지원하는 다른 로컬 에이전트/IDE 사용자

### 2.2 핵심 시나리오

**시나리오 1: 과거 결정 상기**
```
사용자: "JWT 인증 구현해줘"

[기존 방식]
Claude: (과거 맥락 모름) "JWT 구현해드릴게요..."

[Memory System 적용]
시스템: 관련 기억 자동 표면화
 - 3일 전: "세션 기반 인증으로 결정"
 - 1주 전: "JWT 리프레시 토큰 전략 논의"

Claude: "이전에 세션 기반으로 결정하셨는데,
        JWT로 변경하시려는 건가요?"
```

**시나리오 2: 반복 실수 방지**
```
사용자: "배포 스크립트 수정해줘"

시스템: 관련 기억 자동 표면화
 - 2주 전: "PORT 환경변수 빠뜨려서 장애 발생"

Claude: "참고로 지난번 PORT 환경변수 누락으로
        장애가 있었는데, 이번에도 확인할까요?"
```

**시나리오 3: 컨텍스트 연속성**
```
[어제 세션]
사용자: "리팩토링 절반 했어, 나머지는 내일"

[오늘 세션]
사용자: "안녕"

시스템: 관련 기억 자동 표면화
 - 어제: "UserService 리팩토링 진행 중, 50% 완료"

Claude: "안녕하세요! 어제 UserService 리팩토링
        절반 하셨는데, 이어서 진행할까요?"
```

---

## 3. 시스템 아키텍처

### 3.1 전체 구조

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Agent Clients / UI                           │
├───────────────────────────────┬─────────────────────────────────────┤
│ Claude Code / IDE / MCP Client│ Web Dashboard (Next.js + shadcn/ui)│
└───────────────────────┬───────┴──────────────────────┬──────────────┘
                        │                              │
                        ▼                              ▼
┌───────────────────────────────────┐   ┌─────────────────────────────┐
│       MCP Server / Adapter        │   │      Local HTTP API         │
│  • surface/store/end_episode      │   │  • episodes / entities      │
│  • resources (debug/read-only)    │   │  • surfacing logs / stats   │
└───────────────────────┬───────────┘   └──────────────┬──────────────┘
                        │                              │
                        └───────────────┬──────────────┘
                                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Memory Core Service (로컬)                        │
│                                                                     │
│  • Context Analyzer                                                 │
│  • Surfacing Engine                                                 │
│  • Episode Store / Graph Manager                                    │
│  • Evaluation / Feedback Logger                                     │
│                                                                     │
│  ┌───────────────────────┐    ┌─────────────────────────────────┐  │
│  │  Gemma 3 270M (MLX)   │    │     Graph Store (SQLite)       │  │
│  │  엔티티 추출, 분류     │    │  • 엔티티 인덱스               │  │
│  │  ~125MB               │    │  • 에피소드 저장               │  │
│  └───────────────────────┘    │  • 연결 그래프                 │  │
│                                │  • 표면화 로그 / 피드백        │  │
│                                └─────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 컴포넌트 상세

| 컴포넌트 | 역할 | 기술 |
|---------|------|------|
| **Memory Core Service** | 기억 관리 메인 프로세스 | Python |
| **Gemma 3 270M** | 엔티티 추출, 분류 | MLX (Apple Silicon) |
| **Graph Store** | 에피소드, 엔티티, 연결 저장 | SQLite |
| **MCP Server / Adapter** | 범용 에이전트 통합 계층 | Python MCP SDK |
| **Local HTTP API** | 웹 UI/로컬 도구용 읽기/관리 API | FastAPI or similar |
| **Web Dashboard** | 기억 탐색, 로그 확인, 피드백 수집 | Next.js + shadcn/ui |
| **Claude Code Hook** | 초기 클라이언트 통합 | Shell script |

### 3.3 임베딩 없는 설계

**기존 RAG 방식**:
```
쿼리 → 임베딩 → 벡터 유사도 → 검색
```

**본 시스템 방식**:
```
쿼리 → 270M 엔티티 추출 → 그래프 직접 검색 → Spreading Activation
```

**장점**:
- 임베딩 모델 불필요 (메모리/복잡도 절감)
- 엔티티 기반 명확한 연결
- 그래프 구조로 연상 관계 표현

### 3.4 통합 전략

핵심 원칙은 **코어 로직을 특정 에이전트에 종속시키지 않는 것**이다.

- `Memory Core`는 DB, 알고리즘, 정책, 로깅을 담당한다.
- `MCP Server`는 에이전트가 호출하는 표준 인터페이스다.
- `Local HTTP API`는 웹 대시보드와 로컬 운영 도구가 사용한다.
- `Claude Code Hook`는 초기 채널일 뿐이며, 장기적으로는 MCP 클라이언트 중 하나로 본다.

즉, 제품의 확장성은 "MCP로만 구현"이 아니라 "코어 위에 MCP 어댑터를 올리는 구조"로 확보한다.

---

## 4. 데이터 모델

### 4.1 Episode (에피소드)

```sql
CREATE TABLE episodes (
    id TEXT PRIMARY KEY,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    -- 컨텍스트
    project TEXT,
    session_id TEXT,

    -- 내용
    content TEXT,           -- 원본 대화
    summary TEXT,           -- 요약기(summarizer)가 생성한 요약

    -- 메타데이터
    importance TEXT,        -- LOW, MEDIUM, HIGH, CRITICAL
    emotion TEXT,           -- neutral, frustration, excitement, relief

    -- 기억 강도
    strength REAL,          -- 현재 강도 (0-1)
    stability REAL,         -- 안정성 (망각 저항)
    last_accessed TIMESTAMP,
    access_count INTEGER
);
```

### 4.2 Entity (엔티티)

```sql
CREATE TABLE entities (
    id TEXT PRIMARY KEY,
    name TEXT UNIQUE,
    type TEXT,              -- PERSON, CONCEPT, TECHNOLOGY, PROJECT, DECISION
    created_at TIMESTAMP,

    -- 강도
    strength REAL,
    access_count INTEGER
);

CREATE TABLE episode_entities (
    episode_id TEXT,
    entity_id TEXT,
    salience REAL,          -- 해당 에피소드에서의 중요도
    PRIMARY KEY (episode_id, entity_id)
);
```

### 4.3 Connection (연결)

```sql
CREATE TABLE connections (
    source_id TEXT,         -- episode 또는 entity
    target_id TEXT,
    source_type TEXT,       -- 'episode' 또는 'entity'
    target_type TEXT,

    weight REAL,            -- 연결 강도 (0-1)
    relation_type TEXT,     -- 'shared_entity', 'temporal', 'causal', 'similar'
    created_at TIMESTAMP,

    PRIMARY KEY (source_id, target_id)
);
```

---

## 5. 핵심 알고리즘

### 5.1 Spreading Activation

```python
def spread_activation(
    seed_entities: list[str],
    max_hops: int = 3,
    decay: float = 0.7,
    threshold: float = 0.1,
    max_nodes: int = 50
) -> list[ActivatedMemory]:
    """
    연상 활성화 알고리즘

    1. 시드 엔티티에서 시작
    2. 연결된 노드로 활성화 전파
    3. 거리에 따라 감쇠
    4. 임계값 이하 또는 최대 홉 도달 시 중단
    """
    activated = {}
    queue = [(entity, 1.0, 0) for entity in seed_entities]

    while queue:
        node, activation, depth = queue.pop(0)

        if depth > max_hops or activation < threshold:
            continue
        if node in activated:
            continue
        if len(activated) >= max_nodes:
            break

        activated[node] = activation

        # 연결된 노드로 전파
        for neighbor, weight in get_neighbors(node):
            new_activation = activation * weight * decay
            queue.append((neighbor, new_activation, depth + 1))

    # 에피소드만 필터링하여 반환
    return get_episodes_from_activated(activated)
```

### 5.2 망각 곡선 (Forgetting Curve)

```python
def calculate_strength(episode: Episode, current_time: datetime) -> float:
    """
    Ebbinghaus 망각 곡선 + 간격 효과

    R(t) = e^(-t/S)
    S_new = S_old × (1 + α × spacing_factor)
    """
    S = episode.stability

    # 접근 이력 기반 안정성 증가
    for i, access in enumerate(episode.access_history[1:], 1):
        gap = access.time - episode.access_history[i-1].time
        spacing_factor = 1 - math.exp(-gap.total_seconds() / TAU)
        S = S * (1 + ALPHA * spacing_factor)

    # 경과 시간
    t = (current_time - episode.last_accessed).total_seconds()

    # 망각 곡선 적용
    R = math.exp(-t / S)

    return R * episode.base_importance

# 파라미터
ALPHA = 0.2          # 강화 계수
TAU = 86400          # 간격 효과 시간 상수 (1일)
BASE_STABILITY = 3600  # 초기 안정성 (1시간)
```

### 5.3 에피소드 경계 판단

```python
def detect_episode_boundary(
    recent_context: list[Message],
    new_message: Message,
    model: GemmaModel
) -> bool:
    """
    270M 모델로 에피소드 경계 판단

    경계 기준:
    - 주제 전환
    - 목표 변화
    - 시간 간격 (30분 이상)
    - 프로젝트/환경 변화
    """
    # 시간 기반 자동 경계
    if time_gap(recent_context[-1], new_message) > timedelta(minutes=30):
        return True

    # 270M 모델 판단
    prompt = f"""이전 맥락과 새 메시지가 같은 주제인지 판단하세요.

이전: {format_context(recent_context[-3:])}
새 메시지: {new_message.content}

답변 (SAME 또는 NEW):"""

    response = model.generate(prompt, max_tokens=10)
    return "NEW" in response
```

---

## 6. 흐름 상세

### 6.1 입력 시 (UserPromptSubmit Hook)

```
사용자 메시지
    │
    ▼
[1] 270M: 엔티티 추출 (~20ms)
    결과: ["JWT", "인증", "구현"]
    │
    ▼
[2] Graph: 엔티티로 시드 검색 (~5ms)
    결과: [episode_42, episode_78, entity_jwt, ...]
    │
    ▼
[3] Spreading Activation (~10ms)
    시드에서 연결 따라 활성화 전파
    │
    ▼
[4] 270M: 상위 선택 + 관련성 점수 (~20ms)
    │
    ▼
[5] Policy Gate (~1ms)
    ├── confidence < 0.60 → 표면화 안 함
    ├── 인사/짧은 확인/메타 대화 → 표면화 안 함
    ├── 동일 기억 cooldown 30분
    └── mode별 최대 개수 적용
    │
    ▼
[6] 시스템 프롬프트 주입
    <surfaced-memories>
      [HIGH] 3일 전: "세션 기반 인증으로 결정"
      [MED] 1주 전: "JWT 리프레시 토큰 논의"
    </surfaced-memories>
    │
    ▼
Agent/Model API 호출
```

**총 지연: ~50-80ms**

### 6.1.1 표면화 UX 정책

표면화는 "사실 단정"이 아니라 "회상 제안"으로 동작해야 한다.

| 항목 | 정책 |
|-----|------|
| 기본 모드 | `balanced` (기본값), 최대 3개 |
| `quiet` 모드 | 최대 1개, HIGH 중요도 + high confidence만 허용 |
| `aggressive` 모드 | 최대 5개, 디버깅/실험용 |
| trivial prompt | 인사, 짧은 확인, 단순 ACK, 메타 대화에서는 표면화 억제 |
| confidence threshold | 0.60 미만이면 표면화하지 않음 |
| 중간 신뢰도 | 0.60 이상 0.80 미만이면 1개만 짧게 제안형 문구로 표면화 |
| 높은 신뢰도 | 0.80 이상이면 최대 개수까지 허용 |
| 재표면화 제한 | 동일 episode는 30분 cooldown 적용 |
| 사용자 무시/거절 | 현재 세션 동안 suppress |
| stale/conflicting memory | "예전에 이런 결정이 있었는데, 아직 유효한지 확인 필요"처럼 불확실성 명시 |

### 6.1.2 허위 회상 방지 원칙

- 키워드 중복만 있는 기억은 표면화하지 않는다.
- 현재 작업의 결정, 진행 중 상태, 반복 실수 방지에 직접 도움이 되는 기억만 우선한다.
- 오래된 기억이 최근 결정과 충돌하면 최신 기억을 우선하고, 오래된 기억은 참고 사항으로만 제시한다.
- 시스템은 표면화 여부, 억제 사유, 사용자 반응을 모두 로그로 남겨 이후 튜닝에 사용한다.

### 6.2 응답 후 (AssistantResponse Hook)

```
Agent 응답 완료
    │
    ▼
[1] 대화 쌍 임시 저장
    │
    ▼
[2] 270M: 분석 (비동기, ~30ms)
    ├── 엔티티 추출
    ├── 에피소드 경계 판단
    ├── 중요도 분류
    └── 감정 태깅
    │
    ▼
[3] Graph 업데이트
    ├── 엔티티 연결
    ├── 표면화된 기억 강도 증가
    └── 새 연결 생성
```

### 6.3 에피소드 종료 시

```
트리거: 30분 비활성 / 주제 전환 / 세션 종료
    │
    ▼
[1] Summarizer 호출 (기본: 로컬, 선택: 외부 모델)
    "다음 대화를 2-3문장으로 요약..."
    │
    ▼
[2] 에피소드 확정 저장
    │
    ▼
[3] 연결 그래프 업데이트
```

### 6.4 백그라운드 "생각" 프로세스 (후속 연구 트랙, MVP 제외)

인간은 **끊임없이 생각하며 기억을 강화**한다. 본 시스템도 장기적으로는 백그라운드에서 지속적으로 "생각"하는 구조를 목표로 한다.

다만 이 기능은 구현 복잡도와 품질 리스크가 크므로 **MVP와 초기 Core 범위에는 포함하지 않는다**.
도입 전 선행 조건:

- 기본 표면화가 precision@5 > 70%를 안정적으로 달성할 것
- 허위 회상률이 허용 범위 이하로 관리될 것
- 과잉 표면화 억제 정책이 안정화될 것
- 유휴 상태 CPU/메모리 프로파일이 검증될 것

```
┌─────────────────────────────────────────────────────────────────┐
│              Background Thinking Process (상시 실행)             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Mind Wandering Loop (무한 반복)                          │  │
│   │                                                          │  │
│   │  while True:                                             │  │
│   │      seed = pick_random_weighted(recent_memories)        │  │
│   │      path = random_walk(seed, steps=10-30)               │  │
│   │                                                          │  │
│   │      for node in path:                                   │  │
│   │          strengthen(node)           # 방문 = 강화        │  │
│   │          strengthen_edges(node)     # 연결도 강화        │  │
│   │                                                          │  │
│   │      # 새로운 연결 발견                                   │  │
│   │      if path has distant_but_related nodes:              │  │
│   │          create_new_connection()    # 창의적 연결        │  │
│   │                                                          │  │
│   │      sleep(interval)  # 5-30초 간격                      │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Decay Loop (주기적)                                      │  │
│   │                                                          │  │
│   │  every 1 hour:                                           │  │
│   │      for episode in all_episodes:                        │  │
│   │          episode.strength = forgetting_curve(episode)    │  │
│   │                                                          │  │
│   │      # 약해진 기억 아카이브                               │  │
│   │      archive(episodes where strength < 0.01)             │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Consolidation Loop (장기)                                │  │
│   │                                                          │  │
│   │  every 24 hours (또는 시스템 유휴 시):                    │  │
│   │      # 에피소드 → 의미 지식 추상화                        │  │
│   │      extract_patterns(recent_episodes)                   │  │
│   │                                                          │  │
│   │      # 클러스터 재구성                                    │  │
│   │      rebuild_clusters()                                  │  │
│   │                                                          │  │
│   │      # 연결 가지치기                                      │  │
│   │      prune_weak_connections()                            │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 6.4.1 Mind Wandering (마음 방황)

```python
class MindWandering:
    """
    인간의 '멍때리기'를 시뮬레이션
    - 자유로운 연상의 흐름
    - 방문한 기억 강화
    - 새로운 연결 발견
    """

    def __init__(self, graph: MemoryGraph, interval: int = 10):
        self.graph = graph
        self.interval = interval  # 초 단위
        self.running = False

    async def start(self):
        """백그라운드 데몬으로 실행"""
        self.running = True
        while self.running:
            await self.wander_once()
            await asyncio.sleep(self.interval)

    async def wander_once(self):
        # 1. 시드 선택 (최근 + 고중요도 가중)
        seed = self.select_seed()

        # 2. 랜덤 워크 (연상의 흐름)
        path = self.random_walk(seed, steps=random.randint(10, 30))

        # 3. 경로상 노드 강화
        for node in path:
            self.graph.strengthen_node(node, amount=0.01)

        # 4. 경로상 엣지 강화
        for i in range(len(path) - 1):
            self.graph.strengthen_edge(path[i], path[i+1], amount=0.01)

        # 5. 새 연결 발견 (경로의 처음과 끝이 관련있으면)
        if self.should_connect(path[0], path[-1]):
            self.graph.create_connection(
                path[0], path[-1],
                relation_type='discovered',
                weight=0.3
            )

    def select_seed(self) -> str:
        """최근 + 고중요도 기억에서 가중 랜덤 선택"""
        candidates = self.graph.get_recent_memories(limit=100)
        weights = [m.strength * m.importance for m in candidates]
        return random.choices(candidates, weights=weights, k=1)[0]

    def random_walk(self, start: str, steps: int) -> list[str]:
        """그래프에서 랜덤 워크"""
        path = [start]
        current = start

        for _ in range(steps):
            neighbors = self.graph.get_neighbors(current)
            if not neighbors:
                break

            # 연결 강도에 따라 가중 선택
            weights = [n.weight for n in neighbors]
            current = random.choices(neighbors, weights=weights, k=1)[0].id
            path.append(current)

            # 고착 방지: 이미 방문한 노드 재방문 시 점프
            if path.count(current) > 2:
                current = self.select_seed()
                path.append(current)

        return path
```

#### 6.4.2 효과

| 효과 | 설명 |
|-----|------|
| **기억 강화** | 자주 연상되는 기억 = 강도 증가 |
| **연결 강화** | 자주 함께 활성화되는 쌍 = 연결 강화 |
| **새 연결 발견** | 멀리 떨어진 기억 간 관계 발견 |
| **자연스러운 망각** | 연상되지 않는 기억 = 자연 감쇠 |
| **고착 방지** | 랜덤성으로 특정 기억에 편향 방지 |

#### 6.4.3 리소스 사용

```
Mind Wandering 프로세스:
- CPU: ~1% (10초 간격)
- 메모리: 기존 그래프 공유
- 디스크 I/O: 최소 (배치 쓰기)

에너지 효율:
- 유휴 시 더 활발히 (interval ↓)
- 사용 중 덜 활발히 (interval ↑)
- 배터리 모드 감지 → 일시 중지
```

---

## 7. 기술 스택

### 7.1 핵심 의존성

```yaml
runtime: Python 3.11+
llm: MLX + Gemma 3 270M
database: SQLite
agent_integration: MCP
web_dashboard: Next.js + shadcn/ui
local_api: HTTP (localhost)
initial_client: Claude Code Hooks (initial)
```

### 7.2 Python 패키지

```
mlx>=0.10.0
mlx-lm>=0.10.0
sqlite3 (built-in)
fastapi>=0.115.0
uvicorn>=0.30.0
```

### 7.3 모델

```
Model: google/gemma-3-270m-it
Format: MLX 4-bit quantized
Size: ~125MB
Source: mlx-community/gemma-3-270m-it-4bit
```

### 7.4 시스템 요구사항

```
OS: macOS (Apple Silicon)
RAM: 8GB+ (권장 16GB)
Storage: 500MB+
```

### 7.5 프론트엔드

```yaml
framework: Next.js
ui: shadcn/ui
package_manager: pnpm
bootstrap: pnpm dlx shadcn@latest init -t next --preset bKGijv5G
```

---

## 8. 프로젝트 구조

```
memory/
├── docs/
│   ├── README.md                      # 문서 인덱스
│   ├── product/
│   │   ├── PRD.md                     # 이 문서
│   │   └── concept.md                 # 컨셉 문서
│   ├── research/
│   │   ├── prior-art.md               # 기존 서비스 분석
│   │   └── human-memory-research.md   # 인지과학 연구
│   └── archive/
│       ├── deep-analysis.md           # 초기 심층 분석 메모
│       └── slm-architecture.md        # 벡터/SLM 대안 탐색
│
├── apps/
│   └── dashboard/             # Web UI Dashboard (Next.js)
│       ├── app/
│       ├── components/
│       └── lib/
│
├── src/
│   ├── __init__.py
│   ├── main.py                # CLI 엔트리포인트
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   └── server.py          # Local HTTP API
│   │
│   ├── mcp/
│   │   ├── __init__.py
│   │   ├── server.py          # MCP server
│   │   ├── tools.py           # surface/store/end_episode 등
│   │   └── resources.py       # debug/read-only resources
│   │
│   ├── llm/
│   │   ├── __init__.py
│   │   └── gemma.py           # MLX Gemma 래퍼
│   │
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── store.py           # SQLite 저장소
│   │   ├── episode.py         # 에피소드 관리
│   │   ├── entity.py          # 엔티티 관리
│   │   └── graph.py           # 연결 그래프
│   │
│   ├── algorithm/
│   │   ├── __init__.py
│   │   ├── activation.py      # Spreading Activation
│   │   ├── forgetting.py      # 망각 곡선
│   │   └── boundary.py        # 에피소드 경계
│   │
│   ├── daemon/                # Phase 4 이후
│   │   ├── __init__.py
│   │   ├── thinking.py        # Mind Wandering 루프
│   │   ├── decay.py           # 망각 곡선 적용 루프
│   │   └── consolidation.py   # 장기 통합 루프
│   │
│   └── service/
│       ├── __init__.py
│       ├── surface.py         # 기억 표면화
│       ├── store.py           # 기억 저장
│       ├── feedback.py        # 사용자 피드백/억제
│       └── inspect.py         # 대시보드 조회용 read model
│
├── hooks/
│   ├── user-prompt-submit.sh  # 초기 클라이언트 Hook (Claude Code)
│   └── assistant-response.sh
│
├── tests/
│   └── ...
│
├── pyproject.toml
└── README.md
```

---

## 9. API 설계

### 9.1 CLI Commands

```bash
# 기억 표면화 (Hook에서 호출)
$ memory surface --query "JWT 인증 구현해줘"
# stdout: <surfaced-memories>...</surfaced-memories>

# 기억 저장 (Hook에서 호출)
$ memory store --user "JWT 인증 구현해줘" --assistant "..."

# 에피소드 종료
$ memory end-episode --session-id "abc123"

# 로컬 API 서버 시작
$ memory api serve --port 3710

# MCP 서버 시작
$ memory mcp serve

# === 데몬 관리 (Phase 4 이후) ===

# 백그라운드 "생각" 프로세스 시작
$ memory daemon start
# Started: Mind Wandering (interval=10s), Decay (interval=1h)

# 데몬 상태 확인
$ memory daemon status
# Mind Wandering: running (last: 3s ago, walks: 1,247)
# Decay: running (last: 23m ago)
# Consolidation: scheduled (next: 6h)

# 데몬 중지
$ memory daemon stop

# 수동 통합 실행
$ memory consolidate

# === 상태 확인 ===

# 전체 상태
$ memory status
# Episodes: 156, Entities: 423, Connections: 1,247
# Daemon: running
# Memory: 145MB

# 최근 "생각" 로그
$ memory daemon log --tail 10
# [12:34:01] Walk: episode_42 → entity_jwt → episode_78 → ...
# [12:34:01] Strengthened: 12 nodes, 11 edges
# [12:34:01] New connection: episode_42 ↔ episode_156 (discovered)
```

### 9.2 Local HTTP API

```http
GET /episodes?limit=50
GET /episodes/{id}
GET /entities?query=jwt
GET /graph/entity/{name}
GET /surfacing/logs?date=today
GET /stats

POST /feedback
{
  "surface_event_id": "evt_123",
  "action": "helpful" | "irrelevant" | "harmful" | "dismiss"
}
```

### 9.3 MCP Interface

#### Tools

```text
memory.surface
memory.store_turn
memory.end_episode
memory.feedback
memory.get_stats
```

#### Resources

```text
memory://episodes/recent
memory://episode/{id}
memory://graph/entity/{name}
memory://surfacing/logs/today
memory://stats
```

### 9.4 Python API

```python
from memory import MemoryService

service = MemoryService()

# 표면화
memories = service.surface("JWT 인증 구현해줘")
# -> [SurfacedMemory(episode_id, summary, relevance, ...)]

# 저장
service.store(
    user_message="JWT 인증 구현해줘",
    assistant_message="...",
    session_id="abc123"
)

# 에피소드 종료 (요약기 포함)
service.end_episode(session_id="abc123", summarizer=local_summarize)
```

### 9.5 Web Dashboard

주요 화면:

- Overview: 최근 표면화 이벤트, suppress 비율, false recall 피드백
- Episodes: 최근 에피소드 목록 및 상세
- Entities/Graph: 엔티티와 연결 관계 탐색
- Surfacing Log: 왜 특정 기억이 표면화/억제되었는지 확인
- Review Queue: 사용자가 harmful/irrelevant로 표시한 케이스 검토

### 9.6 데몬 자동 시작 (Phase 4 이후)

```bash
# macOS LaunchAgent 등록
$ memory daemon install
# Created: ~/Library/LaunchAgents/com.memory.daemon.plist
# Daemon will start automatically on login

# 또는 초기 클라이언트 시작 시 자동 실행
# 예: Claude Code의 경우 .claude/settings.json
{
  "hooks": {
    "SessionStart": [
      { "command": "memory daemon start --if-not-running" }
    ],
    "SessionEnd": [
      { "command": "memory daemon pause" }  // 세션 종료 시 일시정지
    ]
  }
}
```

### 9.7 에너지 효율 (Phase 4 이후)

```python
# 상황에 따른 interval 자동 조절

INTERVALS = {
    "active_session": 30,      # 에이전트 사용 중: 느리게
    "idle": 5,                 # 유휴 상태: 빠르게 (적극적 통합)
    "battery": 60,             # 배터리 모드: 최소화
    "low_battery": None,       # 저배터리: 일시정지
}
```

---

## 10. 마일스톤

### Phase 1: MVP (2주)

```
목표: 기본 동작하는 시스템

[ ] 프로젝트 셋업 (Python, MLX)
[ ] Gemma 270M 래퍼 구현
[ ] SQLite 스키마 + 기본 CRUD
[ ] 엔티티 추출 (270M)
[ ] 기본 표면화 (엔티티 매칭)
[ ] 초기 클라이언트 Hook 연동 (Claude Code)
[ ] MCP server 초안
[ ] Local HTTP API 초안
[ ] 기본 저장 파이프라인
[ ] 표면화 이벤트 로깅
[ ] 기본 UX 가드레일 (confidence threshold, max-items 제한, trivial prompt suppression, cooldown)
[ ] 평가셋/라벨 포맷 정의
[ ] 최소 Web Dashboard
    - 최근 episode 목록
    - 표면화 로그 목록
    - episode 상세 보기
```

### Phase 2: Core (2주)

```
목표: 핵심 알고리즘 구현

[ ] Spreading Activation 구현
[ ] 망각 곡선 구현
[ ] 에피소드 경계 판단 (270M)
[ ] 중요도/감정 분류 (270M)
[ ] 에피소드 요약 (pluggable summarizer, 기본 로컬)
[ ] 연결 그래프 구축
[ ] 오프라인 평가 harness 구현
[ ] negative test set 구축
[ ] precision / false recall 튜닝
[ ] Dashboard 고도화
    - entity/graph 탐색
    - 피드백 액션
    - suppress / harmful case review
```

### Phase 3: Polish (1주)

```
목표: 안정화 및 최적화

[ ] 성능 최적화 (지연 < 100ms)
[ ] 에러 핸들링
[ ] 표면화 정책 튜닝
[ ] 사용자 피드백 수집
[ ] MCP/HTTP API 안정화
[ ] Dashboard UX polish
[ ] 테스트 작성
[ ] 문서화
```

### Phase 4: Enhancement (이후)

```
[ ] Background Thinking Daemon
[ ] Mind Wandering 루프
[ ] Decay 루프
[ ] Consolidation 루프
[ ] 에너지 효율 모드
[ ] 270M 파인튜닝 (도메인 특화)
[ ] Dashboard 고급 그래프 탐색 / 운영 기능 확장
[ ] 다중 프로젝트 지원
[ ] 기억 내보내기/가져오기
```

---

## 11. 성공 지표

| 지표 | 목표 | 측정 방법 |
|-----|------|----------|
| **표면화 정확도** | precision@5 > 70% | 오프라인 평가셋 200건 이상 |
| **허위 회상률** | top-3 기준 irrelevant/harmful < 10% | 오프라인 평가셋 + 수동 라벨 |
| **과잉 표면화율** | no-memory-needed 프롬프트에서 표면화 < 5% | suppression 테스트셋 |
| **표면화 지연** | < 100ms | 로그 측정 |
| **메모리 사용** | < 200MB | 프로세스 모니터링 |
| **저장 지연** | < 50ms (비동기) | 로그 측정 |
| **사용자 만족** | 유용함 > 방해됨 (2:1 이상) | 세션 피드백 |

### 11.1 평가 프로토콜

1. 실제 세션 로그에서 프롬프트 시점을 추출해 평가셋을 만든다.
2. 각 샘플은 해당 시점 이전에 존재하던 기억만 후보로 사용한다.
3. 평가셋은 최소 4개 카테고리를 포함한다.
   - 과거 결정 상기
   - 반복 실수 방지
   - 진행 중 작업 이어가기
   - 기억이 없어야 하는 프롬프트 (`안녕`, `고마워`, 짧은 ACK 등)
4. 각 샘플에서 시스템의 top-5 표면화 결과를 저장하고 수동 라벨링한다.
5. 주간 단위로 오프라인 평가와 실제 사용자 피드백을 함께 본다.

### 11.2 정답 라벨링 기준

| 라벨 | 정의 | precision 계산 포함 여부 |
|-----|------|-------------------------|
| **Relevant** | 없으면 결정 충돌, 반복 실수, 컨텍스트 단절 가능성이 큰 기억 | 포함 |
| **Helpful** | 있으면 좋지만 없어도 작업 수행에는 큰 문제 없는 기억 | 제외 |
| **Irrelevant** | 키워드만 비슷하거나 현재 작업과 직접 관련 없는 기억 | 제외 |
| **Harmful** | 오래되어 잘못된 결론을 유도하거나, 충돌/혼란/방해를 일으키는 기억 | 제외 |

정의:

- `precision@5 = top-5 중 Relevant 개수 / 5`
- `false recall rate = surfaced 항목 중 Irrelevant + Harmful 비율`
- `과잉 표면화율 = no-memory-needed 샘플에서 하나라도 표면화된 비율`

---

## 12. 리스크 및 대응

| 리스크 | 영향 | 대응 |
|-------|------|------|
| 270M 추출 정확도 낮음 | 관련없는 기억 표면화 | 파인튜닝 또는 규칙 보완 |
| 허위 회상 / 과잉 표면화 | 사용자 신뢰 하락 | policy gate, cooldown, suppress, negative test set |
| 표면화 지연 > 100ms | UX 저하 | 캐싱, 비동기화 |
| 기억 과다 (노이즈) | 관련 기억 묻힘 | 망각 곡선 튜닝 |
| MCP / Dashboard 경계 복잡도 | 구현 복잡도 증가 | core/api/mcp 역할 분리, read-only dashboard부터 시작 |
| Mind Wandering 복잡도 | 일정 지연, 품질 불안정 | Phase 4로 분리, 오프라인 실험 후 도입 |
| Apple Silicon 전용 | 사용자 제한 | 추후 llama.cpp 지원 |

---

## 13. 참고 자료

### 인지과학
- Collins & Loftus (1975) - Spreading Activation Theory
- Ebbinghaus (1885) - Forgetting Curve
- Tulving - Encoding Specificity Principle

### 기술
- [MLX Documentation](https://ml-explore.github.io/mlx/)
- [Gemma 3 270M](https://developers.googleblog.com/en/introducing-gemma-3-270m/)
- [Model Context Protocol](https://modelcontextprotocol.io/introduction)
- [Claude Code Hooks](https://docs.anthropic.com/claude-code/hooks)
- [shadcn/ui](https://ui.shadcn.com/docs/installation/next)

### 관련 프로젝트
- Supermemory
- Mem0
- Zep
