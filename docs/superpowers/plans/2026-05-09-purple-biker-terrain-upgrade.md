# Purple Biker 地面色アップデート実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Purple Bikerの地面ブロックをメタリック・グラデーション + 時間帯対応の発光ラインに変更し、最先端ゲーム風の洗練された質感を実現する

**Architecture:** 
既存の地面描画ループ（1644-1672行）内で、単色塗りをリニアグラデーションに置き換え。昼間は明るいメタリック+薄い白の発光、夜間は暗いメタリック+ネオンマゼンタの発光を適用。`isDaytime`変数で昼夜を判定し、対応する色設定を分岐処理。

**Tech Stack:** 
- Canvas API（`createLinearGradient`）
- JavaScript（既存）
- 色コード（16進数 + rgba）

---

## ファイル構造

**修正対象ファイル：**
- `Purple Biker/index.html`（1644-1672行の地面描画ループ）

**新規作成：** なし

---

## Task 1: 現在のコード構造を確認

**Files:**
- Review: `Purple Biker/index.html:1644-1672`

- [ ] **Step 1: 現在の地面描画コードを確認**

ファイル内の1644-1672行の地面描画ループを確認します。以下の構造であることを確認：
- `for (const block of terrain)` ループ
- `isDaytime` で昼夜分岐
- 昼間：`ctx.fillStyle = '#808080'` + 紫ネオン縁（`#9b5bff`）
- 夜間：`ctx.fillStyle = '#06051b'` + 紫ネオン縁（`#9b5bff`）

**確認項目：**
- ループ内で `block.x`, `block.y`, `block.w`, `block.h` を使用している
- `sx` = スクリーン座標（`block.x - worldOffset`）
- `drawW` = `block.w + 1`
- 現在の縁線描画：`ctx.beginPath()` → `ctx.moveTo()` → `ctx.lineTo()` → `ctx.stroke()`

---

## Task 2: 昼間の地面描画を実装（グラデーション + 薄い白発光ライン）

**Files:**
- Modify: `Purple Biker/index.html:1649-1661`

- [ ] **Step 1: 昼間の地面グラデーションを実装**

1649行の `if (isDaytime) {` ブロック内で、以下のように変更します：

```javascript
if (isDaytime) {
  // グラデーション：上部#3a4a5e（明るめメタリック） → 下部#1a2a3e（暗いメタリック）
  const gradient = ctx.createLinearGradient(sx, block.y, sx, block.y + block.h);
  gradient.addColorStop(0, '#3a4a5e');
  gradient.addColorStop(1, '#1a2a3e');
  ctx.fillStyle = gradient;
  ctx.fillRect(sx, block.y, drawW, block.h);
  
  // 発光ライン：薄い白 rgba(255, 255, 255, 0.6)
  ctx.shadowColor = 'rgba(255, 255, 255, 0.4)';
  ctx.shadowBlur = 8;
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.6)';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(sx, block.y + 0.5);
  ctx.lineTo(sx + drawW, block.y + 0.5);
  ctx.stroke();
  ctx.shadowColor = 'transparent';
  ctx.shadowBlur = 0;
}
```

**確認項目：**
- グラデーション座標が正しい（`sx, block.y` → `sx, block.y + block.h`）
- `addColorStop(0, ...)` が上部、`addColorStop(1, ...)` が下部
- 発光ライン色が `rgba(255, 255, 255, 0.6)` で統一されている
- グロー色が `rgba(255, 255, 255, 0.4)` である

---

## Task 3: 夜間の地面描画を実装（グラデーション + ネオンマゼンタ発光ライン）

**Files:**
- Modify: `Purple Biker/index.html:1662-1671`

- [ ] **Step 1: 夜間の地面グラデーションを実装**

1662行の `else {` ブロックを、以下のように変更します：

