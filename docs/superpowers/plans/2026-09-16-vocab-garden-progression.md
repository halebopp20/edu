# 초록 단어 정원 — 4단계 학습 & 별점 시스템 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 단어별 독립 4단계 학습 흐름 + 정원 별점 시각화 + 수료증/비밀코드 보상 시스템을 `english/vocab-green-unit1_2.html` 한 파일에 구현한다.

**Architecture:** 단일 HTML 파일(vanilla JS)을 수술적으로 수정한다. 새 localStorage 키(`vwgreen_progress_v3`)에 단어별 별점(0~4)을 저장하고, 별점이 곧 현재 단계(0→1단계, 1→2단계, …)를 결정한다. 세션은 미숙달 단어들을 섞어서 한 번씩 진행하고 틀린 단어는 세션 끝에 재시도한다.

**Tech Stack:** HTML/CSS/JS (no build tools), localStorage, Canvas API (기존 setupWritePad 재사용)

---

## 파일 구조

수정 대상: `english/vocab-green-unit1_2.html` 한 파일만.

변경 범위:
- `<style>` 블록: 정원 그리드, 힌트 박스, 스테이지 뱃지, 수료증 CSS 추가
- `<script>` 블록: 상수/스토리지/상태/함수 전면 재작성 (setupWritePad·esc·shuffle은 그대로 유지)

---

## Task 1: 새 상수·스토리지·헬퍼 추가

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — WORDS 배열 바로 아래 상수 블록

- [ ] **Step 1: GROWTH_STAGES 제거 + 새 상수 추가**

`GROWTH_STAGES` 줄과 `STORE_KEY` 줄을 다음으로 교체:

```javascript
var SECRET_CODE = "ROSE🌸";   // ← 엄마가 원하는 코드로 바꾸세요
var STORE_KEY = "vwgreen_progress_v3";
var FLOWER = ["🌱", "🌱", "🌿", "🌷", "🌸"];  // index = stars(0~4)
```

- [ ] **Step 2: loadStore / saveStore 교체**

기존 `loadStore` / `saveStore` 두 함수를 그대로 유지 (내용 변경 없음).  
바로 아래에 별점 헬퍼 두 개 추가:

```javascript
function getStars(wordStr) {
  var store = loadStore();
  return (store.words && store.words[wordStr] !== undefined)
    ? store.words[wordStr] : 0;
}
function setStars(wordStr, n) {
  var store = loadStore();
  if (!store.words) store.words = {};
  store.words[wordStr] = n;
  saveStore(store);
}
```

- [ ] **Step 3: getHint 헬퍼 추가 (3단계 앞-절반 힌트)**

```javascript
function getHint(word) {
  var half = Math.ceil(word.length / 2);
  var shown = word.slice(0, half);
  var blanks = word.slice(half).split("").map(function () { return "_"; }).join(" ");
  return shown + " " + blanks;
  // "force" → "fo _ _ _"   "aim" → "a _ _"
}
```

- [ ] **Step 4: 브라우저에서 콘솔 확인**

파일을 열고 콘솔에서:
```javascript
getStars("force")   // → 0
setStars("force", 2)
getStars("force")   // → 2
getHint("force")    // → "fo _ _ _"
getHint("aim")      // → "a _ _"
```
예상 결과가 나오면 OK.

- [ ] **Step 5: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: add star storage helpers and hint generator"
```

---

## Task 2: state 객체 + buildSessionQueue 교체

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — state 선언 및 pool() 함수

- [ ] **Step 1: state 객체 교체**

기존:
```javascript
var state = { selectedUnits: { 1: true, 2: true }, mode: "mc", questions: [], index: 0, score: 0, wrong: [], answered: false };
```
교체:
```javascript
var state = {
  queue: [],       // 현재 세션 메인 큐
  retryQueue: [],  // 🔁 누른 단어들 (세션 끝에 재시도)
  index: 0,
  answered: false
};
```

- [ ] **Step 2: pool() 함수 제거 + buildSessionQueue 추가**

`function pool()` 줄 전체를 삭제하고 아래로 교체:

```javascript
function buildSessionQueue() {
  return shuffle(WORDS.filter(function (w) { return getStars(w.word) < 4; }));
}
```

- [ ] **Step 3: 브라우저 콘솔 확인**

```javascript
buildSessionQueue().length  // → 20 (처음엔 모두 별 0개)
setStars("allow", 4)
buildSessionQueue().map(function(w){ return w.word; })
// "allow" 가 빠진 19개 배열이면 OK
```

- [ ] **Step 4: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: replace state with queue-based session model"
```

