# Purple Jumper：4番目のキャラクター「餃子」追加デザイン仕様

**作成日:** 2026-05-09  
**対象:** Purple Jumper ゲーム  
**変更内容:** 新キャラクター「餃子」の追加

---

## 概要

Purple Jumperに4番目のキャラクター「餃子」を追加します。キャラクター選択UIは現在の構造を維持しながら、4スロット対応へ拡張し、スワイプ中の連続音再生機能を実装します。

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
- **表示位置:** 他のキャラクターと同じ（キャラクター選択UIの中央スロット内）

### 名前表示

- **表示内容:** 「餃子」
- **スタイル:** 他のキャラクターと統一（小サイズ、画像下部に配置）
- **言語:** 日本語

---

## 2. キャラクター選択UIの拡張

### 現在の構造を維持

- **char-slot-0** — KAMIKI（既存）
- **char-slot-1** — イッショウ（既存）
- **char-slot-2** — パプ太郎（既存）
- **char-slot-3** — 餃子（新規追加）

### 表示方式：円形カルーセル

#### 常時表示
- 左側：1つ前のキャラクター（小さく、暗め）
- 中央：現在選択されているキャラクター（大きく、明るい）
- 右側：1つ先のキャラクター（小さく、暗め）

#### スワイプ動作

ユーザーがスワイプすると、4つのキャラクターが回転して切り替わる：

```
初期状態：        スワイプ右後：
左: イッショウ   →  左: KAMIKI
中: KAMIKI       →  中: イッショウ
右: パプ太郎     →  右: パプ太郎
```

#### 回転サイクル

```
KAMIKI（0）→ イッショウ（1）→ パプ太郎（2）→ 餃子（3）→ KAMIKI（0）...（ループ）
```

### アニメーション

- **速度:** 現在の設定を維持（`--char-swap-dur: 0.72s`）
- **イージング:** 現在の設定を維持（cubic-bezier(0.45, 0, 0.2, 1)）
- **3D効果:** perspective: 820px を保持
- **トランジション:** 既存のアニメーション（promote/demote）を4スロットに対応させる

---

## 3. 音のタイミング（新規仕様）

### スワイプ中の連続音再生

#### 動作概要

スワイプ中にキャラクターが中央位置を通過するたびに「仕掛け作動2.mp3」を再生します。

#### ユーザー体験

- **1回のスワイプ:** 中央を通過するキャラ数分だけ音が鳴る
- **連続スワイプ:** くるくる回すたびに「カタ・カタ・カタ...」と連続で鳴る
- **反応性:** スワイプのビジュアルフィードバック（キャラが中央に来た時点）と音が同期

#### 実装方法

- アニメーション進捗を監視（`requestAnimationFrame` または CSS animation イベント）
- キャラクターが中央位置に達した時点を検出
- その時点で `playCharacterSelectSfx()` を呼び出して音を再生

#### 音の特性

- **ファイル:** 仕掛け作動2.mp3（既存のまま）
- **再生方式:** 既存の `playCharacterSelectSfx()` 関数と同じ
- **音量:** 既存の設定を維持
- **タイミング:** キャラが中央スロットに到達した時点で即座に再生

---

## 4. HTMLの変更

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

### 構造上の注意

- 既存の char-slot-0, char-slot-1, char-slot-2 の後に追加
- DOM要素の順序は、data-char 属性の値と対応させる必要はない（CSSで位置決定）

---

## 5. CSSの変更

### char-slot[data-char="3"] のポジション定義

```css
.char-slot[data-char="3"] { 
  --slot-x: 118px; 
  --slot-rot: -14deg; 
}
```

### 選択時の状態

`char-select-stage[data-selected="3"]` に対応するスタイルを追加（既存のパターンと同じ）：

```css
.char-select-stage[data-selected="3"] .char-slot[data-char="3"] {
  transform: translate3d(var(--slot-x), -6px, 28px) scale(1.14) rotateY(0deg);
  z-index: 4;
  filter: brightness(1.05) saturate(1.08);
  opacity: 1;
}
```

### アニメーション追加

4スロット対応のアニメーション（promote/demote）：
- `char-promote-from-right`（スロット3から0へ）
- `char-demote-to-left`（スロット0から3へ）
- `char-promote-from-left`（スロット0から3へ）
- `char-demote-to-right`（スロット3から0へ）

既存のパターンを参考に、同じアニメーション定義を追加します。

---

## 6. JavaScriptの変更

### 6.1 キャラクター定義

`PLAYABLE_CHARACTERS` 配列に餃子を追加：

