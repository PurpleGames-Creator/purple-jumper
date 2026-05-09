# ゲーム移動速度調整 実装仕様

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Purple Bull ゲームのプレイ中における蛇の移動速度を現在の約1.3倍に高速化する

**Architecture:** ゲーム初期化時に蛇の移動フレーム間隔（TICK値）を計算で調整。ゲーム全体を通して一定の速度を維持する。

**Tech Stack:** JavaScript, Canvas 2D, setInterval

---

## 設計詳細

### 現在の仕様
- **フレーム間隔（TICK）:** 200ミリ秒
- **蛇の移動ペース:** 200msごとに1マス移動
- **ゲーム全体:** 一定速度

### 変更内容
- **新しいTICK値:** `200 / 1.3 ≈ 154ms` に計算
- **実装場所:** `game.js` の `BullGame` クラスコンストラクタ（現在5行目）
- **計算式:** `this.TICK = Math.round(200 / 1.3);` または `this.TICK = 154;`

### 期待される効果
- 蛇の移動速度が約1.3倍速くなる
- ゲーム開始から終了まで一定速度を維持
- ゲームバランスの微調整

### 影響範囲
- `game.js` のコンストラクタのみ
- 他のゲームロジックに影響なし
- スコア計算、肉生成、衝突判定などは変わらず

---

## テスト計画

1. **基本動作確認**
   - ゲーム開始時に蛇が移動開始する
   - 移動速度が従来より速いことを確認

2. **安定性確認**
   - ゲーム開始から終了まで速度が一定か確認
   - 長時間プレイでも速度低下がないか確認

3. **ゲームバランス確認**
   - ハイスコア達成可能か確認
   - ゲーム難易度が適切か確認

---

## 実装の簡潔さ

このは修正は **1行の数値変更** のみです：

```javascript
// 変更前
this.TICK  = 200;

// 変更後
this.TICK = Math.round(200 / 1.3);  // または 154
```

---

## 備考

- 後で速度をさらに調整したい場合、この TICK 値を変更するだけで対応可能
- 難易度段階ごとに異なる速度にしたい場合も、ここを起点に拡張できる
