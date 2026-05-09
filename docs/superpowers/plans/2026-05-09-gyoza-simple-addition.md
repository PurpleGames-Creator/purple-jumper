# 餃子キャラクター追加 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Purple Jumper に4番目のキャラクター「餃子」を追加し、タイトル画面で選択可能にする

**Architecture:** 既存3キャラクターのレイアウトを維持しながら、CSS の位置指定で餃子をパプ太郎の上に配置。JavaScript でクリック選択と状態管理を対応。

**Tech Stack:** HTML5, CSS3 (transform, translate3d), Vanilla JavaScript

---

## ファイル構造

| ファイル | 責務 |
|---------|------|
| `Purple Jumper/index.html` | HTML マークアップ、CSS スタイル、JavaScript ロジック（全て）|

---

## Task 1: HTML に char-slot-3 要素を追加

**Files:**
- Modify: `Purple Jumper/index.html` （HTML セクション、char-select-stage 内）

- [ ] **Step 1: char-slot-3 のマークアップを追加**

`char-select-stage` の末尾（char-slot-2 の後）に以下を追加：

```html
<button type="button" class="char-slot" data-char="3" id="char-slot-3" aria-pressed="false" aria-label="餃子を選択">
  <span class="char-slot-glow" aria-hidden="true"></span>
  <span class="char-slot-inner">
    <img src="gyoza.png" alt="" draggable="false" />
  </span>
  <span class="char-slot-name">餃子</span>
</button>
```

**配置:** 既存の `<button class="char-slot" data-char="2">` （パプ太郎）のすぐ後に追加

- [ ] **Step 2: ブラウザで表示確認（暫定）**

ブラウザを開いて Purple Jumper を読み込み、タイトル画面に `char-slot-3` の要素が DOM に存在することを確認（デベロッパーツールで要素検査）

**確認内容:**
- `<button data-char="3" id="char-slot-3">` が存在する
- 画像 `<img src="gyoza.png">` が読み込まれている
- `<span class="char-slot-name">餃子</span>` が表示されている

- [ ] **Step 3: Commit**

```bash
cd "C:\Users\hayato.takagaki\OneDrive\デスクトップ\Claude（Purple Park）"
git add "Purple Jumper/index.html"
git commit -m "feat: add HTML markup for gyoza character slot (char-slot-3)"
```

---

## Task 2: CSS で char-slot-3 のスタイルを定義

**Files:**
- Modify: `Purple Jumper/index.html` （CSS セクション、style タグ内）

- [ ] **Step 1: char-slot[data-char="3"] の基本スタイルを追加**

既存の `.char-slot[data-char="2"]` スタイルの直後に以下を追加：

```css
.char-slot[data-char="3"] {
  --slot-x: 118px;
  --slot-rot: -14deg;
  transform: translate3d(var(--slot-x), calc(-6px - 120px), 28px) scale(0.9);
  opacity: 0.8;
  z-index: 3;
}
```

**意図:**
- `--slot-x: 118px` — パプ太郎と同じ右側の x 位置
- `--slot-rot: -14deg` — パプ太郎と同じ回転角度
- `transform` の Y 値 `calc(-6px - 120px)` — パプ太郎から 120px 上方に配置
- `scale(0.9)` — 未選択時に若干小さく表示
- `z-index: 3` — パプ太郎（z-index: 2）より上に表示

- [ ] **Step 2: char-select-stage[data-selected="3"] のスタイルを追加**

既存の `.char-select-stage[data-selected="2"]` ブロックの直後に以下を追加：

```css
.char-select-stage[data-selected="3"] .char-slot[data-char="3"] {
  transform: translate3d(var(--slot-x), calc(-6px - 120px), 28px) scale(1);
  z-index: 4;
  filter: brightness(1.05) saturate(1.08);
  opacity: 1;
}
```

