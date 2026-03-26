# Prior Art: 기존 메모리 서비스 분석

## 개요

Human-Like Memory System 설계에 앞서 기존 솔루션들을 분석한다.

---

## 1. Supermemory

> https://github.com/supermemoryai/supermemory

### 개요
MCP 기반 AI 메모리 레이어. Claude, Cursor 등과 통합되는 범용 메모리 서비스.

### 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                    SUPERMEMORY                       │
├─────────────────────────────────────────────────────┤
│  Web App (Next.js) ←→ MCP Server (Cloudflare)       │
│           ↓                    ↓                    │
│     PostgreSQL           Vector DB                  │
│     (Documents,         (Embeddings)                │
│      Metadata)                                      │
└─────────────────────────────────────────────────────┘
```

### 기술 스택
- **Frontend**: Next.js 16, React 19, TypeScript
- **Backend**: Cloudflare Workers, Hono
- **Database**: PostgreSQL + Vector DB
- **Integration**: MCP Protocol, OAuth 2.0

### 핵심 기능
- 메모리 저장 (텍스트, URL, PDF, 이미지)
- 하이브리드 검색 (벡터 + 시맨틱)
- 프로필 자동 생성
- 프로젝트별 구분 (containerTag)

### 한계점
| 문제 | 설명 |
|-----|------|
| **검색 기반** | 명시적 쿼리 필요 |
| **Flat Memory** | 모든 기억이 동등하게 취급 |
| **연결 없음** | 기억 간 연상 관계 없음 |
| **정적 저장** | 시간에 따른 강도 변화 없음 |

---

## 2. claude-mem

> https://github.com/thedotmack/claude-mem

### 개요
Claude Code 전용 메모리 압축 시스템. Lifecycle Hook으로 자동 캡처.

### 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                     CLAUDE-MEM                       │
├─────────────────────────────────────────────────────┤
│  Lifecycle Hooks                                     │
│  ├── SessionStart      (초기화)                     │
│  ├── UserPromptSubmit  (세션 시작)                  │
│  ├── PostToolUse       (관찰 자동 캡처)             │
│  └── Stop              (세션 요약)                  │
│                                                      │
│  Worker Service (port 37777)                         │
│  └── Web Viewer UI                                   │
│                                                      │
│  ┌─────────────┐    ┌─────────────────┐            │
│  │   SQLite    │ ←→ │  Chroma Vector  │            │
│  │ (structured)│    │  (embeddings)   │            │
│  └─────────────┘    └─────────────────┘            │
└─────────────────────────────────────────────────────┘
```

### 기술 스택
- **Runtime**: Bun, Node.js 18+
- **Database**: SQLite (WAL mode, FTS5)
- **Vector**: Chroma (Python, uv managed)
- **SDK**: Claude Agent SDK, MCP SDK

### 핵심 혁신: 3계층 토큰 절약 패턴

```
Layer 1: search()
         → ID + 메타데이터만 반환
         → ~50-100 토큰/결과

Layer 2: timeline()
         → 주변 맥락 확인
         → 최소 토큰

Layer 3: get_observations()
         → 필요한 것만 전체 로드
         → ~500-1000 토큰/결과

결과: ~10x 토큰 절약
```

### Observation 구조

```typescript
interface Observation {
  type: 'decision' | 'bugfix' | 'feature' | 'refactor' | 'discovery' | 'change'
  title: string | null
  subtitle: string | null
  facts: string[]
  narrative: string | null
  concepts: string[]
  files_read: string[]
  files_modified: string[]
}
```

### 강점
| 특징 | 설명 |
|-----|------|
| **자동 캡처** | Hook 기반, 사용자 개입 불필요 |
| **토큰 효율** | Progressive disclosure |
| **로컬 우선** | 프라이버시, 오프라인 가능 |
| **구조화된 관찰** | 타입별 분류 |

### 한계점
| 문제 | 설명 |
|-----|------|
| **검색 기반** | `search()` 호출 필요 |
| **연상 없음** | 기억 간 연결 구조 없음 |
| **자동 표면화 없음** | 기억이 저절로 떠오르지 않음 |
| **Claude Code 전용** | 범용성 부족 |

---

## 3. 비교 분석

### 기능 비교

| 기능 | Supermemory | claude-mem | Human-Like (목표) |
|-----|-------------|-----------|------------------|
| 저장 방식 | 클라우드 | 로컬 | TBD |
| 캡처 | 명시적 | 자동 (Hook) | 자동 |
| 검색 | 하이브리드 | 3계층 | 자동 표면화 |
| 연상 네트워크 | ❌ | ❌ | ✅ |
| 동적 강도 | ❌ | ❌ | ✅ |
| 망각 곡선 | ❌ | ❌ | ✅ |
| 에피소드 구조 | ❌ (청크) | △ (Observation) | ✅ |
| 프라이밍 | ❌ | ❌ | ✅ |

### 검색 vs 표면화

```
[기존 시스템]
사용자 → "JWT 관련 기억 찾아줘" → 검색 → 결과

[Human-Like Memory]
(인증 얘기 중) → 시스템이 자동으로 → "JWT 선택 이유" 표면화
```

---

## 4. 배울 점

### Supermemory에서
- MCP 프로토콜 통합 방식
- 하이브리드 검색 (벡터 + 시맨틱)
- 프로젝트별 격리 (containerTag)

### claude-mem에서
- **자동 캡처** (Lifecycle Hook 패턴)
- **토큰 효율성** (Progressive Disclosure)
- **구조화된 관찰** (타입별 분류)
- **로컬 우선 아키텍처** (SQLite + Chroma)

---

## 5. 차별화 포인트

우리 시스템이 해결해야 할 것:

| 기존의 한계 | 우리의 접근 |
|-----------|-----------|
| 검색 필요 | **자동 표면화** - 컨텍스트 기반 |
| Flat Memory | **연상 네트워크** - 기억 간 연결 |
| 정적 저장 | **동적 강도** - 망각/강화 곡선 |
| 청크 단위 | **에피소드 단위** - 맥락 보존 |
| 명시적 회상 | **프라이밍** - 암묵적 활성화 |

---

## 참고 링크

- [Supermemory GitHub](https://github.com/supermemoryai/supermemory)
- [claude-mem GitHub](https://github.com/thedotmack/claude-mem)
- [Mem0](https://github.com/mem0ai/mem0) - AI 에이전트용 메모리
- [Zep](https://github.com/getzep/zep) - LLM 메모리 서비스
