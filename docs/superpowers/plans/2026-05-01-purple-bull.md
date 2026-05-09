# Purple Bull Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 紫の闘牛キャラクターを操作するスネークゲーム「Purple Bull」を CSS 3D Transform + DOM で実装し、Supabase オンラインランキングを備えたスマホ向けブラウザゲームを完成させる。

**Architecture:** ゲームフィールドは `perspective` + `rotateX/rotateZ` の CSS 3D で斜め俯瞰表示。各セルは `<div>` として DOM 管理し、スネークの位置変化時に class を付け替えてレンダリングする。ゲームロジックは `BullGame` クラス（game.js）、画面制御・スワイプ・ランキングは main.js に分離。

**Tech Stack:** HTML / CSS / Vanilla JS、Supabase JS v2 (CDN)、Google Fonts (Orbitron / Share Tech Mono)

---

## ファイルマップ

| ファイル | 役割 |
|---|---|
| `Purple Bull/index.html` | 全画面の HTML 構造（ホーム・ゲーム・モーダル） |
| `Purple Bull/styles.css` | 全スタイル（3D フィールド・UI・アニメーション） |
| `Purple Bull/game.js` | `BullGame` クラス（グリッド・スネーク・衝突判定・DOM 更新） |
| `Purple Bull/main.js` | 画面切替・スワイプ・ランキング表示・ゲームオーバー処理 |
| `Purple Bull/supabase-api.js` | `bull_scores` テーブルへのスコア投稿・ランキング取得 |
| `Purple Bull/supabase-config.js` | Supabase 接続設定（Purple Diver と同じプロジェクト） |
| `Purple Bull/supabase-config.example.js` | 設定テンプレート |
| `Purple Bull/manifest.json` | PWA マニフェスト |
| `Purple Bull/bull.png` | 牛キャラクター（透過 PNG・ユーザー配置） |

---

## Task 1: プロジェクト scaffold

**Files:**
- Create: `Purple Bull/manifest.json`
- Create: `Purple Bull/supabase-config.example.js`
- Create: `Purple Bull/supabase-config.js`

- [ ] **Step 1: ディレクトリ作成**

```bash
mkdir "Purple Bull"
```

- [ ] **Step 2: manifest.json を作成**

`Purple Bull/manifest.json` を作成:

```json
{
  "name": "Purple Bull",
  "short_name": "Purple Bull",
  "description": "紫の闘牛が肉を食べて成長するスネークゲーム",
  "start_url": "./",
  "display": "standalone",
  "background_color": "#0a0015",
  "theme_color": "#7c3aed",
  "icons": [
    {
      "src": "./bull.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

- [ ] **Step 3: supabase-config.example.js を作成**

`Purple Bull/supabase-config.example.js` を作成:

```js
(function initializeSupabaseClient() {
  const SUPABASE_URL = "https://YOUR_PROJECT.supabase.co";
  const SUPABASE_ANON_KEY = "YOUR_ANON_KEY";

  let resolveReady;
  window.supabaseReadyPromise = new Promise(function (resolve) {
    resolveReady = resolve;
  });

  const init = () => {
    try {
      if (typeof supabase !== "undefined" && supabase.createClient) {
        window.supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
      } else {
        window.supabaseClient = null;
      }
    } catch (e) {
      window.supabaseClient = null;
    }
    if (typeof resolveReady === "function") {
      resolveReady();
      resolveReady = null;
    }
  };

  if (document.readyState === "complete") {
    setTimeout(init, 100);
  } else {
    window.addEventListener("load", init);
  }
})();
```

- [ ] **Step 4: supabase-config.js を作成（Purple Diver の実 URL/KEY をコピー）**

`Purple Bull/supabase-config.js` — Purple Diver の `supabase-config.js` と同じ内容をコピーする（同じ Supabase プロジェクトを使用）:

```js
(function initializeSupabaseClient() {
  const SUPABASE_URL = "https://hefayilffszrczxhnpii.supabase.co";
  const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImhlZmF5aWxmZnN6cmN6eGhucGlpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzE5NDI5NDEsImV4cCI6MjA4NzUxODk0MX0.qUsuQOIZzdlFLXtR-i1d9TX5c3P9QKPdhv34QGt4V_k";

  let resolveReady;
  window.supabaseReadyPromise = new Promise(function (resolve) {
    resolveReady = resolve;
  });

  const init = () => {
    try {
      if (typeof supabase !== "undefined" && supabase.createClient) {
        window.supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
        console.log("Supabase connected.");
      } else {
        console.warn("Supabase library not found. Running in offline mode.");
        window.supabaseClient = null;
      }
    } catch (e) {
      console.error("Supabase init error:", e);
      window.supabaseClient = null;
    }
    if (typeof resolveReady === "function") {
      resolveReady();
      resolveReady = null;
    }
  };

  if (document.readyState === "complete") {
    setTimeout(init, 100);
  } else {
    window.addEventListener("load", init);
  }
})();
```

- [ ] **Step 5: bull.png をゲームディレクトリに配置**

ユーザーから受け取った透過 PNG を `Purple Bull/bull.png` として保存する。

- [ ] **Step 6: コミット**

```bash
git -C "Purple Bull" init
git -C "Purple Bull" add manifest.json supabase-config.example.js supabase-config.js bull.png
git -C "Purple Bull" commit -m "chore: init Purple Bull project scaffold"
```

---

## Task 2: Supabase テーブル作成 + supabase-api.js

**Files:**
- Create: `Purple Bull/supabase-api.js`

- [ ] **Step 1: Supabase ダッシュボードで bull_scores テーブルを作成**

Supabase プロジェクトの SQL エディタで以下を実行:

```sql
create table if not exists bull_scores (
  id          uuid primary key default gen_random_uuid(),
  nickname    text not null,
  score       integer not null default 0,
  created_at  timestamptz not null default now()
);