---

## Task 3: CSS 추가 (정원·힌트·뱃지·수료증)

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — `</style>` 바로 앞에 추가

- [ ] **Step 1: `</style>` 바로 앞에 다음 CSS 블록 삽입**

```css
  /* ── 정원 그리드 ── */
  .garden-grid {
    display: flex; flex-wrap: wrap; gap: 10px;
    padding: 18px; background: var(--surface-2);
    border-radius: 18px; margin-bottom: 14px;
  }
  .garden-flower { font-size: 40px; line-height: 1; cursor: default; }

  .garden-legend {
    display: flex; flex-wrap: wrap; gap: 16px;
    font-size: 20px; color: var(--ink-faint); margin-bottom: 24px;
  }

  /* ── 스테이지 뱃지 ── */
  .stage-badge {
    display: inline-block; font-size: 20px; font-weight: 700;
    padding: 8px 18px; border-radius: 999px; margin-bottom: 18px;
  }
  .stage-1 { background: var(--secondary-tint); color: var(--secondary-strong); }
  .stage-2 { background: #efe0ff; color: #7b2ff7; }
  .stage-3 { background: #fff0c2; color: #b07800; }
  .stage-4 { background: var(--accent-tint); color: var(--accent-strong); }

  /* ── 힌트 박스 ── */
  .hint-box {
    text-align: center; border-radius: 16px; padding: 14px 20px;
    font-family: "Baloo 2", sans-serif; font-size: 36px; font-weight: 700;
    letter-spacing: 3px; margin-bottom: 12px;
  }
  .hint-full  { background: #efe0ff; border: 3px dashed #c49eff; color: #7b2ff7; }
  .hint-partial { background: #fff0c2; border: 3px dashed #ffc93c; color: #b07800; }

  /* ── 수료증 ── */
  .cert-flowers { font-size: 36px; line-height: 1.7; text-align: center; margin-bottom: 16px; }
  .cert-title {
    font-family: "Jua", sans-serif; font-size: 52px; text-align: center;
    color: var(--ink); margin-bottom: 6px;
  }
  .cert-sub { font-size: 24px; color: var(--ink-soft); text-align: center; margin-bottom: 22px; }
  .cert-box {
    background: var(--sun); border-radius: 18px; padding: 18px 24px;
    font-size: 22px; font-weight: 700; color: #5a3a00; text-align: center;
    line-height: 1.6; margin-bottom: 22px;
    box-shadow: 0 4px 0 rgba(180,120,0,0.3);
  }
  .secret-box {
    background: var(--accent-tint); border: 3px dashed var(--accent);
    border-radius: 18px; padding: 22px; text-align: center; margin-bottom: 24px;
  }
  .secret-label { font-size: 22px; font-weight: 700; color: var(--accent-strong); margin-bottom: 10px; }
  .secret-code {
    font-family: "Baloo 2", sans-serif; font-size: 52px; font-weight: 900;
    color: var(--accent-strong); letter-spacing: 8px;
    background: #fff; padding: 12px 24px; border-radius: 14px;
    display: inline-block; margin-bottom: 12px;
  }
  .secret-hint { font-size: 22px; color: var(--ink-soft); line-height: 1.6; }
```

- [ ] **Step 2: 브라우저에서 오류 없이 페이지 로드되는지 확인**

DevTools Console에 CSS 파싱 오류 없으면 OK.

