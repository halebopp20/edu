# Article Quest Intro Animation 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** article-quest.html에 초코 캐릭터(🍫)가 a/an/the/some을 음성으로 설명하는 4슬라이드 인트로 추가

**Architecture:** `renderIntro()` 함수를 article-quest.html에 추가. INTRO_SLIDES 데이터 배열로 슬라이드 정의. Web Speech API로 TTS, TTS 종료 후 자동 다음 슬라이드 진행 (폴백: 3.5초 타이머). Skip 버튼으로 즉시 게임 시작.

**Tech Stack:** Vanilla JS, Web Speech API (SpeechSynthesis), CSS animation

---

## 파일 구조

- **Modify:** `english/article-quest.html`
  - `<style>` 블록에 인트로 CSS 추가 (fade-in, bounce)
  - `<script>` 블록에 `INTRO_SLIDES` 상수 + `renderIntro()` + `speakLine()` 함수 추가
  - `renderHome()` 내 Start 버튼 핸들러 수정: `initQueues(); nextTurn()` → `renderIntro()`

---

### Task 1: INTRO_SLIDES 데이터 + CSS 추가

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: `<style>` 블록 닫는 `</style>` 바로 앞에 인트로 CSS 추가**

```css
  /* 인트로 슬라이드 */
  @keyframes introFadeIn {
    from { opacity: 0; transform: translateY(12px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .intro-card {
    animation: introFadeIn 0.4s ease forwards;
  }
  @keyframes chocoFloat {
    0%, 100% { transform: translateY(0); }
    50%      { transform: translateY(-10px); }
  }
  .choco-float {
    display: inline-block;
    animation: chocoFloat 1.4s ease-in-out infinite;
  }
```

- [ ] **Step 2: `WORDS` 배열 선언 바로 위에 `INTRO_SLIDES` 상수 추가**

```js
  var INTRO_SLIDES = [
    {
      lesson: 'LESSON 1',
      speech: "Hi! I'm Choco! Let's learn articles! Use a or an before one thing.",
      bubbleHtml: "Hi! I'm <strong>Choco</strong>! 🍫<br>Let's learn <strong>articles</strong>!",
      cards: [
        { article: 'a',  emoji: '🐱', word: 'cat' },
        { article: 'an', emoji: '🍎', word: 'apple' },
      ],
    },
    {
      lesson: 'LESSON 2',
      speech: "Use a before a consonant sound. Use an before a vowel sound!",
      bubbleHtml: "<strong>a</strong> + consonant sound 🔤<br><strong>an</strong> + vowel sound 🅰️",
      cards: [
        { article: 'a',  emoji: '🚲', word: 'bicycle' },
        { article: 'an', emoji: '🥚', word: 'egg' },
      ],
    },
    {
      lesson: 'LESSON 3',
      speech: "Only one in the world? Say the! Can't count it? Say some!",
      bubbleHtml: "딱 하나뿐 → <strong>the</strong> 🌍<br>셀 수 없으면 → <strong>some</strong> 💧",
      cards: [
        { article: 'the',  emoji: '☀️', word: 'sun' },
        { article: 'some', emoji: '🥛', word: 'milk' },
      ],
    },
    {
      lesson: 'LESSON 4',
      speech: "a, an, the, some — you've got this! Ready? Let's go!",
      bubbleHtml: "a / an / the / some<br>— you've got this! 🚀",
      cards: [
        { article: 'a',    emoji: '📚', word: 'book' },
        { article: 'an',   emoji: '🥚', word: 'egg' },
        { article: 'the',  emoji: '🌙', word: 'moon' },
        { article: 'some', emoji: '🍞', word: 'bread' },
      ],
    },
  ];
```

- [ ] **Step 3: 브라우저에서 `english/article-quest.html` 열어 콘솔 오류 없는지 확인**

- [ ] **Step 4: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add INTRO_SLIDES data and intro CSS"
```

---

### Task 2: `speakLine()` + `renderIntro()` 함수 추가

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: `renderHome` 함수 선언 바로 위에 `speakLine()` 함수 추가**

```js
  function speakLine(text, onEnd) {
    if (!window.speechSynthesis) { onEnd(); return; }
    if (sessionStorage.getItem('aq-mute') === '1') { onEnd(); return; }
    window.speechSynthesis.cancel();
    var u = new SpeechSynthesisUtterance(text);
    u.lang  = 'en-US';
    u.rate  = 0.9;
    u.pitch = 1.2;
    u.onend = onEnd;
    u.onerror = onEnd;
    window.speechSynthesis.speak(u);
  }
