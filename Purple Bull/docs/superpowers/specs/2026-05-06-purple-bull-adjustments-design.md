# Purple Bull - 仕様微調整設計

**日付**: 2026-05-06  
**プロジェクト**: Purple Bull  
**対象**: 難易度調整、バグ修正、効果音追加

---

## 概要

Purple Bull（スネークゲーム）の3つの仕様微調整を実装します：

1. **肉が出ないバグ修正**：スコア230以降で肉が出現しなくなるバグを修正
2. **難易度アップ**：ハードモード廃止、ノーマルモードを1.5倍速に統一
3. **タイトル画面効果音**：キャラクター増殖時に「ぽよーん」という効果音を再生

---

## 変更1：肉が出ないバグ修正

### 問題
- スコア230～233前後で肉が出現しなくなる
- スマートフォンによって発生スコアが異なる
- 原因：ランダム試行（300回）で見つからないと肉が配置されない

### 根本原因
`game.js` の `_placeMeat()` メソッド：
- フィールド総マス数：12 × 20 = 240
- 95%ルール：蛇が228マス以上を占めたら肉配置を中止
- スコア230（蛇が232マス）で理論上は配置されないはずだが、それ以前にランダム試行が失敗する可能性がある

### 修正内容

**game.js の `_placeMeat()` メソッド**

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

  // ステップ1：ランダム試行で肉配置（見つかれば即座に返る）
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

### 特徴
- ランダム試行（300回）で高速に見つけることがほとんど
- 見つからない場合のみフォールバック（全マススキャン）を実行
- 蛇が239マス以上の極限状態でのみフォールバックが実行される（実用上ほぼなし）
- パフォーマンスへの影響は最小限

### テスト計画
- [ ] スコア230で肉が出現するか確認
- [ ] スコア235、239で肉が出現するか確認
- [ ] スマートフォンでパフォーマンス測定（遅延がないか）
- [ ] 複数デバイスで動作確認

---

## 変更2：難易度アップ（TICK値変更 + 長押し機能削除）

### 内容

**1. ハードモード廃止、ノーマルを1.5倍速に統一**

現在のモード設定：
- ノーマルモード：TICK = 300ms
- ハードモード：TICK = 200ms

変更後：
- すべてのゲームを TICK = 200ms で実行（ハードモード廃止）

**game.js の変更**

```javascript
class BullGame {
  constructor({ fieldEl, scoreEl, nickname, mode = 'normal' }) {
    this.GRID_COLS = 12;
    this.GRID_ROWS = 20;
    // 変更：mode によらず TICK = 200 に統一
    this.TICK = 200;  // ← 固定値（旧ハードモード相当）
    this.fieldEl  = fieldEl;
    this.scoreEl  = scoreEl;
    this.nickname = nickname;
    this.mode = mode;  // mode パラメータは保持（互換性のため）
    
    // ... 他のコード変わらず ...
  }
}
```

**2. スマートフォン長押し1.5倍速機能の削除**

以下のコードを `main.js` から削除：

```javascript
// 削除対象：
// - gameCanvas.addEventListener('touchstart', ...) ブロック
// - gameCanvas.addEventListener('touchend', ...) ブロック
// - gameCanvas.addEventListener('mousedown', ...) ブロック
// - gameCanvas.addEventListener('mouseup', ...) ブロック
// - longPressTimer, isLongPressing 変数
```

**game.js から削除：**

```javascript
// 削除対象：
// - this.isSpeedBoost プロパティ
// - setSpeedBoost() メソッド
// - update() 内の effectiveTick 計算ロジック
```

### 効果
- ゲーム難易度が1.5倍速になる
- ユーザーは難易度調整できない（シンプル化）
- UIから長押し検出の複雑さを削除

### テスト計画
- [ ] ノーマルモード選択時、蛇の移動速度が1.5倍速か確認
- [ ] 長押ししても倍速が発動しないことを確認
- [ ] iOS/Android 両方で動作確認

---

