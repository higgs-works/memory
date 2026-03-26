# 로컬 SLM 기반 메모리 시스템 아키텍처

> Small Language Model을 활용한 Human-Like Memory System 구현 방안
>
> Status: Archived exploration
>
> 이 문서는 벡터/임베딩 기반 대안을 포함한 초기 탐색안이다. 현재 기준 아키텍처는 [PRD](../product/PRD.md)를 따른다.

---

## 1. 왜 로컬 SLM인가?

### 1.1 장점

| 장점 | 설명 |
|-----|------|
| **프라이버시** | 모든 기억이 로컬에 저장, 외부 전송 없음 |
| **저지연** | 네트워크 호출 없이 밀리초 단위 응답 |
| **비용** | API 호출 비용 없음, 일회성 하드웨어 투자 |
| **가용성** | 인터넷 없이도 동작, 항상 사용 가능 |
| **커스터마이징** | 도메인 특화 파인튜닝 가능 |

### 1.2 2024-2025 SLM 현황

[Medium - Why Local LLMs Matter in 2025](https://medium.com/@danieltse/why-local-llms-matter-in-2025-be0b46eb6f8c):

```
2024-2025 코딩 모델 혁명:
"작은 모델이 훨씬 큰 모델에 필적하는 성능 달성"

하드웨어 요구사항:
- 최소: 8GB VRAM (시작점)
- 권장: 16-24GB VRAM (13B-30B 모델)
- 일반 데스크톱: NVMe SSD + 16-64GB RAM + 디스크리트 GPU
```

---

## 2. 아키텍처 개요

### 2.1 역할 분담 설계

```
┌─────────────────────────────────────────────────────────────────┐
│                    Human-Like Memory System                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │   Embedding SLM   │    │  Generation SLM  │                   │
│  │  (308M - 1B)     │    │   (1B - 7B)      │                   │
│  │                  │    │                  │                   │
│  │  • 텍스트 임베딩   │    │  • 에피소드 요약   │                   │
│  │  • 유사도 검색    │    │  • 엔티티 추출    │                   │
│  │  • 클러스터링     │    │  • 연결 생성      │                   │
│  │  • 시맨틱 분석    │    │  • 컨텍스트 분석  │                   │
│  └────────┬─────────┘    └────────┬─────────┘                   │
│           │                       │                              │
│           └───────────┬───────────┘                              │
│                       ▼                                          │
│           ┌───────────────────────┐                              │
│           │   Memory Controller   │                              │
│           │                       │                              │
│           │  • Spreading Activation│                             │
│           │  • 망각 곡선 관리       │                             │
│           │  • 표면화 결정         │                             │
│           └───────────┬───────────┘                              │
│                       ▼                                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Storage Layer                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │   │
│  │  │ Vector DB   │  │  Graph DB   │  │  Episode Store  │   │   │
│  │  │ (Embeddings)│  │ (Relations) │  │  (Raw Data)     │   │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 각 컴포넌트별 SLM 선택

| 역할 | 권장 모델 | 크기 | 용도 |
|-----|----------|-----|------|
| **임베딩** | EmbeddingGemma | 308M | 벡터 생성, 유사도 |
| **임베딩 (대안)** | nomic-embed-text | 137M | 경량 임베딩 |
| **생성/분석** | Gemma 2B | 2B | 요약, 추출 |
| **생성 (고품질)** | Phi-3-mini | 3.8B | 복잡한 추론 |
| **생성 (최대)** | Gemma 7B / Llama 3 8B | 7-8B | 고품질 분석 |

---

## 3. 임베딩 SLM 활용

### 3.1 EmbeddingGemma (권장)

[Google Developers Blog](https://developers.googleblog.com/en/introducing-embeddinggemma/):

```
EmbeddingGemma 특징:
- 파라미터: 308M (매우 경량)
- 용도: On-device RAG, 시맨틱 검색
- 통합: Ollama, LangChain, LlamaIndex, sentence-transformers 등

장점:
- 모바일/엣지 디바이스에서도 구동 가능
- 빠른 임베딩 생성
- 메모리 효율적
```

### 3.2 임베딩 활용 시나리오

```python
# Ollama + EmbeddingGemma 예시

from ollama import embeddings

class MemoryEmbedder:
    def __init__(self):
        self.model = "embeddinggemma"  # 또는 "nomic-embed-text"

    def embed_episode(self, episode: Episode) -> list[float]:
        """에피소드를 벡터로 변환"""
        text = self.episode_to_text(episode)
        response = embeddings(model=self.model, prompt=text)
        return response['embedding']

    def find_similar(self, query_embedding, top_k=10):
        """유사한 기억 검색 - Spreading Activation의 시드로 활용"""
        # Vector DB에서 유사 벡터 검색
        return self.vector_db.search(query_embedding, top_k)

    def cluster_memories(self, embeddings):
        """관련 기억 클러스터링"""
        # HDBSCAN 등으로 자동 클러스터링
        pass
```

### 3.3 성능 기대치

```
EmbeddingGemma (308M):
- 임베딩 생성: ~10-50ms per text
- 메모리: ~500MB RAM
- GPU 불필요 (CPU로 충분)

nomic-embed-text (137M):
- 임베딩 생성: ~5-20ms per text
- 메모리: ~300MB RAM
```

---

## 4. 생성 SLM 활용

### 4.1 역할별 모델 선택

```
┌─────────────────────────────────────────────────────────────┐
│  작업 복잡도에 따른 모델 선택                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  간단한 작업 (Gemma 2B / Phi-3-mini)                        │
│  ├── 엔티티 추출 (Named Entity Recognition)                 │
│  ├── 키워드 추출                                            │
│  ├── 간단한 요약 (1-2문장)                                  │
│  └── 감정/중요도 분류                                       │
│                                                             │
│  중간 작업 (Gemma 7B / Llama 3 8B)                          │
│  ├── 에피소드 경계 판단                                     │
│  ├── 연결 관계 추론                                         │
│  ├── 컨텍스트 분석                                          │
│  └── 정교화 리허설 시뮬레이션                               │
│                                                             │
│  복잡한 작업 (필요시 외부 LLM 또는 더 큰 로컬 모델)         │
│  ├── 복잡한 추론                                            │
│  ├── 긴 문맥 처리                                           │
│  └── 창의적 연결 생성                                       │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 엔티티/연결 추출

[SMALLM Framework](https://link.springer.com/article/10.1007/s40747-025-02074-6) 접근법 참고:

```python
# 로컬 SLM으로 엔티티 추출

class EntityExtractor:
    def __init__(self):
        self.model = "gemma:2b"  # Ollama 모델명

    def extract_entities(self, text: str) -> list[Entity]:
        prompt = f"""다음 텍스트에서 중요한 엔티티를 추출하세요.

텍스트: {text}

다음 형식으로 응답:
- PERSON: [이름들]
- CONCEPT: [개념들]
- TECHNOLOGY: [기술들]
- PROJECT: [프로젝트들]
- DECISION: [결정 사항들]
"""
        response = self.generate(prompt)
        return self.parse_entities(response)

    def extract_connections(self, episode: Episode) -> list[Connection]:
        prompt = f"""다음 에피소드에서 엔티티 간 관계를 추출하세요.

에피소드:
{episode.content}

엔티티들: {episode.entities}

관계 형식: [엔티티1] -[관계유형]-> [엔티티2]
"""
        response = self.generate(prompt)
        return self.parse_connections(response)
```

### 4.3 에피소드 요약 및 경계 판단

```python
class EpisodeAnalyzer:
    def __init__(self):
        self.model = "gemma:7b"  # 더 복잡한 분석용

    def summarize_episode(self, episode: Episode) -> str:
        prompt = f"""다음 대화 에피소드를 1-2문장으로 요약하세요.
핵심 결정, 문제 해결, 중요 발견에 초점을 맞추세요.

에피소드:
{episode.content}

요약:"""
        return self.generate(prompt, max_tokens=100)

    def detect_episode_boundary(self, recent_context: str, new_message: str) -> bool:
        prompt = f"""새 메시지가 이전 맥락과 같은 에피소드인지 판단하세요.

이전 맥락:
{recent_context}

새 메시지:
{new_message}

경계 기준:
- 주제 전환
- 목표 변화
- 시간 간격 (30분 이상)
- 프로젝트/환경 변화

답변 (SAME_EPISODE 또는 NEW_EPISODE):"""
        response = self.generate(prompt, max_tokens=20)
        return "NEW_EPISODE" in response
```

---

## 5. 하이브리드 파이프라인

### 5.1 기억 저장 파이프라인

```
새 입력 → [임베딩 SLM] → 벡터 생성
       → [생성 SLM]  → 엔티티 추출
                     → 연결 추론
                     → 에피소드 경계 판단
                     → 요약 생성
       → [저장소]    → Vector DB (임베딩)
                     → Graph DB (연결)
                     → Episode Store (원본)
```

### 5.2 기억 인출 파이프라인

```
현재 컨텍스트 → [임베딩 SLM] → 쿼리 벡터 생성
             → [Vector DB]  → 유사 기억 검색 (시드)
             → [Graph DB]   → Spreading Activation
             → [생성 SLM]   → 관련성 순위화
                            → 작업 기억 선택 (7±2개)
             → 표면화
```

### 5.3 백그라운드 통합 파이프라인

```
주기적 실행 (유휴 시간):
1. [Graph DB] → 최근 고활성 기억 선택
2. [생성 SLM] → 마음 방황 시뮬레이션
              → 새 연결 발견
              → 기존 연결 강화/약화
3. [임베딩 SLM] → 클러스터 재계산
4. [저장소] → 업데이트
```

---

## 6. 구현 기술 스택

### 6.1 권장 스택

```yaml
# 핵심 컴포넌트
inference_runtime: Ollama
embedding_model: embeddinggemma  # 또는 nomic-embed-text
generation_model: gemma:2b       # 기본, gemma:7b 복잡한 작업

# 저장소
vector_db: ChromaDB              # 로컬, 임베딩 검색
graph_db: SQLite + 인접 리스트    # 간단, 또는 Neo4j (복잡)
episode_store: SQLite            # 로컬 파일 기반

# 프레임워크
orchestration: LangChain         # 또는 LlamaIndex
```

### 6.2 Ollama 설정

```bash
# 모델 설치
ollama pull embeddinggemma
ollama pull gemma:2b
ollama pull gemma:7b  # 선택적

# 동시 로드 설정 (RAG 파이프라인용)
# Ollama는 여러 모델 동시 로드 지원
```

### 6.3 메모리 최적화

```
4-bit 양자화 활용:
- gemma:7b-q4 → ~4GB VRAM
- 80-120 tokens/second 달성 가능

최적 레이어 오프로드:
- 13B 모델 기준 20-25 레이어 GPU
- Attention/Embedding → GPU
- Feed-Forward → CPU
```

---

## 7. 성능 예측

### 7.1 지연 시간 추정

| 작업 | 모델 | 예상 지연 |
|-----|------|----------|
| 임베딩 생성 | EmbeddingGemma | 10-50ms |
| 엔티티 추출 | Gemma 2B | 100-300ms |
| 에피소드 요약 | Gemma 2B | 200-500ms |
| 연결 추론 | Gemma 7B | 500ms-1s |
| 전체 저장 파이프라인 | - | ~1-2s |
| 인출 + 표면화 | - | ~200-500ms |

### 7.2 하드웨어 요구사항

```
최소 사양:
- CPU: 4코어 이상
- RAM: 16GB
- 저장: SSD 50GB+
- GPU: 없어도 동작 (느림)

권장 사양:
- CPU: 8코어 이상
- RAM: 32GB
- 저장: NVMe SSD 100GB+
- GPU: 8GB+ VRAM (RTX 3060 이상)

최적 사양:
- CPU: 12코어 이상
- RAM: 64GB
- 저장: NVMe SSD 256GB+
- GPU: 16-24GB VRAM (RTX 4080/4090)
```

---

## 8. 확장 고려사항

### 8.1 파인튜닝 가능성

```
로컬 파인튜닝 장점:
- 도메인 특화 엔티티 인식
- 개인화된 연결 패턴 학습
- 사용자별 중요도 기준 학습

LoRA/QLoRA로 효율적 파인튜닝:
- 전체 모델 재학습 불필요
- 수백~수천 개 파라미터만 업데이트
- 몇 시간 내 로컬에서 가능
```

### 8.2 점진적 확장

```
Phase 1 (MVP):
- Ollama + Gemma 2B + ChromaDB
- 기본 저장/인출 기능

Phase 2:
- Gemma 7B 추가 (복잡한 분석)
- Spreading Activation 구현
- 망각 곡선 적용

Phase 3:
- 백그라운드 통합 프로세스
- 파인튜닝 파이프라인
- 성능 최적화
```

---

## 9. 대안 및 비교

### 9.1 클라우드 LLM vs 로컬 SLM

| 측면 | 클라우드 LLM | 로컬 SLM |
|-----|------------|---------|
| 성능 | 높음 (GPT-4급) | 중간 (충분) |
| 지연 | 1-5초 | 0.1-1초 |
| 비용 | 사용량 비례 | 초기 투자 후 무료 |
| 프라이버시 | 데이터 외부 전송 | 완전 로컬 |
| 가용성 | 인터넷 필요 | 항상 가능 |
| 커스터마이징 | 제한적 | 완전 자유 |

### 9.2 결론

```
Human-Like Memory System에 로컬 SLM 적합한 이유:

1. 빈번한 호출
   - 매 대화마다 임베딩, 분석, 인출
   - 클라우드 API 비용 급증 방지

2. 프라이버시 핵심
   - 개인 기억 = 민감 데이터
   - 외부 전송 없이 처리

3. 저지연 필수
   - 자동 표면화 = 실시간 반응
   - 네트워크 지연 허용 불가

4. 작업 특성
   - 추출, 요약, 분류 = SLM 강점
   - 창의적 생성 아님
```

---

## 참고 자료

- [Ollama GitHub](https://github.com/ollama/ollama)
- [EmbeddingGemma - Google Developers](https://developers.googleblog.com/en/introducing-embeddinggemma/)
- [Local RAG with Ollama](https://dev.to/apideck/build-a-local-rag-with-ollama-huggingface-faiss-and-google-gemma-3-2e0n)
- [Why Local LLMs Matter in 2025](https://medium.com/@danieltse/why-local-llms-matter-in-2025-be0b46eb6f8c)
- [Memory for LLM Agents - GitHub Collection](https://github.com/TsinghuaC3I/Awesome-Memory-for-Agents)
- [SMALLM Framework - Springer](https://link.springer.com/article/10.1007/s40747-025-02074-6)
- [Intel IPEX-LLM](https://github.com/intel/ipex-llm)
