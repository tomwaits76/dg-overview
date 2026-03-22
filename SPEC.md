# SPEC.md — 모두의 벗 랜딩 페이지

## 개요

한국·일본 시니어를 위한 AI 보이스 동반자 서비스 "모두의 벗"의 투자자 대상 랜딩 페이지.
이 페이지는 "읽히는 것"이 아니라 **"느껴지는 것"**이다.
정보 전달이 아니라 감성적 인상 형성이 목적이다.
투자자 의사결정 흐름: 감성적 관심(이 페이지) → 논리적 검증(피치 덱) → 미팅.

---

## 기술 스택 + 아키텍처

### 기본
- 단일 index.html (CSS + JS 인라인)
- GSAP + ScrollTrigger (CDN)
- Lenis v1.3.17+ (CDN) — Cosmos 동일 버전

### Cosmos 아키텍처 (현재 구현 완료, 유지)

```
html, body { overflow: hidden }
#scroll-wrapper (fixed, inset:0, 100dvh, overflow-y:auto, overscroll-behavior:none)
  └── #scroll-content
       ├── section#hero
       ├── section#problem-1
       ├── section#problem-2
       ├── section#solution-intro
       ├── section#solution-demo
       ├── section#solution-bridge
       ├── section#kick
       └── section#ending
```

- Lenis가 #scroll-wrapper의 scrollTop을 제어
- ScrollTrigger는 scroller: wrapper를 통해 스크롤 위치를 읽음
- sectionSnap: 휠 누적 delta가 threshold를 넘으면 lenis.scrollTo()로 다음 섹션 offsetTop에 착지
- 배경: .bg-stage (fixed, z:0)에서 Terracotta→dark crossfade. 섹션들은 background: transparent.

### Lenis 설정 (Cosmos 일치)
```javascript
{ lerp: 0.1, smoothWheel: true, smoothTouch: false, wheelMultiplier: 1, touchMultiplier: 2 }
```

### 애니메이션 패턴
- **섹션 간 전환**: scroll-driven (scrub: true). 퇴장 텍스트는 opacity + translateY(-80px)로 위로 올라가며 사라짐. 등장 텍스트는 같은 중앙에서 opacity만으로 출현 (중앙 앵커 비대칭 패턴).
- **섹션 내 연출**: whileInView time-based. 섹션 진입 시 자동 재생, 역스크롤 시 reverse.
- **"1,000만"**: whileInView + onLeaveBack에 gsap.to(duration:0.3) 역재생. gsap.set 즉시 소멸 금지.

---

## 디자인 시스템

### 톤
"기술적이면서 인간적". Stripe의 톤을 참고.
AI 기술 기업의 세련됨 + 소셜 벤처의 따뜻함이 공존해야 한다.
에디토리얼이 아니다. 테크 쪽에 60~70% 무게를 둔다.

### 컬러
```
--terracotta:       #D1684E    ← 키 컬러. 히어로/엔딩 배경, 디바이스 화면 배경.
--terracotta-light: #E57960    ← 히어로 gradient 밝은 쪽.
--dark:             #1C1C1E    ← 섹션 2~7 배경, 디바이스 프레임.
--white:            #FFFFFF    ← 텍스트(다크/Terracotta 배경 위), 파동 연출.
--text-on-dark-sub: #B8A99E    ← 보조 텍스트(다크 배경 위).
--text-sub:         #7A6A62    ← 보조 텍스트(필요 시 검토).

```
Terracotta 사용처: 히어로 배경, 엔딩 배경, 디바이스 목업 화면 내부. 그 외에는 사용하지 않는다.

### 배경 흐름 (v4 변경)
```
Terracotta → 다크 → Terracotta
```
내부 섹션(문제, 해법, 킥)은 **전부 다크 배경으로 통일**. 섹션 구분은 콘텐츠 전환으로만 체감.
bg-stage에서 Terracotta→dark crossfade (히어로→섹션2 전환).
엔딩에서 dark→Terracotta 복귀 (bg-stage에 Terracotta 오버레이 추가 또는 섹션 자체 배경).