- [ ] **Step 3: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: add CSS for garden grid, hint boxes, stage badges, certificate"
```

---

## Task 4: renderStart() — 정원 홈 화면으로 교체

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — `renderStart` 함수 전체

- [ ] **Step 1: renderStart 함수 전체 교체**

기존 `function renderStart() { ... }` 전체를 아래로 교체:

```javascript
function renderStart() {
  var remaining = WORDS.filter(function (w) { return getStars(w.word) < 4; }).length;

  var gardenHtml = '<div class="garden-grid">' +
    WORDS.map(function (w) {
      var stars = getStars(w.word);
      var emoji = FLOWER[stars];
      var style = stars === 0 ? ' style="opacity:0.25"' : '';
      return '<span class="garden-flower"' + style + ' title="' + esc(w.word) + '">' + emoji + '</span>';
    }).join("") +
  '</div>';

  var legendHtml = '<div class="garden-legend">' +
    '<span style="opacity:0.25">🌱 아직 안 봄</span>' +
    '<span>🌱 1단계 통과</span><span>🌿 2단계</span>' +
    '<span>🌷 3단계</span><span>🌸 완전히 외움</span>' +
  '</div>';

  var actionHtml = remaining > 0
    ? '<button class="primary-btn" id="startBtn" type="button">🚀 오늘의 연습 시작! (' + remaining + '개 남음)</button>'
    : '<button class="primary-btn" id="certBtn" type="button">🏆 수료증 보기!</button>';

  screenEl.innerHTML =
    '<div class="mascot-row"><span class="mascot">🐰</span>' +
      '<div class="speech">안녕! 오늘도 같이 단어를 키워보자! 🌸</div></div>' +
    '<div class="card">' +
      '<h2 class="section-title">🌸 내 단어 정원</h2>' +
      gardenHtml + legendHtml + actionHtml +
    '</div>';

  var startBtn = document.getElementById("startBtn");
  if (startBtn) startBtn.addEventListener("click", startQuiz);
  var certBtn = document.getElementById("certBtn");
  if (certBtn) certBtn.addEventListener("click", renderCertificate);
}
```

- [ ] **Step 2: 브라우저에서 홈 화면 확인**

- 🌱 × 20개 (흐리게) 표시되는지
- "오늘의 연습 시작! (20개 남음)" 버튼 보이는지
- 단원 선택 UI가 사라졌는지

- [ ] **Step 3: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: redesign home screen with garden grid, remove unit selector"
```

---

## Task 5: startQuiz() + handleCorrect() + handleWrong() + goNext() 교체

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — startQuiz, markAnswered, markAnsweredPen, goNext 함수들

- [ ] **Step 1: startQuiz 교체**

기존 `function startQuiz() { ... }` 교체:

```javascript
function startQuiz() {
  var q = buildSessionQueue();
  if (!q.length) { renderCertificate(); return; }
  state.queue = q;
  state.retryQueue = [];
  state.index = 0;
  state.answered = false;
  renderQuiz();
}
```

- [ ] **Step 2: handleCorrect + handleWrong 추가 (기존 markAnswered·markAnsweredPen 대체)**

기존 `markAnswered`, `markAnsweredPen` 두 함수를 삭제하고 아래 두 함수로 교체:

```javascript
function handleCorrect(w) {
  if (state.answered) return;
  state.answered = true;
  setStars(w.word, getStars(w.word) + 1);
  var fb = document.getElementById("feedback");
  fb.innerHTML = "🐰✨ 잘했어요! 별이 하나 더 생겼어요 ⭐";
  fb.className = "feedback-row correct";
  document.getElementById("nextBtn").classList.add("show");
}

function handleWrong(w) {
  if (state.answered) return;
  state.answered = true;
  state.retryQueue.push(w);
  var fb = document.getElementById("feedback");
  fb.innerHTML = "🐰🌱 괜찮아요, 나중에 다시 해봐요!";
  fb.className = "feedback-row wrong";
  document.getElementById("nextBtn").classList.add("show");
}
```

- [ ] **Step 3: goNext 교체**

기존 `function goNext() { ... }` 교체:

```javascript
function goNext() {
  state.index++;
  if (state.index < state.queue.length) {
    renderQuiz();
  } else if (state.retryQueue.length > 0) {
    state.queue = state.queue.concat(shuffle(state.retryQueue));
    state.retryQueue = [];
    renderQuiz();
  } else {
    var allDone = WORDS.every(function (w) { return getStars(w.word) >= 4; });
    if (allDone) renderCertificate();
    else renderResult();
  }
}
```

