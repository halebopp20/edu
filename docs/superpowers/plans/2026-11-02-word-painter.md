# Word Painter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 8세 아이가 문장의 단어를 품사(noun/verb/adjective/adverb)에 맞는 색으로 칠하며 연습하는 단일 HTML 앱

**Architecture:** 단일 파일 `english/word-painter.html`. Vanilla JS 상태 머신(splash→quiz→result). 전역 STATE 객체로 현재 화면·문제 번호·채점 관리. DOM을 직접 조작해 화면 전환.

**Tech Stack:** HTML/CSS/Vanilla JS, Google Fonts (Jua, Baloo 2), 외부 의존 없음

---

### Task 1: HTML 뼈대 + CSS 전체

**Files:**
- Create: `english/word-painter.html`

- [ ] **Step 1: 파일 생성 — HTML 뼈대 + CSS 작성**

`english/word-painter.html` 전체를 아래 내용으로 작성한다.  
CSS만 포함하고 JS·HTML body는 Task 2에서 추가.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>🎨 Word Painter</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Jua&family=Baloo+2:wght@400;600;700;800&display=swap">
<style>
:root{
  --sky1:#38BDF8; --sky2:#818CF8;
  --ink:#1a1a1a; --ink2:#1e3a5f;
  --surface:#FFFFFF;
  --noun-bg:#BBDEFB; --noun-bd:#1565C0; --noun-dot:#1565C0;
  --verb-bg:#FFCDD2; --verb-bd:#C62828; --verb-dot:#C62828;
  --adj-bg:#DCEDC8;  --adj-bd:#33691E;  --adj-dot:#33691E;
  --adv-bg:#FFF9C4;  --adv-bd:#F57F17;  --adv-dot:#F57F17;
  --skip-bg:#F1F5F9; --skip-bd:#CBD5E1;
  --blank-bg:#EFF6FF; --blank-bd:#93C5FD;
  --correct:#16A34A; --wrong:#DC2626;
  --radius:22px; --cshadow:0 5px 0 rgba(0,0,0,.25);
}
*{box-sizing:border-box;margin:0;padding:0}
html,body{min-height:100vh}
body{background:linear-gradient(160deg,var(--sky1),var(--sky2));color:var(--ink);font-family:"Baloo 2",sans-serif;font-size:18px}
#app{max-width:520px;margin:0 auto;padding:24px 16px 150px}