```

- [ ] **Step 2: `speakLine` 바로 아래에 `renderIntro()` 함수 추가**

```js
  function renderIntro() {
    var slideIndex = 0;

    function showSlide(idx) {
      var slide    = INTRO_SLIDES[idx];
      var total    = INTRO_SLIDES.length;
      var pct      = Math.round(((idx + 1) / total) * 100);
      var isMuted  = sessionStorage.getItem('aq-mute') === '1';

      /* 점 인디케이터 HTML */
      var dotsHtml = INTRO_SLIDES.map(function(_, i) {
        return '<div class="dot' + (i === idx ? ' dot-active' : '') + '"></div>';
      }).join('');

      /* 예문 카드 HTML */
      var cardsHtml = slide.cards.map(function(c) {
        return '<div class="intro-ex-card">' +
          '<span class="intro-article">' + esc(c.article) + '</span>' +
          '<span class="intro-word">' + c.emoji + ' ' + esc(c.word) + '</span>' +
        '</div>';
      }).join('');

      screenEl.innerHTML =
        '<div class="intro-card card" style="margin-top:14px">' +
          /* 음소거 버튼 */
          '<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">' +
            '<span style="font-size:12px;font-weight:700;letter-spacing:2px;color:var(--cream2)">📖 ' + esc(slide.lesson) + '</span>' +
            '<button id="muteBtn" style="background:transparent;border:1px solid var(--gold2);color:var(--cream2);border-radius:10px;padding:4px 10px;font-size:16px;cursor:pointer">' +
              (isMuted ? '🔇' : '🔊') +
            '</button>' +
          '</div>' +
          /* 진행 바 */
          '<div style="background:var(--surface2);border-radius:999px;height:8px;overflow:hidden;border:1px solid var(--gold2);margin-bottom:20px">' +
            '<div style="height:100%;width:' + pct + '%;background:linear-gradient(90deg,var(--gold),var(--gold2));border-radius:999px;transition:width 0.4s ease"></div>' +
          '</div>' +
          /* 캐릭터 + 말풍선 */
          '<div style="display:flex;align-items:flex-start;gap:14px;margin-bottom:24px">' +
            '<div class="choco-float" style="font-size:56px;line-height:1;flex-shrink:0">🍫</div>' +
            '<div style="background:var(--gold);color:var(--bg);border-radius:18px 18px 18px 4px;padding:14px 16px;font-size:18px;font-weight:700;line-height:1.5;flex:1">' +
              slide.bubbleHtml +
            '</div>' +
          '</div>' +
          /* 예문 카드 */
          '<div style="display:flex;gap:10px;flex-wrap:wrap;justify-content:center;margin-bottom:24px">' +
            cardsHtml +
          '</div>' +
          /* 하단: 점 + Skip */
          '<div style="display:flex;justify-content:space-between;align-items:center">' +
            '<div style="display:flex;gap:6px">' + dotsHtml + '</div>' +
            '<button id="skipBtn" style="background:transparent;border:1px solid var(--gold2);color:var(--cream2);border-radius:12px;padding:8px 14px;font-size:14px;font-family:\'Baloo 2\',sans-serif;cursor:pointer">Skip ⏩</button>' +
          '</div>' +
        '</div>';

      /* 예문 카드 인라인 스타일 (동적 생성이므로 JS로 적용) */
      document.querySelectorAll('.intro-ex-card').forEach(function(el) {
        el.style.cssText = 'background:var(--surface2);border:2px solid var(--gold2);border-radius:14px;padding:10px 16px;text-align:center;font-size:18px;font-weight:700';
      });
      document.querySelectorAll('.intro-article').forEach(function(el) {
        el.style.cssText = 'color:var(--gold);font-size:22px;font-weight:900;display:block';
      });
      document.querySelectorAll('.intro-word').forEach(function(el) {
        el.style.cssText = 'color:var(--cream2)';
      });
      document.querySelectorAll('.dot').forEach(function(el) {
        el.style.cssText = 'width:8px;height:8px;border-radius:50%;background:var(--surface2);border:2px solid var(--gold2)';
      });
      document.querySelectorAll('.dot-active').forEach(function(el) {
        el.style.cssText = 'width:8px;height:8px;border-radius:50%;background:var(--gold);border:2px solid var(--gold)';
      });

      /* 이벤트 바인딩 */
      var skipped = false;

      function goNext() {
        if (skipped) return;
        skipped = true;
        window.speechSynthesis && window.speechSynthesis.cancel();
        if (idx + 1 < total) {
          showSlide(idx + 1);
        } else {
          initQueues();
          nextTurn();
        }
      }

      document.getElementById('skipBtn').addEventListener('click', function() {
        skipped = true;
        window.speechSynthesis && window.speechSynthesis.cancel();
        initQueues();
        nextTurn();
      });

      document.getElementById('muteBtn').addEventListener('click', function() {
        var nowMuted = sessionStorage.getItem('aq-mute') === '1';
        sessionStorage.setItem('aq-mute', nowMuted ? '0' : '1');
        window.speechSynthesis && window.speechSynthesis.cancel();
        showSlide(idx); /* 아이콘 갱신을 위해 재렌더 */
      });

      /* TTS 재생 → 종료 후 1초 뒤 자동 다음 슬라이드 */
      var fallbackTimer = setTimeout(function() { goNext(); }, 5000); /* 최대 폴백 */
      speakLine(slide.speech, function() {
        clearTimeout(fallbackTimer);
        setTimeout(goNext, 1000);
      });
    }

    showSlide(0);
  }
