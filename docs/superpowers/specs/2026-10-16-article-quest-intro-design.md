# Article Quest — Intro Animation 스펙

## 개요

Article Quest 게임 시작 전, 🍫 초코 캐릭터가 a/an/the/some 개념을 설명하는 4슬라이드 인트로 애니메이션 추가.

## 트리거

홈 화면 "Start Quest 🚀" 클릭 → `renderIntro()` → 4슬라이드 완료 또는 Skip → `initQueues()` + `nextTurn()`

## 슬라이드 구성

| # | 주제 | 초코 대사 (TTS 재생) | 예문 카드 |
|---|------|----------------------|-----------|
| 1 | a / an 소개 | "Hi! I'm Choco! Let's learn articles! Use a or an before one thing." | a 🐱 cat / an 🍎 apple |
| 2 | a vs an 규칙 | "Use a before a consonant sound. Use an before a vowel sound!" | a 🚲 bicycle / an 🥚 egg |
| 3 | the / some | "Only one in the world? Say the! Can't count it? Say some!" | the ☀️ sun / some 🥛 milk |
| 4 | 복습 요약 | "a, an, the, some — you've got this! Ready? Let's go!" | 4개 article 한눈에 요약 카드 |

## UI 구성 (슬라이드 공통)

- **진행 바**: 슬라이드별 25% / 50% / 75% / 100%
- **점 인디케이터**: ●○○○ 하단 배치
- **초코 캐릭터**: 🍫 이모지 + 바운스 애니메이션 + 금색 말풍선
- **Skip 버튼**: 우하단, 클릭 시 즉시 게임 시작
- **음소거 버튼**: 우상단 🔊/🔇, 토글

## 자동 진행 로직

1. 슬라이드 렌더링과 동시에 TTS 재생 시작
2. TTS 종료 후 1초 대기 → 다음 슬라이드로 자동 이동
3. TTS가 지원되지 않는 환경에서는 3.5초 고정 타이머로 폴백

## 음성 (Web Speech API)

- `SpeechSynthesisUtterance` 사용, 외부 의존성 없음
- `lang: 'en-US'`, `rate: 0.9`, `pitch: 1.2` (밝고 또렷한 톤)
- 음소거 상태는 `sessionStorage`에 저장 (페이지 새로고침 시 초기화)

## 구현 범위

- `article-quest.html` 내 `renderIntro()` 함수 추가
- `renderHome()`의 Start 버튼 핸들러에서 `initQueues()` 대신 `renderIntro()` 호출
- 기존 함수(`renderSort`, `renderDrill` 등) 변경 없음

## 범위 외

- localStorage로 "인트로 본 적 있음" 스킵 로직 (요청되지 않음)
- 자체 음성 파일 사용 (Web Speech API로 충분)
- 슬라이드 애니메이션 커스터마이즈 (기본 fade-in으로 충분)