**意図:**
- `scale(1)` — 選択時は標準サイズで表示
- `z-index: 4` — 選択時はパプ太郎より前面に表示
- `filter` — 選択時に明るく、彩度を上げる
- `opacity: 1` — 選択時は完全な不透明度

- [ ] **Step 3: 画面で餃子の位置を確認**

ブラウザで Purple Jumper を再読み込みして、以下を確認：

**確認内容:**
- 餃子がパプ太郎の上に表示されている
- パプ太郎の文字「パプ太郎」と餃子の画像・文字が重なっていない
- 十分なギャップ（視覚的に 20～30px 以上）がある
- 未選択時は若干小さく見える

**調整が必要な場合:**
- Y 値の `120px` を増減（例：`130px` に変更）してギャップを調整
- または、パプ太郎のスケール値を確認して、計算に反映

- [ ] **Step 4: Commit**

```bash
git add "Purple Jumper/index.html"
git commit -m "feat: add CSS styles for gyoza character slot positioning and selection state"
```

---

## Task 3: JavaScript で PLAYABLE_CHARACTERS に餃子を追加

**Files:**
- Modify: `Purple Jumper/index.html` （JavaScript セクション）

- [ ] **Step 1: PLAYABLE_CHARACTERS 配列に餃子を追加**

現在のコード：

```javascript
const PLAYABLE_CHARACTERS = [
  { id: "purple", name: "KAMIKI", preview: "透過済.png", base: "透過済.png", jump1: "透過済1.png", jump2: "透過済2.png" },
  { id: "gen", name: "イッショウ", preview: "元透過済.png", base: "元透過済.png", jump1: "元透過済.png", jump2: "元透過済.png" },
  { id: "paputaro", name: "パプ太郎", preview: "GAMEOVER21.png", base: "GAMEOVER21.png", jump1: "GAMEOVER21.png", jump2: "GAMEOVER21.png" }
];
```

新しいコード：

```javascript
const PLAYABLE_CHARACTERS = [
  { id: "purple", name: "KAMIKI", preview: "透過済.png", base: "透過済.png", jump1: "透過済1.png", jump2: "透過済2.png" },
  { id: "gen", name: "イッショウ", preview: "元透過済.png", base: "元透過済.png", jump1: "元透過済.png", jump2: "元透過済.png" },
  { id: "paputaro", name: "パプ太郎", preview: "GAMEOVER21.png", base: "GAMEOVER21.png", jump1: "GAMEOVER21.png", jump2: "GAMEOVER21.png" },
  { id: "gyoza", name: "餃子", preview: "gyoza.png", base: "gyoza.png", jump1: "gyoza.png", jump2: "gyoza.png" }
];
```

**配置:** 既存の3つのオブジェクトの後、配列の末尾に追加

- [ ] **Step 2: Commit**

```bash
git add "Purple Jumper/index.html"
git commit -m "feat: add gyoza character to PLAYABLE_CHARACTERS array"
```

---

## Task 4: JavaScript で DOM 参照を追加

**Files:**
- Modify: `Purple Jumper/index.html` （JavaScript セクション）

- [ ] **Step 1: charSlot3 の参照を追加**

現在のコード（例）：

```javascript
const charSlot0 = document.getElementById("char-slot-0");
const charSlot1 = document.getElementById("char-slot-1");
const charSlot2 = document.getElementById("char-slot-2");
```

新しいコード：

```javascript
const charSlot0 = document.getElementById("char-slot-0");
const charSlot1 = document.getElementById("char-slot-1");
const charSlot2 = document.getElementById("char-slot-2");
const charSlot3 = document.getElementById("char-slot-3");
```

**配置:** 既存の3つの参照の直後に追加

- [ ] **Step 2: Commit**

```bash
git add "Purple Jumper/index.html"
git commit -m "feat: add charSlot3 DOM reference"
```

---

## Task 5: syncCharSelectDom() 関数を更新

**Files:**
- Modify: `Purple Jumper/index.html` （JavaScript セクション、syncCharSelectDom 関数内）

