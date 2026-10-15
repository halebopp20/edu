# Article Quest Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 다크 초콜릿 테마의 단일 HTML 퀴즈 앱 `english/article-quest.html`을 만든다. Sort(분류)와 Drill(빈칸 드릴)을 번갈아 출제하며, 모든 문제를 맞혀야 게임이 끝나는 마스터리 기반 학습 앱.

**Architecture:** 단일 HTML 파일 — CSS 변수로 다크 초콜릿 테마 정의, 바닐라 JS로 상태 관리, `innerHTML` 렌더링. grammar-quiz.html과 동일한 패턴. 두 개의 독립 큐(sortQueue, drillQueue)를 번갈아 소진하는 마스터리 루프.

**Tech Stack:** HTML5, CSS3 (CSS variables, keyframe animations), Vanilla JS (ES5 호환), Google Fonts (Jua + Baloo 2), localStorage

---

## 파일 구조

| 파일 | 역할 |
|------|------|
| `english/article-quest.html` | 앱 전체 (생성) |

---

### Task 1: HTML 스켈레톤 + CSS 변수/기본 스타일

**Files:**
- Create: `english/article-quest.html`

- [ ] **Step 1: 파일 생성 — HTML 스켈레톤 + CSS 변수**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>🍫 Choco Quest</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Jua&family=Baloo+2:wght@400;600;700&display=swap">
<style>
  :root {
    --bg:        #3b1a08;
    --bg2:       #5c2d0e;
    --surface:   #2c1206;
    --surface2:  #3b2010;
    --gold:      #fbbf24;
    --gold2:     #d97706;
    --cream:     #fef3c7;
    --cream2:    #fde68a;
    --correct:   #22c55e;
    --wrong:     #dc2626;
    --radius:    20px;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: linear-gradient(160deg, var(--bg), var(--bg2));
    min-height: 100vh;
    color: var(--cream);
    font-family: "Baloo 2", sans-serif;
    font-size: 18px;
  }

  .app {
    max-width: 680px;
    margin: 0 auto;
    padding: 24px 16px 60px;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  /* 카드 */
  .card {
    background: var(--surface);
    border: 3px solid var(--gold2);
    border-radius: var(--radius);
    padding: 28px 22px;
    box-shadow: 0 6px 24px rgba(0,0,0,0.4);
  }

  /* 진행 바 */
  .prog-wrap {
    background: var(--surface);
    border-radius: 999px;
    height: 14px;
    overflow: hidden;
    border: 2px solid var(--gold2);
  }
  .prog-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--gold), var(--gold2));
    border-radius: 999px;
    transition: width 0.35s ease;
  }
  .prog-label {
    display: flex;
    justify-content: space-between;
    font-size: 16px;
    color: var(--cream2);
    font-weight: 700;
    margin-bottom: 6px;
  }

  /* 기본 버튼 */
  .primary-btn {
    width: 100%;
    border: none;
    border-bottom: 5px solid var(--gold2);
    background: linear-gradient(135deg, var(--gold), var(--gold2));
    color: var(--bg);
    font-family: "Jua", sans-serif;
    font-size: 26px;
    padding: 18px;
    border-radius: var(--radius);
    cursor: pointer;
    transition: transform 0.1s;
  }
  .primary-btn:active { transform: translateY(3px); border-bottom-width: 2px; }

  /* 흔들림 애니메이션 (오답) */
  @keyframes shake {
    0%,100% { transform: translateX(0); }
    20% { transform: translateX(-8px); }
    40% { transform: translateX(8px); }
    60% { transform: translateX(-6px); }
    80% { transform: translateX(6px); }
  }
  .shake { animation: shake 0.4s ease; }

  /* 폭죽 파티클 */
  @keyframes burst {
    0%   { transform: translate(0,0) scale(1); opacity: 1; }
    100% { transform: translate(var(--tx), var(--ty)) scale(0); opacity: 0; }
  }
  .confetti {
    position: fixed;
    width: 12px; height: 12px;
    border-radius: 50%;
    pointer-events: none;
    animation: burst 1.2s ease-out forwards;
  }
</style>
</head>
<body>
<div class="app" id="screen"></div>
<script>
  // Task 2 이후에 채울 예정