- [ ] **Step 4: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: add handleCorrect/handleWrong with star tracking, update goNext with retry queue"
```

---

## Task 6: renderQuiz() — 4단계 디스패처로 교체

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — renderQuiz 함수 + 새 stage 렌더러 3개 추가

- [ ] **Step 1: renderQuiz 함수 교체 (디스패처만)**

기존 `function renderQuiz() { ... }` 전체를 교체:

```javascript
function renderQuiz() {
  var w = state.queue[state.index];
  state.answered = false;
  var stars = getStars(w.word);
  if (stars === 0) renderStage1(w);
  else if (stars === 1) renderStage2(w);
  else if (stars === 2) renderStage3(w);
  else renderStage4(w);
}
```

- [ ] **Step 2: renderStage1 추가 (기존 MC 로직 이식)**

`renderQuiz` 바로 아래에 추가:

```javascript
function renderStage1(w) {
  var total = state.queue.length;
  var distractors = shuffle(WORDS.filter(function (x) { return x.word !== w.word; })).slice(0, 3);
  var options = shuffle([w].concat(distractors));

  screenEl.innerHTML =
    '<div class="progress-row"><span>문항 ' + (state.index + 1) + ' / ' + total + '</span><span class="growth">🌱</span></div>' +
    '<div class="quiz-card">' +
      '<span class="stage-badge stage-1">🧩 1단계 · 객관식</span>' +
      '<p class="meaning-en">' + esc(w.en) + '</p>' +
      '<p class="meaning-kr">' + esc(w.kr) + '</p>' +
      '<div class="options-grid" id="optionsGrid">' +
        options.map(function (o) {
          return '<button class="option-btn" data-word="' + esc(o.word) + '" type="button">' + esc(o.word) + '</button>';
        }).join("") +
      '</div>' +
      '<div class="feedback-row" id="feedback"></div>' +
      '<div class="quiz-footer">' +
        '<button class="ghost-btn" id="quitBtn" type="button">나가기</button>' +
        '<button class="next-btn" id="nextBtn" type="button">다음 →</button>' +
      '</div>' +
    '</div>';

  document.getElementById("quitBtn").addEventListener("click", renderStart);
  document.getElementById("nextBtn").addEventListener("click", goNext);
  document.querySelectorAll("#optionsGrid .option-btn").forEach(function (btn) {
    btn.addEventListener("click", function () {
      if (state.answered) return;
      var isCorrect = btn.getAttribute("data-word") === w.word;
      document.querySelectorAll("#optionsGrid .option-btn").forEach(function (b) {
        b.disabled = true;
        if (b.getAttribute("data-word") === w.word) b.classList.add("is-correct");
        else if (b === btn && !isCorrect) b.classList.add("is-wrong");
      });
      if (isCorrect) handleCorrect(w); else handleWrong(w);
    });
  });
}
```

- [ ] **Step 3: renderStage2 추가 (단어 보면서 쓰기)**

```javascript
function renderStage2(w) {
  var total = state.queue.length;
  screenEl.innerHTML =
    '<div class="progress-row"><span>문항 ' + (state.index + 1) + ' / ' + total + '</span><span class="growth">🌿</span></div>' +
    '<div class="quiz-card">' +
      '<span class="stage-badge stage-2">✏️ 2단계 · 보면서 쓰기</span>' +
      '<p class="meaning-en">' + esc(w.en) + '</p>' +
      '<p class="meaning-kr">' + esc(w.kr) + '</p>' +
      '<div class="hint-box hint-full word-font">' + esc(w.word) + '</div>' +
      '<p class="type-hint">위 단어를 보면서 아래에 써봐요</p>' +
      renderCanvasHtml() +
      '<div class="feedback-row" id="feedback"></div>' +
      '<div class="quiz-footer">' +
        '<button class="ghost-btn" id="quitBtn" type="button">나가기</button>' +
        '<button class="next-btn" id="nextBtn" type="button">다음 →</button>' +
      '</div>' +
    '</div>';
  bindCanvasEvents(w);
}
```

- [ ] **Step 4: renderStage3 추가 (앞 절반 힌트)**

```javascript
function renderStage3(w) {
  var total = state.queue.length;
  screenEl.innerHTML =
    '<div class="progress-row"><span>문항 ' + (state.index + 1) + ' / ' + total + '</span><span class="growth">🌷</span></div>' +
    '<div class="quiz-card">' +
      '<span class="stage-badge stage-3">🌟 3단계 · 힌트 보고 쓰기</span>' +
      '<p class="meaning-en">' + esc(w.en) + '</p>' +
      '<p class="meaning-kr">' + esc(w.kr) + '</p>' +
      '<div class="hint-box hint-partial word-font">' + esc(getHint(w.word)) + '</div>' +
      '<p class="type-hint">힌트를 보고 전체 단어를 아래에 써봐요</p>' +
      renderCanvasHtml() +
      '<div class="feedback-row" id="feedback"></div>' +
      '<div class="quiz-footer">' +
        '<button class="ghost-btn" id="quitBtn" type="button">나가기</button>' +
        '<button class="next-btn" id="nextBtn" type="button">다음 →</button>' +
      '</div>' +
    '</div>';
  bindCanvasEvents(w);
}
```

- [ ] **Step 5: renderStage4 추가 (뜻만 보고 쓰기)**

```javascript
function renderStage4(w) {
  var total = state.queue.length;
  screenEl.innerHTML =
    '<div class="progress-row"><span>문항 ' + (state.index + 1) + ' / ' + total + '</span><span class="growth">🌸</span></div>' +
    '<div class="quiz-card">' +
      '<span class="stage-badge stage-4">🖊️ 4단계 · 완전 암기</span>' +
      '<p class="meaning-en">' + esc(w.en) + '</p>' +
      '<p class="meaning-kr">' + esc(w.kr) + '</p>' +
      renderCanvasHtml() +
      '<div class="feedback-row" id="feedback"></div>' +
      '<div class="quiz-footer">' +
        '<button class="ghost-btn" id="quitBtn" type="button">나가기</button>' +
        '<button class="next-btn" id="nextBtn" type="button">다음 →</button>' +
      '</div>' +
    '</div>';
  bindCanvasEvents(w);
}
```

- [ ] **Step 6: renderCanvasHtml + bindCanvasEvents 헬퍼 추가**

캔버스 HTML과 이벤트 바인딩을 2·3·4단계가 공유하는 헬퍼 함수:

```javascript
function renderCanvasHtml() {
  return '<div class="pad-card"><canvas id="writeCanvas" class="write-pad"></canvas></div>' +
    '<div class="reveal-box" id="revealBox" hidden>' +
      '<span class="reveal-label">정답</span>' +
      '<span class="reveal-word word-font" id="revealWord"></span>' +
    '</div>' +
    '<div class="pad-toolbar" id="preToolbar">' +
      '<button class="ghost-btn" id="clearPad" type="button">🧹 다시 쓰기</button>' +
      '<button class="primary-btn" id="revealBtn" type="button">👀 정답 확인하기</button>' +
    '</div>' +
    '<div class="pad-toolbar" id="postToolbar" hidden>' +
      '<button class="ghost-btn" id="clearPad2" type="button">🧹 다시 쓰기</button>' +
      '<button class="ghost-btn" id="missedBtn" type="button">🔁 다시 볼게요</button>' +
      '<button class="primary-btn btn-correct" id="gotItBtn" type="button">✅ 맞았어요</button>' +
    '</div>' +
    '<p class="type-hint">펜이나 손가락으로 써봐요. 다 썼으면 정답 확인을 눌러주세요.</p>';
}

