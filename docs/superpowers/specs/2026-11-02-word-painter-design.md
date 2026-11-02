# Word Painter — Design Spec

**Date:** 2026-11-02  
**File:** `english/word-painter.html`  
**Target user:** 8세 아이, noun/verb/adjective/adverb 개념 입문

---

## 목적

문장에서 품사(noun, verb, adjective, adverb)를 찾아 색깔로 칠하는 인터랙티브 연습 앱.  
설명 슬라이드 없이 바로 문제로 시작하고, 틀렸을 때만 8세 눈높이 힌트를 보여준다.

---

## 앱 흐름

```
스플래시 → 퀴즈 (×20) → 결과
```

### 1. 스플래시 화면
- 제목: 🎨 Word Painter
- 곰 캐릭터 + 말풍선: "단어를 색깔로 칠해봐!"
- "시작!" 버튼

### 2. 퀴즈 화면 (20문제)
- 상단: 진행 바 + "n / 20" 표시
- 곰 캐릭터 말풍선 (문맥별 멘트)
- 문장 카드: 단어들이 칩(chip) 형태로 표시
  - **skip 단어** (the, a, an, is, are): 회색, 탭 불가, 처음부터 회색 처리
  - **미칠한 단어**: 연한 파란 배경 (아직 안 골랐음을 표시)
  - **칠한 단어**: 해당 품사 색
- 하단 고정 팔레트: Noun / Verb / Adjective / Adverb
  - 선택된 색: scale up + shadow 강조
- "확인!" 버튼

**정답 처리:**
- 전부 맞으면: 단어들 pop 애니메이션 + "🎉 완벽해!" → 1.2초 후 다음 문제
- 틀린 단어: shake 애니메이션 + 카드 하단에 힌트 박스 등장
  - 힌트 확인 후 수정 → "확인!" 재시도 가능
  - 두 번째 시도는 틀려도 "정답은 __ 야!" 로 알려주고 다음 문제로 넘어감

**점수 집계:** 문장당 첫 번째 시도에서 모두 맞추면 +1점 (만점 20점)

### 3. 결과 화면
- 별점: 18~20점=★★★, 14~17점=★★☆, ~13점=★☆☆
- 틀린 단어 복습 카드 (단어 + 품사 + 힌트 1줄)
- "다시 도전!" 버튼

---

## 품사 색상 시스템

| 품사 | 영어 | 칩 배경 | 테두리 | 팔레트 점 |
|------|------|---------|--------|----------|
| 명사 | Noun | #BBDEFB | #1565C0 | #1565C0 |
| 동사 | Verb | #FFCDD2 | #C62828 | #C62828 |
| 형용사 | Adjective | #DCEDC8 | #33691E | #33691E |
| 부사 | Adverb | #FFF9C4 | #F57F17 | #F57F17 |
| skip | — | #F1F5F9 | #CBD5E1 | — |

---

## 힌트 문구 (틀렸을 때, 8세 눈높이)

- **Noun 힌트:** "사람, 동물, 물건의 이름이야 → 🟦 Noun!"
- **Verb 힌트:** "무엇을 하는지 나타내는 단어야 → 🟥 Verb!"
- **Adjective 힌트:** "명사 옆에 붙어서 어떤지 설명해줘 → 🟩 Adjective!"
- **Adverb 힌트:** "동사 옆에서 어떻게 하는지 말해줘 → 🟨 Adverb!"

각 단어별로 구체적인 힌트 추가 가능 (예: `"fast"` → "runs가 **어떻게** 일어나는지 말해줘 → 🟨 Adverb!")

---

## 20문제 문장 목록

| # | 문장 | 주요 품사 |
|---|------|----------|
| 1 | Dogs run fast. | noun, verb, adverb |
| 2 | The cat sleeps quietly. | noun, verb, adverb |
| 3 | She eats lunch. | noun, verb, noun |
| 4 | The red ball bounces. | adj, noun, verb |
| 5 | He runs quickly. | noun, verb, adverb |
| 6 | Birds sing sweetly. | noun, verb, adverb |
| 7 | The big dog barks. | adj, noun, verb |
| 8 | She reads books daily. | noun, verb, noun, adverb |
| 9 | The happy child laughs. | adj, noun, verb |
| 10 | He jumps high. | noun, verb, adverb |
| 11 | The tiny mouse hides. | adj, noun, verb |
| 12 | She writes neatly. | noun, verb, adverb |
| 13 | The cold wind blows. | adj, noun, verb |
| 14 | He talks loudly. | noun, verb, adverb |
| 15 | The brave lion roars. | adj, noun, verb |
| 16 | She dances beautifully. | noun, verb, adverb |
| 17 | The tall tree stands firmly. | adj, noun, verb, adverb |
| 18 | He eats slowly. | noun, verb, adverb |
| 19 | The yellow sun shines brightly. | adj, noun, verb, adverb |
| 20 | She sings a beautiful song softly. | noun, verb, adj, noun, adverb |

---

## 디자인 토큰

- **배경:** `linear-gradient(160deg, #38BDF8, #818CF8)` (Sky+Lavender)
- **캐릭터:** SVG 곰 (inline)
- **폰트:** Jua (한글/타이틀), Baloo 2 (본문)
- **단어 칩 글씨:** 26px, Jua
- **팔레트 버튼 글씨:** 17px, Jua
- **확인 버튼:** 28px, Jua
- **카드:** white, border 2.5px #1a1a1a, border-radius 22px, box-shadow 0 5px 0 rgba(0,0,0,.25)

---

## 기술 스택

- 단일 HTML 파일 (`english/word-painter.html`)
- 외부 의존: Google Fonts (Jua, Baloo 2)
- 프레임워크 없음, Vanilla JS
- 기존 grammar-quest.html 스타일 패턴 준수