</script>
</body>
</html>
```

- [ ] **Step 2: 브라우저에서 확인**

`english/article-quest.html`을 브라우저로 열어 다크 초콜릿 배경이 보이는지 확인. 빈 페이지여도 배경색이 정상이면 OK.

- [ ] **Step 3: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: scaffold article-quest HTML skeleton with dark choco theme"
```

---

### Task 2: 데이터 정의 (WORDS + SENTENCES)

**Files:**
- Modify: `english/article-quest.html` — `<script>` 안에 데이터 추가

- [ ] **Step 1: WORDS 배열 추가** (`<script>` 안에 작성)

```js
var WORDS = [
  { word: 'milk',    emoji: '🥛', type: 'noncount' },
  { word: 'cheese',  emoji: '🧀', type: 'noncount' },
  { word: 'bread',   emoji: '🍞', type: 'noncount' },
  { word: 'oil',     emoji: '🫙', type: 'noncount' },
  { word: 'water',   emoji: '💧', type: 'noncount' },
  { word: 'apple',   emoji: '🍎', type: 'count' },
  { word: 'book',    emoji: '📚', type: 'count' },
  { word: 'cat',     emoji: '🐱', type: 'count' },
  { word: 'bicycle', emoji: '🚲', type: 'count' },
  { word: 'banana',  emoji: '🍌', type: 'count' },
  { word: 'salt',    emoji: '🧂', type: 'noncount' },
  { word: 'egg',     emoji: '🥚', type: 'count' },
];
```

- [ ] **Step 2: SENTENCES 배열 추가**

```js
var SENTENCES = [
  { text: 'I drink ___ milk.',                    ans: 'some', hint: 'milk은 셀 수 없어요 → some!' },
  { text: 'She has ___ umbrella.',                ans: 'an',   hint: 'u로 시작하는 소리 → an!' },
  { text: 'Look at ___ sun.',                     ans: 'the',  hint: '태양은 세상에 하나뿐이에요 → the!' },
  { text: 'Tom has a dog. ___ dog is cute.',      ans: 'The',  hint: '앞에서 말한 바로 그 강아지 → The!' },
  { text: 'We need ___ onion.',                   ans: 'an',   hint: 'o로 시작하는 소리 → an!' },
  { text: 'I need ___ cheese.',                   ans: 'some', hint: 'cheese는 셀 수 없어요 → some!' },
  { text: 'Jack has ___ smartphone.',             ans: 'a',    hint: 's는 자음 소리 → a!' },
  { text: '___ sky is clear.',                    ans: 'The',  hint: '하늘은 세상에 하나뿐이에요 → The!' },
  { text: 'She has a cat. ___ cat is small.',     ans: 'The',  hint: '앞에서 말한 바로 그 고양이 → The!' },
  { text: 'I see ___ elephant.',                  ans: 'an',   hint: 'e로 시작하는 소리 → an!' },
  { text: 'He drinks ___ water every day.',       ans: 'some', hint: 'water는 셀 수 없어요 → some!' },
  { text: 'I have ___ idea!',                     ans: 'an',   hint: 'i로 시작하는 소리 → an!' },
  { text: 'I like ___ moon.',                     ans: 'the',  hint: '달은 세상에 하나뿐이에요 → the!' },
  { text: 'Ann has a bag. ___ bag is red.',       ans: 'The',  hint: '앞에서 말한 바로 그 가방 → The!' },
  { text: 'Can I have ___ bread?',               ans: 'some', hint: 'bread는 셀 수 없어요 → some!' },
  { text: 'She is ___ honest girl.',              ans: 'an',   hint: 'h는 묵음 → an honest!' },
];
```

- [ ] **Step 3: 상태 객체 + 큐 초기화 함수 추가**

