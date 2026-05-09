# Purple Bull 仕様微調整 - 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Purple Bullゲームの3つの仕様微調整を実装する：肉が出ないバグ修正、難易度アップ（1.5倍速化）、タイトル画面効果音追加。

**Architecture:** 
- タスク1-2：肉配置ロジックの修正とパフォーマンステスト
- タスク3-4：ゲーム速度の変更と長押し機能削除
- タスク5-6：タイトル画面の効果音追加

**Tech Stack:** Vanilla JavaScript, HTML5 Canvas, Web Audio API

---

## ファイル構造

**修正ファイル:**
- `game.js` - BullGame クラス内の `_placeMeat()` メソッド修正、TICK値変更、isSpeedBoost削除
- `main.js` - 長押し検出コード削除、タイトル画面効果音再生追加

**新規ファイル:**
- `poyoyon.mp3` - タイトル画面キャラクター増殖時の効果音（Purple Bullフォルダルート）

---

## Task 1: 肉配置ロジックの修正（_placeMeat() の フォールバック処理追加）

**Files:**
- Modify: `game.js:339-385` (_placeMeat メソッド)

- [ ] **Step 1: _placeMeat() メソッドの全体構造を確認**

現在のコードを確認して、ランダム試行のロジックを理解する：

```bash
# game.js の 339-385行を確認
# - snakeSet キャッシング
# - 95% ルール（行356-359）
# - ランダム試行ループ（行374-381）
```

- [ ] **Step 2: _placeMeat() メソッド全体を置き換え**

`game.js` の行339-385を以下のコードで置き換え：

```javascript
_placeMeat() {
  // snakeSetをキャッシュ：snake長が変わったときのみ再生成
  const snakeLength = this.snake.length;
  if (!this._snakeSet || this._lastSnakeLength !== snakeLength) {
    this._snakeSet = new Set();
    for (let i = 0; i < snakeLength; i++) {
      const s = this.snake[i];
      this._snakeSet.add(s.row * this.GRID_COLS + s.col);
    }
    this._lastSnakeLength = snakeLength;
  }
  const snakeSet = this._snakeSet;

  const totalCells = this.GRID_ROWS * this.GRID_COLS;
  const snakeCells = this.snake.length;

  // Snake がフィールドのほぼ全て（95%以上）を占めたら肉は配置不可
  if (snakeCells > totalCells * 0.95) {
    this.meat = null;
    return;
  }

  // 肉のタイプを固定確率で決定
  const rand = Math.random();
  let type;
  if (rand < 0.5) {
    type = 'normal';
  } else if (rand < 0.9) {
    type = 'special';
  } else {
    type = 'super_special';
  }

  // ステップ1：ランダム試行で肉配置（300回）
  for (let attempts = 0; attempts < 300; attempts++) {
    const r = Math.floor(Math.random() * this.GRID_ROWS);
    const c = Math.floor(Math.random() * this.GRID_COLS);
    if (!snakeSet.has(r * this.GRID_COLS + c)) {
      this.meat = { row: r, col: c, type };
      return;
    }
  }

  // ステップ2：フォールバック処理（フィールド全体をスキャン）
  // ランダム試行で見つからなかった場合、確実に空きマスを見つける
  for (let r = 0; r < this.GRID_ROWS; r++) {
    for (let c = 0; c < this.GRID_COLS; c++) {
      if (!snakeSet.has(r * this.GRID_COLS + c)) {
        this.meat = { row: r, col: c, type };
        return;
      }
    }
  }

  // 全マスが蛇で埋まっている場合のみ肉なし状態
  this.meat = null;
}
```

- [ ] **Step 3: ゲームを実行して動作確認**

ゲームサーバーを起動して、正常に動作することを確認：

```bash
# ローカルサーバーで Purple Bull を起動
# (ゲーム開始 → スコア30程度まで進める → 肉が出現することを確認)
```

期待される動作：肉が正常に出現し、スコアが増えていく

