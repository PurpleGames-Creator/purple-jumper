# Purple Jumper：4番目のキャラクター「餃子」追加デザイン仕様（簡易版）

**作成日:** 2026-05-09  
**対象:** Purple Jumper ゲーム  
**変更内容:** 新キャラクター「餃子」の追加（回転アニメーションなし、静的配置）

---

## 概要

Purple Jumperに4番目のキャラクター「餃子」を追加します。既存3キャラクター（KAMIKI、イッショウ、パプ太郎）の位置・大きさ・スタイルは変更せず、餃子をパプ太郎の真上に小さく配置します。選択方法はクリック/タップで自由に選択可能です。

---

## 1. 新キャラクター「餃子」の定義

### キャラクター情報

```javascript
{
  id: "gyoza",
  name: "餃子",
  preview: "gyoza.png",
  base: "gyoza.png",
  jump1: "gyoza.png",
  jump2: "gyoza.png"
}
```

### 画像仕様

- **ファイル名:** gyoza.png
- **形式:** PNG（透過背景）
- **用途:** 全状態（待機、ジャンプ1、ジャンプ2）で同じ画像を使用
- **サイズ:** パプ太郎と同じ「小さいスロット」用サイズを想定

---

## 2. キャラクター選択UIの構成

### 現在の構造を維持

```
        [餃子]（新規追加 - パプ太郎の上）
         ↓

[イッショウ]  [KAMIKI]  [パプ太郎]
 （左・小暗）  （中央・大明）（右・小暗）
```

**既存3スロットの特性（変更なし）:**
- 左側：イッショウ（小さく、暗めの表示）
- 中央：KAMIKI（大きく、明るい表示）
- 右側：パプ太郎（小さく、暗めの表示）

**新規スロット:**
- 上側：餃子（パプ太郎の上に配置、小さいサイズ）

### 重なり回避のための配置

- 餃子を `translate3d` で上方向（負の Y 値）に配置
- パプ太郎の文字「パプ太郎」と餃子の画像・文字「餃子」が重ならないようマージンを設定
- 十分なギャップ（推奨：20～30px）を確保

---

## 3. HTMLの変更

### char-slot-3 の追加

`char-select-stage` に以下の要素を追加：

```html
<button type="button" class="char-slot" data-char="3" id="char-slot-3" aria-pressed="false" aria-label="餃子を選択">
  <span class="char-slot-glow" aria-hidden="true"></span>
  <span class="char-slot-inner">
    <img src="gyoza.png" alt="" draggable="false" />
  </span>
  <span class="char-slot-name">餃子</span>
</button>
```

**配置:** 既存の char-slot-0, 1, 2 の後に追加

---

## 4. CSSの変更

### char-slot[data-char="3"] のポジション定義

パプ太郎（char-slot-2）の上に配置：

```css
.char-slot[data-char="3"] {
  /* パプ太郎と同じ右側の x 位置を基準に、上方向に配置 */
  transform: translate3d(var(--slot-x), calc(-6px - 120px), 28px) scale(0.9);
  /* 120px は上方向のギャップ（パプ太郎との距離） */
  z-index: 3;
  opacity: 0.8;
}
```

### 選択時の状態

`char-select-stage[data-selected="3"]` に対応するスタイル：

```css
.char-select-stage[data-selected="3"] .char-slot[data-char="3"] {
  transform: translate3d(var(--slot-x), calc(-6px - 120px), 28px) scale(1);
  z-index: 4;
  filter: brightness(1.05) saturate(1.08);
  opacity: 1;
}
```

**調整項目:**
- `calc(-6px - 120px)` の `120px` 値は、パプ太郎との距離に応じて調整
- `scale(0.9)` は未選択時の小さなサイズ
- `scale(1)` は選択時の標準サイズ

---

## 5. JavaScriptの変更

### 5.1 キャラクター定義

`PLAYABLE_CHARACTERS` 配列に餃子を追加（末尾）：

```javascript
const PLAYABLE_CHARACTERS = [
  { id: "purple", name: "KAMIKI", preview: "透過済.png", base: "透過済.png", jump1: "透過済1.png", jump2: "透過済2.png" },
  { id: "gen", name: "イッショウ", preview: "元透過済.png", base: "元透過済.png", jump1: "元透過済.png", jump2: "元透過済.png" },
  { id: "paputaro", name: "パプ太郎", preview: "GAMEOVER21.png", base: "GAMEOVER21.png", jump1: "GAMEOVER21.png", jump2: "GAMEOVER21.png" },
  { id: "gyoza", name: "餃子", preview: "gyoza.png", base: "gyoza.png", jump1: "gyoza.png", jump2: "gyoza.png" }
];
```

### 5.2 DOM参照の追加

```javascript
const charSlot3 = document.getElementById("char-slot-3");
```

### 5.3 syncCharSelectDom() 関数の更新

`aria-pressed` 属性の更新に char-slot-3 を追加：

```javascript
if (charSlot3) charSlot3.setAttribute("aria-pressed", selectedCharacterIndex === 3 ? "true" : "false");
```

### 5.4 既存ロジックの確認

以下の関数が既に 4 スロット対応していることを確認：
- `initCharacterFromStorage()` — `PLAYABLE_CHARACTERS.length` をチェック済み
- `runCharacterSwap()` — インデックス範囲チェック（`idx >= PLAYABLE_CHARACTERS.length`）
- クリックイベントリスナー — 既存のイベント委譲で自動対応

---

## 6. ローカルストレージ

### 選択状態の保存

- **キー:** `"murasaki_ojisan_character"`（既存のまま）
- **値:** 0～3（インデックス）
  - 0: KAMIKI
  - 1: イッショウ
  - 2: パプ太郎
  - 3: 餃子

### 初期化

`initCharacterFromStorage()` が既存の `PLAYABLE_CHARACTERS.length` をチェックしているため、自動的に 4 スロット対応になります。

---

## 7. ゲームプレイ中の表示

### 餃子選択時

```javascript
// プレイ中のキャラクター表示の例
var curCh = PLAYABLE_CHARACTERS[selectedCharacterIndex];
if (curCh && curCh.id === "gyoza") {
  // 通常サイズで表示（sizeScale = 1.0）
  // 他の特別なルールはなし
}
```

---

## 8. テスト観点

### 機能テスト

- [ ] 餃子がタイトル画面に表示される
- [ ] 餃子をクリック/タップで選択できる
- [ ] 選択状態がローカルストレージに保存される
- [ ] ブラウザリロード後も選択状態が復元される
- [ ] パプ太郎の画像・文字と餃子が重ならない
- [ ] 他の3キャラの表示は変わらない

### 視覚テスト

- [ ] 餃子がパプ太郎の上に適切な位置に表示されている
- [ ] 餃子の文字「餃子」がしっかり読める
- [ ] 選択時に明るく光る効果が適切に動作している

---

## 9. 注記

- **gyoza.png の準備状況:** 既に提供済み（透過背景）
- **回転アニメーション:** なし（静的配置）
- **ブラウザ互換性:** 既存コードの範囲内で対応
- **将来の拡張性:** 5番目以降のキャラクター追加も同じパターンで対応可能