function bindCanvasEvents(w) {
  document.getElementById("quitBtn").addEventListener("click", renderStart);
  document.getElementById("nextBtn").addEventListener("click", goNext);
  var pad = setupWritePad();
  document.getElementById("clearPad").addEventListener("click", function () { if (pad) pad.clear(); });
  document.getElementById("revealBtn").addEventListener("click", function () {
    document.getElementById("revealWord").textContent = w.word;
    document.getElementById("revealBox").hidden = false;
    document.getElementById("preToolbar").hidden = true;
    document.getElementById("postToolbar").hidden = false;
  });
  document.getElementById("clearPad2").addEventListener("click", function () { if (pad) pad.clear(); });
  document.getElementById("gotItBtn").addEventListener("click", function () { handleCorrect(w); });
  document.getElementById("missedBtn").addEventListener("click", function () { handleWrong(w); });
}
```

- [ ] **Step 7: 브라우저에서 4단계 전부 테스트**

1. 페이지 열고 시작 클릭
2. 1단계 문제(객관식) 나오는지 확인
3. 정답 클릭 → "별이 하나 더 생겼어요" 피드백 확인
4. 다음 → 같은 단어가 나중에 2단계(보라색 힌트박스 "force")로 나오는지 확인 (localStorage에서 `getStars("force")` 직접 확인)
5. 홈으로 나갔다 다시 들어오면 정원 꽃이 🌱→🌿로 바뀌는지 확인

- [ ] **Step 8: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: implement 4-stage quiz dispatcher with canvas helpers"
```