create index if not exists bull_scores_score_idx on bull_scores (score desc);
create index if not exists bull_scores_created_at_idx on bull_scores (created_at desc);
```

- [ ] **Step 2: supabase-api.js を作成**

`Purple Bull/supabase-api.js` を作成:

```js
const RANKING_TABLE = "bull_scores";

async function waitForSupabaseConnection(maxWaitMs = 5000) {
  if (window.supabaseReadyPromise) {
    await window.supabaseReadyPromise;
  }
  const client = await getSupabaseClientWithRetry(maxWaitMs, 100);
  return { connected: !!client };
}
if (typeof window !== "undefined") {
  window.waitForSupabaseConnection = waitForSupabaseConnection;
}

async function getSupabaseClientWithRetry(maxWaitMs = 5000, intervalMs = 100) {
  const start = Date.now();
  if (window.supabaseClient) return window.supabaseClient;

  while (Date.now() - start < maxWaitMs) {
    await new Promise((resolve) => setTimeout(resolve, intervalMs));
    if (window.supabaseClient) return window.supabaseClient;
  }

  console.warn("Supabaseクライアントが指定時間内に初期化されませんでした。オフラインモードで動作します。");
  return null;
}

async function submitScore({ nickname, score }) {
  const client = await getSupabaseClientWithRetry();
  if (!client) return { error: null, skipped: true };

  const { data, error } = await client
    .from(RANKING_TABLE)
    .insert({ nickname, score })
    .select()
    .single();

  return { data, error };
}

async function fetchRanking(range) {
  const client = await getSupabaseClientWithRetry();
  if (!client) return { data: [], error: null, skipped: true };

  const now = new Date();
  let fromDate = null;
  if (range === "today") {
    fromDate = new Date(now.getFullYear(), now.getMonth(), now.getDate());
  } else if (range === "week") {
    fromDate = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 7);
  }

  let query = client
    .from(RANKING_TABLE)
    .select("*")
    .order("score", { ascending: false })
    .order("created_at", { ascending: true })
    .limit(500);

  if (fromDate) {
    query = query.gte("created_at", fromDate.toISOString());
  }

  const { data, error } = await query;
  if (error || !Array.isArray(data)) return { data: [], error, skipped: false };

  // 同一ニックネームは最高スコアのみ残す
  const bestByName = data.reduce((acc, row) => {
    const name = String(row.nickname ?? "").trim();
    if (!name) return acc;
    const sc = Number(row.score ?? 0);
    if (!acc[name] || sc > Number(acc[name].score ?? 0)) acc[name] = row;
    return acc;
  }, {});

  const deduped = Object.values(bestByName)
    .sort((a, b) => {
      const diff = Number(b.score ?? 0) - Number(a.score ?? 0);
      if (diff !== 0) return diff;
      return new Date(a.created_at || 0) - new Date(b.created_at || 0);
    })
    .slice(0, 10);

  return { data: deduped, error: null };
}
```

- [ ] **Step 3: コミット**

```bash
git -C "Purple Bull" add supabase-api.js
git -C "Purple Bull" commit -m "feat: add supabase-api for bull_scores ranking"
```

---

## Task 3: index.html

**Files:**
- Create: `Purple Bull/index.html`

- [ ] **Step 1: index.html を作成**

`Purple Bull/index.html` を作成:

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <title>Purple Bull</title>
  <meta name="apple-mobile-web-app-title" content="🐂Purple Bull🐂" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2" crossorigin defer></script>
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700;800;900&family=Share+Tech+Mono&display=swap" rel="stylesheet" />
  <link rel="stylesheet" href="./styles.css" />
  <link rel="manifest" href="./manifest.json" />
</head>
<body>
  <div id="app">
    <div id="error-banner" aria-live="polite"></div>

    <!-- ホーム画面 -->
    <main id="screen-home" class="screen screen--active">
      <header class="header">
        <a class="home-logo-link" href="https://purple-park.vercel.app/" aria-label="Purple Park へ移動">
          <img src="./logo.png" alt="Purple Park ロゴ" class="home-logo-image" />
        </a>
        <h1 class="title">🐂 PURPLE BULL</h1>
      </header>

      <section class="hero">
        <img src="./bull.png" alt="紫の闘牛" class="hero-image" />
      </section>

      <section class="form-section">
        <input id="nickname" class="input" type="text" maxlength="12" placeholder="例：むらさき太郎" />
        <button id="start-button" class="button button--primary">GAME START</button>
      </section>

      <section class="ranking-section">
        <div class="tabs" role="tablist">
          <button class="tab tab--active" data-tab="today" role="tab" aria-selected="true">今日</button>
          <button class="tab" data-tab="week" role="tab" aria-selected="false">週間</button>
          <button class="tab" data-tab="all" role="tab" aria-selected="false">歴代</button>
        </div>
        <div class="ranking-lists">
          <div class="ranking-list ranking-list--active" data-panel="today" role="tabpanel">
            <ul id="ranking-today" class="ranking-items">
              <li class="ranking-item ranking-item--placeholder">今日のランキングを読み込み中…</li>
            </ul>
          </div>
          <div class="ranking-list" data-panel="week" role="tabpanel">
            <ul id="ranking-week" class="ranking-items">
              <li class="ranking-item ranking-item--placeholder">週間ランキングを読み込み中…</li>
            </ul>
          </div>
          <div class="ranking-list" data-panel="all" role="tabpanel">
            <ul id="ranking-all" class="ranking-items">
              <li class="ranking-item ranking-item--placeholder">歴代ランキングを読み込み中…</li>
            </ul>
          </div>
        </div>
      </section>

      <footer class="footer"></footer>
    </main>

    <!-- ゲーム画面 -->
    <main id="screen-game" class="screen">
      <header class="game-header">
        <span class="game-score-label">🥩 <span id="score-value" class="game-score-value">0</span></span>
        <span class="game-best-label">BEST: <span id="best-value">0</span></span>
        <button id="quit-button" class="quit-button" type="button" aria-label="ゲームを終了">✕</button>
      </header>
      <section class="game-section">
        <div class="game-field-wrapper">
          <div id="game-field" class="game-field"></div>
        </div>
        <p class="swipe-hint" id="swipe-hint">← SWIPE TO MOVE →</p>
      </section>
    </main>

    <!-- ゲームオーバーモーダル -->
    <div id="gameover-overlay" class="gameover-overlay" aria-hidden="true">
      <div class="gameover-dialog">
        <h2 class="gameover-title">GAME OVER</h2>
        <p class="gameover-score">
          <span class="gameover-score-label">今回の記録：</span>
          <span id="gameover-score-value" class="gameover-score-value">0</span>
          <span class="gameover-score-unit">🥩</span>
        </p>
        <p id="gameover-badge" class="gameover-badge gameover-badge--hidden">New Record!</p>
        <div class="gameover-actions">
          <button id="gameover-retry" class="gameover-button gameover-button--primary" type="button">再挑戦</button>
          <button id="gameover-home" class="gameover-button gameover-button--secondary" type="button">トップへ戻る</button>
        </div>
      </div>
    </div>
  </div>

  <script src="./supabase-config.js" defer></script>
  <script src="./supabase-api.js" defer></script>
  <script src="./game.js" defer></script>
  <script src="./main.js" defer></script>
</body>
</html>
```