```javascript
} else {
  // グラデーション：上部#1a2a3e（暗いメタリック） → 下部#000005（ほぼ黒）
  const gradient = ctx.createLinearGradient(sx, block.y, sx, block.y + block.h);
  gradient.addColorStop(0, '#1a2a3e');
  gradient.addColorStop(1, '#000005');
  ctx.fillStyle = gradient;
  ctx.fillRect(sx, block.y, drawW, block.h);
  
  // 発光ライン：ネオンマゼンタ #ff00ff
  ctx.shadowColor = '#ff00ff';
  ctx.shadowBlur = 12;
  ctx.globalAlpha = 0.8;
  ctx.strokeStyle = '#ff00ff';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(sx, block.y + 0.5);
  ctx.lineTo(sx + drawW, block.y + 0.5);
  ctx.stroke();
  ctx.globalAlpha = 1.0;
  ctx.shadowColor = 'transparent';
  ctx.shadowBlur = 0;
}
```

**確認項目：**
- グラデーション座標が正しい（`sx, block.y` → `sx, block.y + block.h`）
- `addColorStop(0, ...)` が上部、`addColorStop(1, ...)` が下部
- 発光ライン色が `#ff00ff`（マゼンタ）である
- グロー色も `#ff00ff` で統一
- `globalAlpha = 0.8` で透明度80%を設定
- 最後に `globalAlpha = 1.0` でリセット

---

## Task 4: コード修正内容を確認＆視認性テスト

**Files:**
- Review: `Purple Biker/index.html:1644-1672`（変更後）

- [ ] **Step 1: 修正内容の確認**

以下の点を確認：
- 1644行目の注釈を更新（必要に応じて）
- 昼間ブロック（1649-1661行）がグラデーション + 薄い白発光に変わっている
- 夜間ブロック（1662-1671行）がグラデーション + マゼンタ発光に変わっている
- 古い単色コード（`#808080`, `#06051b`）が削除されている
- グロー設定が昼夜で異なっている（昼：`shadowBlur = 8`、夜：`shadowBlur = 12`）

---

## Task 5: プレビューで昼間プレイを確認

**Files:**
- Test: ブラウザプレビュー（昼間）

- [ ] **Step 1: ローカルサーバーを起動**

```bash
cd "/c/Users/hayato.takagaki/OneDrive/デスクトップ/Claude（Purple Park）/Purple Biker"
python -m http.server 8000
```

- [ ] **Step 2: ブラウザで http://localhost:8000 を開く**

- [ ] **Step 3: ゲーム開始してプレイ（昼間を確認）**

**確認項目：**
- 地面の色が明るいメタリック・グラデーション（上部#3a4a5e → 下部#1a2a3e）
- 地面上部に薄い白の発光ライン（光の反射）が見える
- グロー効果で白い光が上部に見える
- プレイヤーが地面に対して視認性良好
- フレームレート低下がない

**問題あれば：** コンソール（F12）でエラーをチェック。グラデーション座標が間違っていないか確認。

---

## Task 6: プレビューで夜間プレイを確認

**Files:**
- Test: ブラウザプレビュー（夜間）

- [ ] **Step 1: ゲームをリセット＆プレイを続ける（時間経過で夜間へ）**

夜間に自動的に切り替わるまでプレイするか、背景が暗くなったことを確認。

**確認項目：**
- 背景が暗くなった（夜間判定が正しく働いている）
- 地面の色が暗いメタリック・グラデーション（上部#1a2a3e → 下部#000005）
- 地面上部にネオンマゼンタの発光ライン（#ff00ff）が見える
- マゼンタのグロー効果（shadowBlur = 12）で派手な光が見える
- プレイヤーがマゼンタ発光でも視認性良好
- Purple Bikerの世界観に合っている

**問題あれば：** 
- マゼンタ色が見えない → `globalAlpha = 0.8` の設定を確認
- グロー効果が弱い → `shadowBlur = 12` を増やす（最大20程度）

---

## Task 7: 昼夜切り替え時の色遷移を確認

**Files:**
- Test: ブラウザプレビュー（昼夜交互）

- [ ] **Step 1: ゲームをプレイして背景が昼→夜に切り替わる瞬間を観察**

背景色が変わるのと同時に、地面の色も変わることを確認。

**確認項目：**
- 昼夜の切り替わり時に地面の色もスムーズに変わる
- 色が急激に変わらず、自然に見える
- グロー効果の色も切り替わる
- 切り替わり時にチラつきがない

---

## Task 8: パフォーマンス確認