---

## Task 7: renderResult() 단순화

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — renderResult 함수 전체

- [ ] **Step 1: renderResult 교체**

기존 `function renderResult() { ... }` 전체를 교체:

```javascript
function renderResult() {
  var mastered = WORDS.filter(function (w) { return getStars(w.word) >= 4; }).length;
  var remaining = WORDS.length - mastered;

  screenEl.innerHTML =
    '<div class="card">' +
      '<div class="score-hero">' +
        '<div class="score-mascot">🐰🌱</div>' +
        '<div class="score-num">' + mastered + ' / 20</div>' +
        '<div class="score-label">꽃이 ' + mastered + '송이 피었어요! 계속 키워봐요 🌸</div>' +
      '</div>' +
      '<div class="result-actions">' +
        (remaining > 0
          ? '<button class="primary-btn" id="continueBtn" type="button">🌱 계속 연습하기 (' + remaining + '개 남음)</button>'
          : '') +
        '<button class="ghost-btn" id="homeBtn" type="button">🏠 정원으로 돌아가기</button>' +
      '</div>' +
    '</div>';

  var continueBtn = document.getElementById("continueBtn");
  if (continueBtn) continueBtn.addEventListener("click", startQuiz);
  document.getElementById("homeBtn").addEventListener("click", renderStart);
}
```

- [ ] **Step 2: 세션 종료 후 결과 화면 동작 확인**

몇 개 단어 풀고 세션 끝 → 결과 화면에 "꽃이 N송이" 표시, "계속 연습하기" 누르면 새 세션 시작되는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: simplify result screen with mastery count"
```

---

## Task 8: renderCertificate() 수료증 화면 추가

**Files:**
- Modify: `english/vocab-green-unit1_2.html` — renderResult 아래에 추가

- [ ] **Step 1: renderCertificate 함수 추가**

`renderResult` 함수 바로 아래에 추가:

```javascript
function renderCertificate() {
  screenEl.innerHTML =
    '<div class="card">' +
      '<div class="cert-flowers">🌸🌸🌸🌸🌸<br>🌸🌸🌸🌸🌸<br>🌸🌸🌸🌸🌸<br>🌸🌸🌸🌸🌸</div>' +
      '<div class="cert-title">정원 완성! 🎉</div>' +
      '<div class="cert-sub">Unit 1 · 2 단어 20개를 모두 외웠어요!</div>' +
      '<div class="cert-box">이 친구는 초록 단어 정원의<br>모든 꽃을 피웠습니다 🌸</div>' +
      '<div class="secret-box">' +
        '<div class="secret-label">🔑 비밀코드</div>' +
        '<div class="secret-code">' + esc(SECRET_CODE) + '</div>' +
        '<div class="secret-hint">엄마한테 이 코드를 말해봐요!<br><strong>선물이 기다리고 있어요 🎁</strong></div>' +
      '</div>' +
      '<button class="primary-btn" id="shareBtn" type="button">📸 엄마한테 보여주기</button>' +
      '<button class="ghost-btn" id="resetBtn" type="button" style="margin-top:12px">🔄 처음부터 다시 하기</button>' +
    '</div>';

  document.getElementById("shareBtn").addEventListener("click", function () {
    window.scrollTo({ top: 0, behavior: "smooth" });
  });
  document.getElementById("resetBtn").addEventListener("click", function () {
    var store = loadStore();
    store.words = {};
    saveStore(store);
    renderStart();
  });
}
```

- [ ] **Step 2: 수료증 화면 강제 테스트**

콘솔에서 모든 단어를 별 4개로 설정한 뒤 확인:

```javascript
// 콘솔에 붙여넣기
(function() {
  var store = JSON.parse(localStorage.getItem("vwgreen_progress_v3") || "{}");
  if (!store.words) store.words = {};
  ["allow","bitter","common","faint","firm","force","goal","patient","prefer","trace",
   "aim","aware","defeat","drift","mild","pause","refuse","route","ruin","solid"]
    .forEach(function(w) { store.words[w] = 4; });
  localStorage.setItem("vwgreen_progress_v3", JSON.stringify(store));
  location.reload();
})();
```

페이지 새로고침 후 → "수료증 보기!" 버튼 클릭 → 비밀코드 표시 확인.  
"처음부터 다시 하기" 클릭 → 별점 초기화되고 정원이 🌱×20으로 돌아오는지 확인.

- [ ] **Step 3: SECRET_CODE 변경 안내 footer 업데이트**

기존 footer:
```html
<footer class="note">Unit 1 · Unit 2 들어있어요 (모두 20개) · Unit 3는 준비되면 함께 넣어줄게요.</footer>
```
교체:
```html
<footer class="note">Unit 1 · Unit 2 · 20개 단어 · 비밀코드는 소스 맨 위 SECRET_CODE 변수에서 바꿔요.</footer>
```

- [ ] **Step 4: 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: add certificate screen with secret code and reset button"
```