- [ ] **Step 2: ブラウザで開いてホーム画面の構造を確認**

`Purple Bull/index.html` をブラウザで開く。スタイルなしの骨格 HTML が表示されればOK。

- [ ] **Step 3: コミット**

```bash
git -C "Purple Bull" add index.html
git -C "Purple Bull" commit -m "feat: add index.html structure"
```

---

## Task 4: styles.css

**Files:**
- Create: `Purple Bull/styles.css`

- [ ] **Step 1: styles.css を作成**

`Purple Bull/styles.css` を作成:

```css
:root {
  --bg: #0a0015;
  --bg-field: #051205;
  --accent: #7c3aed;
  --accent-soft: rgba(124, 58, 237, 0.18);
  --accent-strong: #c084fc;
  --text: #f7f5ff;
  --text-muted: #a6a0c0;
  --radius-lg: 20px;
  --radius-md: 14px;
  --shadow-soft: 0 18px 45px rgba(0, 0, 0, 0.5);

  /* grass colors */
  --grass-a: #166534;
  --grass-b: #15803d;
  --grass-dark: #052e16;

  /* snake colors */
  --bull-body: #a855f7;
  --bull-body-dark: #7c3aed;
  --bull-shadow: #4c1d95;
}

*, *::before, *::after { box-sizing: border-box; }

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: radial-gradient(circle at top, #1a0a2e 0, var(--bg) 55%, #000 100%);
  color: var(--text);
  overflow: hidden;
  position: fixed;
  top: 0; left: 0;
  overscroll-behavior: none;
}

body {
  display: flex;
  justify-content: center;
  align-items: stretch;
}

#app {
  width: 100%;
  max-width: 520px;
  min-height: 100vh;
  padding: 12px 12px 18px;
  display: flex;
  flex-direction: column;
}

#error-banner {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 100;
  padding: 6px 10px;
  font-size: 0.8rem;
  background: rgba(239, 68, 68, 0.9);
  color: #fef2f2;
  text-align: center;
  display: none;
}

.screen { display: none; flex: 1; }
.screen--active { display: flex; flex-direction: column; }

/* ========== ホーム画面 ========== */

.header {
  position: relative;
  text-align: center;
  padding-top: 8px;
}

.home-logo-link {
  position: absolute;
  top: 0; left: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  text-decoration: none;
}

.home-logo-image {
  display: block;
  width: clamp(56px, 14vw, 78px);
  height: auto;
}

.title {
  margin: 0;
  font-family: "Orbitron", sans-serif;
  font-size: clamp(1.4rem, 4vw, 2rem);
  font-weight: 900;
  letter-spacing: 0.08em;
  color: var(--accent-strong);
  text-shadow: 0 0 18px rgba(157, 93, 255, 0.8);
  animation: title-float 3.6s ease-in-out infinite;
}

@keyframes title-float {
  0%, 100% { transform: translateY(0); }
  50%       { transform: translateY(-6px); }
}

.hero {
  margin-top: 14px;
  display: flex;
  justify-content: center;
}

.hero-image {
  display: block;
  width: clamp(90px, 25vw, 140px);
  height: auto;
  filter: drop-shadow(0 8px 24px rgba(168, 85, 247, 0.6));
  animation: title-float 3.6s ease-in-out infinite;
}

.form-section {
  margin-top: 12px;
  padding: 14px 14px 16px;
  border-radius: var(--radius-lg);
  background: radial-gradient(circle at top left, rgba(124, 58, 237, 0.25), rgba(16, 6, 46, 0.9));
  box-shadow: var(--shadow-soft);
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.input {
  padding: 10px 12px;
  border-radius: 999px;
  border: 1px solid rgba(192, 132, 252, 0.4);
  background: rgba(5, 0, 24, 0.9);
  color: var(--text);
  font-size: 0.95rem;
  outline: none;
}

.input::placeholder { color: rgba(166, 160, 192, 0.8); }
.input:focus {
  border-color: var(--accent-strong);
  box-shadow: 0 0 0 1px rgba(124, 58, 237, 0.7);
}

.button {
  margin-top: 6px;
  padding: 10px 16px;
  border-radius: 999px;
  border: none;
  font-family: "Orbitron", sans-serif;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
  transition: transform 0.08s ease-out, box-shadow 0.08s ease-out;
}

.button--primary {
  background: linear-gradient(135deg, #c084fc, #7c3aed);
  color: #fff;
  box-shadow: 0 4px 0 #4c1d95, 0 8px 24px rgba(124, 58, 237, 0.5);
}

.button--primary:active {
  transform: translateY(2px);
  box-shadow: 0 2px 0 #4c1d95, 0 4px 12px rgba(0, 0, 0, 0.5);
}

.button:disabled { opacity: 0.55; cursor: default; box-shadow: none; }

.ranking-section {
  margin-top: 14px;
  padding: 14px 12px 6px;
  border-radius: var(--radius-lg);
  background: rgba(6, 2, 28, 0.92);
  box-shadow: var(--shadow-soft);
  display: flex;
  flex-direction: column;
  gap: 10px;
  flex: 1;
  min-height: 0;
}

.tabs {
  display: flex;
  gap: 6px;
  padding: 3px;
  border-radius: 999px;
  background: rgba(19, 8, 55, 0.95);
}

.tab {
  flex: 1;
  border-radius: 999px;
  border: none;
  background: transparent;
  color: var(--text-muted);
  padding: 6px;
  font-size: 0.82rem;
  cursor: pointer;
}

.tab--active {
  background: radial-gradient(circle at top, #c084fc, #7c3aed);
  color: #fff;
  font-weight: 600;
}

.ranking-lists { flex: 1; min-height: 0; }

.ranking-list { display: none; height: 100%; }
.ranking-list--active { display: block; }

.ranking-items {
  list-style: none;
  margin: 0;
  padding: 3px 0 6px;
  max-height: 200px;
  overflow-y: auto;
}

.ranking-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 7px 10px;
  border-radius: var(--radius-md);
  background: linear-gradient(90deg, rgba(19, 8, 55, 0.95), rgba(33, 12, 85, 0.9));
  color: var(--text);
  font-size: 0.9rem;
}

.ranking-item + .ranking-item { margin-top: 6px; }

.ranking-item--placeholder {
  justify-content: center;
  color: var(--text-muted);
  font-size: 0.82rem;
}

.ranking-item--gold   { background: linear-gradient(90deg, #facc15, #b45309); color: #1f2933; }
.ranking-item--silver { background: linear-gradient(90deg, #e5e7eb, #9ca3af); color: #111827; }
.ranking-item--bronze { background: linear-gradient(90deg, #f97316, #7c2d12); color: #111827; }

.ranking-rank { min-width: 32px; font-weight: 700; }
.ranking-name { flex: 1; padding: 0 8px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.ranking-score { font-weight: 600; }

.footer { margin-top: 10px; text-align: center; }

/* ========== ゲーム画面 ========== */

.game-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 12px;
  background: rgba(0, 0, 0, 0.4);
  border-bottom: 1px solid rgba(124, 58, 237, 0.2);
  border-radius: var(--radius-md) var(--radius-md) 0 0;
}

.game-score-label {
  font-family: "Share Tech Mono", monospace;
  font-size: 1.1rem;
  color: #f59e0b;
}

.game-score-value { color: #fff; font-weight: bold; }

.game-best-label {
  font-family: "Share Tech Mono", monospace;
  font-size: 0.85rem;
  color: var(--accent-strong);
}

.quit-button {
  background: none;
  border: none;
  color: #6b7280;
  font-size: 1.1rem;
  cursor: pointer;
  padding: 4px 8px;
  line-height: 1;
}

.game-section {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(160deg, #0a0015, #051205);
  border-radius: 0 0 var(--radius-md) var(--radius-md);
  overflow: hidden;
}

/* ========== 3D フィールド ========== */

.game-field-wrapper {
  perspective: 500px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
}

.game-field {
  transform: rotateX(35deg) rotateZ(-10deg);
  transform-style: preserve-3d;
  display: grid;
  grid-template-columns: repeat(15, 1fr);
  gap: 2px;
  width: min(78vw, 380px);
  aspect-ratio: 1;
  padding: 4px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 8px;
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.8);
}

/* グラスセル */
.cell {
  border-radius: 3px;
  aspect-ratio: 1;
  position: relative;
  transition: background 0.08s;
}

.cell--grass   { background: linear-gradient(135deg, var(--grass-a), var(--grass-b)); box-shadow: 0 3px 0 var(--grass-dark); }
.cell--grass-b { background: linear-gradient(135deg, var(--grass-b), #1a4731);        box-shadow: 0 3px 0 var(--grass-dark); }

/* 体セル */
.cell--body {
  background: linear-gradient(135deg, var(--bull-body), var(--bull-body-dark)) !important;
  box-shadow: 0 5px 0 var(--bull-shadow), 0 0 10px rgba(168, 85, 247, 0.4), inset 0 1px 0 rgba(255, 255, 255, 0.25) !important;
  border-radius: 4px !important;
}

/* 頭セル */
.cell--head {
  background: none !important;
  box-shadow: none !important;
  background-image: url('./bull.png') !important;
  background-size: 130% !important;
  background-repeat: no-repeat !important;
  background-position: center !important;
  transform: rotate(var(--head-rotate, 90deg));
  z-index: 1;
  filter: drop-shadow(0 4px 8px rgba(168, 85, 247, 0.7));
}

/* 肉セル */
.cell--meat {
  background: radial-gradient(circle at 40% 35%, #f87171, #dc2626) !important;
  box-shadow: 0 4px 0 #7f1d1d, 0 0 12px rgba(239, 68, 68, 0.6) !important;
  border-radius: 4px !important;
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  font-size: clamp(8px, 1.8vw, 14px);
}

.cell--meat::after { content: '🥩'; }

/* スワイプヒント */
.swipe-hint {
  margin: 6px 0 0;
  font-size: 0.7rem;
  color: #374151;
  letter-spacing: 2px;
  text-align: center;
  font-family: "Share Tech Mono", monospace;
  transition: opacity 0.5s;
}

/* ========== ゲームオーバー ========== */

.gameover-overlay {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle at top, rgba(76, 29, 149, 0.85), rgba(15, 6, 45, 0.96));
  backdrop-filter: blur(10px);
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease-out;
  z-index: 50;
}

.gameover-overlay--visible {
  opacity: 1;
  pointer-events: auto;
}

.gameover-dialog {
  width: 100%;
  max-width: 420px;
  margin: 0 16px;
  padding: 24px 20px 26px;
  border-radius: 20px;
  background: radial-gradient(circle at top, #a855f7, #4c1d95);
  box-shadow: 0 22px 50px rgba(0, 0, 0, 0.75);
  text-align: center;
  color: #fdf4ff;
}

.gameover-title {
  margin: 0 0 16px;
  font-family: "Orbitron", sans-serif;
  font-size: 2.2rem;
  font-weight: 900;
  letter-spacing: 0.12em;
  text-shadow: 0 0 10px rgba(250, 250, 255, 0.9), 0 0 24px rgba(192, 132, 252, 0.8);
}

.gameover-score {
  margin: 6px 0;
  font-size: 1.3rem;
  font-weight: 700;
}

.gameover-score-label { color: #fdfdfd; }
.gameover-score-value { color: #fde68a; margin-left: 4px; }
.gameover-score-unit  { margin-left: 2px; }

.gameover-badge {
  margin: 8px 0 12px;
  font-size: 1.4rem;
  font-weight: 800;
  color: #fef9c3;
  text-shadow: 0 0 10px rgba(251, 191, 36, 0.9), 0 0 22px rgba(245, 158, 11, 0.9);
  animation: badge-bounce 0.9s ease-out infinite, badge-rainbow 2.2s linear infinite;
}

.gameover-badge--hidden { display: none; }

@keyframes badge-bounce {
  0%, 100% { transform: translateY(0) scale(1); }
  30%       { transform: translateY(-6px) scale(1.08); }
  60%       { transform: translateY(2px) scale(0.98); }
}

@keyframes badge-rainbow {
  0%   { color: #fef9c3; }
  25%  { color: #bfdbfe; }
  50%  { color: #f9a8d4; }
  75%  { color: #a7f3d0; }
  100% { color: #fef9c3; }
}

.gameover-actions {
  margin-top: 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.gameover-button {
  padding: 14px 20px;
  border-radius: 999px;
  border: none;
  font-family: "Orbitron", sans-serif;
  font-weight: 700;
  font-size: 1.1rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  cursor: pointer;
  min-width: 100%;
}

.gameover-button--primary {
  background: linear-gradient(135deg, #f9a8ff, #c084fc);
  color: #0b0014;
  box-shadow: 0 12px 28px rgba(192, 132, 252, 0.6);
}

.gameover-button--secondary {
  background: rgba(15, 23, 42, 0.9);
  color: #e5e7eb;
  border: 1px solid rgba(226, 232, 240, 0.45);
}

@media (min-width: 768px) {
  #app { padding: 20px 18px 24px; }
  .form-section { margin-top: 20px; }
}
```