**Files:**
- Test: ブラウザコンソール（Performance）

- [ ] **Step 1: ブラウザの開発者ツールを開く（F12）**

- [ ] **Step 2: Performance タブで記録開始**

5秒間ゲームをプレイして記録終了。

**確認項目：**
- フレームレートが60fps近辺を維持（グラフが平坦）
- グラデーション生成による処理負荷がない（各フレームで一度だけ生成）
- メモリ使用量が増加していない
- `createLinearGradient` の呼び出しが過度ではない（ループ内で毎フレーム実行は OK）

**問題あれば：** 
- フレームレートが低下 → グラデーション生成をループ外に移動することを検討（ただし、スクリーン座標が動的なので現在の実装が正しい）

---

## Task 9: コンソールエラーを確認＆修正

**Files:**
- Review: ブラウザコンソール（F12 Console タブ）

- [ ] **Step 1: コンソールにエラーが出ていないか確認**

**確認項目：**
- 「色コード不正」エラーがない
- 「undefined」エラーがない
- グラデーション生成エラーがない

**もしエラーがあれば：**
- 色コード形式を確認（`#RRGGBB` または `rgba(r, g, b, a)` 形式か）
- グラデーション座標が正しいか（`sx, block.y, sx, block.y + block.h`）
- `createLinearGradient` の引数が4個か

---

## Task 10: コード形式・スタイルを確認

**Files:**
- Review: `Purple Biker/index.html:1644-1672`

- [ ] **Step 1: コード形式の最終確認**

以下を確認：
- インデント統一（既存コードと同じ2スペース）
- 色コードが統一（16進数 `#RRGGBB` または `rgba(r, g, b, a)` 形式）
- コメント明記（グラデーション、発光ラインの説明）
- 古いコード完全削除（単色塗りコード）
- 境界線描画のコードが重複していない

---

## Task 11: Git コミット

**Files:**
- Modify: `Purple Biker/index.html`

- [ ] **Step 1: 変更内容をステージング**

```bash
cd "/c/Users/hayato.takagaki/OneDrive/デスクトップ/Claude（Purple Park）/Purple Biker"
git add index.html
git status
```

**確認：** `index.html` が "Changes to be committed" に表示される

- [ ] **Step 2: コミットメッセージを作成＆コミット**

```bash
git commit -m "feat: upgrade terrain with metallic gradient and time-of-day glow

- Daytime: bright metallic gradient (#3a4a5e -> #1a2a3e) + white glow line
- Nighttime: dark metallic gradient (#1a2a3e -> #000005) + neon magenta glow
- Add shadowBlur effects for glow (daytime: 8, nighttime: 12)
- Improve visual quality and align with latest game aesthetics"
```

- [ ] **Step 3: ログで確認**

```bash
git log --oneline -1
```

**確認：** 新しいコミットが表示される

---

## Task 12: GitHub にプッシュ

**Files:**
- Push: `Purple Biker` リポジトリ

- [ ] **Step 1: プッシュ実行**

```bash
git push origin main
```

**確認：** 
- エラーなくプッシュ完了
- "Your branch is up to date" と表示される

---

## 自己レビュー

**スペック内容との対応確認：**
1. ✅ グラデーション色（昼夜別）→ Task 2-3 で実装
2. ✅ 発光ライン（昼は薄い白、夜はマゼンタ）→ Task 2-3 で実装
3. ✅ グロー効果（昼：8、夜：12）→ Task 2-3 で実装
4. ✅ バグ防止措置 → Task 4（確認）、Task 9（エラー確認）
5. ✅ テスト項目（昼夜確認、視認性、パフォーマンス）→ Task 5-8

**プレースホルダースキャン：**
- ✅ すべてのコード例が完全
- ✅ "TBD" や "TODO" がない
- ✅ 確認項目が具体的

**型・メソッド一貫性：**
- ✅ グラデーション座標一貫（`sx, block.y, sx, block.y + block.h`）
- ✅ 色コード形式統一（16進数 + rgba）
- ✅ グロー設定一貫（`shadowColor`, `shadowBlur` セット）

---

## 実行準備完了

計画ファイルを保存しました。Subagent-Driven Development で Task 1 から順に実装を進めます。
