# Purple Bull — ゲーム設計書

**作成日：** 2026-05-01  
**ステータス：** 承認済み

---

## 概要

Purple Park シリーズの新作ブラウザゲーム。紫の闘牛キャラクターを操作するスネークゲーム。肉を食べると体が1マスずつ伸び、壁または自分の体に衝突するとゲームオーバー。スマートフォン向けスワイプ操作、Supabaseオンラインランキング対応。

---

## ゲームメカニクス

| 項目 | 仕様 |
|---|---|
| グリッドサイズ | 15×15 |
| スタート体長 | 3マス |
| 移動速度 | 300ms/マス（終始一定・速度変化なし） |
| スコア単位 | 食べた肉の枚数 |
| ゲームオーバー条件 | 壁への衝突 / 自分の体への衝突 |
| アイテム | 肉（🥩）が1個ずつランダム配置 |

### 操作

- **スワイプ**（上下左右）で進行方向を変更
- 判定距離：30px 以上のタッチ移動
- 逆方向への転換は無効（即死防止）
- タッチイベント優先のスマホ最適化

---

## 画面構成

### ① ホーム画面

- Purple Park ロゴリンク（左上）
- ゲームタイトル「🐂 PURPLE BULL」
- ニックネーム入力フィールド（最大12文字）
- GAME START ボタン
- オンラインランキング（今日 / 週間 / 歴代 タブ、各10件）

### ② ゲーム画面

- ヘッダー：スコア表示（🥩 枚数）/ ベストスコア / 終了ボタン
- 3Dフィールド：CSS `rotateX` + `rotateZ` による斜め俯瞰表示
- スワイプヒントラベル（初回のみ表示）

### ③ ゲームオーバーモーダル

- 今回のスコア
- New Record バッジ（自己ベスト更新時）
- 「再挑戦」ボタン / 「トップへ戻る」ボタン
- Supabase へのスコア自動投稿

---

## ビジュアルデザイン

### レンダリング方式

**CSS DOM + 3D Transform**（Canvas 不使用）

- 各セルを `<div>` で管理
- フィールド全体に `perspective: 400px` + `rotateX(35deg) rotateZ(-10deg)` を適用
- セルに `box-shadow` でブロックの厚みを表現

### カラースキーム

| 要素 | カラー |
|---|---|
| 背景 | `#0a0015` → `#051205` グラデーション |
| フィールドタイル（明） | `#166534` → `#15803d` |
| フィールドタイル（暗） | `#15803d` → `#1a4731` |
| 牛の体ブロック | `#a855f7` → `#7c3aed`、影 `#4c1d95` |
| 牛の頭 | カスタム画像（`bull.png`、透過PNG） |
| 肉アイテム | `#ef4444` → `#dc2626`、赤グロー |
| UI アクセント | `#7c3aed`（紫）|

### キャラクター

- **頭セル**：`bull.png`（透過PNG、ユーザー提供）をセルサイズにフィット表示
- 進行方向に応じて `rotate` で向きを変える（右:0°, 下:90°, 左:180°, 上:270°）
- **体セル**：紫グラデーションの角丸ブロック、`box-shadow` で立体感

---

## データ設計（Supabase）

Purple Diver（`diver_scores`）と同じ構造で、専用テーブル `bull_scores` を新規作成。

```sql
create table bull_scores (
  id          uuid primary key default gen_random_uuid(),
  nickname    text,
  score       integer,
  created_at  timestamptz default now()
);
```

- ランキング取得：`bull_scores` テーブルをクエリ
- 今日：`created_at >= today`
- 週間：`created_at >= 7日前`
- 歴代：全件
- 同一ニックネームは最高スコアのみ表示（Purple Diver と同じ重複排除ロジック）

---

## ファイル構成

```
Purple Bull/
├── index.html          ← 画面HTML（ホーム・ゲーム・モーダル）
├── styles.css          ← スタイル（3Dフィールド・UI）
├── game.js             ← ゲームロジック（スネーク・衝突判定・DOM更新）
├── main.js             ← 画面制御・スワイプイベント・ランキング表示
├── supabase-api.js     ← スコア投稿・ランキング取得
├── supabase-config.js  ← Supabase接続設定（.gitignore対象）
├── supabase-config.example.js
├── bull.png            ← 牛キャラクター画像（透過PNG）
└── manifest.json       ← PWA対応
```

---

## 技術スタック

- **言語**：HTML / CSS / Vanilla JS（フレームワーク不使用）
- **3D**：CSS Transform（`perspective` / `rotateX` / `rotateZ` / `box-shadow`）
- **DB**：Supabase（既存プロジェクト流用）
- **フォント**：Orbitron（タイトル）/ Share Tech Mono（スコア）

---

## 既存ゲームとの違い

| | Purple Diver | Purple Bull |
|---|---|---|
| レンダリング | HTML5 Canvas | CSS DOM + 3D Transform |
| 操作 | タップ | スワイプ |
| キャラ | Canvas 描画 | 画像ファイル（PNG） |
| スコア単位 | 深度（m） | 肉の枚数 |