- [ ] **Step 2: ブラウザで開いてスタイルを確認**

`index.html` をリロードし、ダーク紫背景でホーム画面のレイアウトが整っていることを確認。

- [ ] **Step 3: コミット**

```bash
git -C "Purple Bull" add styles.css
git -C "Purple Bull" commit -m "feat: add styles with 3D field CSS"
```

---

## Task 5: game.js（BullGame クラス）

**Files:**
- Create: `Purple Bull/game.js`

- [ ] **Step 1: game.js を作成**

`Purple Bull/game.js` を作成:

```js
class BullGame {
  constructor({ fieldEl, scoreEl, nickname }) {
    this.GRID  = 15;
    this.TICK  = 300;
    this.fieldEl  = fieldEl;
    this.scoreEl  = scoreEl;
    this.nickname = nickname;

    this.cells    = [];   // cells[row][col] = HTMLElement
    this.snake    = [];   // [{row, col}, ...] head は index 0
    this.dir      = { dr: 0, dc: 1 };
    this.nextDir  = { dr: 0, dc: 1 };
    this.meat     = null; // {row, col}
    this.score    = 0;
    this.running  = false;
    this.timerId  = null;
  }

  start() {
    this._buildGrid();
    this._placeSnake();
    this._placeMeat();
    this._render();
    this.running = true;
    this.timerId = setInterval(() => this._tick(), this.TICK);
  }

  destroy() {
    clearInterval(this.timerId);
    this.running = false;
  }

  setDirection(dr, dc) {
    // 逆方向転換を無視（即死防止）
    if (dr === -this.dir.dr && dc === -this.dir.dc) return;
    this.nextDir = { dr, dc };
  }

  // ---- private ----

  _buildGrid() {
    this.fieldEl.innerHTML = '';
    this.cells = [];
    const frag = document.createDocumentFragment();
    for (let r = 0; r < this.GRID; r++) {
      this.cells[r] = [];
      for (let c = 0; c < this.GRID; c++) {
        const el = document.createElement('div');
        el.className = 'cell ' + ((r + c) % 2 === 0 ? 'cell--grass' : 'cell--grass-b');
        this.cells[r][c] = el;
        frag.appendChild(el);
      }
    }
    this.fieldEl.appendChild(frag);
  }

  _placeSnake() {
    const mid = Math.floor(this.GRID / 2);
    this.snake = [
      { row: mid, col: mid + 1 },
      { row: mid, col: mid },
      { row: mid, col: mid - 1 },
    ];
    this.dir     = { dr: 0, dc: 1 };
    this.nextDir = { dr: 0, dc: 1 };
    this.score   = 0;
  }

  _placeMeat() {
    const snakeSet = new Set(this.snake.map(s => s.row * this.GRID + s.col));
    const empty = [];
    for (let r = 0; r < this.GRID; r++) {
      for (let c = 0; c < this.GRID; c++) {
        if (!snakeSet.has(r * this.GRID + c)) empty.push({ row: r, col: c });
      }
    }
    this.meat = empty[Math.floor(Math.random() * empty.length)] ?? null;
  }

  _render() {
    // 全セルをグラスに戻す
    for (let r = 0; r < this.GRID; r++) {
      for (let c = 0; c < this.GRID; c++) {
        const el = this.cells[r][c];
        el.className = 'cell ' + ((r + c) % 2 === 0 ? 'cell--grass' : 'cell--grass-b');
        el.style.removeProperty('--head-rotate');
      }
    }

    // 肉
    if (this.meat) {
      this.cells[this.meat.row][this.meat.col].classList.add('cell--meat');
    }

    // 体（末尾から先頭の順で塗ることで頭が上に重なる）
    for (let i = this.snake.length - 1; i >= 0; i--) {
      const { row, col } = this.snake[i];
      const el = this.cells[row][col];
      if (i === 0) {
        el.classList.add('cell--head');
        el.style.setProperty('--head-rotate', this._headRotateDeg() + 'deg');
      } else {
        el.classList.add('cell--body');
      }
    }
  }

  _headRotateDeg() {
    const { dr, dc } = this.dir;
    if (dc === 1)  return 90;   // right
    if (dc === -1) return 270;  // left
    if (dr === 1)  return 180;  // down
    return 0;                   // up
  }

  _tick() {
    this.dir = this.nextDir;
    const head = this.snake[0];
    const next = { row: head.row + this.dir.dr, col: head.col + this.dir.dc };

    // 壁衝突
    if (next.row < 0 || next.row >= this.GRID || next.col < 0 || next.col >= this.GRID) {
      this._gameOver(); return;
    }

    // 自己衝突
    if (this.snake.some(s => s.row === next.row && s.col === next.col)) {
      this._gameOver(); return;
    }

    const ate = this.meat && next.row === this.meat.row && next.col === this.meat.col;
    this.snake.unshift(next);
    if (ate) {
      this.score++;
      if (this.scoreEl) this.scoreEl.textContent = this.score;
      this._placeMeat();
    } else {
      this.snake.pop();
    }

    this._render();
  }

  _gameOver() {
    this.destroy();
    if (typeof window.handleBullGameOver === 'function') {
      window.handleBullGameOver({ nickname: this.nickname, score: this.score });
    }
  }
}

window.BullGame = BullGame;
```

