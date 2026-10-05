# Grammar Quiz 앱 설계 스펙

**날짜:** 2026-10-05  
**파일:** `english/grammar-quiz.html`  
**대상:** 8살 여자아이

---

## 개요

영어 문법 복습 시험을 재미있게 풀 수 있는 단일 HTML 퀴즈 앱.  
섹션 8개를 순서대로 진행하고, 문제마다 즉시 피드백. 마지막에 전체 성적표 제공.

---

## 화면 흐름

```
홈 → 섹션 인트로 → 문제(즉시 피드백) → 섹션 완료 → (반복) → 최종 성적표
```

### 1. 홈 화면
- 마스코트 🌟 + 말풍선 ("안녕! 같이 문법 시험 연습해보자! ✨")
- 앱 제목: **Grammar Quiz** / 부제: 영어 문법 도전
- "시험 시작! 🚀" 버튼 (전체 75문제)

### 2. 섹션 인트로 화면
- 섹션 이모지 + 이름 + 문제 수
- 응원 메시지 한 줄 ("전치사는 어디에 있는지 알려주는 단어야! 💪")
- "시작!" 버튼

### 3. 문제 화면
- 상단: 전체 진행 바 (0~75) + 현재 섹션 레이블
- 문제 카드: 힌트 텍스트 + 문제 문장 (`___` 강조 표시)
- 선택지 버튼 (유형별 레이아웃 상이)
- 즉시 피드백: 정답 시 ✨ 칭찬 / 오답 시 💡 정답 표시
- "다음 →" 버튼 (피드백 후 나타남)

### 4. 섹션 완료 화면
- 섹션 점수 (예: 7/7 ⭐⭐⭐)
- 격려 메시지
- "다음 섹션 →" 버튼

### 5. 최종 성적표
- 전체 점수 (예: 68/75)
- 섹션별 결과 리스트
- 최종 격려 메시지 + 귀여운 이모지 폭탄

---

## 섹션 구성 (75문제)

| # | 섹션 | 이모지 | 문제 수 | 선택지 형태 |
|---|---|---|---|---|
| 1 | 전치사 — 장소 | 📍 | 7 | 단어 4개 |
| 2 | 전치사 — 시간 | ⏰ | 10 | in / on / at 3개 대형 |
| 3 | 대문자 | 🔠 | 8 | 문장 3개 택1 |
| 4 | 문장부호 | ❓ | 10 | . ! ? 특대형 |
| 5 | 쉼표 | ✏️ | 5 | 문장 3개 택1 |
| 6 | 축약형 | ✂️ | 12 | 단어 4개 |
| 7 | 인용부호 | 💬 | 8 | 문장 3개 택1 |
| 8 | 접두사/접미사 | 🔬 | 15 | 단어 4개 |

---

## 문제 데이터

### Section 1 — 전치사 장소
Word bank: next to, in, above, between, behind, under, on

1. The dog sleeps ___ the bed. → **under**
2. The cup is ___ the plate. → **on**
3. The girl is ___ the park. → **in**
4. The ball flew ___ the boy. → **above**
5. The cat is ___ the wall. → **behind**
6. The fridge is ___ the oven. → **next to**
7. The school is ___ the bank and the hospital. → **between**

### Section 2 — 전치사 시간 (in/on/at)
1. The class starts ___ Monday. → **on**
2. We have a meeting ___ 3:45 PM. → **at**
3. She was born ___ July. → **in**
4. The train arrives ___ platform 5. → **at**
5. The flowers bloom ___ spring. → **in**
6. The store opens ___ 8:00 AM. → **at**
7. We'll have dinner ___ the restaurant. → **at**
8. The event is ___ the weekend. → **on**
9. The cat sleeps ___ the bed. → **on**
10. We play games ___ the evening. → **in**

### Section 3 — 대문자 (3지선다: 올바른 문장 고르기)
1. alice enjoys playing → **Alice enjoys playing with her pet rabbit.**
2. namsan tower in seoul → **Next summer, we plan to visit Namsan Tower in Seoul.**
3. aunt jane's → **We went to my aunt Jane's house for dinner.**
4. monday and thursday → **We have science class every Monday and Thursday.**
5. harry potter → **"Harry Potter and the Philosopher's Stone" is a fantastic book.**
6. my cousin tom → **My cousin Tom lives in London.**
7. the cat walked → **The cat walked over the road.**
8. everland on saturday → **I went to Everland on Saturday.**

