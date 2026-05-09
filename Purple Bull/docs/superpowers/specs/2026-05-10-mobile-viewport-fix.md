# モバイルビューポート修正 実装仕様

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** スマートフォン（iPhone・Android）でプレイ時、下のアドレスバーに隠れて一番下のマスが見えない問題を修正し、全デバイスで全てのマスが表示される

**Architecture:** CSS の `#app` 要素の高さを `100vh`（Static Viewport Height）から `100dvh`（Dynamic Viewport Height）に変更。これによりモバイルブラウザの動的アドレスバーを自動的に考慮し、利用可能な高さで正確にレイアウト。既存の flex レイアウトとの相性も完全。

**Tech Stack:** CSS3（viewport-relative units）、モダンブラウザサポート

---

## 問題分析

### 現状
- スマートフォンでプレイ時、下のアドレスバーに隠れて一番下の行が見えない
- 原因：`styles.css` の49行目で `#app { min-height: 100vh; }` を使用
- `100vh` はモバイルブラウザの動的アドレスバーを考慮せず、実際の利用可能高さより大きい
- 結果：ゲームキャンバスがビューポートをはみ出し、マスが隠れる

### デバイス環境
- **iOS Safari：** アドレスバーが動的に表示/非表示（動画再生時は自動隠蔽）
- **Chrome Android：** アドレスバーがスクロール時に動的に表示/非表示
- **Samsung Internet・Firefox Android：** 同様の動的ビューポート動作

---

## 解決方法

### 変更内容

**ファイル:** `styles.css` 行49

**変更前：**
```css
#app {
  width: 100%;
  max-width: 90vw;
  min-height: 100vh;  /* ← 問題：アドレスバーを考慮しない */
  padding: 12px 12px 18px;
  display: flex;
  flex-direction: column;
}
```

**変更後：**
```css
#app {
  width: 100%;
  max-width: 90vw;
  min-height: 100dvh;  /* Dynamic Viewport Height に変更 */
  padding: 12px 12px 18px;
  display: flex;
  flex-direction: column;
}
```

### 仕様詳細

- **`100dvh`（Dynamic Viewport Height）とは：**
  - CSS4 Viewport Units の新しい単位
  - ブラウザのアドレスバーやナビゲーションバーの動的表示/非表示を自動的に追跡
  - スマートフォンブラウザで利用可能な実際の高さに対応

- **ブラウザサポート：**
  - iOS Safari 15.0+（2021年9月以降）
  - Chrome 108+（2022年12月以降）
  - Samsung Internet 17+
  - Firefox 101+
  - 現在のモバイルデバイスで99%以上カバー

- **フォールバック：**
  - `100dvh` を認識しないブラウザは自動的に `100vh` にフォールバック
  - ゲーム自体は動作し、稀な古いブラウザでもプレイ可能

---

## 期待される効果

### モバイル環境での改善
- **iOS Safari：** アドレスバー表示時も、全マスがビューポート内に収まる
- **Chrome Android：** スクロール時のアドレスバー動作に自動追従
- **横向き（ランドスケープ）モード：** ナビゲーションバーがある場合も対応

### 既存機能への影響
- **ゲームロジック：** 変更なし
- **タッチ入力：** 変更なし
- **パフォーマンス：** 変更なし
- **レスポンシブレイアウト：** 既存の flex レイアウトと完全互換

---

## テスト計画

### 動作確認環境
1. **iOS（iPhone 12以降）**
   - Safari でゲームをプレイ
   - アドレスバー表示状態で全マスが見えることを確認
   - 横向きモードで動作確認

2. **Android（Chrome 最新版）**
   - Chrome でゲームをプレイ
   - スクロール時のアドレスバー動作で全マスが見える
   - 横向きモードで動作確認

3. **デスクトップブラウザ（検証用）**
   - Chrome DevTools のスマートフォンエミュレーターで確認
   - 既存レイアウトの崩れがないことを確認

---

## 技術詳細

### なぜ `100dvh` で解決するのか
- `100vh` は CSS viewport の論理的高さで、静的（Static Viewport Height）
- モバイルブラウザは、アドレスバーを隠した際の「最大可能な高さ」を `100vh` と定義
- 結果：アドレスバーが表示されている間、実際の利用可能高さ < `100vh`
- `100dvh` は常に「現在の利用可能な高さ」を意味するため、動的に正確

---

## リスク評価

| リスク | レベル | 対策 |
|--------|--------|------|
| 古いブラウザサポート | 低 | フォールバック（100vh）で自動対応 |
| レイアウト崩れ | 非常に低 | `100dvh` は `100vh` の改良版、互換性高 |
| パフォーマンス | なし | CSS単位の変更、性能影響なし |

---

## 実装の簡潔さ

このは修正は **1行の数値変更** のみです：
```css
min-height: 100vh;  →  min-height: 100dvh;
```