### 타이포그래피
```
히어로 "모두의 벗"        : Noto Serif KR, 300(light), 추적 0.06em
엔딩 브랜드 문장           : Noto Serif KR, 300(light)
대화 스니펫 (섹션 5)       : Noto Serif KR, 300, 15~16px — "서비스의 목소리" 위계
                            가독성 문제 시 재검토 (브라우저에서 판단)
그 외 모든 텍스트          : Pretendard (weight 300~800, size로 위계)
```

### 레이아웃
- **풀스크린 섹션 기반**. 각 섹션이 최소 100vh.
- **텍스트 영역만 max-width 제한** (약 ~900px). 배경과 시각 요소는 화면 전체를 사용.
- **뷰포트 단위: dvh 사용** (모바일 주소창 대응).

---

## 섹션별 명세 (8섹션)

### 섹션 1 — 오프닝 / 히어로 (section#hero)

**배경**: Terracotta. radial-gradient(중앙 밝음).
**구성**: 서비스명 "모두의 벗" + 파동 연출 + "한국, 일본 시니어를 위한 AI 보이스 동반자"
**파동**: samantha_02.svg의 파동 부분 (디바이스 프레임 없이). 히어로 전용.
**등장 시퀀스**: Terracotta 배경 → 서비스명 fade-up → 설명 letter-spacing 수렴 → 파동 materialize.
**로드 중 스크롤 차단**: lenis.stop() → heroTL 완료 후 lenis.start() + setupScrollAnimations().

### 섹션 2 — 문제 #1 (section#problem-1)

**배경**: 다크 (bg-stage crossfade로 전환).
**카피**:
- 메인: "1,000만이 넘는 어르신이 혼자입니다."
- 보조: "시니어 인구 한국 1,084만, 일본 3,625만."
**모션**: 메인은 중앙 앵커 패턴으로 opacity fade in. "1,000만"은 whileInView time-based 극적 등장(scale 0.95→1, duration 0.8s).

### 섹션 3 — 문제 #2 (section#problem-2)

**배경**: 다크 유지.
**카피**:
- 메인: "일찍 일어나 하루를 바쁘게 보내도, 적적한 마음은 가시지 않습니다."
- 보조: "먼저 연락을 하기도, 받기도 어려운 일상의 반복."
**모션**: 중앙 텍스트 fade in. 여백이 감정을 만든다.

### 섹션 4 — 해법 #1 (section#solution-intro)

**배경**: 다크 유지.
**카피**:
- 메인: "만약 누군가 먼저 말을 걸어준다면,"
- 보조: "나를 기억하고, 언제 어디서든 늘 함께한다면..."
**모션**: 중앙 텍스트 fade in. 기대감을 만드는 여백.

### 섹션 5 — 해법 #2 / 목업 데모 (section#solution-demo)

**이 섹션이 전체 랜딩 페이지의 핵심.**

**배경**: 다크 유지.
**구성**: 중앙에 samantha_02.svg 디바이스 목업 + 주변에 대화 스니펫이 나타났다 사라짐.
**레퍼런스**: cosmos.so 섹션 3 ("Image by image") — 중앙 텍스트 고정 + 주변에 이미지 카드가 방사형으로 분포.

**디바이스 목업**:
- samantha_02.svg 기반. 앱 실행 시퀀스(홈 화면 → 아이콘 확장 → 사만다 구동)를 포함.
- SVG 내장 애니메이션의 시작 시점을 섹션 진입에 동기화.
- 디바이스 크기: clamp(300px, 35vw, 500px). 뷰포트 너비에 비례하여 자연스럽게 축소, 상하한 고정. 종횡비 440:920. 내부 요소는 calc(--device-h * N / 920) 비례 계산.