```js
var state = {
  sortQueue:  [],   // { word, emoji, type, firstTry: true }
  drillQueue: [],   // { text, ans, hint, firstTry: true }
  turn: 'sort',     // 'sort' | 'drill'
  totalSort:  0,
  totalDrill: 0,
  firstCorrectSort:  0,  // 첫 시도 정답 카운트
  firstCorrectDrill: 0,
  startTime: null,
};

function initQueues() {
  state.sortQueue = WORDS.map(function(w) {
    return { word: w.word, emoji: w.emoji, type: w.type, firstTry: true };
  });
  state.drillQueue = SENTENCES.map(function(s) {
    return { text: s.text, ans: s.ans, hint: s.hint, firstTry: true };
  });
  state.turn = 'sort';
  state.totalSort  = state.sortQueue.length;
  state.totalDrill = state.drillQueue.length;
  state.firstCorrectSort  = 0;
  state.firstCorrectDrill = 0;
  state.startTime = new Date();
}

function esc(s) {
  return String(s).replace(/[&<>"']/g, function(c) {
    return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c];
  });
}

var screenEl = document.getElementById('screen');
```

- [ ] **Step 4: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add WORDS, SENTENCES data and state object"
```

---

### Task 3: 홈 화면 renderHome()

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: renderHome() 함수 추가**

```js
function renderHome() {
  var best = localStorage.getItem('aq-best') || null;
  var bestHtml = best
    ? '<p style="color:var(--cream2);font-size:18px;margin-top:12px">🏆 최고 기록: <strong>' + best + '%</strong> 첫 시도 정답률</p>'
    : '';

  screenEl.innerHTML =
    '<div style="text-align:center;padding:20px 0 10px">' +
      '<div style="font-size:72px;line-height:1">🍫</div>' +
      '<h1 style="font-family:Jua,sans-serif;font-size:42px;color:var(--gold);margin:8px 0 4px">Choco Quest</h1>' +
      '<p style="color:var(--cream2);font-size:20px;font-weight:700">Master a, an, the &amp; some!</p>' +
    '</div>' +
    '<div class="card">' +
      '<p style="color:var(--cream);font-size:20px;line-height:1.7;margin-bottom:20px">' +
        '🍫 Sort — 단어를 Count / Noncount로 분류해봐!<br>' +
        '✏️ Drill — 빈칸에 알맞은 관사를 골라봐!<br>' +
        '🏆 모두 맞혀야 Choco Master 완성! 틀리면 다시 나와!' +
      '</p>' +
      bestHtml +
      '<button class="primary-btn" id="startBtn" style="margin-top:20px">Start Quest 🚀</button>' +
    '</div>';

  document.getElementById('startBtn').addEventListener('click', function() {
    initQueues();
    nextTurn();
  });
}
```

- [ ] **Step 2: 진입점 추가 (스크립트 맨 아래)**

```js
renderHome();
```

- [ ] **Step 3: 브라우저에서 확인**

브라우저로 열어 홈 화면이 정상 표시되는지, Start Quest 버튼이 보이는지 확인. (버튼 클릭 시 에러 나도 OK — nextTurn은 다음 Task에서 구현)

- [ ] **Step 4: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add home screen with best score display"
```

---

### Task 4: 진행 바 + nextTurn() 라우터

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: remainingCount() + progressHtml() + nextTurn() 추가**

```js
function remainingCount() {
  return state.sortQueue.length + state.drillQueue.length;
}

function totalCount() {
  return state.totalSort + state.totalDrill;
}

function progressHtml() {
  var remaining = remainingCount();
  var done = totalCount() - remaining;
  var pct = Math.round((done / totalCount()) * 100);
  return (
    '<div class="prog-label">' +
      '<span>🍫 남은 문제 <strong>' + remaining + '개</strong></span>' +
      '<span>' + pct + '%</span>' +
    '</div>' +
    '<div class="prog-wrap"><div class="prog-fill" style="width:' + pct + '%"></div></div>'
  );
}

function nextTurn() {
  if (state.sortQueue.length === 0 && state.drillQueue.length === 0) {
    renderEnding();
    return;
  }
  if (state.turn === 'sort' && state.sortQueue.length > 0) {
    renderSort();
    return;
  }
  if (state.turn === 'drill' && state.drillQueue.length > 0) {
    renderDrill();
    return;
  }
  // 한쪽 큐가 빈 경우 남은 쪽으로 전환
  if (state.sortQueue.length > 0) { state.turn = 'sort'; renderSort(); }
  else { state.turn = 'drill'; renderDrill(); }
}
```