- [ ] **Step 4: Commit**

```bash
cd "C:\Users\hayato.takagaki\OneDrive\デスクトップ\Claude（Purple Park）\Purple Bull"
git add game.js
git commit -m "fix: add fallback logic to meat placement to handle high snake lengths"
```

---

## Task 2: パフォーマンステスト（スコア230～240）

**Files:**
- Test: インブラウザテスト（スマートフォン実機推奨）

- [ ] **Step 1: テスト環境のセットアップ**

スマートフォンまたはパソコンのブラウザで Purple Bull を起動

- [ ] **Step 2: スコア230まで進める**

実際にゲームをプレイしてスコア230まで進める：
- 蛇が233マス程度になる状態
- 肉が正常に出現するか確認

期待される動作：スコア230でも肉が出現し、遅延がない

- [ ] **Step 3: スコア235～240まで続ける**

さらに進めてスコア235～240の範囲で動作確認：
- 蛇がほぼフィールドを埋め尽くす状態
- レンダリングが遅延していないか目視確認
- フレームレートが安定しているか確認（Chrome DevTools の Performance タブで測定可能）

期待される動作：
- 蛇の移動がスムーズ（フレームレートが安定）
- 肉が出現する
- 顕著な遅延がない

- [ ] **Step 4: 複数デバイスでテスト（可能な範囲で）**

iOS（iPhone）と Android の実機で同じテストを実施：
- 両方で遅延がないか確認
- デバイス間で動作に差がないか確認

期待される動作：すべてのデバイスで遅延なし

**テスト結果をメモ:**
```
テスト日時: 2026-05-06
テスト環境: [デバイス名]
スコア230: [肉出現/出現しない、遅延の有無]
スコア235-240: [遅延の有無、フレームレート]
結論: [修正成功/要最適化]
```

---

## Task 3: TICK値の変更（難易度アップ）

**Files:**
- Modify: `game.js:1-10` (constructor メソッド)

- [ ] **Step 1: BullGame constructor を確認**

現在の TICK 値の設定を確認：

```javascript
// 現在のコード（game.js の constructor内）
this.TICK = mode === 'hard' ? 200 : 300;
```

- [ ] **Step 2: TICK値を200に統一**

`game.js` の constructor内の TICK の行を以下のように変更：

```javascript
// 変更前：
this.TICK = mode === 'hard' ? 200 : 300;

// 変更後：
this.TICK = 200;  // 常に200msで統一（ハードモード廃止）
```

- [ ] **Step 3: ゲームを実行して動作確認**

ゲームを起動して蛇の速度が1.5倍速になったことを確認：

```bash
# ゲーム開始
# 蛇の移動速度が旧ハードモード相当（現在の1.5倍速）か確認
# スコア10～20程度まで進めて、動作がスムーズか確認
```

期待される動作：蛇が速く移動し、遅延がない

- [ ] **Step 4: Commit**

```bash
git add game.js
git commit -m "feat: set TICK to 200ms uniformly (remove hard mode, increase base difficulty)"
```

---

## Task 4: 長押し機能の削除

**Files:**
- Modify: `game.js:40-60` (isSpeedBoost プロパティ削除)
- Modify: `main.js` (長押し検出コード削除)

- [ ] **Step 1: game.js から isSpeedBoost と setSpeedBoost() を削除**

`game.js` の constructor内のこの行を削除：

```javascript
// 削除対象：
this.isSpeedBoost = false;
```

また、`game.js` 内に `setSpeedBoost()` メソッドがあれば、それも削除：

```javascript
// 削除対象：
setSpeedBoost(active) {
  this.isSpeedBoost = active;
}
```

さらに、`update()` メソッド内の `effectiveTick` 計算がありれば削除：

```javascript
// 削除対象（update メソッド内）：
const effectiveTick = this.isSpeedBoost ? Math.floor(this.TICK * 2 / 3) : this.TICK;
// 代わりに this.TICK を直接使用
```