/* 버튼 */
.btn{display:block;width:100%;border:2.5px solid var(--ink);border-bottom:6px solid var(--ink);background:#0EA5E9;color:#fff;font-family:"Jua",sans-serif;font-size:28px;padding:18px;border-radius:var(--radius);cursor:pointer;text-align:center;transition:transform .1s}
.btn:active{transform:translateY(4px);border-bottom-width:2px}

/* 진행 바 */
.prog-wrap{background:rgba(255,255,255,.25);border-radius:999px;height:14px;overflow:hidden;border:2px solid rgba(255,255,255,.6);margin-bottom:6px}
.prog-fill{height:100%;background:#fff;border-radius:999px;transition:width .3s ease}
.prog-label{display:flex;justify-content:flex-end;font-size:14px;color:#E0F2FE;font-weight:700;margin-bottom:14px}

/* 상단 바 */
.topbar{display:flex;align-items:center;gap:12px;margin-bottom:12px}
.topbar-title{font-family:"Jua",sans-serif;font-size:26px;color:#fff;flex:1;text-shadow:2px 2px 0 rgba(0,0,0,.2)}

/* 스플래시 */
.splash{text-align:center;padding:40px 0 20px}
.splash h1{font-family:"Jua",sans-serif;font-size:52px;color:#fff;-webkit-text-stroke:2px var(--ink);text-shadow:4px 4px 0 var(--ink);margin-bottom:10px}
.splash p{font-size:20px;font-weight:700;color:#E0F2FE;margin-bottom:30px}

/* 캐릭터 + 말풍선 */
.char-row{display:flex;align-items:flex-end;gap:12px;margin-bottom:16px}
.bubble{background:var(--surface);border:2.5px solid var(--ink);border-radius:20px 20px 20px 4px;padding:13px 16px;font-size:18px;font-weight:700;line-height:1.6;flex:1;box-shadow:0 3px 0 rgba(0,0,0,.2);color:var(--ink2)}

/* 카드 */
.card{background:var(--surface);border:2.5px solid var(--ink);border-radius:var(--radius);padding:24px 18px;box-shadow:var(--cshadow);margin-bottom:16px}
.card-label{font-size:14px;font-weight:700;letter-spacing:2px;color:#64748B;margin-bottom:16px}

/* 단어 칩 */
.sent-area{display:flex;flex-wrap:wrap;gap:10px;justify-content:center;margin-bottom:4px}
.chip{padding:12px 20px;border-radius:14px;font-size:26px;font-family:"Jua",sans-serif;border:2.5px solid var(--ink);cursor:pointer;box-shadow:0 4px 0 var(--ink);transition:transform .1s;user-select:none}
.chip:active:not(.skip){transform:translateY(3px);box-shadow:0 1px 0 var(--ink)}
.chip.blank{background:var(--blank-bg);border-color:var(--blank-bd);color:#3B82F6}
.chip.noun{background:var(--noun-bg);border-color:var(--noun-bd);color:var(--noun-dot)}
.chip.verb{background:var(--verb-bg);border-color:var(--verb-bd);color:var(--verb-dot)}
.chip.adj {background:var(--adj-bg); border-color:var(--adj-bd); color:var(--adj-dot)}
.chip.adv {background:var(--adv-bg); border-color:var(--adv-bd); color:var(--adv-dot)}
.chip.skip{background:var(--skip-bg);border-color:var(--skip-bd);color:#94A3B8;cursor:default;font-size:22px}
@keyframes shake{0%,100%{transform:translateX(0)}25%{transform:translateX(-8px)}75%{transform:translateX(8px)}}
.chip.shake{animation:shake .35s ease}
@keyframes pop{0%{transform:scale(1)}50%{transform:scale(1.25)}100%{transform:scale(1)}}
.chip.pop{animation:pop .3s ease}

/* 힌트 박스 */
.hint-box{background:#FFF7ED;border:2.5px solid #FB923C;border-radius:16px;padding:14px 16px;font-size:18px;font-weight:700;color:#9A3412;margin-top:16px;display:flex;align-items:flex-start;gap:12px;line-height:1.6}
.hint-icon{font-size:34px;flex:none}

/* 팔레트 */
.palette{position:fixed;bottom:0;left:0;right:0;background:rgba(255,255,255,.97);border-top:2.5px solid var(--ink);padding:12px 10px 20px;display:flex;gap:8px;justify-content:center;z-index:100}
.pal{flex:1;max-width:116px;padding:10px 4px 8px;border-radius:16px;border:2.5px solid var(--ink);font-family:"Jua",sans-serif;font-size:17px;font-weight:700;text-align:center;cursor:pointer;box-shadow:0 4px 0 var(--ink);transition:transform .12s;line-height:1.3}
.pal:active{transform:translateY(2px)}
.pal.noun{background:var(--noun-bg);color:var(--noun-dot)}
.pal.verb{background:var(--verb-bg);color:var(--verb-dot)}
.pal.adj {background:var(--adj-bg); color:var(--adj-dot)}
.pal.adv {background:var(--adv-bg); color:var(--adv-dot)}
.pal.active{transform:scale(1.1) translateY(-4px);box-shadow:0 7px 0 var(--ink);border-width:3px}
.pal-dot{width:20px;height:20px;border-radius:50%;border:2px solid rgba(0,0,0,.2);margin:0 auto 4px}

/* 결과 */
.result-hero{text-align:center;padding:30px 0 16px}
.result-title{font-family:"Jua",sans-serif;font-size:44px;color:#fff;-webkit-text-stroke:1.5px var(--ink);text-shadow:3px 3px 0 var(--ink)}
.stars-big{font-size:40px;letter-spacing:8px;margin:14px 0}
.review-item{background:var(--surface);border:2px solid var(--ink);border-radius:14px;padding:12px 16px;margin-bottom:10px;font-size:16px;font-weight:700;line-height:1.7;box-shadow:0 3px 0 var(--ink)}
.review-word{font-size:20px;font-family:"Jua",sans-serif}

@keyframes fadeUp{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:translateY(0)}}
.card,.result-hero,.splash{animation:fadeUp .3s ease}
</style>
</head>
<body>
<div id="app"></div>
<script>
/* Task 2~6에서 채워짐 */
</script>
</body>
</html>
```

- [ ] **Step 2: 브라우저에서 확인**

`english/word-painter.html` 을 브라우저로 열어 흰 화면(빈 #app)이 오류 없이 로드되는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter HTML skeleton and CSS"
```

---

### Task 2: 데이터 레이어 (20문장)

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부

단어 배열 구조:  
`{ w: "단어", pos: "noun"|"verb"|"adj"|"adv"|"skip" }`  
`pos:"skip"` = the/a/an/is/are (처음부터 회색, 채점 제외)

- [ ] **Step 1: SENTENCES 배열 작성**

`<script>` 내용을 아래로 교체:

```js
var BEAR = (function(){
  var s = '<svg viewBox="0 0 100 110" width="76" height="90" xmlns="http://www.w3.org/2000/svg">';
  s += '<circle cx="22" cy="24" r="14" fill="#C8956C" stroke="#1a1a1a" stroke-width="2.5"/>';
  s += '<circle cx="78" cy="24" r="14" fill="#C8956C" stroke="#1a1a1a" stroke-width="2.5"/>';
  s += '<circle cx="22" cy="24" r="9" fill="#E8B090"/>';
  s += '<circle cx="78" cy="24" r="9" fill="#E8B090"/>';
  s += '<ellipse cx="50" cy="60" rx="38" ry="36" fill="#C8956C" stroke="#1a1a1a" stroke-width="2.5"/>';
  s += '<ellipse cx="50" cy="70" rx="18" ry="12" fill="#E8B090"/>';
  s += '<circle cx="36" cy="52" r="7" fill="#1a1a1a"/>';
  s += '<circle cx="64" cy="52" r="7" fill="#1a1a1a"/>';
  s += '<circle cx="38" cy="50" r="2.5" fill="white"/>';
  s += '<circle cx="66" cy="50" r="2.5" fill="white"/>';
  s += '<ellipse cx="22" cy="63" rx="8" ry="5.5" fill="rgba(255,140,140,0.5)"/>';
  s += '<ellipse cx="78" cy="63" rx="8" ry="5.5" fill="rgba(255,140,140,0.5)"/>';
  s += '<ellipse cx="50" cy="65" rx="5" ry="3.5" fill="#1a1a1a"/>';
  s += '<path d="M44,70 Q50,76 56,70" fill="none" stroke="#1a1a1a" stroke-width="2.2" stroke-linecap="round"/>';
  s += '<ellipse cx="50" cy="100" rx="28" ry="12" fill="#C8956C" stroke="#1a1a1a" stroke-width="2"/>';
  s += '</svg>';
  return s;
})();

/* 각 단어의 pos: noun/verb/adj/adv/skip
   hint: 틀렸을 때 보여줄 8세 눈높이 설명 */
var SENTENCES = [
  { words: [
    {w:'Dogs',  pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'run',   pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'fast',  pos:'adv',  hint:'run이 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',     pos:'skip'},
    {w:'cat',     pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'sleeps',  pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'quietly', pos:'adv',  hint:'sleeps가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'She',   pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'eats',  pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'lunch', pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'}
  ]},
  { words: [
    {w:'The',      pos:'skip'},
    {w:'red',      pos:'adj',  hint:'ball이 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'ball',     pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'bounces',  pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'He',      pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'runs',    pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'quickly', pos:'adv',  hint:'runs가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'Birds',   pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'sing',    pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'sweetly', pos:'adv',  hint:'sing이 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',   pos:'skip'},
    {w:'big',   pos:'adj',  hint:'dog이 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'dog',   pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'barks', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'She',   pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'reads', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'books', pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'daily', pos:'adv',  hint:'reads가 언제 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',    pos:'skip'},
    {w:'happy',  pos:'adj',  hint:'child가 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'child',  pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'laughs', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'He',    pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'jumps', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'high',  pos:'adv',  hint:'jumps가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',   pos:'skip'},
    {w:'tiny',  pos:'adj',  hint:'mouse가 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'mouse', pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'hides', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'She',    pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'writes', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'neatly', pos:'adv',  hint:'writes가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',   pos:'skip'},
    {w:'cold',  pos:'adj',  hint:'wind가 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'wind',  pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'blows', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'He',     pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'talks',  pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'loudly', pos:'adv',  hint:'talks가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',   pos:'skip'},
    {w:'brave', pos:'adj',  hint:'lion이 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'lion',  pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'roars', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'}
  ]},
  { words: [
    {w:'She',         pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'dances',      pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'beautifully', pos:'adv',  hint:'dances가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',    pos:'skip'},
    {w:'tall',   pos:'adj',  hint:'tree가 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'tree',   pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'stands', pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'firmly', pos:'adv',  hint:'stands가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'He',     pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'eats',   pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'slowly', pos:'adv',  hint:'eats가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'The',      pos:'skip'},
    {w:'yellow',   pos:'adj',  hint:'sun이 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'sun',      pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'shines',   pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'brightly', pos:'adv',  hint:'shines가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]},
  { words: [
    {w:'She',       pos:'noun', hint:'사람 이름 대신 쓰는 단어야 → 🟦 Noun!'},
    {w:'sings',     pos:'verb', hint:'무엇을 하는지 나타내는 단어야 → 🟥 Verb!'},
    {w:'a',         pos:'skip'},
    {w:'beautiful', pos:'adj',  hint:'song이 어떤 모습인지 설명해줘, 명사 옆에 붙는 단어야 → 🟩 Adjective!'},
    {w:'song',      pos:'noun', hint:'사람·동물·물건 이름이야 → 🟦 Noun!'},
    {w:'softly',    pos:'adv',  hint:'sings가 어떻게 일어나는지 말해줘 → 🟨 Adverb!'}
  ]}
];
```

- [ ] **Step 2: 브라우저 콘솔에서 확인**

브라우저 콘솔에서 `SENTENCES.length` 입력 → `20` 출력 확인.

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter data layer (20 sentences)"
```

---

### Task 3: 상태 머신 + 스플래시 화면

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부 (SENTENCES 배열 아래에 추가)

- [ ] **Step 1: STATE 객체 + 렌더 함수 + 스플래시 추가**

SENTENCES 배열 아래에 이어서 작성:

```js
var STATE = {
  screen: 'splash',   // 'splash' | 'quiz' | 'result'
  qIdx: 0,            // 현재 문제 인덱스 (0~19)
  picks: [],          // 현재 문제의 각 단어에 선택한 pos ('blank'|'noun'|'verb'|'adj'|'adv')
  activePal: null,    // 현재 선택한 팔레트 색
  tried: 0,           // 현재 문제 시도 횟수
  score: 0,           // 첫 시도 전부 맞춘 문제 수
  wrongWords: []      // 결과 화면용 {w, correctPos, hint}
};

var app = document.getElementById('app');

function render() {
  if (STATE.screen === 'splash') renderSplash();
  else if (STATE.screen === 'quiz') renderQuiz();
  else renderResult();
}

/* ── 곰 말풍선 HTML 반환 ── */
function bearBubble(msg) {
  return '<div class="char-row">' +
    '<div style="flex:none">' + BEAR + '</div>' +
    '<div class="bubble">' + msg + '</div>' +
    '</div>';
}

/* ── 스플래시 ── */
function renderSplash() {
  app.innerHTML =
    '<div class="splash">' +
      bearBubble('단어를 색깔로<br>칠해봐! 🖌️') +
      '<h1>🎨 Word Painter</h1>' +
      '<p>알맞은 색으로 모든 단어를 칠해봐!</p>' +
      '<button class="btn" onclick="startQuiz()">시작! 🐻</button>' +
    '</div>';
}

function startQuiz() {
  STATE.screen = 'quiz';
  STATE.qIdx = 0;
  STATE.score = 0;
  STATE.wrongWords = [];
  loadQuestion();
}

render();
```

- [ ] **Step 2: 브라우저에서 스플래시 확인**

`english/word-painter.html` 새로고침 → 곰 + 말풍선 + "시작!" 버튼 보이는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter splash screen"
```

---

### Task 4: 퀴즈 화면 렌더링

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부 (render 함수들 아래)

- [ ] **Step 1: loadQuestion + renderQuiz 함수 추가**

```js
/* ── 문제 로드 ── */
function loadQuestion() {
  var q = SENTENCES[STATE.qIdx];
  STATE.picks = q.words.map(function(w) {
    return w.pos === 'skip' ? 'skip' : 'blank';
  });
  STATE.activePal = null;
  STATE.tried = 0;
  renderQuiz();
}

/* ── 퀴즈 렌더 ── */
function renderQuiz() {
  var pct = Math.round((STATE.qIdx / 20) * 100);
  var bubbleMsgs = [
    '색을 먼저 고르고,<br>단어를 탭해서 칠해봐! 🖌️',
    '잘 하고 있어! 계속해봐! 💪',
    '어떤 색일까? 잘 생각해봐! 🤔',
    '거의 다 왔어! 조금만 더! 🌟'
  ];
  var msgIdx = STATE.qIdx < 5 ? 0 : STATE.qIdx < 10 ? 1 : STATE.qIdx < 15 ? 2 : 3;

  var html =
    '<div class="topbar"><span class="topbar-title">🎨 Word Painter</span></div>' +
    '<div class="prog-wrap"><div class="prog-fill" style="width:' + pct + '%"></div></div>' +
    '<div class="prog-label">' + (STATE.qIdx + 1) + ' / 20</div>' +
    bearBubble(bubbleMsgs[msgIdx]) +
    '<div class="card" id="qcard">' +
      '<div class="card-label">색을 골라서 모든 단어를 칠해봐!</div>' +
      '<div class="sent-area" id="sent-area">' + renderChips() + '</div>' +
    '</div>' +
    '<button class="btn" onclick="checkAnswer()">✅ 확인!</button>' +
    renderPalette();

  app.innerHTML = html;
}

/* ── 단어 칩 HTML ── */
function renderChips() {
  var q = SENTENCES[STATE.qIdx];
  return q.words.map(function(word, i) {
    var cls = STATE.picks[i];
    return '<div class="chip ' + cls + '" id="chip' + i + '" ' +
      (cls !== 'skip' ? 'onclick="paintChip(' + i + ')"' : '') +
      '>' + word.w + '</div>';
  }).join('');
}

/* ── 팔레트 HTML ── */
function renderPalette() {
  var pals = [
    {key:'noun', label:'Noun'},
    {key:'verb', label:'Verb'},
    {key:'adj',  label:'Adjective'},
    {key:'adv',  label:'Adverb'}
  ];
  var dotColors = {noun:'#1565C0', verb:'#C62828', adj:'#33691E', adv:'#F57F17'};
  var html = '<div class="palette">';
  pals.forEach(function(p) {
    var active = STATE.activePal === p.key ? ' active' : '';
    html += '<button class="pal ' + p.key + active + '" onclick="selectPal(\'' + p.key + '\')">' +
      '<div class="pal-dot" style="background:' + dotColors[p.key] + '"></div>' +
      p.label +
      '</button>';
  });
  html += '</div>';
  return html;
}
```

- [ ] **Step 2: 브라우저에서 확인**

"시작!" 클릭 → 문장 칩들 + 팔레트 버튼 4개 보이는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter quiz render"
```

---

### Task 5: 팔레트 선택 + 단어 색칠 인터랙션

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부

- [ ] **Step 1: selectPal + paintChip 함수 추가**

```js
/* ── 팔레트 선택 ── */
function selectPal(key) {
  STATE.activePal = key;
  /* 팔레트 버튼 active 클래스 갱신 */
  document.querySelectorAll('.pal').forEach(function(btn) {
    btn.classList.toggle('active', btn.classList.contains(key));
  });
}

/* ── 단어 칩 색칠 ── */
function paintChip(idx) {
  if (!STATE.activePal) return; /* 색 안 고른 상태 */
  STATE.picks[idx] = STATE.activePal;
  var chip = document.getElementById('chip' + idx);
  /* 기존 품사 클래스 제거 후 새 클래스 적용 */
  chip.className = 'chip ' + STATE.activePal;
  chip.setAttribute('onclick', 'paintChip(' + idx + ')');
  /* pop 애니메이션 */
  chip.classList.add('pop');
  setTimeout(function() { chip.classList.remove('pop'); }, 300);
}
```

- [ ] **Step 2: 브라우저에서 확인**

1. 팔레트에서 "Noun" 클릭 → 버튼 scale up 확인
2. 단어 칩 탭 → 파란 배경으로 변경 + pop 확인
3. 다른 색으로 바꾼 뒤 같은 칩 탭 → 색 변경 확인

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter palette + paint interaction"
```

---

### Task 6: 정답 확인 + 힌트 + 다음 문제 흐름

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부

- [ ] **Step 1: checkAnswer 함수 추가**

```js
/* ── 정답 확인 ── */
function checkAnswer() {
  var q = SENTENCES[STATE.qIdx];
  var wrongIdxs = [];

  q.words.forEach(function(word, i) {
    if (word.pos === 'skip') return;
    if (STATE.picks[i] !== word.pos) wrongIdxs.push(i);
  });

  if (wrongIdxs.length === 0) {
    /* 전부 정답 */
    if (STATE.tried === 0) STATE.score++;
    celebrateAndNext();
  } else {
    STATE.tried++;
    shakeWrong(wrongIdxs, q);
  }
}

/* ── 틀린 칩 흔들기 + 힌트 표시 ── */
function shakeWrong(wrongIdxs, q) {
  wrongIdxs.forEach(function(i) {
    var chip = document.getElementById('chip' + i);
    chip.classList.add('shake');
    setTimeout(function() { chip.classList.remove('shake'); }, 400);
  });

  /* 힌트 박스 — 첫 번째 틀린 단어 힌트만 표시 */
  var firstWrong = wrongIdxs[0];
  var hintText = q.words[firstWrong].hint;

  /* 두 번째 시도 이상이면 정답도 함께 공개 */
  if (STATE.tried >= 2) {
    var correctLabel = {noun:'🟦 Noun', verb:'🟥 Verb', adj:'🟩 Adjective', adv:'🟨 Adverb'};
    hintText += '<br><b>정답: "' + q.words[firstWrong].w + '" → ' +
      correctLabel[q.words[firstWrong].pos] + '</b>';

    /* wrongWords 기록 (결과 화면용, 중복 방지) */
    wrongIdxs.forEach(function(i) {
      var already = STATE.wrongWords.some(function(x){ return x.w === q.words[i].w; });
      if (!already) {
        STATE.wrongWords.push({w: q.words[i].w, correctPos: q.words[i].pos, hint: q.words[i].hint});
      }
    });

    setTimeout(function() { nextQuestion(); }, 2200);
  }

  var card = document.getElementById('qcard');
  /* 기존 힌트 제거 후 추가 */
  var old = document.getElementById('hint-box');
  if (old) old.remove();
  var hintEl = document.createElement('div');
  hintEl.id = 'hint-box';
  hintEl.className = 'hint-box';
  hintEl.innerHTML = '<span class="hint-icon">🐻</span><div>' + hintText + '</div>';
  card.appendChild(hintEl);
}

/* ── 정답 축하 → 다음 문제 ── */
function celebrateAndNext() {
  var area = document.getElementById('sent-area');
  area.querySelectorAll('.chip:not(.skip)').forEach(function(chip) {
    chip.classList.add('pop');
  });
  /* 말풍선 변경 */
  var bubble = document.querySelector('.bubble');
  if (bubble) bubble.innerHTML = '🎉 완벽해!';
  setTimeout(function() { nextQuestion(); }, 1200);
}

/* ── 다음 문제 또는 결과 ── */
function nextQuestion() {
  STATE.qIdx++;
  if (STATE.qIdx >= 20) {
    STATE.screen = 'result';
    renderResult();
  } else {
    loadQuestion();
  }
}
```

- [ ] **Step 2: 브라우저에서 확인**

1. 칩 일부만 칠한 뒤 "확인!" → 틀린 칩 흔들림 + 힌트 박스 등장
2. 전부 맞게 칠한 뒤 "확인!" → 칩 pop + "🎉 완벽해!" → 다음 문제 전환
3. 두 번 틀리면 정답 공개 후 자동 전환

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter answer check, hints, flow"
```

---

### Task 7: 결과 화면

**Files:**
- Modify: `english/word-painter.html` — `<script>` 내부

- [ ] **Step 1: renderResult 함수 추가**

```js
/* ── 결과 화면 ── */
function renderResult() {
  var stars = STATE.score >= 18 ? '★★★' : STATE.score >= 14 ? '★★☆' : '★☆☆';
  var msg   = STATE.score >= 18 ? '최고야! 🎉' : STATE.score >= 14 ? '잘 했어! 👍' : '다시 해봐! 💪';

  var reviewHtml = '';
  if (STATE.wrongWords.length > 0) {
    reviewHtml = '<div class="card" style="margin-top:16px">' +
      '<div class="card-label">다시 확인해봐!</div>';
    STATE.wrongWords.forEach(function(item) {
      var colorLabel = {noun:'🟦 Noun', verb:'🟥 Verb', adj:'🟩 Adjective', adv:'🟨 Adverb'};
      reviewHtml +=
        '<div class="review-item">' +
          '<span class="review-word">' + item.w + '</span> → ' + colorLabel[item.correctPos] + '<br>' +
          '<span style="font-size:15px;color:#64748B">' + item.hint + '</span>' +
        '</div>';
    });
    reviewHtml += '</div>';
  }

  app.innerHTML =
    '<div class="result-hero">' +
      bearBubble(msg) +
      '<div class="stars-big">' + stars + '</div>' +
      '<div class="result-title">' + STATE.score + ' / 20</div>' +
    '</div>' +
    reviewHtml +
    '<button class="btn" style="margin-top:20px" onclick="restartApp()">🔄 다시 도전!</button>';
}

function restartApp() {
  STATE.screen = 'splash';
  render();
}
```

- [ ] **Step 2: 브라우저에서 전체 플로우 확인**

1. 시작 → 20문제 완료 → 결과 화면 (점수·별 표시)
2. 틀린 단어 복습 카드 표시 확인
3. "다시 도전!" → 스플래시로 돌아오는지 확인

- [ ] **Step 3: 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter result screen"
```

---

### Task 8: 최종 polish + 모바일 점검

**Files:**
- Modify: `english/word-painter.html`

- [ ] **Step 1: 빈 칸 검사 추가 (확인 버튼 클릭 시 미칠한 단어 있으면 경고)**

`checkAnswer()` 함수 상단에 아래 코드를 추가:

```js
/* 미칠한 단어 있으면 흔들기 */
var unpainted = [];
q.words.forEach(function(word, i) {
  if (word.pos !== 'skip' && STATE.picks[i] === 'blank') unpainted.push(i);
});
if (unpainted.length > 0) {
  unpainted.forEach(function(i) {
    var chip = document.getElementById('chip' + i);
    chip.classList.add('shake');
    setTimeout(function(){ chip.classList.remove('shake'); }, 400);
  });
  var bubble = document.querySelector('.bubble');
  if (bubble) bubble.innerHTML = '먼저 모든 단어를 칠해봐! 🎨';
  return;
}
```

- [ ] **Step 2: 모바일 뷰 확인**

브라우저 DevTools → 모바일 뷰(375px 기준):
- 단어 칩 줄바꿈 자연스러운지 확인
- 하단 팔레트 버튼 4개 가로로 잘 맞는지 확인
- "확인!" 버튼이 팔레트에 가려지지 않는지 확인 (`padding-bottom:150px` 이미 적용됨)

- [ ] **Step 3: 최종 커밋**

```bash
git add english/word-painter.html
git commit -m "feat: word-painter complete - paint-bucket POS quiz"
```