- [ ] **Step 2: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add progress bar and nextTurn router"
```

---

### Task 5: Sort 턴 renderSort()

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: renderSort() 추가**

```js
function renderSort() {
  var item = state.sortQueue[0];

  screenEl.innerHTML =
    progressHtml() +
    '<div class="card" style="margin-top:14px">' +
      '<div style="font-size:13px;font-weight:700;letter-spacing:2px;color:var(--cream2);margin-bottom:16px">📦 SORT THE WORD</div>' +
      '<div id="wordCard" style="text-align:center;background:var(--surface2);border:3px solid var(--gold);border-radius:var(--radius);padding:28px 20px;margin-bottom:24px">' +
        '<div style="font-size:60px;line-height:1;margin-bottom:10px">' + esc(item.emoji) + '</div>' +
        '<div style="font-size:28px;font-weight:700;color:var(--cream)">' + esc(item.word) + '</div>' +
      '</div>' +
      '<div style="display:flex;gap:14px">' +
        '<button class="sort-btn" data-type="count" style="flex:1;background:var(--surface2);border:3px solid var(--gold);border-radius:var(--radius);padding:20px 10px;cursor:pointer;color:var(--cream);font-family:\'Baloo 2\',sans-serif;font-size:20px;font-weight:700">' +
          '<div style="font-size:32px">📦</div>' +
          '<div>Count</div>' +
          '<div style="font-size:14px;color:var(--cream2);margin-top:4px">→ a / an</div>' +
        '</button>' +
        '<button class="sort-btn" data-type="noncount" style="flex:1;background:var(--surface2);border:3px solid var(--gold2);border-radius:var(--radius);padding:20px 10px;cursor:pointer;color:var(--cream);font-family:\'Baloo 2\',sans-serif;font-size:20px;font-weight:700">' +
          '<div style="font-size:32px">💧</div>' +
          '<div>Noncount</div>' +
          '<div style="font-size:14px;color:var(--cream2);margin-top:4px">→ some</div>' +
        '</button>' +
      '</div>' +
      '<div id="sortFb" style="margin-top:18px;font-size:20px;font-weight:700;text-align:center;min-height:28px"></div>' +
    '</div>';

  var answered = false;
  document.querySelectorAll('.sort-btn').forEach(function(btn) {
    btn.addEventListener('click', function() {
      if (answered) return;
      answered = true;

      var chosen = btn.getAttribute('data-type');
      var correct = item.type;
      var isCorrect = chosen === correct;
      var fb = document.getElementById('sortFb');

      if (isCorrect) {
        if (item.firstTry) state.firstCorrectSort++;
        state.sortQueue.shift();             // 정답 → 큐에서 제거
        btn.style.borderColor = 'var(--correct)';
        btn.style.background = 'rgba(34,197,94,0.15)';
        fb.style.color = 'var(--correct)';
        fb.textContent = '✓ ' + (correct === 'count' ? 'Count!' : 'Noncount!') + ' 🍫';
        state.turn = 'drill';
        setTimeout(nextTurn, 1400);
      } else {
        item.firstTry = false;
        state.sortQueue.shift();
        state.sortQueue.push(item);          // 오답 → 맨 뒤로
        btn.style.borderColor = 'var(--wrong)';
        btn.style.background = 'rgba(220,38,38,0.15)';
        btn.classList.add('shake');
        fb.style.color = 'var(--wrong)';
        fb.textContent = '✗ 정답은 ' + (correct === 'count' ? 'Count (a/an)' : 'Noncount (some)') + '!';
        state.turn = 'drill';
        setTimeout(nextTurn, 2000);
      }
    });
  });
}
```

- [ ] **Step 2: 브라우저에서 확인**

Start Quest → Sort 화면 표시 확인. 단어 카드와 두 버튼이 보이면 OK. 버튼 클릭 시 콘솔 에러 없이 다음으로 넘어가면 OK. (renderDrill 없어도 에러 메시지만 나오면 됨)

- [ ] **Step 3: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add Sort turn UI with mastery re-queue logic"
```

---

### Task 6: Drill 턴 renderDrill()

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: renderDrill() 추가**