```javascript
const PLAYABLE_CHARACTERS = [
  { id: "purple", name: "KAMIKI", preview: "透過済.png", base: "透過済.png", jump1: "透過済1.png", jump2: "透過済2.png" },
  { id: "gen", name: "イッショウ", preview: "元透過済.png", base: "元透過済.png", jump1: "元透過済.png", jump2: "元透過済.png" },
  { id: "paputaro", name: "パプ太郎", preview: "GAMEOVER21.png", base: "GAMEOVER21.png", jump1: "GAMEOVER21.png", jump2: "GAMEOVER21.png" },
  { id: "gyoza", name: "餃子", preview: "gyoza.png", base: "gyoza.png", jump1: "gyoza.png", jump2: "gyoza.png" }
];
```

### 6.2 DOM参照の追加

```javascript
const charSlot3 = document.getElementById("char-slot-3");
```

### 6.3 syncCharSelectDom() 関数の更新

`aria-pressed` 属性の更新に char-slot-3 を追加：

```javascript
if (charSlot3) charSlot3.setAttribute("aria-pressed", selectedCharacterIndex === 3 ? "true" : "false");
```

### 6.4 スワイプ中の中央通過検出ロジック

**新規追加：** キャラクターがアニメーション中に中央位置を通過したことを検出し、その時点で音を再生する機能

**実装方法（推奨）:**

1. `runCharacterSwap()` 関数でスワイプアニメーション開始時に監視を開始
2. `requestAnimationFrame` で毎フレーム各スロットの位置を確認
3. キャラクターが中央スロット（`--slot-x: 0px`）の位置に到達した時点を検出
4. 前フレームでは中央にいなかったが、現フレームで中央に到達した場合、`playCharacterSelectSfx()` を呼び出す
5. アニメーション完了時に監視を終了

**検出のポイント:**

- 各 `char-slot` の `transform` から `translate3d` の値を取得
- `--slot-x` が `0px` 付近（中央）に到達したかを判定
- 連続スワイプで複数キャラが中央を通過する場合、各通過ごとに音を再生

### 6.5 アニメーション条件の確認

既存の以下のロジックが4スロットに対応しているか検証：
- `isSlotSwapAnimation()` — アニメーション名のチェック
- `runCharacterSwap()` — インデックス範囲チェック（`idx >= PLAYABLE_CHARACTERS.length`）
- アニメーション `data-anim` 属性の遷移ロジック（0-1, 1-0 など）

4スロットの場合、追加の遷移パターン：
- 3 → 0（餃子 → KAMIKI）
- 0 → 3（KAMIKI → 餃子）
- 2 → 3（パプ太郎 → 餃子）
- 3 → 2（餃子 → パプ太郎）

---

## 7. ローカルストレージ

### 選択状態の保存

- **キー:** `"murasaki_ojisan_character"`（既存のまま）
- **値:** 0 ～ 3（インデックス）
  - 0: KAMIKI
  - 1: イッショウ
  - 2: パプ太郎
  - 3: 餃子

### 初期化

`initCharacterFromStorage()` 関数が既に `PLAYABLE_CHARACTERS.length` をチェックしているので、4スロット対応として動作します。

---

## 8. テスト観点

### 機能テスト

- [ ] キャラクター選択時に「餃子」が正しく表示される
- [ ] スワイプで「餃子」を選択できる
- [ ] スワイプ中に複数のキャラが中央を通過する場合、その都度音が鳴る
- [ ] 連続スワイプで「カタ・カタ・カタ...」と連続音が鳴る
- [ ] 選択状態がローカルストレージに保存される
- [ ] ブラウザリロード後も選択状態が復元される

### パフォーマンステスト

- [ ] スワイプ中の中央通過検出がスムーズで、フレームドロップがない
- [ ] 連続スワイプで音再生のタイミングが安定している

### UI/UXテスト

- [ ] 「餃子」の表示サイズが他のキャラと統一されている
- [ ] 名前「餃子」がキャラクターと視覚的にマッチしている
- [ ] アニメーションが4スロット対応で違和感がない

---

## 9. 今後の拡張性

### 5番目以降のキャラクター追加

同じパターンで対応可能：
1. `PLAYABLE_CHARACTERS` に新キャラを追加
2. HTML に `char-slot-4` を追加
3. CSS に `--slot-x` と `--slot-rot` を定義
4. JavaScript に DOM参照と `aria-pressed` 更新を追加

アニメーション調整が必要な場合のみ、CSS の keyframes を拡張します。

---

## 10. 注記

- **gyoza.png の準備状況:** 既に提供済み（透過背景）
- **音ファイル:** 既存の「仕掛け作動2.mp3」を再利用
- **ブラウザ互換性:** 既存コードの範囲内で対応（特別な API 不要）