- [ ] **Step 2: ブラウザコンソールで動作確認**

`index.html` をリロードし、ブラウザコンソールで以下を実行してエラーがないことを確認:

```js
const g = new BullGame({ fieldEl: document.getElementById('game-field'), scoreEl: document.getElementById('score-value'), nickname: 'test' });
console.log(g);  // BullGame インスタンスが表示されること
```

- [ ] **Step 3: コミット**

```bash
git -C "Purple Bull" add game.js
git -C "Purple Bull" commit -m "feat: add BullGame class with snake logic"
```

---

## Task 6: main.js（画面制御・スワイプ・ランキング）

**Files:**
- Create: `Purple Bull/main.js`

- [ ] **Step 1: main.js を作成**

`Purple Bull/main.js` を作成:

```js
document.addEventListener('DOMContentLoaded', () => {
  const screenHome    = document.getElementById('screen-home');
  const screenGame    = document.getElementById('screen-game');
  const nicknameInput = document.getElementById('nickname');
  const startButton   = document.getElementById('start-button');
  const tabButtons    = document.querySelectorAll('.tab');
  const gameoverOverlay     = document.getElementById('gameover-overlay');
  const gameoverScoreValue  = document.getElementById('gameover-score-value');
  const gameoverBadge       = document.getElementById('gameover-badge');
  const retryButton   = document.getElementById('gameover-retry');
  const homeButton    = document.getElementById('gameover-home');
  const scoreEl       = document.getElementById('score-value');
  const bestEl        = document.getElementById('best-value');
  const swipeHint     = document.getElementById('swipe-hint');
  const quitButton    = document.getElementById('quit-button');
  const errorBanner   = document.getElementById('error-banner');

  const BEST_KEY = 'purpleBullBest';

  let currentGame = null;
  let lastNickname = null;
  let gameoverAnimId = null;

  // エラー表示
  window.showGameError = (msg) => {
    if (!errorBanner) { console.error(msg); return; }
    errorBanner.textContent = String(msg);
    errorBanner.style.display = 'block';
    setTimeout(() => { if (errorBanner.textContent === msg) errorBanner.style.display = 'none'; }, 5000);
  };

  // ダブルタップズーム防止
  let lastTouchEnd = 0;
  document.addEventListener('touchend', (e) => {
    const now = performance.now();
    if (now - lastTouchEnd < 300) e.preventDefault();
    lastTouchEnd = now;
  }, { passive: false });

  // ベストスコア表示更新
  const updateBestDisplay = () => {
    const best = Number(localStorage.getItem(BEST_KEY) || '0');
    if (bestEl) bestEl.textContent = best;
  };
  updateBestDisplay();

  // タブ切り替え
  tabButtons.forEach((btn) => {
    btn.addEventListener('click', () => {
      const target = btn.dataset.tab;
      tabButtons.forEach((b) => {
        b.classList.toggle('tab--active', b === btn);
        b.setAttribute('aria-selected', b === btn ? 'true' : 'false');
      });
      document.querySelectorAll('.ranking-list').forEach((p) => {
        p.classList.toggle('ranking-list--active', p.dataset.panel === target);
      });
      loadRankingAfterConnection(target);
    });
  });

  // ゲーム開始
  const startGame = (nickname) => {
    if (currentGame) { currentGame.destroy(); currentGame = null; }

    lastNickname = nickname;
    scoreEl.textContent = '0';
    updateBestDisplay();

    // スワイプヒントを初回のみ表示
    if (swipeHint) swipeHint.style.opacity = '1';

    currentGame = new BullGame({
      fieldEl:  document.getElementById('game-field'),
      scoreEl,
      nickname,
    });
    currentGame.start();
  };

  startButton.addEventListener('click', () => {
    const nickname = nicknameInput.value.trim();
    if (!nickname) { alert('ニックネームを入力してください。'); nicknameInput.focus(); return; }

    screenHome.classList.remove('screen--active');
    screenGame.classList.add('screen--active');
    startGame(nickname);
  });

  // ゲームオーバー処理
  window.handleBullGameOver = async ({ nickname, score }) => {
    // スコア表示カウントアップ
    if (gameoverScoreValue) {
      if (gameoverAnimId != null) { cancelAnimationFrame(gameoverAnimId); gameoverAnimId = null; }
      const duration = 700;
      const start = performance.now();
      const tick = (now) => {
        const t = Math.min(1, (now - start) / duration);
        const eased = 1 - Math.pow(1 - t, 3);
        gameoverScoreValue.textContent = Math.floor(score * eased);
        if (t < 1) {
          gameoverAnimId = requestAnimationFrame(tick);
        } else {
          gameoverAnimId = null;
          gameoverScoreValue.textContent = score;
        }
      };
      gameoverScoreValue.textContent = '0';
      gameoverAnimId = requestAnimationFrame(tick);
    }

    // ベストスコア判定
    const prev = Number(localStorage.getItem(BEST_KEY) || '0');
    const isNew = score > prev;
    if (isNew) localStorage.setItem(BEST_KEY, String(score));

    if (gameoverBadge) {
      gameoverBadge.textContent = 'New Record!';
      gameoverBadge.classList.toggle('gameover-badge--hidden', !isNew);
    }

    if (gameoverOverlay) {
      gameoverOverlay.classList.add('gameover-overlay--visible');
      gameoverOverlay.setAttribute('aria-hidden', 'false');
    }

    // Supabase スコア投稿
    if (typeof submitScore === 'function') {
      const { error } = await submitScore({ nickname, score });
      if (error) console.error('スコア投稿エラー:', error);
    }
  };

  const hideGameover = () => {
    if (gameoverOverlay) {
      gameoverOverlay.classList.remove('gameover-overlay--visible');
      gameoverOverlay.setAttribute('aria-hidden', 'true');
    }
  };

  retryButton?.addEventListener('click', () => {
    hideGameover();
    startGame(lastNickname || nicknameInput.value.trim());
  });

  homeButton?.addEventListener('click', () => {
    hideGameover();
    if (currentGame) { currentGame.destroy(); currentGame = null; }
    screenGame.classList.remove('screen--active');
    screenHome.classList.add('screen--active');
    updateBestDisplay();
    loadRankingAfterConnection('today');
  });

  quitButton?.addEventListener('click', () => {
    if (currentGame) { currentGame.destroy(); currentGame = null; }
    screenGame.classList.remove('screen--active');
    screenHome.classList.add('screen--active');
    updateBestDisplay();
    loadRankingAfterConnection('today');
  });

  // スワイプ操作
  const SWIPE_MIN = 30;
  let touchStartX = 0, touchStartY = 0;

  document.getElementById('screen-game')?.addEventListener('touchstart', (e) => {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
  }, { passive: true });

  document.getElementById('screen-game')?.addEventListener('touchend', (e) => {
    if (!currentGame) return;
    const dx = e.changedTouches[0].clientX - touchStartX;
    const dy = e.changedTouches[0].clientY - touchStartY;
    if (Math.abs(dx) < SWIPE_MIN && Math.abs(dy) < SWIPE_MIN) return;

    if (Math.abs(dx) > Math.abs(dy)) {
      currentGame.setDirection(0, dx > 0 ? 1 : -1);
    } else {
      currentGame.setDirection(dy > 0 ? 1 : -1, 0);
    }

    // 初回スワイプでヒントを消す
    if (swipeHint) swipeHint.style.opacity = '0';
  }, { passive: true });

  // 初期ランキング読み込み
  (async () => { await loadRankingAfterConnection('today'); })();
});

// ランキング取得・表示
async function loadRankingAfterConnection(range) {
  const map = {
    today: document.getElementById('ranking-today'),
    week:  document.getElementById('ranking-week'),
    all:   document.getElementById('ranking-all'),
  };
  const listEl = map[range];
  if (!listEl) return;

  listEl.innerHTML = '<li class="ranking-item ranking-item--placeholder">読み込み中…</li>';

  if (typeof window.waitForSupabaseConnection === 'function') {
    const { connected } = await window.waitForSupabaseConnection(5000);
    if (!connected) {
      listEl.innerHTML = '<li class="ranking-item ranking-item--placeholder">オフラインのためランキングを表示できません。</li>';
      return;
    }
  }

  if (typeof fetchRanking !== 'function') {
    listEl.innerHTML = '<li class="ranking-item ranking-item--placeholder">Supabase未設定のため表示できません。</li>';
    return;
  }

  const { data, error, skipped } = await fetchRanking(range);
  if (skipped || error || !data || data.length === 0) {
    listEl.innerHTML = '<li class="ranking-item ranking-item--placeholder">' +
      (skipped ? 'オフラインのためランキングを表示できません。' : error ? 'ランキングの取得に失敗しました。' : 'まだスコアが登録されていません。') +
      '</li>';
    return;
  }

  listEl.innerHTML = '';
  data.forEach((row, idx) => {
    const rank = idx + 1;
    let extraClass = '', medal = '';
    if (rank === 1) { extraClass = ' ranking-item--gold';   medal = '🥇'; }
    else if (rank === 2) { extraClass = ' ranking-item--silver'; medal = '🥈'; }
    else if (rank === 3) { extraClass = ' ranking-item--bronze'; medal = '🥉'; }

    const li = document.createElement('li');
    li.className = 'ranking-item' + extraClass;
    li.innerHTML =
      `<span class="ranking-rank">${medal || rank}</span>` +
      `<span class="ranking-name">${escapeHtml(row.nickname ?? 'No Name')}</span>` +
      `<span class="ranking-score">${row.score ?? 0} 🥩</span>`;
    listEl.appendChild(li);
  });
}

function escapeHtml(str) {
  return str
    .replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;').replace(/'/g, '&#039;');
}
```