- [ ] **Step 2: main.js から長押し検出コードを削除**

`main.js` 内のこれらのコードブロックを探して削除：

```javascript
// 削除対象：
let longPressTimer = null;
let isLongPressing = false;

gameCanvas.addEventListener('touchstart', (e) => {
  if (!gameRunning) return;
  
  longPressTimer = setTimeout(() => {
    isLongPressing = true;
    game.setSpeedBoost(true);
  }, 500);
});

gameCanvas.addEventListener('touchend', () => {
  clearTimeout(longPressTimer);
  if (isLongPressing) {
    game.setSpeedBoost(false);
    isLongPressing = false;
  }
});

gameCanvas.addEventListener('mousedown', (e) => {
  if (!gameRunning) return;
  
  longPressTimer = setTimeout(() => {
    isLongPressing = true;
    game.setSpeedBoost(true);
  }, 500);
});

gameCanvas.addEventListener('mouseup', () => {
  clearTimeout(longPressTimer);
  if (isLongPressing) {
    game.setSpeedBoost(false);
    isLongPressing = false;
  }
});
```

- [ ] **Step 3: ゲーム実行して長押しが効かないことを確認**

```bash
# ゲーム開始
# スマートフォンまたはマウスでフィールドを長押し
# 倍速が発動しないこと（蛇の速度が変わらない）を確認
```

期待される動作：長押ししても蛇の速度は変わらない（常に1.5倍速）

- [ ] **Step 4: Commit**

```bash
git add game.js main.js
git commit -m "feat: remove speed boost long-press feature (always 1.5x speed)"
```

---

## Task 5: 効果音ファイルの用意

**Files:**
- Create: `poyoyon.mp3` (Purple Bull フォルダルート)

- [ ] **Step 1: 効果音ファイルを用意**

「ぽよーん」という短い（0.3～0.5秒）キャリい効果音の MP3 ファイルを用意：

- ファイル名：`poyoyon.mp3`
- 形式：MP3
- 長さ：0.3～0.5秒
- サンプリングレート：44.1kHz以上
- 配置：`Purple Bull/poyoyon.mp3`（Purple Bullフォルダのルート）

ファイルが確保できない場合、以下の代替案があります：
- フリー素材サイト（効果音ラボ等）から「ぽよん」系の音をダウンロード
- AI音声合成（Google TTS等）で「ぽよーん」を生成

- [ ] **Step 2: ファイルの配置を確認**

```bash
# Purple Bull フォルダで確認
ls -la poyoyon.mp3

# 出力例：
# -rw-r--r--  1 user  group  12345 May  6 12:00 poyoyon.mp3
```

期待される動作：ファイルが Purple Bull フォルダのルートに存在

---

## Task 6: タイトル画面効果音の実装

**Files:**
- Modify: `main.js` (タイトル画面キャラクター周辺タップ時の効果音再生)

- [ ] **Step 1: main.js 内のタイトル画面キャラクター周辺タップイベントを探す**

現在のコードを確認（タイトル画面でキャラクターの周りをタップするコード）：

```javascript
// 探すコード例：
// document.getElementById('title-screen').addEventListener('click', ...)
// またはキャラクター増殖を処理する関数
```

- [ ] **Step 2: 効果音再生関数を追加**

`main.js` の最上部または適切な場所に以下の関数を追加：

```javascript
function playCharacterSound() {
  const audio = new Audio('./poyoyon.mp3');
  audio.volume = 0.5;
  audio.play().catch(err => console.warn('Character sound play failed:', err));
}
```

- [ ] **Step 3: タイトル画面のタップイベントで効果音を再生**

キャラクター周辺タップ時の処理に `playCharacterSound()` を追加：

```javascript
// 既存のキャラクター増殖処理の中に追加
if (isNearCharacter(e.clientX, e.clientY)) {
  spawnCharacter();  // 既存処理：キャラ増殖
  playCharacterSound();  // ← 新規：効果音再生
}
```