- [ ] **Step 1: syncCharSelectDom() 関数に charSlot3 の aria-pressed 更新を追加**

既存のコード（例）：

```javascript
function syncCharSelectDom() {
  charSelectStage.dataset.selected = String(selectedCharacterIndex);
  if (charSlot0) charSlot0.setAttribute("aria-pressed", selectedCharacterIndex === 0 ? "true" : "false");
  if (charSlot1) charSlot1.setAttribute("aria-pressed", selectedCharacterIndex === 1 ? "true" : "false");
  if (charSlot2) charSlot2.setAttribute("aria-pressed", selectedCharacterIndex === 2 ? "true" : "false");
}
```

新しいコード：

```javascript
function syncCharSelectDom() {
  charSelectStage.dataset.selected = String(selectedCharacterIndex);
  if (charSlot0) charSlot0.setAttribute("aria-pressed", selectedCharacterIndex === 0 ? "true" : "false");
  if (charSlot1) charSlot1.setAttribute("aria-pressed", selectedCharacterIndex === 1 ? "true" : "false");
  if (charSlot2) charSlot2.setAttribute("aria-pressed", selectedCharacterIndex === 2 ? "true" : "false");
  if (charSlot3) charSlot3.setAttribute("aria-pressed", selectedCharacterIndex === 3 ? "true" : "false");
}
```

**変更内容:** 最後に1行追加

- [ ] **Step 2: Commit**

```bash
git add "Purple Jumper/index.html"
git commit -m "feat: update syncCharSelectDom to handle charSlot3"
```

---

## Task 6: 選択機能が既に対応していることを確認

**Files:**
- No modifications needed (verify existing code)

- [ ] **Step 1: 既存コードが 4 スロット対応していることを確認**

デベロッパーツールのコンソールで、以下のコマンドを実行して動作確認：

```javascript
// 餃子を選択
applyPlayableCharacter(3);
// 結果: selectedCharacterIndex === 3 になること
console.log(selectedCharacterIndex); // 3 と表示される
```

**確認内容:**
- `applyPlayableCharacter(3)` で selectedCharacterIndex が 3 に更新される
- `syncCharSelectDom()` が自動的に呼ばれて、DOM が更新される
- char-select-stage に `data-selected="3"` がセットされる

- [ ] **Step 2: runCharacterSwap() が 4 スロット対応していることを確認**

既存コードを grep で検索：

```javascript
// 検索対象: runCharacterSwap 関数内の以下の行
if (toIndex < 0 || toIndex >= PLAYABLE_CHARACTERS.length) return;
```

**確認内容:**
- `PLAYABLE_CHARACTERS.length` が 4 になっているので、`toIndex < 4` なら有効
- インデックス 0, 1, 2, 3 全てで `runCharacterSwap()` が動作することを確認

**テスト方法:**
コンソールで以下を実行：

```javascript
// 各スロットをクリック
document.getElementById("char-slot-0").click(); // KAMIKI 選択
document.getElementById("char-slot-1").click(); // イッショウ 選択
document.getElementById("char-slot-2").click(); // パプ太郎 選択
document.getElementById("char-slot-3").click(); // 餃子 選択
```

全てのクリックが正しく反応することを確認

- [ ] **Step 3: No commit needed (verification only)**

---

## Task 7: クリック選択とローカルストレージのテスト

**Files:**
- No file modifications (testing only)

- [ ] **Step 1: 餃子をクリックして選択**

Purple Jumper をブラウザで開き、タイトル画面で：

1. 餃子（char-slot-3）をクリック
2. 餃子が選択されて、ハイライト（明るく、大きく）表示されることを確認

**期待される動作:**
- 餃子が他のスロットより明るく、サイズが大きくなる（未選択時の 0.9 倍から 1.0 倍へ）
- 選択状態が視覚的に区別できる

- [ ] **Step 2: ローカルストレージに保存されることを確認**

コンソールで以下を実行：