```js
function renderDrill() {
  var item = state.drillQueue[0];
  var opts = ['a', 'an', 'the', 'some'];
  var qHtml = esc(item.text).replace(/___/, '<span style="background:var(--gold);color:var(--bg);padding:2px 14px;border-radius:8px;font-weight:900;font-size:28px">___</span>');

  screenEl.innerHTML =
    progressHtml() +
    '<div class="card" style="margin-top:14px">' +
      '<div style="font-size:13px;font-weight:700;letter-spacing:2px;color:var(--cream2);margin-bottom:16px">✏️ FILL IN THE BLANK</div>' +
      '<p style="font-size:24px;font-weight:600;color:var(--cream);line-height:1.7;margin-bottom:24px;text-align:center">' + qHtml + '</p>' +
      '<div style="display:flex;gap:10px;flex-wrap:wrap;justify-content:center" id="drillOpts">' +
        opts.map(function(o) {
          return '<button class="drill-btn" data-ans="' + esc(o) + '" ' +
            'style="min-width:100px;padding:14px 20px;border-radius:var(--radius);background:var(--surface2);border:3px solid var(--gold2);color:var(--cream2);font-family:\'Baloo 2\',sans-serif;font-size:22px;font-weight:900;cursor:pointer">' +
            esc(o) + '</button>';
        }).join('') +
      '</div>' +
      '<div id="drillFb" style="margin-top:20px;font-size:18px;font-weight:700;text-align:center;min-height:48px;color:var(--cream2)"></div>' +
    '</div>';

  var answered = false;
  document.querySelectorAll('.drill-btn').forEach(function(btn) {
    btn.addEventListener('click', function() {
      if (answered) return;
      answered = true;

      var chosen = btn.getAttribute('data-ans');
      var correct = item.ans;
      var isCorrect = chosen === correct;
      var fb = document.getElementById('drillFb');

      // 모든 버튼 비활성화
      document.querySelectorAll('.drill-btn').forEach(function(b) {
        b.disabled = true;
        if (b.getAttribute('data-ans') === correct) {
          b.style.background = 'rgba(34,197,94,0.2)';
          b.style.borderColor = 'var(--correct)';
          b.style.color = 'var(--correct)';
        }
      });

      if (isCorrect) {
        if (item.firstTry) state.firstCorrectDrill++;
        state.drillQueue.shift();
        btn.style.background = 'rgba(34,197,94,0.3)';
        fb.style.color = 'var(--correct)';
        fb.textContent = '✓ 정답! 💡 ' + esc(item.hint);
        state.turn = 'sort';
        setTimeout(nextTurn, 1600);
      } else {
        item.firstTry = false;
        state.drillQueue.shift();
        state.drillQueue.push(item);
        btn.style.background = 'rgba(220,38,38,0.2)';
        btn.style.borderColor = 'var(--wrong)';
        btn.style.color = 'var(--wrong)';
        btn.classList.add('shake');
        fb.style.color = 'var(--gold)';
        fb.innerHTML = '💡 ' + esc(item.hint);
        state.turn = 'sort';
        setTimeout(nextTurn, 2200);
      }
    });
  });
}
```

- [ ] **Step 2: 브라우저에서 확인**

Sort → Drill 번갈아 진행되는지 확인. 정답 시 1.6초 후 다음으로, 오답 시 2.2초 후 다음으로. 남은 문제 수가 줄어드는지 확인.

- [ ] **Step 3: 전체 게임 플로우 확인**

마지막 문제를 다 맞혀 큐가 비었을 때 콘솔에 에러 없이 renderEnding 호출되는지 확인. (renderEnding 없어도 OK — 다음 Task에서 구현)