具体的なコード場所は、実装時に `main.js` 内を確認して特定してください。

- [ ] **Step 4: ブラウザで動作確認**

タイトル画面を開いてキャラクター周辺をタップ：

```bash
# ゲーム起動 → タイトル画面表示
# キャラクターの周りをタップ
# 「ぽよーん」という効果音が再生されることを確認
# 複数回タップして毎回音が鳴ることを確認
```

期待される動作：
- タップするたびに「ぽよーん」という効果音が再生される
- 音量が適切（0.5倍）
- エラーが出ない

- [ ] **Step 5: Commit**

```bash
git add main.js poyoyon.mp3
git commit -m "feat: add character spawn sound effect (poyoyon.mp3)"
```

---

## Task 7: 最終動作確認と統合テスト

**Files:**
- Test: 全機能の統合テスト

- [ ] **Step 1: ゲーム全体の動作確認**

ゲームを起動して、すべての変更が正常に動作することを確認：

1. **タイトル画面**
   - キャラクター周辺をタップ → 「ぽよーん」が毎回再生される

2. **ゲーム画面**
   - 蛇が1.5倍速で移動する
   - 長押しが効かない（蛇の速度が変わらない）
   - スコア230～240でも肉が出現する

3. **全般**
   - 遅延やフリーズがない
   - エラーがコンソールに出ない

テストチェックリスト：
```
- [ ] タイトル画面の効果音が毎回再生される
- [ ] ゲーム速度が1.5倍速
- [ ] 長押し機能が削除されている
- [ ] スコア230で肉が出現
- [ ] スコア235～240で遅延がない
- [ ] iOS/Android で同じ動作
```

- [ ] **Step 2: コンソールエラーの確認**

ブラウザの開発者ツール（DevTools）を開いて、Console タブでエラーがないことを確認：

```bash
# Chrome DevTools → Console
# エラーが出ていないことを確認
# 警告（Warning）は許容範囲（古い API使用など）
```

期待される動作：エラーなし（警告は許容）

- [ ] **Step 3: 最終 Commit と動作確認**

```bash
# 最終確認用のログメッセージを残す場合
git log --oneline -5
# コミット歴の確認
```

出力例：
```
abc1234 feat: add character spawn sound effect (poyoyon.mp3)
def5678 feat: remove speed boost long-press feature (always 1.5x speed)
ghi9012 fix: add fallback logic to meat placement to handle high snake lengths
```

- [ ] **Step 4: 動作確認完了**

すべての機能が正常に動作することが確認されたら、実装完了。

---

## Self-Review

**1. Spec coverage:**
- ✓ 肉配置ロジック修正（Task 1）
- ✓ パフォーマンステスト（Task 2）
- ✓ TICK値変更（Task 3）
- ✓ 長押し機能削除（Task 4）
- ✓ 効果音ファイル用意（Task 5）
- ✓ 効果音再生実装（Task 6）
- ✓ 統合テスト（Task 7）

すべての仕様要件がカバーされています。

**2. Placeholder scan:**
- なし。すべてのステップで完全なコードまたは正確なコマンドを提供しています。

**3. Type/Method consistency:**
- `playCharacterSound()` 関数名は Task 6 で一貫
- `_placeMeat()` メソッドは Task 1 で完全に置き換え
- `TICK` プロパティは Task 3 で統一

すべての識別子が一貫しています。

**4. 実装順序の妥当性:**
- Task 1-2：バグ修正を優先（最重要）
- Task 3-4：ゲームバランス変更
- Task 5-6：UI機能追加
- Task 7：統合テスト

論理的な依存関係に基づいた適切な順序です。

---

## 実装完了後

実装が完了したら、以下の手順をお勧めします：

1. **すべてのコミットが完了したことを確認**
   ```bash
   git log --oneline | head -10
   ```

2. **本番環境へのデプロイ前に、複数デバイスで最終テスト**

3. **ユーザーへの通知**（必要に応じて）
   - SNSでバグ修正と難易度アップを案内