**대화 스니펫**:
- 형태: 꼬리 없는 미니멀 버블 (둥근 모서리 + 극미세한 반투명 배경 rgba white 0.05~0.08). 카카오톡 스타일 아님. iMessage/ChatGPT 스타일의 절제된 블록. 대화임을 즉각 인식 + 덩어리 구별 가능.
- 서체: Noto Serif KR 300, 15~16px. (가독성 문제 시 Pretendard로 대체)
- 한 줄 최대: 20자 (한국어). 최대 2줄.
- 배치: 디바이스 중심으로 13존 방사형 분포. 상단 5 / 중단 4 / 하단 4. 
  Cosmos처럼 뷰포트 사분면에 고르게, 약간의 비대칭.
  디바이스 좌우 근접 존은 디바이스 에지(50vw ± device-width/2) 기준
  고정 간격(30px)으로 JS 추적. 뷰포트와 디바이스 크기 변화에 자동 대응.
- 총 스니펫 수: 13개. time-based로 각 존에 fade in → 유지 → fade out 순환.
  동시 표시 3~4개.
- 스케일링: 스니펫은 고정 px. 뷰포트 변화 시 간격만 조정. 
  모바일(768px 이하)에서 폰트/패딩 한 단계 축소.
  디바이스는 clamp(300px, 35vw, 500px). 뷰포트 너비 비례. 모바일 별도 breakpoint 없이 clamp 연속 적용.

**대화 스니펫 내용 (확정, 13개):**
선제적 대화:
1. "오늘 날씨가 꽤 쌀쌀하네요. 외출 계획 있으세요?" (2줄)
2. "오늘 병원 가시는 날이죠?" (1줄)
3. "점심은 드셨어요?" (1줄)
4. "좋아하시는 프로그램 곧 시작해요." (1줄)

맥락 기억:
5. "무릎은 좀 나아지셨어요? 저번에 많이 아프셨잖아요." (2줄)
6. "손주 시험 잘 봤다면서요?" (1줄)
7. "지난번에 좋아하신다던 그 노래, 찾아두었어요." (2줄)

관계 진화:
8. "요즘 기운이 없어 보이세요. 무슨 일 있으신 건 아니죠?" (2줄)
9. "그럴 수 있어요. 너무 속상해 마세요." (1줄)
10. "오늘따라 목소리가 한결 밝으시네요." (1줄)

일상 동반:
11. "오후 2시쯤에는 좀 따뜻해질 거예요." (1줄)
12. "내일 비 온대요. 빨래는 오늘 하시는 게 좋겠어요." (2줄)
13. "어제 산책은 즐거우셨어요?" (1줄)

**작동 메커니즘** (Cosmos S3 동일 — whileInView time-based):
- 섹션 높이 100dvh. PIN 없음. sectionSnap이 섹션 체류를 보장.
- 디바이스는 CSS로 뷰포트 중앙 배치 (다른 섹션의 중앙 텍스트와 동일).
- 섹션 진입(whileInView onEnter) → 단일 GSAP timeline 자동 재생:
  1. 앱 실행 시퀀스 (~3.3s): 홈 화면 → 아이콘 확장 → 사만다 구동
  2. 스니펫 순환 (~10~12s, 브라우저 테스트 후 조정): stagger로 각 존에 fade in → 유지 → fade out
  3. 두 시퀀스는 하나의 timeline 내에서 순차 배치 (독립 관리, 동일 timeline)
- 역스크롤(onLeaveBack) → timeline pause(0) + reset. 재진입 시 restart.
- 스크롤 progress와 무관. Cosmos S3과 동일하게 시간 기반으로만 동작.

### 섹션 6 — 해법 #3 / 기술 브릿지 (section#solution-bridge)

**배경**: 다크 유지.
**구성**: 3열 인포그래픽 (Linear 스타일).

| 항목 | 제목 | 설명 |
|------|------|------|
| 1 | 선제적 대화 | 생활 패턴을 학습하여 먼저 말을 거는 AI |
| 2 | 사실적인 맥락 | 현실 정보와 사건까지 담은 복합 기억 |
| 3 | 진화하는 관계 | 대화가 쌓일수록 깊어지는 역할, 그리고 관계 |