```

- [ ] **Step 3: 브라우저에서 열고 콘솔 오류 없는지 확인 (Start는 아직 누르지 않아도 됨)**

- [ ] **Step 4: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add renderIntro and speakLine functions"
```

---

### Task 3: Start 버튼을 `renderIntro()`로 연결

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: `renderHome()` 내 Start 버튼 클릭 핸들러 수정**

현재 코드 (`renderHome` 함수 내부):
```js
    document.getElementById('startBtn').addEventListener('click', function() {
      initQueues();
      nextTurn();
    });
```

변경 후:
```js
    document.getElementById('startBtn').addEventListener('click', function() {
      renderIntro();
    });
```

- [ ] **Step 2: 브라우저에서 수동 테스트**

1. `english/article-quest.html` 열기
2. "Start Quest 🚀" 클릭
3. 확인 항목:
   - 슬라이드 1이 fade-in과 함께 등장하는가
   - 🍫 이모지가 위아래로 float 애니메이션 하는가
   - TTS 음성이 재생되는가 (소리가 안 나면 브라우저 설정 확인)
   - TTS 종료 후 약 1초 뒤 슬라이드 2로 자동 이동하는가
   - 슬라이드 4 완료 후 게임(Sort turn)이 시작되는가
4. Skip ⏩ 클릭 시 즉시 게임으로 이동하는가
5. 🔊 → 🔇 토글 후 음성이 재생되지 않는가

- [ ] **Step 3: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: wire Start button to renderIntro"
```

---

## Self-Review

**스펙 커버리지 체크:**
- [x] 트리거: Start → renderIntro (Task 3)
- [x] 슬라이드 4개 / 대사 / 예문 카드 (Task 1, INTRO_SLIDES)
- [x] 진행 바 25/50/75/100% (Task 2, renderIntro)
- [x] 점 인디케이터 (Task 2)
- [x] 🍫 바운스 애니메이션 (Task 1 CSS + Task 2)
- [x] Skip 버튼 (Task 2)
- [x] 음소거 버튼 + sessionStorage (Task 2)
- [x] TTS 종료 후 1초 대기 자동 이동 (Task 2)
- [x] TTS 미지원 환경 폴백 타이머 (Task 2, 5초 fallbackTimer)
- [x] 기존 renderSort/renderDrill 변경 없음 (Task 3만 renderHome 수정)

**플레이스홀더 없음 확인:** 모든 스텝에 실제 코드 포함됨.

**타입 일관성:** `speakLine(text, onEnd)` → Task 2에서 정의, Task 2 내부에서만 사용. `INTRO_SLIDES` → Task 1에서 정의, Task 2에서 참조. 일관됨.