### Section 4 — 문장부호 (. ! ?)
1. Aaaargh, a monster___ → **!**
2. Why are you leaving so early___ → **?**
3. I went to the shops yesterday___ → **.**
4. Did you remember to buy milk___ → **?**
5. Wow___ This is very exciting. → **!**
6. I need to study tonight for the test___ → **.**
7. That's amazing news___ good job. → **!**
8. Where did you put the orange juice___ → **?**
9. I think I need to go to school early tomorrow___ → **.**
10. When is the flight going to leave___ → **?**

### Section 5 — 쉼표 (3지선다)
1. sleeping eating and → **Cats enjoy sleeping, eating, and playing with string.**
2. museum but → **I wanted to go to the museum, but it's closed today.**
3. apples bananas → **My favorite fruits are apples, bananas, and oranges.**
4. hour so → **The concert starts in an hour, so don't be late.**
5. books so → **She likes reading books, so she reads lots of them.**

### Section 6 — 축약형 (4지선다)
They are→They're, She is→She's, You are→You're, Did not→Didn't,
Cannot→Can't, Is not→Isn't, Does not→Doesn't, It is→It's,
We are→We're, Do not→Don't, He is→He's, I am→I'm

### Section 7 — 인용부호 (3지선다)
1. Can we have pizza for dinner? Jack asked. → **"Can we have pizza for dinner?" Jack asked.**
2. Where is my backpack? questioned Lily. → **"Where is my backpack?" questioned Lily.**
3. Pass me the remote, asked Dad → **"Pass me the remote," asked Dad during the movie.**
4. I love playing video games, said Jake. → **"I love playing video games," said Jake.**
5. My favorite book is the Wizard of Oz. → **My favorite book is "the Wizard of Oz."**
6. Did you finish your homework? asked the teacher. → **"Did you finish your homework?" asked the teacher.**
7. I can't find my keys anywhere! shouted Sarah. → **"I can't find my keys anywhere!" shouted Sarah.**
8. Meet me at the park at 4 PM, Tom said. → **"Meet me at the park at 4 PM," Tom said.**

### Section 8 — 접두사/접미사
매칭 (9문제):
- inter- → between or among
- over- → over or above
- sub- → under or below
- un- → not or opposite of
- re- → again or back
- -ful → full of
- -ic → of or related to
- -er/-or → a person doing the action
- -ly → turns adjective into adverb

문장 채우기 (6문제):
- chaos + -ic → chaotic
- over + paid → overpaid
- inter + national → international
- bake + -er → baker
- poison + -ous → poisonous
- use + -ful → useful

---

## 비주얼 / UX

### 테마 (8살 여아 대상)
- **배경:** 연한 라벤더/하늘색 (`#f0f4ff`)
- **포인트 컬러:** 인디고+보라 (`#6366f1`) + 골드 (`#f59e0b`)
- **강조:** 핑크 액센트 (`#ec4899`) — 귀여운 느낌 추가
- **폰트:** Jua (제목) + Gaegu (UI) + Baloo 2 (문제)
- **마스코트:** 🌟 별 or 🦋 나비 (귀여움 우선)
- **이모지 피드백:** 정답 "🌟 정답이에요!", 오답 "💡 정답은:"

### 버튼 크기
- 문장부호 버튼 (. ! ?): 특대형, 패딩 넉넉히
- 단어 버튼: 큰 텍스트
- 문장 버튼: 보통 크기, 줄바꿈 허용

### 애니메이션 (CSS only)
- 정답 시 버튼 초록 flash
- 오답 시 버튼 빨간 shake
- 섹션 완료 시 별 이모지 팡파르

---

## 기술 스펙

- 단일 HTML 파일 (의존성 없음, Google Fonts만)
- 바닐라 JS IIFE 패턴 (vocab-green과 동일)
- localStorage 없음 (시험 세션은 페이지 메모리만)
- 모바일 우선 반응형

---

## 파일 위치

```
english/grammar-quiz.html
```