## 変更3：タイトル画面効果音追加

### 内容

タイトル画面でキャラクターの周りをタップしたときに、「ぽよーん」という効果音を再生します。

### 必要なファイル

効果音ファイル：`./poyoyon.mp3`
- 音声：「ぽよーん」というかわいい音
- 長さ：0.3～0.5秒程度
- フォーマット：MP3
- 配置：Purple Bull フォルダのルートに配置

### 実装方法

**main.js での実装**

```javascript
// 効果音再生関数（新規追加）
function playCharacterSound() {
  const audio = new Audio('./poyoyon.mp3');
  audio.volume = 0.5;
  audio.play().catch(err => console.warn('Character sound play failed:', err));
}

// タイトル画面のキャラクター周辺タップイベント（既存コード内）
// タップでキャラが増殖する部分を探して、以下を追加：

document.getElementById('title-screen').addEventListener('click', (e) => {
  // 既存：キャラクター周辺判定ロジック
  if (isNearCharacter(e.clientX, e.clientY)) {
    spawnCharacter();  // 既存処理：キャラ増殖
    playCharacterSound();  // ← 新規：効果音再生
  }
});
```

**game.js での処理（既存コードの拡張）**

`_preloadSounds()` メソッドに 'poyoyon.mp3' を追加（オプション）：

```javascript
_preloadSounds() {
  const sounds = ['bashi.mp3', 'dosu.mp3', 'paku.mp3', 'kabe.mp3', 'ushi.mp3', 'poyoyon.mp3'];
  sounds.forEach(filename => {
    const key = filename.replace('.mp3', '');
    if (!this.soundPool[key]) {
      const audio = new Audio('./' + filename);
      audio.volume = filename === 'ushi.mp3' ? 0.25 : 0.5;
      this.soundPool[key] = audio;
    }
  });
}
```

### 特徴
- タップするたびに「ぽよーん」という効果音が鳴る
- タップの反応がより直感的になる
- 複数回タップでも毎回効果音が鳴る

### テスト計画
- [ ] タイトル画面でキャラ周辺タップで効果音が鳴るか
- [ ] 複数回タップで毎回効果音が鳴るか
- [ ] iOS/Android で音量バランスが適切か

---

## ファイル変更一覧

| ファイル | 変更内容 |
|---------|--------|
| `game.js` | ・`_placeMeat()` メソッドをフォールバック処理対応に変更 ・`TICK = 200` に統一 ・`isSpeedBoost`, `setSpeedBoost()` 削除 ・`_preloadSounds()` に 'poyoyon.mp3' 追加（オプション） |
| `main.js` | ・長押し検出コード削除 ・タイトル画面キャラクター周辺タップで効果音再生 |
| 新規ファイル | ・`poyoyon.mp3` （効果音ファイル、Purple Bull フォルダルート） |

---

## 実装順序

1. **肉配置ロジック修正** → パフォーマンステスト実施
2. **難易度アップ**（TICK変更 + 長押し機能削除）
3. **タイトル画面効果音追加**

各ステップ後にテストを実施し、問題がないことを確認してから次へ進みます。

---

## パフォーマンステスト計画

肉配置ロジックのフォールバック処理追加後、以下をテストします：

- **スマートフォン実機テスト**：スコア235～240前後でゲームが遅延しないか
- **複数デバイステスト**：iOS（iPhone）と Android で遅延の有無を確認
- **フレームレート測定**：ブラウザの開発者ツールで FPS 低下がないか確認

測定方法：
1. スコア230～240まで進める
2. フレームレート監視（Chrome DevTools の Performance タブ）
3. 遅延が検出された場合、フォールバック処理の最適化を検討

---

## 備考

- **互換性**：`mode` パラメータは保持するため、既存コードとの互換性が保たれる
- **効果音**：Web Audio API ではなく HTML5 Audio 要素を使用（シンプルで高い互換性）
- **バグの根本解決**：フォールバック処理により、蛇が全マスを占めない限り肉が必ず出現
