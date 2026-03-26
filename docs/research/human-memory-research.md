# 인간 기억 메커니즘 연구 조사

> Human-Like Memory System 설계를 위한 인지과학/신경과학 기반 조사

---

## 1. 기억의 신경생물학적 기반

### 1.1 해마(Hippocampus)의 역할

해마는 에피소드 기억 처리의 핵심 허브다. [Nature Human Behaviour (2023)](https://www.nature.com/articles/s41562-023-01706-6) 연구에 따르면:

```
감각 입력
    ↓
내측측두엽 피질 (Multi-modal representations)
├── 비주위피질 (Perirhinal): 객체 정보
└── 해마방피질 (Parahippocampal): 공간/맥락 정보
    ↓
내후각피질 (Entorhinal Cortex): 게이팅
    ↓
해마 형성체 (Hippocampal Formation)
├── 치상회 (DG): 패턴 분리 (Pattern Separation)
├── CA3: 자기연상 네트워크 (Auto-associative Network)
└── CA1 + 해마지각: 신피질로 피드백
    ↓
신피질 (Neocortex): 안정적 통합
```

### 1.2 에피소드 특이적 뉴런 (Episode-Specific Neurons)

[Nature Human Behaviour 연구](https://www.nature.com/articles/s41562-023-01706-6)에서 발견된 핵심 사실:

| 특성 | 설명 |
|-----|------|
| **위치** | 해마에서만 관찰됨 |
| **코딩 방식** | 발화율(rate) 또는 시간적 발화(temporal) 코드 사용 |
| **핵심 발견** | 개별 요소(개념, 시간)가 아닌 **요소들의 결합(conjunction)**을 코딩 |

### 1.3 기억 형성의 점진적 안정화

[Nature Neuroscience (2025)](https://www.nature.com/articles/s41593-025-01986-3) 연구:

```
Day 1 → Day 7 학습 과정:
- 안정적 장소장(place field)을 유지하는 장소 세포 수 ↑
- 개별 장소 세포의 안정성 ↑

메커니즘:
"희소한 시냅스 집합의 점진적 안정화가 인출 시
과거 활동 패턴을 복구하는 새로운 가소성을 유도"
→ 에피소드 기억의 인코딩과 유지를 위한 효율적 메커니즘
```

---

## 2. Spreading Activation 이론 심화

### 2.1 원본 이론 (Collins & Loftus, 1975)

[원본 논문](https://psycnet.apa.org/record/1976-03421-001)의 핵심:

```
1. 개념 = 노드, 의미적 관련성 = 엣지
2. 하나의 노드 활성화 → 연결된 노드로 병렬적 전파
3. 전파 강도: 링크 가중치 × 거리에 따라 감쇠
4. 두 개념 동시 활성화 → 연결 강화
```

### 2.2 ACT-R 모델 (Anderson)

```
기억 활성화 수준: A = B + Σ(Wj × Sji)

Where:
- A: 총 활성화 수준
- B: 기본 활성화 (사용 빈도/최근성)
- W: 주의 가중치
- S: 연상 강도
```

### 2.3 최신 도구: SpreadPy (2024-2025)

[SpreadPy](https://arxiv.org/pdf/2507.09628) - 인지 네트워크 과학용 도구:
- Collins & Loftus 이론 기반 구현
- 다양한 지식 모델링 접근법 지원
- 인지 현상 연구에 네트워크 과학 적용

### 2.4 Fan Effect (팬 효과)

```
노드의 연결 수 ↑ → 개별 연결 강도 ↓

예시:
- "사과" 노드가 3개 연결: 각 연결 강도 = 0.33
- "사과" 노드가 10개 연결: 각 연결 강도 = 0.10

시스템 시사점:
- 과도한 연결은 개별 연상 품질 저하
- 연결 수 제한 또는 가중치 정규화 필요
```

---

## 3. 맥락 의존적 기억 (Context-Dependent Memory)

### 3.1 인코딩 특수성 원리 (Encoding Specificity Principle)

[Tulving & Thomson](https://en.wikipedia.org/wiki/Encoding_specificity_principle)의 핵심 원리:

```
인출 성공 = f(인코딩 시 맥락, 인출 시 맥락의 일치도)
```

### 3.2 클래식 연구: 수중 실험 (Godden & Baddeley, 1975)

| 학습 환경 | 테스트 환경 | 회상 성과 |
|----------|-----------|----------|
| 수중 | 수중 | **높음** |
| 수중 | 지상 | 낮음 |
| 지상 | 지상 | **높음** |
| 지상 | 수중 | 낮음 |

### 3.3 맥락의 종류

| 맥락 유형 | 설명 | AI 시스템 적용 |
|----------|------|--------------|
| **환경적 맥락** | 물리적 장소, 시간 | 프로젝트, 파일 경로, 세션 |
| **상태 의존적** | 생리/심리 상태 | 감정 상태, 작업 모드 |
| **언어적 맥락** | 사용 언어, 어휘 | 도메인 용어, 코드 스타일 |
| **사회적 맥락** | 함께 있던 사람 | 팀원, 대화 상대 |

### 3.4 맥락 회상 기법

[Frontiers in Psychology (2024)](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2024.1489039/full):

```
물리적 환경 재현 불가 시:
→ "맥락 회상 기법(Context Recall Technique)" 사용
→ 기억에서 과거 환경 단서를 의식적으로 생성

AI 시스템 적용:
- 직접 맥락 매칭 우선
- 실패 시 관련 맥락 특징 기반 검색
```

---

## 4. 기억 통합과 수면 (Memory Consolidation)

### 4.1 통합의 두 단계

[PMC 연구](https://pmc.ncbi.nlm.nih.gov/articles/PMC2680680/):

| 단계 | 시간 범위 | 메커니즘 |
|-----|----------|---------|
| **시냅스 통합** | 학습 후 수 시간 | Late-phase LTP |
| **시스템 통합** | 수 주 ~ 수 년 | 해마 → 신피질 이동 |

### 4.2 수면의 역할

```
NREM 수면 (비REM):
├── 1-4 단계로 세분
├── 서파 수면(SWS)에서 기억 재생
└── 해마 → 신피질 전송

REM 수면:
├── 90분 주기로 NREM과 교대
├── 감정 기억 처리에 중요
└── 기억 통합 기여 (Nature, 2025)

Sharp Wave Ripples:
- 수면/휴식 중 발생
- 인코딩 시 활동 패턴 재생
- 해마 및 연결 구조에서 기억 인출/통합
```

### 4.3 재통합 (Reconsolidation)

[Cell/Trends in Neurosciences](https://www.cell.com/trends/neurosciences/abstract/S0166-2236(05)00159-1):

```
통합된 기억 → 재활성화 → 불안정 상태 → 재통합
                ↓
          수정/강화 가능한 창

Nader et al. (2000) 실험:
- 재활성화된 기억 + 단백질 합성 억제 → 기억 손상
- "기억은 단순히 형성되는 것이 아니라 진화한다"
```

**시스템 시사점**:
- 기억 접근 시 업데이트 기회
- 오래된 기억도 새 정보와 통합 가능

---

## 5. 프라이밍과 암묵적 기억 (Priming & Implicit Memory)

### 5.1 명시적 vs 암묵적 기억

| 구분 | 명시적 기억 | 암묵적 기억 |
|-----|-----------|-----------|
| **의식** | 의식적 회상 | 무의식적 영향 |
| **예시** | "어제 뭘 했지?" | 단어 완성, 기술 수행 |
| **뇌 영역** | 내측측두엽 (해마) | 신피질 (감각/운동 영역) |
| **기억상실증** | 손상됨 | 보존됨 |

### 5.2 프라이밍의 종류

[Wikipedia - Priming](https://en.wikipedia.org/wiki/Priming_(psychology)):

| 유형 | 설명 | 예시 |
|-----|------|-----|
| **지각적** | 물리적 형태 유사성 | "DOCTOR" → "D_CT_R" |
| **의미적/개념적** | 의미적 관련성 | "의사" → "간호사" |
| **연상적** | 동시 발생 경험 | "소금" → "후추" |
| **반복적** | 동일 자극 재노출 | 광고 반복 효과 |
| **긍정적** | 처리 속도 향상 | 관련 단어 빠른 인식 |
| **부정적** | 처리 속도 저하 | 무시했던 자극 느린 처리 |

### 5.3 Two-Mechanism Account

[UCPress 연구](https://online.ucpress.edu/collabra/article/3/1/13/112372/Implicit-and-Explicit-Memory-Factors-in-Cumulative):

```
구조적 프라이밍 = 암묵적 학습 + 명시적 기억

기본 프라이밍 효과: 암묵적 학습 과정
단어 반복 시 증폭: 명시적 기억 과정
```

### 5.4 AI 시스템 시사점

```
1. 암묵적 활성화:
   - 사용자가 의식하지 않아도 관련 기억 활성화
   - 직접 인출 요청 없이 연상 네트워크 준비

2. 프라이밍 체인:
   컨텍스트 A → 관련 기억 활성화 → 연상된 기억 프라이밍
                                    ↓
                            다음 인출에 영향

3. 부정적 프라이밍 고려:
   - 자주 무시되는 기억의 접근성 저하
   - 억제 메커니즘 구현 필요
```

---

## 6. 감정과 기억 (Emotional Memory)

### 6.1 편도체-해마 상호작용

[PMC 연구](https://pmc.ncbi.nlm.nih.gov/articles/PMC1906714/):

```
감정적 사건
    ↓
편도체 활성화 (감정 경보 시스템)
    ↓ 교차 대화
해마 신호 (시간 기록 시스템)
    ↓
강화된 통합
    ↓
선명한 기억 + 맥락 결합
```

### 6.2 섬광 기억 (Flashbulb Memory)

[ScienceDirect](https://www.sciencedirect.com/topics/neuroscience/flashbulb-memory):

```
특징:
- 놀랍고 감정적으로 각성되는 사건
- "어디서, 누구와, 무엇을, 어떻게 느꼈는지" 상세 기억
- 지속적이고 상세함 (정확성과는 별개)

신경 메커니즘:
1. 스트레스 반응 → 전전두엽 기능 방해
2. 편도체가 실행 제어 담당
3. 1-2분간 "섬광 기억 모드" 작동
```

### 6.3 스트레스 호르몬의 역할

```
카테콜아민 (노르에피네프린):
└── 시냅스 가소성 조절

글루코코르티코이드 (코르티솔):
└── 감정적 정보의 적절한 인코딩

임상 연구:
- 편도체 손상 환자 → 섬광 기억 회상 품질 ↓
- 비언어 우세 반구 손상 시 더 큰 영향
```

### 6.4 감정 가중치 시스템 설계 제안

```python
def calculate_emotional_weight(event):
    # 기본 중요도
    base_salience = event.explicit_importance or 1.0

    # 감정 신호 (대화 분석에서 추출)
    emotional_markers = detect_emotional_markers(event.content)

    # 감정 강도 계산
    emotional_intensity = sum([
        markers.surprise * 1.5,      # 놀람
        markers.frustration * 1.3,   # 좌절감
        markers.excitement * 1.4,    # 흥분
        markers.relief * 1.2,        # 안도감
    ]) / 4

    # 결정/해결 순간 부스트
    if event.type in ['decision', 'breakthrough', 'error_resolution']:
        emotional_intensity *= 1.5

    return base_salience * (1 + emotional_intensity)
```

---

## 7. 의미 기억 네트워크 (Semantic Memory Network)

### 7.1 뇌의 의미 처리 네트워크

[PMC 연구](https://pmc.ncbi.nlm.nih.gov/articles/PMC3350748/) 120개 연구 메타분석:

```
일반 의미 네트워크 (좌반구 우세):
├── 각회 (Angular Gyrus)
├── 외측/복측 측두 피질
├── 복내측 전전두 피질
├── 하전두회
├── 배내측 전전두 피질
└── 후대상회
```

### 7.2 의미 기억 모델 종류

[Springer 리뷰](https://link.springer.com/article/10.3758/s13423-020-01792-x):

| 모델 | 표현 방식 | 장점 | 단점 |
|-----|----------|-----|-----|
| **연상 네트워크** | 노드 + 링크 | 직관적, 해석 가능 | 관계 수동 정의 |
| **분산 표현** | 벡터 공간 | 자동 학습, 유사도 계산 | 해석 어려움 |
| **특징 기반** | 속성 집합 | 범주화 설명 | 특징 선택 어려움 |

### 7.3 접지된 인지 (Grounded Cognition)

```
단어의 의미 = 감각운동 시스템에 접지

예: "배" 생각할 때:
├── 시각: 배 이미지 시뮬레이션
├── 미각: 달콤한 맛 시뮬레이션
├── 촉각: 손에 쥐는 느낌
└── 운동: 베어 무는 동작

AI 시스템 시사점:
- 추상적 벡터만이 아닌 다양한 모달리티 연결
- 경험적 맥락이 의미를 풍부하게 함
```

### 7.4 범주 특이적 손상

```
측두엽 손상 → 특정 범주 의미 기억 선택적 손상

예:
- 생물 범주만 손상 (동물, 식물)
- 도구 범주만 손상 (인공물)

시사점:
의미 기억이 범주별로 다른 영역에 분산 저장
```

---

## 8. 망각의 메커니즘

### 8.1 망각의 원인

| 이론 | 메커니즘 | 특징 |
|-----|---------|-----|
| **소멸 (Decay)** | 시간에 따른 흔적 약화 | Ebbinghaus 망각 곡선 |
| **간섭 (Interference)** | 유사 기억 간 경쟁 | 순행/역행 간섭 |
| **인출 실패** | 단서 부족으로 접근 불가 | 기억은 존재, 경로만 차단 |
| **인출 유도 망각** | 특정 기억 반복 인출 → 관련 기억 억제 | 경쟁적 억제 |

### 8.2 인출 유도 망각 (Retrieval-Induced Forgetting)

```
실험 패러다임:
1. 학습: 과일-사과, 과일-바나나, 과일-체리...
2. 연습: "과일-사___" (사과만 반복 인출)
3. 테스트: 모든 과일 회상

결과:
- 사과: 회상 향상
- 바나나, 체리: 회상 저하 (억제됨)

AI 시스템 주의점:
- 자동 표면화 시 특정 기억만 반복 활성화 → 다른 기억 억제 가능
- 다양한 경로로 접근하여 편향 방지
```

### 8.3 간격 효과와 최적 복습

```
최적 복습 간격 (Spacing Effect):
┌────────────────────────────────────────┐
│  보유 기간    최적 복습 간격            │
│  ─────────   ────────────              │
│  1주일        1-2일                    │
│  1개월        1주일                    │
│  1년          1개월                    │
└────────────────────────────────────────┘

원리:
- 너무 빠른 복습: 강화 효과 적음
- 너무 늦은 복습: 이미 망각됨
- 최적점: 망각 직전 복습 시 최대 강화
```

---

## 9. 작업 기억 모델 (Working Memory)

### 9.1 Baddeley 모델

```
┌─────────────────────────────────────────────────┐
│              중앙 실행기                          │
│         (Central Executive)                      │
│    주의 제어, 조정, 억제, 전환                    │
└────────┬─────────────┬────────────┬─────────────┘
         │             │            │
    ┌────▼────┐  ┌─────▼─────┐  ┌──▼───────────┐
    │ 음운 루프 │  │에피소드 버퍼│  │시공간 스케치패드│
    │Phonological│ │ Episodic  │  │ Visuospatial │
    │   Loop   │  │  Buffer   │  │  Sketchpad   │
    └──────────┘  └───────────┘  └──────────────┘
     언어/청각      장기기억 연결    시각/공간 정보
      정보 유지     멀티모달 통합     일시적 저장
```

### 9.2 용량 제한: 7±2 규칙

```
Miller (1956):
- 작업 기억 용량: 7±2 청크
- 청크의 크기는 가변적

청킹 (Chunking):
- "FBICIAUSAIBM" → "FBI CIA USA IBM" (4청크)
- 전문성 증가 → 더 큰 청크 가능

AI 시스템 적용:
- 표면화되는 기억 수 제한 (5-9개)
- 관련 기억을 청크로 그룹화하여 제시
```

---

## 10. 시스템 설계 시사점 요약

### 10.1 핵심 인사이트

| 인지과학 원리 | AI 시스템 구현 방향 |
|-------------|-------------------|
| 에피소드 특이적 뉴런 | 요소 결합을 단위로 저장 |
| 점진적 안정화 | 반복 접근 시 기억 강도 증가 |
| 맥락 의존적 인출 | 현재 맥락과 유사한 기억 우선 |
| 수면 통합 | 주기적 정리 및 연결 강화 |
| 재통합 | 접근 시 업데이트 허용 |
| 암묵적 프라이밍 | 무의식적 연상 활성화 |
| 감정 가중치 | 중요 순간 기억 강화 |
| 인출 유도 망각 | 다양한 접근 경로 확보 |
| 작업 기억 제한 | 표면화 개수 제한 |

### 10.2 구현 우선순위

```
Phase 1 (MVP):
├── Spreading Activation 기본 구현
├── 맥락 기반 초기 활성화
└── 망각 곡선 (Ebbinghaus)

Phase 2:
├── 에피소드 구조화 저장
├── 감정 가중치 시스템
└── 재통합 메커니즘

Phase 3:
├── 암묵적 프라이밍 체인
├── 인출 유도 망각 방지
└── 주기적 통합 프로세스
```

---

## 11. 생각을 통한 기억 강화 (Thinking & Memory Strengthening)

### 11.1 리허설의 두 종류

[Wikipedia - Memory Rehearsal](https://en.wikipedia.org/wiki/Memory_rehearsal):

| 유형 | 설명 | 효과 |
|-----|------|-----|
| **유지 리허설** (Maintenance) | 단순 반복 (전화번호 되뇌기) | 단기기억 유지, 장기기억 전이 X |
| **정교화 리허설** (Elaborative) | 의미 연결, 기존 지식과 통합 | 장기기억 형성에 효과적 |

```
유지 리허설:
"010-1234-5678, 010-1234-5678..." → 통화 후 망각

정교화 리허설:
"010-1234-5678... 1234는 순서대로, 5678도 순서대로네"
→ 의미 부여 → 장기 기억화
```

### 11.2 정교화의 메커니즘

[APA Dictionary of Psychology](https://onefocusapp.com/blog/long-term-memory-elaborative-rehearsal/) 기반:

```
정교화 리허설 = 인코딩 전략

핵심 요소:
1. 의미 처리 (Semantic Processing)
2. 기존 지식과 연결 (Association)
3. 맥락 부여 (Contextualization)
4. 자기 참조 (Self-Reference)

효과:
- 항목 내 특징 강화 (Intraitem Feature Strengthening)
- 의식적 회상 능력 향상
- 친숙성 느낌 증가
```

### 11.3 마음 방황과 기억 통합 (Mind Wandering)

[Nature Reviews Neuroscience](https://www.nature.com/articles/nrn.2016.113) 연구:

```
마음 방황 (Mind Wandering):
= 자발적 사고 (Spontaneous Thought)
= 자유로운 생각의 흐름

핵심 발견:
"자발적 사고의 신경 역학이 기억 재생(Memory Replay)과 유사"
→ 기억 통합에 최적화된 프로세스
```

### 11.4 Default Variability Hypothesis (Mills et al., 2018)

```
┌─────────────────────────────────────────────────────┐
│  생각의 자유로운 이동이 기억 통합을 촉진한다         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  마음 방황 특성:        기억 통합 효과:              │
│  ├── 변동성(Variability) → 에피소드 기억 인코딩     │
│  ├── 반복(Repetition)    → 의미 추상화              │
│  └── 유사 무작위 순서    → 기억 재생과 유사          │
│                                                     │
│  메커니즘:                                          │
│  기억 흔적이 새롭고 다양한 순서로 재활성화          │
│  → 의미/에피소드 기억 통합 최적화                   │
└─────────────────────────────────────────────────────┘
```

### 11.5 반추 vs 마음 방황

| 특성 | 마음 방황 | 반추 (Rumination) |
|-----|----------|------------------|
| **흐름** | 자유롭고 변화함 | 한 주제에 고착 |
| **주의** | 비유도적 (Unguided) | 자동/의도적 유도 |
| **결과** | 창의성, 기억 통합 | 기분 장애 위험 |
| **신경 기반** | DMN 전반 | DMN 일부 + 정서 회로 |

```
건강한 처리:
생각 A → 연상 → 생각 B → 연상 → 생각 C → ...
(자유로운 전환 = 기억 통합)

문제적 처리:
생각 A → 생각 A → 생각 A → 생각 A → ...
(고착 = 기억 강화이지만 부정적 편향)
```

### 11.6 관련 뇌 영역

[Science Advances (2022)](https://www.science.org/doi/10.1126/sciadv.abn8616):

```
자기 생성 개념 처리 시 활성화 영역:
├── 내측 전전두 피질 (mPFC): 자기 참조
├── 내측 측두엽: 자전적 기억
├── 해마: 가상적 시나리오 구성
└── 전정맥/후대상회: 디폴트 모드 네트워크

Default Mode Network (DMN):
- 외부 과제 없을 때 활성화
- 자전적 기억 회상
- 미래 시뮬레이션
- 사회적 인지
```

### 11.7 AI 시스템 설계 시사점

```python
# 사용자가 "생각을 통해 기억 강화"하는 것을 모방

class ThinkingBasedConsolidation:
    """
    사용자의 자발적 사고 패턴을 시뮬레이션하여
    기억 네트워크를 강화하는 시스템
    """

    def simulate_mind_wandering(self, seed_memory):
        """
        하나의 기억에서 시작해 연상 네트워크를 따라
        자유롭게 이동하며 연결 강화
        """
        current = seed_memory
        visited = []

        for _ in range(MAX_WANDERING_STEPS):
            visited.append(current)

            # 연결된 기억 중 랜덤하게 (가중치 기반) 다음 선택
            next_memory = self.weighted_random_walk(current)

            # 연결 강화 (정교화 효과)
            self.strengthen_connection(current, next_memory)

            # 반추 방지: 같은 곳 반복 시 탈출
            if self.is_stuck(visited):
                current = self.jump_to_distant_memory()
            else:
                current = next_memory

        return visited

    def elaborative_rehearsal(self, memory, context):
        """
        의미적 연결을 통한 기억 강화
        """
        # 1. 현재 컨텍스트와 연결
        self.connect_to_context(memory, context)

        # 2. 기존 관련 지식과 연결
        related = self.find_semantic_neighbors(memory)
        for r in related:
            self.strengthen_connection(memory, r)

        # 3. 자기 참조 연결 (사용자의 이전 경험)
        self_ref = self.find_autobiographical_links(memory)
        for s in self_ref:
            self.strengthen_connection(memory, s, weight=1.5)

        memory.stability *= ELABORATION_BOOST

    def periodic_consolidation(self):
        """
        유휴 시간에 기억 네트워크 순회하며 통합
        (수면 중 기억 재생과 유사)
        """
        seeds = self.get_recent_high_salience_memories()

        for seed in seeds:
            # 마음 방황 시뮬레이션
            self.simulate_mind_wandering(seed)

            # 재통합 체크
            self.check_reconsolidation_opportunities(seed)
```

### 11.8 핵심 원리 요약

```
인간의 생각 → 기억 강화 메커니즘:

1. 정교화 리허설
   - 단순 반복 X, 의미 연결 O
   - 기존 지식과 통합할수록 강화

2. 마음 방황
   - 자유로운 연상의 흐름
   - 기억 재생과 유사한 신경 패턴
   - 에피소드 → 의미 기억 추상화

3. 자기 참조 처리
   - "나와 관련짓기" → 최강 인코딩
   - 자전적 기억과 연결 시 강화

4. 변동성의 중요성
   - 한 곳에 고착 = 반추 (비생산적)
   - 자유로운 전환 = 통합 (생산적)

AI 시스템 적용:
→ 주기적으로 기억 네트워크 "방황"
→ 연결 강화 및 새로운 연결 발견
→ 반추 패턴 감지 시 강제 전환
```

---

## 참고 문헌

### 주요 논문
- [Collins & Loftus (1975) - Spreading Activation Theory](https://psycnet.apa.org/record/1976-03421-001)
- [Tulving & Thomson - Encoding Specificity](https://en.wikipedia.org/wiki/Encoding_specificity_principle)
- [Nature Human Behaviour (2023) - Episode-Specific Neurons](https://www.nature.com/articles/s41562-023-01706-6)
- [Nature Neuroscience (2025) - Memory Formation](https://www.nature.com/articles/s41593-025-01986-3)
- [PMC - Sleep-Dependent Memory Consolidation](https://pmc.ncbi.nlm.nih.gov/articles/PMC2680680/)
- [PMC - Temporal Dynamics of Emotional Memory](https://pmc.ncbi.nlm.nih.gov/articles/PMC1906714/)
- [PMC - Neurobiology of Semantic Memory](https://pmc.ncbi.nlm.nih.gov/articles/PMC3350748/)

### 도구 및 리뷰
- [SpreadPy (2024-2025)](https://arxiv.org/pdf/2507.09628)
- [Semantic Memory Review - Psychonomic Bulletin](https://link.springer.com/article/10.3758/s13423-020-01792-x)
- [Context-Dependent Memory - Frontiers (2024)](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2024.1489039/full)

### 마음 방황 및 리허설
- [Nature Reviews Neuroscience - Mind Wandering as Spontaneous Thought](https://www.nature.com/articles/nrn.2016.113)
- [Science Advances - Self-Generated Concepts in Spontaneous Thought](https://www.science.org/doi/10.1126/sciadv.abn8616)
- [PMC - Why Do We Think?](https://pmc.ncbi.nlm.nih.gov/articles/PMC11210302/)
- [Wikipedia - Memory Rehearsal](https://en.wikipedia.org/wiki/Memory_rehearsal)
- [Elaborative Rehearsal - 1Focus](https://onefocusapp.com/blog/long-term-memory-elaborative-rehearsal/)

### Wikipedia 참고
- [Spreading Activation](https://en.wikipedia.org/wiki/Spreading_activation)
- [Context-Dependent Memory](https://en.wikipedia.org/wiki/Context-dependent_memory)
- [Priming (Psychology)](https://en.wikipedia.org/wiki/Priming_(psychology))
- [Memory Consolidation](https://en.wikipedia.org/wiki/Memory_consolidation)
- [Semantic Memory](https://en.wikipedia.org/wiki/Semantic_memory)
