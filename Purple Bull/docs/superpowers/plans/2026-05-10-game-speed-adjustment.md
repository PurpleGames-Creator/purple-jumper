# ゲーム移動速度調整 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Purple Bullゲームのプレイ中における蛇の移動速度を現在の約1.3倍に高速化する

**Architecture:** BullGameクラスのコンストラクタで蛇の移動フレーム間隔（TICK値）を200msから約154msに変更。この単一の値変更により、ゲーム全体の速度が1.3倍になる。

**Tech Stack:** JavaScript, Canvas 2D, setInterval

---

## ファイル構造

- **修正:** `game.js:5` - BullGameクラスコンストラクタのTICK値

---

## 実装タスク

### Task 1: TICK値の変更

**Files:**
- Modify: `game.js:5`

- [ ] **Step 1: game.jsのTICK値を確認**

`game.js`の5行目を確認します：

```javascript
this.TICK  = 200;  // 常に200msで統一（ハードモード廃止）
```

現在のTICK値が200msに設定されていることを確認します。

- [ ] **Step 2: TICK値を計算値に変更**

game.jsの5行目を以下のように変更します：

```javascript
this.TICK = Math.round(200 / 1.3);  // 約154ms（1.3倍速）
```

変更理由：
- 200 / 1.3 ≈ 153.85ms
- `Math.round()` で四捨五入して154msに
- フレーム間隔が短くなるため、蛇の移動速度が1.3倍になる

- [ ] **Step 3: ゲームをプレイして速度確認**

プレビューサーバーでゲームをテストして、以下を確認します：
- ゲーム開始直後に蛇が移動開始する
- 従来より速く移動していることを視認確認
- ゲーム全体を通して速度が一定か確認
- 肉の食べやすさ、ゲームバランスが適切か確認

- [ ] **Step 4: Commit**

以下のコマンドで変更をコミットします：

```bash
git add game.js
git commit -m "feat: Increase game speed to 1.3x by adjusting TICK value from 200ms to 154ms"
```

---

## テスト計画

1. **基本動作確認**
   - ゲーム画面を開き、GAME STARTボタンをクリック
   - 蛇が移動開始することを確認
   - 移動速度が従来より速いことを視認確認

2. **安定性確認**
   - ゲーム開始から終了まで速度が一定か確認
   - キー入力による方向変更が正常に動作するか確認

3. **ゲームバランス確認**
   - 新しい速度でハイスコア達成可能か試す
   - 難易度が適切か（難しすぎないか、易しすぎないか）確認

---

## 実装の簡潔さ

このタスクは極めてシンプルです：
- **変更行数:** 1行
- **変更内容:** 200 → Math.round(200 / 1.3)
- **影響範囲:** ゲームの全体速度のみ
- **副作用:** なし（他のロジックへの影響なし）

---

## 備考

- TICK値はゲームループの基本的なタイミング値です
- この値を小さくすると、より短い時間間隔で蛇が移動します
- 将来さらに速度を調整したい場合は、この値を変更するだけで対応可能です