- [ ] **Step 4: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add Drill turn UI with mastery re-queue logic"
```

---

### Task 7: 엔딩 화면 + 폭죽 애니메이션

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: 폭죽 함수 추가**

```js
function launchConfetti() {
  var colors = ['#fbbf24','#d97706','#fef3c7','#f97316','#fde68a','#ffffff'];
  var count = 60;
  for (var i = 0; i < count; i++) {
    (function() {
      var el = document.createElement('div');
      el.className = 'confetti';
      var angle = Math.random() * 2 * Math.PI;
      var dist  = 80 + Math.random() * 180;
      el.style.setProperty('--tx', Math.cos(angle) * dist + 'px');
      el.style.setProperty('--ty', Math.sin(angle) * dist - 120 + 'px');
      el.style.left = (20 + Math.random() * 60) + 'vw';
      el.style.top  = (30 + Math.random() * 30) + 'vh';
      el.style.background = colors[Math.floor(Math.random() * colors.length)];
      el.style.animationDelay = (Math.random() * 0.5) + 's';
      el.style.animationDuration = (0.9 + Math.random() * 0.6) + 's';
      document.body.appendChild(el);
      setTimeout(function() { el.remove(); }, 2000);
    })();
  }
}
```

- [ ] **Step 2: renderEnding() 추가**

```js
function renderEnding() {
  // 첫 시도 정답률 계산
  var total = state.totalSort + state.totalDrill;
  var firstCorrect = state.firstCorrectSort + state.firstCorrectDrill;
  var rate = Math.round((firstCorrect / total) * 100);

  // 최고 기록 저장
  var prev = parseInt(localStorage.getItem('aq-best') || '0', 10);
  if (rate > prev) localStorage.setItem('aq-best', String(rate));

  launchConfetti();
  setTimeout(launchConfetti, 700);

  screenEl.innerHTML =
    '<div style="text-align:center;padding:30px 10px">' +
      '<div style="font-size:80px;line-height:1;margin-bottom:16px">🏆</div>' +
      '<h1 style="font-family:Jua,sans-serif;font-size:40px;color:var(--gold);margin-bottom:8px">Choco Master!</h1>' +
      '<p style="font-size:26px;color:var(--cream);margin-bottom:4px">All Clear! 🍫✨</p>' +
      '<p style="font-size:60px;line-height:1.2;margin:16px 0">⭐⭐⭐</p>' +
    '</div>' +
    '<div class="card" style="text-align:center">' +
      '<p style="font-size:22px;color:var(--cream);margin-bottom:8px">총 <strong style="color:var(--gold)">' + total + '문제</strong> 완료!</p>' +
      '<p style="font-size:20px;color:var(--cream2);margin-bottom:20px">첫 시도 정답률: <strong style="color:var(--gold);font-size:28px">' + rate + '%</strong></p>' +
      '<button id="parentBtn" class="primary-btn" style="margin-bottom:12px">부모님께 보여주기 📣</button>' +
      '<div id="parentCard" style="display:none"></div>' +
      '<button id="retryBtn" class="primary-btn" style="margin-top:8px;background:var(--surface2);border-bottom-color:var(--surface);color:var(--cream2)">다시 도전! 🔄</button>' +
    '</div>';

  document.getElementById('parentBtn').addEventListener('click', showParentCard);
  document.getElementById('retryBtn').addEventListener('click', function() {
    initQueues();
    nextTurn();
  });
}
```

- [ ] **Step 3: showParentCard() 추가**

```js
function showParentCard() {
  var total = state.totalSort + state.totalDrill;
  var firstCorrect = state.firstCorrectSort + state.firstCorrectDrill;
  var rate = Math.round((firstCorrect / total) * 100);
  var now = new Date();
  var timeStr = now.getHours() + ':' + String(now.getMinutes()).padStart(2,'0');

  // 가장 많이 틀린 카테고리 분석
  var sortWrong  = state.totalSort  - state.firstCorrectSort;
  var drillWrong = state.totalDrill - state.firstCorrectDrill;
  var weakNote = '';
  if (sortWrong > drillWrong) weakNote = '📦 Count/Noncount 분류 (' + sortWrong + '번 재도전)';
  else if (drillWrong > sortWrong) weakNote = '✏️ 빈칸 채우기 (' + drillWrong + '번 재도전)';
  else weakNote = '골고루 잘 했어요!';

  var card = document.getElementById('parentCard');
  card.style.display = 'block';
  card.innerHTML =
    '<div style="background:var(--surface2);border:2px solid var(--gold2);border-radius:var(--radius);padding:20px;margin-top:16px;text-align:left">' +
      '<div style="font-size:16px;font-weight:700;color:var(--cream2);letter-spacing:1px;margin-bottom:14px">📋 학습 결과 리포트</div>' +
      '<div style="display:flex;flex-direction:column;gap:10px;font-size:18px;color:var(--cream)">' +
        '<div>📅 완료 시각: <strong style="color:var(--gold)">' + timeStr + '</strong></div>' +
        '<div>📊 총 문제: <strong style="color:var(--gold)">' + total + '문제</strong></div>' +
        '<div>🎯 첫 시도 정답률: <strong style="color:var(--gold)">' + rate + '%</strong></div>' +
        '<div>🔍 약점: <strong style="color:var(--gold)">' + weakNote + '</strong></div>' +
      '</div>' +
    '</div>';

  document.getElementById('parentBtn').style.display = 'none';
}
```

- [ ] **Step 4: 브라우저에서 전체 게임 플레이 확인**

1. 홈 → Start Quest
2. Sort + Drill 번갈아 진행
3. 모든 문제 정답 처리 (몇 개 일부러 오답 → 재출제 확인)
4. 모두 맞히면 엔딩 화면 + 폭죽 확인
5. "부모님께 보여주기" 클릭 → 리포트 카드 확인
6. "다시 도전" 클릭 → 홈으로 돌아가기 확인

- [ ] **Step 5: 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: add ending screen with confetti and parent report card"
```