```javascript
// ローカルストレージ確認
localStorage.getItem("murasaki_ojisan_character")
// 結果: "3" が返される
```

**確認内容:**
- 餃子選択時にローカルストレージに "3" が保存されていること

- [ ] **Step 3: ブラウザを再読み込みしてリストア確認**

1. Purple Jumper をブラウザで開く（餃子が選択状態で保存されている）
2. ページを再読み込み（F5 または Ctrl+R）
3. 再読み込み後、餃子が選択状態のまま表示されることを確認

**期待される動作:**
- `initCharacterFromStorage()` が実行され、ローカルストレージから "3" を読み込む
- 餃子がハイライト表示される

- [ ] **Step 4: 他のキャラクターも正常に動作することを確認**

各キャラクターをクリックして、選択状態が正常に切り替わることを確認：

```
KAMIKI → イッショウ → パプ太郎 → 餃子 → KAMIKI ...
```

全ての遷移が滑らかに動作し、ローカルストレージが正常に更新されることを確認

- [ ] **Step 5: No commit needed (testing only)**

---

## Task 8: ゲームプレイ中の動作確認

**Files:**
- No file modifications (testing only)

- [ ] **Step 1: 餃子を選択してゲームをプレイ**

タイトル画面で餃子を選択し、「PLAY」ボタンをクリックしてゲームを開始

**確認内容:**
- ゲーム画面で餃子が正常に表示される
- ジャンプ時のアニメーションが正常に動作する
- ゲームプレイに支障がない

- [ ] **Step 2: 他のキャラクターと比較**

各キャラクター（KAMIKI、イッショウ、パプ太郎）を選択してプレイし、餃子の表示サイズが他と一貫していることを確認

**確認内容:**
- パプ太郎だけ小さいサイズ（0.7 倍）で表示される
- 餃子は通常サイズ（1.0 倍）で表示される

- [ ] **Step 3: No commit needed (testing only)**

---

## Task 9: 最終確認とドキュメント更新

**Files:**
- No file modifications (verification only)

- [ ] **Step 1: 全ての要件が実装されたことを確認**

設計仕様書の以下の項目を確認：

- ✅ HTML に char-slot-3 が追加されている
- ✅ CSS で餃子がパプ太郎の上に配置されている
- ✅ JavaScript で PLAYABLE_CHARACTERS に餃子が追加されている
- ✅ クリック選択が動作している
- ✅ ローカルストレージで選択状態が保存されている
- ✅ ブラウザリロード後も選択状態が復元される
- ✅ ゲームプレイ中に餃子が表示される
- ✅ 既存3キャラクターの位置・大きさが変わらない
- ✅ パプ太郎の画像・文字と餃子が重ならない

- [ ] **Step 2: No commit needed (final verification)**

---

## 補足

### CSS の Y 値調整ガイド

設計仕様で `calc(-6px - 120px)` を使用していますが、実装後に視覚的な調整が必要な場合があります：

**調整方法:**
1. デベロッパーツールで char-slot-3 のスタイルを確認
2. Y 値を変更（例：`110px`, `130px`）
3. パプ太郎とのギャップが十分か（20px 以上）視覚的に確認
4. 修正したら CSS を更新してコミット

**最終調整コマンド:**
```bash
git add "Purple Jumper/index.html"
git commit -m "style: adjust gyoza positioning for optimal spacing"
```

---

## 実装時の注意点

1. **ファイル保存時の改行コード:** Windows の CRLF で保存される場合があります。git commit 時に warning が出ても問題ありません。

2. **画像ファイル:** `gyoza.png` は既に `Purple Jumper/` フォルダに存在することを確認してください。

3. **ブラウザキャッシュ:** CSS 変更後、ブラウザのキャッシュをクリアして再読み込み（Ctrl+Shift+R）してください。

4. **テストツール:** ブラウザのデベロッパーツール（F12）を活用して、要素検査とコンソール実行でテストしてください。