---

## Task 9: 구버전 코드 정리 + 전체 검증

**Files:**
- Modify: `english/vocab-green-unit1_2.html`

- [ ] **Step 1: 남은 구버전 참조 제거**

파일에서 아래 항목이 남아있으면 삭제:
- `currentWord()` 함수 (이제 `state.queue[state.index]`로 직접 접근)
- `handleMcAnswer()` 함수 (renderStage1 내부로 이식됨)
- `state.score`, `state.wrong`, `state.mode`, `state.questions`, `state.selectedUnits` 참조 잔재

검색 명령:
```bash
grep -n "currentWord\|handleMcAnswer\|state\.score\|state\.wrong\|state\.mode\|selectedUnits\|pool()" english/vocab-green-unit1_2.html
```
출력이 없으면 OK.

- [ ] **Step 2: 전체 시나리오 수동 테스트 (아이패드 환경 체크리스트)**

1. 앱 첫 로드 → 정원 🌱×20(흐리게) + "20개 남음" 버튼
2. 시작 → 1단계 객관식 20문제 풀기 (몇 개 일부러 틀리기)
3. 세션 종료 → 결과 화면 "꽃이 N송이" 표시
4. 홈 → 정원에서 맞힌 단어는 🌱(진하게), 틀린 것은 🌱(흐리게) 유지
5. 다시 시작 → 틀린 단어는 1단계 재도전, 맞힌 단어는 2단계(보라 힌트박스)
6. 2단계 통과 → 홈에서 🌿 확인
7. 3단계 → 앞 절반 힌트(`fo _ _ _`) 표시 확인
8. 4단계 → 힌트 없이 뜻만 표시 확인
9. 모든 단어 별 4개 → 자동으로 수료증 화면 이동
10. 비밀코드 표시, "처음부터 다시 하기" 동작 확인
11. 다크 모드 토글 유지되는지 확인

- [ ] **Step 3: 최종 커밋**

```bash
git add english/vocab-green-unit1_2.html
git commit -m "feat: complete vocab garden 4-stage progression system"
```

---

## 자기 검토 체크

**스펙 커버리지:**
- [x] 4단계 구성 (Task 6)
- [x] 모든 쓰기 단계 캔버스 (Task 6 — renderCanvasHtml)
- [x] 3단계 앞 절반 힌트 (Task 6 — getHint)
- [x] 세션: 단어별 독립 진행, 인터리브 (Task 2, 5)
- [x] ✅ 맞았어요 → 별 즉시 저장, 이번 세션 종료 (Task 5 — handleCorrect)
- [x] 🔁 다시 볼게요 → 세션 끝 재등장 (Task 5 — handleWrong + goNext)
- [x] 별 4개 = 숙달, 다음 세션 제외 (Task 2 — buildSessionQueue)
- [x] 정원 그리드 시각화 (Task 4)
- [x] 단원 선택 제거 (Task 4)
- [x] 수료증 + 비밀코드 (Task 8)
- [x] SECRET_CODE 변수 (Task 1)
- [x] localStorage v3 키 (Task 1)