---

### Task 8: 마무리 polish — 폰트 크기 / 모바일 / UX

**Files:**
- Modify: `english/article-quest.html`

- [ ] **Step 1: `<style>`에 반응형 + UX polish 추가**

`</style>` 바로 위에 추가:

```css
  /* 터치 친화적 버튼 */
  button { touch-action: manipulation; -webkit-tap-highlight-color: transparent; }

  /* 버튼 hover 효과 (마우스 환경) */
  .sort-btn:hover:not(:disabled),
  .drill-btn:hover:not(:disabled) {
    border-color: var(--gold);
    color: var(--gold);
    transform: translateY(-2px);
  }

  /* 좁은 화면에서 Sort 버튼 세로 배치 */
  @media (max-width: 380px) {
    .sort-btns { flex-direction: column !important; }
  }
```

- [ ] **Step 2: sort-btn과 drill-btn에 transition 추가**

`renderSort()`와 `renderDrill()` 안의 버튼 inline style에 `transition:all 0.15s` 추가:

sort-btn style 속성 끝에:
```
transition:all 0.15s;
```

drill-btn style 속성 끝에:
```
transition:all 0.15s;
```

- [ ] **Step 3: 브라우저에서 최종 확인**

모바일 크기(375px)로 DevTools에서 확인:
- 버튼이 탭하기 충분히 큰지 (최소 48px 높이)
- 텍스트가 잘리지 않는지
- 폰트 크기가 편안한지

- [ ] **Step 4: 최종 커밋**

```bash
git add english/article-quest.html
git commit -m "feat: complete article-quest with polish and mobile support"
```

---

## 셀프 리뷰

**스펙 커버리지 체크:**
- [x] a vs an — Drill 문제 다수 포함
- [x] Count vs Noncount — Sort 턴 전담
- [x] some — Drill 문제 + Sort Noncount 분류
- [x] the (only one) — Drill 문제 포함 (sun, moon, sky)
- [x] the (specific) — Drill 문제 포함 (앞 문장 언급 패턴)
- [x] 마스터리 루프 — sortQueue/drillQueue 재추가
- [x] 진행 바 "남은 문제 N개" — progressHtml()
- [x] 엔딩 — 폭죽 + Choco Master
- [x] 부모 요약 카드 — showParentCard()
- [x] localStorage 최고 기록 — renderEnding()에서 저장, renderHome()에서 표시
- [x] 다크 초콜릿 테마 — CSS variables
- [x] 큰 폰트 — 24px 본문, 22px 버튼, 28px 빈칸 강조
- [x] 모바일 — 클릭 방식, 큰 버튼, @media 반응형

**타입 일관성:**
- `state.sortQueue` / `state.drillQueue` — Task 2에서 정의, Task 5·6에서 동일 키 사용
- `item.firstTry` — Task 2 initQueues()에서 true로 초기화, Task 5·6에서 false로 수정
- `state.firstCorrectSort` / `state.firstCorrectDrill` — Task 2에서 0으로 초기화, Task 7 renderEnding()에서 합산
- `state.turn` — Task 4 nextTurn()에서 라우팅, Task 5·6에서 'sort'/'drill'로 전환
