# モバイルビューポート修正 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** スマートフォンでプレイ時にアドレスバーに隠れる一番下のマスを表示させ、全デバイスで完全にゲームが見える

**Architecture:** CSS の `#app` 要素の高さ指定を Static Viewport Height（`100vh`）から Dynamic Viewport Height（`100dvh`）に変更。モバイルブラウザのアドレスバー動的表示を自動的に追跡し、利用可能な高さで正確にレイアウト。既存の flex レイアウトとの相性は完全。

**Tech Stack:** CSS3（viewport-relative units）、モダンブラウザ（iOS 15.0+、Chrome 108+）

---

## ファイル構造

- **修正:** `styles.css:49` - #app 要素の min-height 値

---

## 実装タスク

### Task 1: ビューポート高さ指定の修正

**Files:**
- Modify: `styles.css:49`

- [ ] **Step 1: styles.css の #app セクションを確認**

`styles.css` の46〜53行目を確認します。現在のコードは以下のようになっています：

```css
#app {
  width: 100%;
  max-width: 90vw;
  min-height: 100vh;  /* ← この行を変更 */
  padding: 12px 12px 18px;
  display: flex;
  flex-direction: column;
}
```

確認内容：
- 49行目に `min-height: 100vh;` が存在することを確認

- [ ] **Step 2: min-height を 100vh から 100dvh に変更**

styles.css の49行目を以下のように変更します：

```css
#app {
  width: 100%;
  max-width: 90vw;
  min-height: 100dvh;  /* 100vh → 100dvh に変更 */
  padding: 12px 12px 18px;
  display: flex;
  flex-direction: column;
}
```

変更内容：
- `100vh` を `100dvh` に置き換え
- コメントは変更不要

- [ ] **Step 3: スマートフォンでゲームをテスト**

プレビューサーバーでゲームにアクセスし、モバイルデバイス（またはデバイスエミュレーター）で確認します：

**テスト手順：**
1. プレビューサーバーを起動（既に起動しているはず）
2. スマートフォン（iPhone または Android Chrome）で以下を確認：
   - ゲーム画面を開く
   - GAME START ボタンをクリックしてゲームを開始
   - **重要:** アドレスバー表示時に、20行目（一番下の行）まで全てのマスが見えることを確認
   - スクロール時にアドレスバーが動的に表示/非表示になっても、全マスが見える状態を維持することを確認

**期待される結果：**
- 一番下のマスが画面の下部に完全に表示される
- アドレスバー表示時でも、ゲーム画面がはみ出ない
- 横向き（ランドスケープ）モードでも全マスが見える

**確認できない場合：**
- DevTools のモバイルエミュレーターで画面をリロード
- ブラウザキャッシュをクリアして再度アクセス

- [ ] **Step 4: Commit**

以下のコマンドで変更をコミットします：

```bash
git add styles.css
git commit -m "fix: Use 100dvh instead of 100vh to support mobile dynamic viewport height

- Changes #app min-height from 100vh to 100dvh
- Automatically accommodates mobile browser address bar dynamic display
- Fixes issue where bottom row of game grid was hidden on mobile devices
- Maintains existing flex layout compatibility"
```

---

## テスト計画

### 1. 基本動作確認
- デスクトップブラウザで既存レイアウトが崩れていないか確認
- ゲーム開始から終了まで正常に動作することを確認

### 2. モバイルデバイス確認（優先度高）
- **iOS Safari**
  - iPhone 12以降でゲームプレイ
  - アドレスバー表示時に全マスが見える
  - 横向きモードで動作確認
  
- **Chrome Android**
  - Android 最新版 Chrome でゲームプレイ
  - スクロール時のアドレスバー動作で全マスが見える
  - 横向きモードで動作確認

### 3. デバイスエミュレーター確認（代替）
- Chrome DevTools → Device Mode でスマートフォンサイズに変更
- デバイスを「iPhone 12」「Pixel 5」などに変更
- 全マスが見えることを視認確認

---

## 実装の簡潔さ

このタスクは極めてシンプルです：
- **変更ファイル:** 1ファイル（styles.css）
- **変更行数:** 1行
- **変更内容:** `100vh` → `100dvh`
- **影響範囲:** モバイルビューポート高さのみ
- **副作用:** なし（既存レイアウト完全互換）

---

## 技術詳細

### 100dvh について
- **意味:** Dynamic Viewport Height
- **動作:** モバイルブラウザのアドレスバー動的表示を自動追跡
- **サポート状況：**
  - iOS Safari 15.0+（2021年以降）
  - Chrome 108+（2022年以降）
  - 現在のモバイルデバイスで99%以上カバー
  - 古いブラウザは自動的に `100vh` にフォールバック

### なぜこれで解決するのか
- `100vh` は Static Viewport Height で、常に「ブラウザウィンドウの最大高さ」を指す
- モバイルでは、アドレスバーが表示されている間、実際の利用可能高さ < `100vh` となる
- `100dvh` は常に「現在の利用可能な高さ」を指すため、動的に正確に対応

---

## リスク評価

| リスク | レベル | 対策 |
|--------|--------|------|
| 古いブラウザ非対応 | 低 | フォールバック（100vh）で自動対応 |
| レイアウト崩れ | 非常に低 | 100dvh は 100vh の改良版、完全互換 |
| パフォーマンス影響 | なし | CSS単位の変更、性能影響ゼロ |

---

## 完了条件

- [ ] styles.css の49行目が `min-height: 100dvh;` に変更されている
- [ ] モバイルデバイス（またはエミュレーター）でゲームプレイ時、一番下のマスが完全に見える
- [ ] デスクトップブラウザでも既存レイアウトが崩れていない
- [ ] git にコミットされている