- 각 항목: SVG 아이콘 + 제목 + 1줄 설명.
- whileInView로 stagger 등장.
- 밀도: Linear 수준. 과도한 상세 금지.

### 섹션 7 — 킥 (section#kick)

**배경**: 다크 유지.
**카피**: "20여년간 게임에서 사람이 떠나지 않는 구조를 설계했습니다. 이제 그 설계를 사회에서 가장 필요한 곳에 씁니다."
**모션**: 느린 fade. 여백이 말하게 한다. 시각 요소 없음.

### 섹션 8 — 엔딩 (section#ending)

**배경**: Terracotta 복귀. 히어로와 수미상관.

**구성요소 (위→아래)**:
1. "모두의 벗이 그리는 앞날이 담긴 자료를 보내드립니다." — 안내 텍스트.
2. "연락하기" 버튼 → 클릭 시 폼 표시.
3. 폼 (Web3Forms AJAX, 인라인 제출):
   - 이름 (필수)
   - 소속 (필수)
   - 이메일 (필수)
   - 연락처 (선택)
   - 메모 (선택, textarea 자유 입력)
   - botcheck honeypot (hidden, UI 노출 없음)
   - 제출 후 인라인 성공 메시지. 페이지 이동 없음. 외부 서비스 브랜딩 노출 없음.
4. "모두에게 열려 있지만, 당신에게만 깊어지는 벗" — Noto Serif KR, light. 브랜드 문장.
5. "(주)디지털가든" — Pretendard 400, 작은 사이즈.

**파동 연출**: 히어로와 동일 파동을 배경에 깔되, opacity 낮춤.

---

## 모션 원칙

1. 섹션마다 다른 모션 언어. 전부 fade-in + translateY 금지.
2. 섹션 간 전환: 중앙 앵커 비대칭 (퇴장=올라가며 fade, 등장=제자리 fade).
3. 스크롤 연동(scrub: true)은 히어로 퇴장 + 배경 전환에만 사용. 나머지 모든 섹션 내 연출은 whileInView time-based.
4. 파동 연출: CSS @keyframes 또는 SVG animate. GSAP이 아닌 자체 루프.
5. sectionSnap: 휠 누적 delta 기반 섹션 점프. 현재 구현 유지.

---

## 반응형

- **데스크탑 우선**. 투자자는 주로 데스크탑/노트북에서 확인.
- **모바일 대응 필수**. 디바이스 목업 크기, 텍스트 사이즈, 레이아웃 조정.
- **디바이스 목업**: 모바일에서 크기 축소, 스니펫 배치 조정.

---

## 접근성

- `prefers-reduced-motion` 지원. 모션 비활성화 시 모든 콘텐츠가 즉시 표시.
- Lenis는 중지하지 않음 (normal flow이므로 자연 스크롤 필요).
- 시맨틱 HTML. 각 섹션에 `aria-label`.
- 색상 대비: WCAG AA 이상.

---

## 참조 에셋

| 파일 | 역할 |
|------|------|
| _refs/samantha_02.svg | 디바이스 목업 시각 기준 (v4에서 변경). 앱 실행 시퀀스 포함. |
| _refs/references.md | 레퍼런스 URL + 참고 포인트 |
| cosmos.so | 핵심 UX 레퍼런스. 특히 섹션 3 "Image by image"의 배치 패턴. |

---

## 폐기 사항 (절대 참조 금지)

- 이전 SPEC의 라이트 배경(#F5F2EE) 섹션 — 삭제. 내부 섹션은 전부 다크.
- 이전 SPEC의 장면 A/B 대화 시퀀스 — 삭제. Cosmos S3 스타일(중앙 목업 + 주변 스니펫)로 대체.
- 텍스트 말풍선(카카오톡 스타일) — 이전 폐기 결정은 재검토됨. 미니멀 버블(꼬리 없음)은 허용.
- samantha.svg (v1) — samantha_02.svg로 대체.
- ScrollTrigger PIN + scroll progress 기반 스니펫 제어 — Cosmos S3 실제 동작과 불일치. whileInView time-based로 확정.