- [ ] **Step 2: ブラウザで動作確認（主要フロー）**

1. `index.html` をブラウザで開く
2. ニックネームを入力して「GAME START」→ ゲーム画面に切り替わること
3. 3D フィールドに 15×15 グリッドが斜め俯瞰で表示されること
4. 牛の頭画像（`bull.png`）が頭セルに表示されること
5. 画面をスワイプすると牛が方向転換すること
6. 🥩 を食べると体が1マス伸び、スコアが増加すること
7. 壁または自分の体に当たるとゲームオーバーモーダルが表示されること
8. 「再挑戦」で同じニックネームで再スタートできること
9. 「トップへ戻る」でホーム画面に戻ること

- [ ] **Step 3: コミット**

```bash
git -C "Purple Bull" add main.js
git -C "Purple Bull" commit -m "feat: add main.js with swipe control and ranking display"
```

---

## Task 7: ロゴ画像配置と最終確認

**Files:**
- Modify: `Purple Bull/` （logo.png 配置）

- [ ] **Step 1: logo.png を配置**

Purple Diver の `logo.png` を `Purple Bull/logo.png` にコピーする（Purple Park ロゴ共通素材）:

```bash
cp "Purple Diver/logo.png" "Purple Bull/logo.png"
```

- [ ] **Step 2: スマートフォンでの動作確認**

ローカルサーバーを立ち上げてスマホからアクセスし確認する:

```bash
cd "Purple Bull" && python -m http.server 8080
```

スマホのブラウザで `http://<PC の IP アドレス>:8080` を開き、以下を確認:
- スワイプ操作が正常に動作すること
- 3D フィールドが正しく表示されること
- ランキングタブが表示されること（Supabase 接続時）

- [ ] **Step 3: 最終コミット**

```bash
git -C "Purple Bull" add logo.png
git -C "Purple Bull" commit -m "feat: Purple Bull complete - snake game with 3D field and ranking"
```

---

## チェックリスト（完了条件）

- [ ] 15×15 の CSS 3D フィールドが斜め俯瞰で表示される
- [ ] 牛の頭画像が進行方向に回転する
- [ ] スワイプで方向転換できる（逆方向への転換は無視される）
- [ ] 肉を食べると体が1マス伸び、スコアが増加する
- [ ] 壁・自己衝突でゲームオーバーになる
- [ ] ゲームオーバー時に Supabase にスコアが投稿される
- [ ] 今日・週間・歴代のランキングが表示される
- [ ] New Record バッジが自己ベスト更新時に表示される
- [ ] 再挑戦・トップへ戻るが正常に動作する
