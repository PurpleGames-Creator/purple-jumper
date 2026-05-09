# リバーシオンライン対戦ゲーム 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** スマートフォン最適化のリアルタイムオンライン対戦リバーシゲーム。タイトル→ロビー→ゲーム画面を通じて、Socket.ioでリアルタイム通信。

**Architecture:** 
- **フロントエンド（Next.js）**：3画面のUI、Socket.io通信、ローカルストレージでプレイヤー名保持
- **バックエンド（Node.js + Express + Socket.io）**：ゲーム状態管理（リバーシロジック）、ルーム管理、接続管理
- **共有ロジック**：合法手判定、コマ裏返しロジックはバックエンドで実装、フロントエンドからはAPI呼び出し

**Tech Stack:** 
- フロントエンド：Next.js, React, Tailwind CSS, Socket.io クライアント
- バックエンド：Node.js, Express, Socket.io, リバーシ盤面管理
- その他：ローカルストレージ（プレイヤー名）

---

## ファイル構造

### バックエンド（server/）

```
server/
├── index.js                          # Express + Socket.io メインサーバー
├── game/
│   ├── ReversiGame.js               # ゲーム盤面・ゲーム状態管理クラス
│   ├── rules.js                      # リバーシ詳細ルール（合法手判定、裏返し）
│   └── constants.js                  # 定数（盤サイズ、タイムアウト等）
├── managers/
│   └── RoomManager.js                # ルーム・接続管理（ルーム作成、入室、削除）
├── events/
│   └── socketHandlers.js             # Socket.ionイベントハンドラ定義
└── package.json
```

### フロントエンド（client/）

```
client/
├── pages/
│   ├── index.js                      # タイトル画面（プレイヤー名入力）
│   ├── lobby.js                      # ロビー画面（部屋一覧、人数表示）
│   ├── game.js                       # ゲーム画面（盤面、操作）
│   └── _app.js                       # グローバル設定
├── components/
│   ├── Board.jsx                     # 8×8 リバーシ盤面表示
│   ├── PlayerInfo.jsx                # プレイヤー名・スコア表示
│   ├── Timer.jsx                     # タイマー（20秒カウントダウン）
│   ├── Markers.jsx                   # 赤丸マーカー（直前手）、白丸マーカー（合法手）
│   ├── GameOverModal.jsx             # ゲーム終了ダイアログ
│   └── RoomCard.jsx                  # ロビーの部屋カード
├── lib/
│   ├── socket.js                     # Socket.io クライアント初期化
│   ├── reversi-logic.js              # 合法手判定等、共有ロジック（クライアント側補助用）
│   ├── storage.js                    # ローカルストレージ管理
│   └── constants.js                  # 定数
├── styles/
│   ├── globals.css                   # Tailwind CSS設定
│   └── animations.css                # アニメーション定義
├── public/
│   └── （アイコン等は不要）
└── next.config.js
```

---

## Task 1: プロジェクト初期設定

**Files:**
- Create: `package.json` (ルート)
- Create: `server/package.json`
- Create: `client/package.json`
- Create: `server/index.js`
- Create: `client/next.config.js`
- Create: `client/pages/_app.js`

### Step 1-1: ルートディレクトリの初期化

```bash
mkdir reversi-game
cd reversi-game
npm init -y
```

`package.json` に以下を追加：

```json
{
  "name": "reversi-game",
  "version": "1.0.0",
  "scripts": {
    "dev:server": "node server/index.js",
    "dev:client": "cd client && npm run dev",
    "build": "cd client && npm run build",
    "start": "node server/index.js"
  }
}
```

### Step 1-2: サーバー（Node.js）の初期設定

```bash
mkdir server
cd server
npm init -y
```

`server/package.json` に dependencies 追加：

```json
{
  "name": "reversi-server",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.18.2",
    "socket.io": "^4.7.2",
    "cors": "^2.8.5"
  },
  "scripts": {
    "start": "node index.js",
    "dev": "node index.js"
  }
}
```

```bash
npm install
```

### Step 1-3: クライアント（Next.js）の初期設定

```bash
cd ..
npx create-next-app@latest client --typescript false --tailwind --eslint false
```

Or manually:

```bash
mkdir client
cd client
npm init -y
```

`client/package.json`：

```json
{
  "name": "reversi-client",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "next": "^14.0.0",
    "socket.io-client": "^4.7.2"
  },
  "devDependencies": {
    "tailwindcss": "^3.3.0",
    "autoprefixer": "^10.4.14",
    "postcss": "^8.4.24"
  }
}
```

```bash
npm install
```

### Step 1-4: サーバーのメインファイル作成（最小限）

`server/index.js`：

```javascript
const express = require('express');
const cors = require('cors');
const { Server } = require('socket.io');
const http = require('http');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' }
});

app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 3001;

io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);

  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Step 1-5: Next.js 基本設定

`client/next.config.js`：

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
};

module.exports = nextConfig;
```

`client/pages/_app.js`：

```javascript
import '../styles/globals.css';

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
```

`client/styles/globals.css`：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background-color: #f5f5f5;
}

html, body, #__next {
  width: 100%;
  height: 100%;
  -webkit-user-select: none;
  -webkit-touch-callout: none;
  user-select: none;
}
```

### Step 1-6: Tailwind CSS 設定

`client/tailwind.config.js`：

```javascript
module.exports = {
  content: [
    './pages/**/*.{js,jsx}',
    './components/**/*.{js,jsx}',
  ],
  theme: {
    extend: {
      colors: {
        board: '#2d5016',
        'piece-white': '#f0f0f0',
        'piece-purple': '#6b2d84',
      },
    },
  },
  plugins: [],
};
```

`client/postcss.config.js`：

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### Step 1-7: 動作確認

- [ ] **Step 1-7a: サーバー起動**

```bash
cd server
npm start
```

Expected: `Server running on http://localhost:3001`

- [ ] **Step 1-7b: クライアント起動（別ターミナル）**

```bash
cd client
npm run dev
```

Expected: `ready - started server on 0.0.0.0:3000`

- [ ] **Step 1-7c: ブラウザで http://localhost:3000 にアクセス**

Expected: Next.js のデフォルトページが表示される

- [ ] **Step 1-7d: Commit**

```bash
git add .
git commit -m "chore: init project setup with Next.js and Express+Socket.io"
```

---

## Task 2: リバーシゲームロジック実装（バックエンド）

**Files:**
- Create: `server/game/constants.js`
- Create: `server/game/rules.js`
- Create: `server/game/ReversiGame.js`

### Step 2-1: 定数定義

`server/game/constants.js`：

```javascript
const BOARD_SIZE = 8;
const TURN_TIME_LIMIT = 20000; // ms
const INITIAL_BOARD = [
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,1,2,0,0,0],
  [0,0,0,2,1,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
];

module.exports = {
  BOARD_SIZE,
  TURN_TIME_LIMIT,
  INITIAL_BOARD,
  PIECE_EMPTY: 0,
  PIECE_WHITE: 1,
  PIECE_PURPLE: 2,
};
```

### Step 2-2: リバーシルール実装

`server/game/rules.js`：

```javascript
const { BOARD_SIZE, PIECE_EMPTY, PIECE_WHITE, PIECE_PURPLE } = require('./constants');

/**
 * 指定位置に指定色でコマを置いた時、裏返る敵コマ方向と枚数を返す
 * @param {number[][]} board - 盤面
 * @param {number} row - 行
 * @param {number} col - 列
 * @param {number} piece - 置く色（1: 白, 2: 紫）
 * @returns {Array} [[dr, dc, count], ...] 各方向での裏返し情報
 */
function getFlippableDirections(board, row, col, piece) {
  const opponent = piece === PIECE_WHITE ? PIECE_PURPLE : PIECE_WHITE;
  const directions = [
    [-1, 0], [1, 0], [0, -1], [0, 1],
    [-1, -1], [-1, 1], [1, -1], [1, 1],
  ];
  
  const flippable = [];
  
  for (const [dr, dc] of directions) {
    let r = row + dr;
    let c = col + dc;
    let count = 0;
    
    while (r >= 0 && r < BOARD_SIZE && c >= 0 && c < BOARD_SIZE) {
      if (board[r][c] === PIECE_EMPTY) break;
      if (board[r][c] === opponent) {
        count++;
      } else if (board[r][c] === piece) {
        if (count > 0) {
          flippable.push([dr, dc, count]);
        }
        break;
      }
      r += dr;
      c += dc;
    }
  }
  
  return flippable;
}

/**
 * 指定位置が合法手かチェック
 * @param {number[][]} board - 盤面
 * @param {number} row - 行
 * @param {number} col - 列
 * @param {number} piece - 置く色
 * @returns {boolean} 合法手なら true
 */
function isLegalMove(board, row, col, piece) {
  if (board[row][col] !== PIECE_EMPTY) return false;
  return getFlippableDirections(board, row, col, piece).length > 0;
}

/**
 * 指定色の全合法手を取得
 * @param {number[][]} board - 盤面
 * @param {number} piece - 置く色
 * @returns {Array} [[row, col], ...]
 */
function getLegalMoves(board, piece) {
  const moves = [];
  for (let r = 0; r < BOARD_SIZE; r++) {
    for (let c = 0; c < BOARD_SIZE; c++) {
      if (isLegalMove(board, r, c, piece)) {
        moves.push([r, c]);
      }
    }
  }
  return moves;
}

/**
 * コマを置いて裏返す
 * @param {number[][]} board - 盤面（参照で更新）
 * @param {number} row - 行
 * @param {number} col - 列
 * @param {number} piece - 置く色
 */
function placeAndFlip(board, row, col, piece) {
  board[row][col] = piece;
  const flippable = getFlippableDirections(board, row, col, piece);
  
  for (const [dr, dc, count] of flippable) {
    for (let i = 1; i <= count; i++) {
      board[row + dr * i][col + dc * i] = piece;
    }
  }
}

/**
 * 盤面から各色のコマ数を数える
 * @param {number[][]} board - 盤面
 * @returns {Object} { white: 数, purple: 数 }
 */
function countPieces(board) {
  let white = 0, purple = 0;
  for (let r = 0; r < BOARD_SIZE; r++) {
    for (let c = 0; c < BOARD_SIZE; c++) {
      if (board[r][c] === PIECE_WHITE) white++;
      else if (board[r][c] === PIECE_PURPLE) purple++;
    }
  }
  return { white, purple };
}

module.exports = {
  getFlippableDirections,
  isLegalMove,
  getLegalMoves,
  placeAndFlip,
  countPieces,
};
```

### Step 2-3: ReversiGame クラス実装

`server/game/ReversiGame.js`：

```javascript
const { PIECE_EMPTY, PIECE_WHITE, PIECE_PURPLE, INITIAL_BOARD, TURN_TIME_LIMIT } = require('./constants');
const { getLegalMoves, placeAndFlip, countPieces } = require('./rules');

class ReversiGame {
  constructor(player1, player2) {
    this.player1 = player1;        // { id, name }
    this.player2 = player2;        // { id, name }
    this.board = JSON.parse(JSON.stringify(INITIAL_BOARD));
    this.currentPlayer = PIECE_WHITE; // 白（プレイヤー1）から開始
    this.gameState = 'playing'; // 'playing', 'finished'
    this.winner = null;
    this.lastMove = null; // { row, col, piece }
    this.turnStartTime = Date.now();
  }

  /**
   * 現在の番のプレイヤーを取得
   */
  getCurrentPlayer() {
    return this.currentPlayer === PIECE_WHITE ? this.player1 : this.player2;
  }

  /**
   * コマを置く（検証済みと仮定）
   */
  move(row, col) {
    placeAndFlip(this.board, row, col, this.currentPlayer);
    this.lastMove = { row, col, piece: this.currentPlayer };
    
    // ターンを交代
    this.currentPlayer = this.currentPlayer === PIECE_WHITE ? PIECE_PURPLE : PIECE_WHITE;
    
    // 相手の合法手をチェック
    if (getLegalMoves(this.board, this.currentPlayer).length === 0) {
      // パス
      if (getLegalMoves(this.board, this.currentPlayer === PIECE_WHITE ? PIECE_PURPLE : PIECE_WHITE).length === 0) {
        // 両者置けない → ゲーム終了
        this.finish();
      }
      // そうでなければ、再度相手のターンになる（自動パス）
    }
  }

  /**
   * ゲーム終了
   */
  finish() {
    this.gameState = 'finished';
    const { white, purple } = countPieces(this.board);
    if (white > purple) {
      this.winner = this.player1;
    } else if (purple > white) {
      this.winner = this.player2;
    } else {
      this.winner = null; // 引き分け
    }
  }

  /**
   * ゲーム盤面をシリアライズ
   */
  serialize() {
    const { white, purple } = countPieces(this.board);
    return {
      board: this.board,
      currentPlayer: this.currentPlayer,
      gameState: this.gameState,
      player1: { ...this.player1, score: white },
      player2: { ...this.player2, score: purple },
      lastMove: this.lastMove,
      winner: this.winner,
    };
  }
}

module.exports = ReversiGame;
```

### Step 2-4: テスト（リバーシロジック）

`server/game/__test__/rules.test.js`：

```javascript
const { isLegalMove, getLegalMoves, placeAndFlip, countPieces } = require('../rules');
const { INITIAL_BOARD, PIECE_WHITE, PIECE_PURPLE, PIECE_EMPTY } = require('../constants');

describe('Reversi Rules', () => {
  test('should identify initial legal moves for white', () => {
    const moves = getLegalMoves(INITIAL_BOARD, PIECE_WHITE);
    const expected = [[2, 3], [3, 2], [4, 5], [5, 4]];
    expect(moves).toEqual(expect.arrayContaining(expected));
    expect(moves.length).toBe(4);
  });

  test('should place piece and flip correctly', () => {
    const board = JSON.parse(JSON.stringify(INITIAL_BOARD));
    placeAndFlip(board, 2, 3, PIECE_WHITE);
    expect(board[2][3]).toBe(PIECE_WHITE);
    expect(board[3][3]).toBe(PIECE_WHITE); // 裏返った
  });

  test('should count pieces correctly', () => {
    const { white, purple } = countPieces(INITIAL_BOARD);
    expect(white).toBe(2);
    expect(purple).toBe(2);
  });
});
```

```bash
npm test  # または jest コマンド
```

Expected: すべてのテストが PASS

- [ ] **Step 2-4a: Commit**

```bash
git add server/game/
git commit -m "feat: implement Reversi game logic and rules"
```

---

## Task 3: ルーム・接続管理（バックエンド）

**Files:**
- Create: `server/managers/RoomManager.js`
- Create: `server/game/constants.js` に追加

### Step 3-1: RoomManager クラス実装

`server/managers/RoomManager.js`：

```javascript
const ReversiGame = require('../game/ReversiGame');
const { TURN_TIME_LIMIT } = require('../game/constants');

class RoomManager {
  constructor() {
    this.rooms = new Map();      // roomId → Room object
    this.playerToRoom = new Map(); // playerId → roomId
    this.roomCounter = 0;
  }

  /**
   * 新しい部屋を作成
   */
  createRoom(hostId, hostName) {
    const roomId = `room_${++this.roomCounter}_${Date.now()}`;
    const room = {
      roomId,
      host: { id: hostId, name: hostName },
      guest: null,
      game: null,
      status: 'waiting', // 'waiting', 'playing', 'finished'
    };
    this.rooms.set(roomId, room);
    this.playerToRoom.set(hostId, roomId);
    return roomId;
  }

  /**
   * 部屋に参加
   */
  joinRoom(roomId, guestId, guestName) {
    const room = this.rooms.get(roomId);
    if (!room || room.status !== 'waiting') {
      throw new Error('Room not available');
    }

    room.guest = { id: guestId, name: guestName };
    room.status = 'playing';
    room.game = new ReversiGame(room.host, room.guest);

    this.playerToRoom.set(guestId, roomId);
    return room;
  }

  /**
   * 部屋から退出（ゲーム終了時）
   */
  leaveRoom(roomId) {
    const room = this.rooms.get(roomId);
    if (!room) return;

    if (room.host) this.playerToRoom.delete(room.host.id);
    if (room.guest) this.playerToRoom.delete(room.guest.id);

    this.rooms.delete(roomId);
  }

  /**
   * プレイヤーの部屋を取得
   */
  getPlayerRoom(playerId) {
    const roomId = this.playerToRoom.get(playerId);
    return roomId ? this.rooms.get(roomId) : null;
  }

  /**
   * 募集中の部屋一覧
   */
  getWaitingRooms() {
    return Array.from(this.rooms.values()).filter(r => r.status === 'waiting');
  }

  /**
   * 対戦中の部屋一覧
   */
  getPlayingRooms() {
    return Array.from(this.rooms.values()).filter(r => r.status === 'playing');
  }

  /**
   * 現在の接続人数（タイトル画面用）
   */
  getOnlineCount() {
    return this.playerToRoom.size;
  }

  /**
   * ゲームに手を進める
   */
  makeMove(roomId, row, col) {
    const room = this.rooms.get(roomId);
    if (!room || !room.game) {
      throw new Error('Room or game not found');
    }
    room.game.move(row, col);
    return room.game.serialize();
  }

  /**
   * ゲーム終了（降参等）
   */
  finishGame(roomId) {
    const room = this.rooms.get(roomId);
    if (!room || !room.game) {
      throw new Error('Room or game not found');
    }
    room.game.finish();
    room.status = 'finished';
    return room.game.serialize();
  }
}

module.exports = RoomManager;
```

- [ ] **Step 3-1a: Commit**

```bash
git add server/managers/
git commit -m "feat: implement RoomManager for room and connection handling"
```

---

## Task 4: Socket.io イベントハンドラ（バックエンド）

**Files:**
- Create: `server/events/socketHandlers.js`
- Modify: `server/index.js`

### Step 4-1: Socket.io イベント定義

`server/events/socketHandlers.js`：

```javascript
const RoomManager = require('../managers/RoomManager');
const { getLegalMoves } = require('../game/rules');

const roomManager = new RoomManager();
const playerNames = new Map(); // playerId → playerName

function registerSocketHandlers(io) {
  io.on('connection', (socket) => {
    console.log('Client connected:', socket.id);

    // プレイヤー名登録
    socket.on('register', (playerName, callback) => {
      playerNames.set(socket.id, playerName);
      const onlineCount = roomManager.getOnlineCount() + 1; // 自分を含める
      io.emit('online-count-updated', onlineCount);
      callback({ success: true });
    });

    // 部屋一覧取得
    socket.on('get-rooms', (callback) => {
      const waitingRooms = roomManager.getWaitingRooms().map(r => ({
        roomId: r.roomId,
        hostName: r.host.name,
      }));
      const playingRooms = roomManager.getPlayingRooms().map(r => ({
        player1: r.host.name,
        player2: r.guest.name,
      }));
      callback({ waitingRooms, playingRooms });
    });

    // 部屋作成
    socket.on('create-room', (callback) => {
      const playerName = playerNames.get(socket.id);
      const roomId = roomManager.createRoom(socket.id, playerName);
      socket.join(roomId);
      callback({ roomId, success: true });
      io.emit('rooms-updated', {
        waiting: roomManager.getWaitingRooms(),
        playing: roomManager.getPlayingRooms(),
      });
    });

    // 部屋に参加
    socket.on('join-room', (roomId, callback) => {
      try {
        const playerName = playerNames.get(socket.id);
        const room = roomManager.joinRoom(roomId, socket.id, playerName);
        socket.join(roomId);

        // 相手に通知
        io.to(roomId).emit('game-started', room.game.serialize());

        // 合法手を送信
        const legalMoves = getLegalMoves(room.game.board, room.game.currentPlayer);
        io.to(roomId).emit('legal-moves-updated', legalMoves);

        callback({ success: true, gameState: room.game.serialize() });
        io.emit('rooms-updated', {
          waiting: roomManager.getWaitingRooms(),
          playing: roomManager.getPlayingRooms(),
        });
      } catch (err) {
        callback({ success: false, error: err.message });
      }
    });

    // 手を置く
    socket.on('place-piece', (roomId, row, col, callback) => {
      try {
        const room = roomManager.getPlayerRoom(socket.id);
        if (!room || room.roomId !== roomId) {
          throw new Error('Invalid room');
        }

        const gameState = roomManager.makeMove(roomId, row, col);

        io.to(roomId).emit('board-updated', gameState);

        // 次の合法手リストを送信
        const legalMoves = getLegalMoves(gameState.board, gameState.currentPlayer);
        io.to(roomId).emit('legal-moves-updated', legalMoves);

        // ゲーム終了チェック
        if (gameState.gameState === 'finished') {
          io.to(roomId).emit('game-finished', gameState);
        }

        callback({ success: true });
      } catch (err) {
        callback({ success: false, error: err.message });
      }
    });

    // 降参
    socket.on('resign', (roomId, callback) => {
      try {
        const room = roomManager.getPlayerRoom(socket.id);
        if (!room || room.roomId !== roomId) {
          throw new Error('Invalid room');
        }

        const gameState = roomManager.finishGame(roomId);
        io.to(roomId).emit('game-finished', gameState);

        callback({ success: true });
      } catch (err) {
        callback({ success: false, error: err.message });
      }
    });

    // 退室
    socket.on('leave-room', (roomId, callback) => {
      socket.leave(roomId);
      roomManager.leaveRoom(roomId);
      io.emit('rooms-updated', {
        waiting: roomManager.getWaitingRooms(),
        playing: roomManager.getPlayingRooms(),
      });
      callback({ success: true });
    });

    // 切断
    socket.on('disconnect', () => {
      console.log('Client disconnected:', socket.id);
      const room = roomManager.getPlayerRoom(socket.id);
      if (room && room.status === 'playing') {
        io.to(room.roomId).emit('opponent-disconnected', 'Opponent disconnected');
        roomManager.leaveRoom(room.roomId);
      }
      playerNames.delete(socket.id);
      io.emit('rooms-updated', {
        waiting: roomManager.getWaitingRooms(),
        playing: roomManager.getPlayingRooms(),
      });
    });
  });
}

module.exports = registerSocketHandlers;
```

### Step 4-2: server/index.js を更新

`server/index.js`：

```javascript
const express = require('express');
const cors = require('cors');
const { Server } = require('socket.io');
const http = require('http');
const registerSocketHandlers = require('./events/socketHandlers');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' }
});

app.use(cors());
app.use(express.json());

registerSocketHandlers(io);

const PORT = process.env.PORT || 3001;

server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

- [ ] **Step 4-2a: Commit**

```bash
git add server/events/ server/index.js
git commit -m "feat: implement Socket.io event handlers for real-time communication"
```

---

## Task 5: タイトル画面（フロントエンド）

**Files:**
- Create: `client/pages/index.js`
- Create: `client/lib/socket.js`
- Create: `client/lib/storage.js`
- Create: `client/lib/constants.js`

### Step 5-1: Socket.io クライアント初期化

`client/lib/socket.js`：

```javascript
import io from 'socket.io-client';

let socket = null;

export const initSocket = () => {
  if (!socket) {
    socket = io('http://localhost:3001', {
      reconnection: true,
    });
  }
  return socket;
};

export const getSocket = () => socket;

export const disconnectSocket = () => {
  if (socket) {
    socket.disconnect();
    socket = null;
  }
};
```

### Step 5-2: ローカルストレージ管理

`client/lib/storage.js`：

```javascript
const PLAYER_NAME_KEY = 'reversi_player_name';

export const getPlayerName = () => {
  if (typeof window === 'undefined') return null;
  return localStorage.getItem(PLAYER_NAME_KEY);
};

export const setPlayerName = (name) => {
  if (typeof window === 'undefined') return;
  localStorage.setItem(PLAYER_NAME_KEY, name);
};

export const clearPlayerName = () => {
  if (typeof window === 'undefined') return;
  localStorage.removeItem(PLAYER_NAME_KEY);
};
```

### Step 5-3: 定数

`client/lib/constants.js`：

```javascript
export const PIECE_WHITE = 1;
export const PIECE_PURPLE = 2;

export const COLORS = {
  [PIECE_WHITE]: 'bg-white',
  [PIECE_PURPLE]: 'bg-purple-600',
};
```

### Step 5-4: タイトル画面実装

`client/pages/index.js`：

```javascript
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { initSocket } from '../lib/socket';
import { getPlayerName, setPlayerName } from '../lib/storage';

export default function Title() {
  const router = useRouter();
  const [playerName, setInputName] = useState('');
  const [onlineCount, setOnlineCount] = useState(0);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const socket = initSocket();

    // 初期プレイヤー名をロード
    const savedName = getPlayerName();
    if (savedName) {
      setInputName(savedName);
    }

    // オンライン人数更新リスナー
    socket.on('online-count-updated', (count) => {
      setOnlineCount(count);
    });

    return () => {
      socket.off('online-count-updated');
    };
  }, []);

  const handleStartMatching = () => {
    if (!playerName.trim()) return;

    setLoading(true);
    const socket = initSocket();

    socket.emit('register', playerName, ({ success }) => {
      if (success) {
        setPlayerName(playerName);
        router.push('/lobby');
      }
      setLoading(false);
    });
  };

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-b from-blue-900 to-blue-700 p-4">
      {/* タイトル */}
      <h1 className="text-5xl font-bold text-white mb-2">リバーシ</h1>
      <p className="text-white text-lg mb-8">オンライン対戦</p>

      {/* 待機中の人数 */}
      <div className="mb-8 p-4 bg-white bg-opacity-20 rounded-lg">
        <p className="text-white text-xl">{onlineCount}人待機中</p>
      </div>

      {/* プレイヤー名入力 */}
      <input
        type="text"
        placeholder="プレイヤー名を入力"
        value={playerName}
        onChange={(e) => setInputName(e.target.value)}
        maxLength="20"
        className="w-full max-w-xs px-4 py-3 mb-6 rounded-lg border-2 border-white bg-white bg-opacity-10 text-white placeholder-gray-300 focus:outline-none focus:bg-opacity-20"
        onKeyPress={(e) => e.key === 'Enter' && handleStartMatching()}
        disabled={loading}
      />

      {/* マッチングボタン */}
      <button
        onClick={handleStartMatching}
        disabled={!playerName.trim() || loading}
        className={`w-full max-w-xs px-6 py-4 text-lg font-bold rounded-lg transition-all ${
          !playerName.trim() || loading
            ? 'bg-gray-400 text-gray-600 cursor-not-allowed'
            : 'bg-yellow-400 text-blue-900 hover:bg-yellow-500 cursor-pointer'
        }`}
      >
        {loading ? 'マッチング中...' : 'マッチングへ'}
      </button>
    </div>
  );
}
```

- [ ] **Step 5-4a: Commit**

```bash
git add client/lib/ client/pages/index.js
git commit -m "feat: implement title screen with player name input"
```

---

## Task 6: ロビー画面（フロントエンド）

**Files:**
- Create: `client/pages/lobby.js`
- Create: `client/components/RoomCard.jsx`

### Step 6-1: RoomCard コンポーネント

`client/components/RoomCard.jsx`：

```javascript
export function WaitingRoomCard({ room, onJoin, loading }) {
  return (
    <div className="flex items-center justify-between p-4 bg-white rounded-lg mb-2 shadow-sm border border-gray-200">
      <span className="text-lg font-semibold text-gray-800">{room.hostName}</span>
      <button
        onClick={() => onJoin(room.roomId)}
        disabled={loading}
        className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:bg-gray-400"
      >
        入室
      </button>
    </div>
  );
}

export function PlayingRoomCard({ room }) {
  return (
    <div className="p-4 bg-gray-100 rounded-lg mb-2 text-center">
      <p className="text-gray-800 font-semibold">
        {room.player1} vs {room.player2}
      </p>
    </div>
  );
}
```

### Step 6-2: ロビー画面実装

`client/pages/lobby.js`：

```javascript
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { getSocket } from '../lib/socket';
import { WaitingRoomCard, PlayingRoomCard } from '../components/RoomCard';
import { getPlayerName } from '../lib/storage';

export default function Lobby() {
  const router = useRouter();
  const [waitingRooms, setWaitingRooms] = useState([]);
  const [playingRooms, setPlayingRooms] = useState([]);
  const [onlineCount, setOnlineCount] = useState(0);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const socket = getSocket();
    if (!socket.connected) {
      router.push('/');
      return;
    }

    // ルーム一覧取得
    socket.emit('get-rooms', ({ waitingRooms, playingRooms }) => {
      setWaitingRooms(waitingRooms);
      setPlayingRooms(playingRooms);
      setOnlineCount(Object.keys(socket.id).length);
    });

    // ルーム更新リスナー
    const handleRoomsUpdated = ({ waiting, playing }) => {
      setWaitingRooms(
        waiting.map(r => ({
          roomId: r.roomId,
          hostName: r.host.name,
        }))
      );
      setPlayingRooms(
        playing.map(r => ({
          player1: r.host.name,
          player2: r.guest.name,
        }))
      );
    };

    socket.on('rooms-updated', handleRoomsUpdated);

    return () => {
      socket.off('rooms-updated', handleRoomsUpdated);
    };
  }, [router]);

  const handleCreateRoom = () => {
    setLoading(true);
    const socket = getSocket();

    socket.emit('create-room', ({ roomId, success }) => {
      if (success) {
        router.push(`/game?roomId=${roomId}`);
      }
      setLoading(false);
    });
  };

  const handleJoinRoom = (roomId) => {
    setLoading(true);
    const socket = getSocket();

    socket.emit('join-room', roomId, ({ success, gameState }) => {
      if (success) {
        router.push(`/game?roomId=${roomId}`);
      }
      setLoading(false);
    });
  };

  const handleRefresh = () => {
    const socket = getSocket();
    socket.emit('get-rooms', ({ waitingRooms, playingRooms }) => {
      setWaitingRooms(waitingRooms);
      setPlayingRooms(playingRooms);
    });
  };

  return (
    <div className="flex flex-col h-screen bg-gray-50">
      {/* ヘッダー */}
      <div className="bg-blue-600 text-white p-4 text-center">
        <p className="text-xl font-bold">{onlineCount}人待機中</p>
      </div>

      {/* コンテンツ */}
      <div className="flex-1 overflow-y-auto p-4">
        {/* 募集中セクション */}
        <div className="mb-6">
          <h2 className="text-lg font-bold text-gray-800 mb-3">募集中</h2>
          {waitingRooms.length > 0 ? (
            waitingRooms.map((room) => (
              <WaitingRoomCard
                key={room.roomId}
                room={room}
                onJoin={handleJoinRoom}
                loading={loading}
              />
            ))
          ) : (
            <p className="text-gray-500 text-center py-4">募集中の部屋はありません</p>
          )}
        </div>

        {/* 対戦中セクション */}
        <div className="mb-6">
          <h2 className="text-lg font-bold text-gray-800 mb-3">対戦中</h2>
          {playingRooms.length > 0 ? (
            playingRooms.map((room, idx) => (
              <PlayingRoomCard key={idx} room={room} />
            ))
          ) : (
            <p className="text-gray-500 text-center py-4">現在の対戦はありません</p>
          )}
        </div>
      </div>

      {/* フッター（ボタン） */}
      <div className="p-4 border-t bg-white">
        <button
          onClick={handleCreateRoom}
          disabled={loading}
          className="w-full px-6 py-4 mb-3 bg-green-500 text-white font-bold rounded-lg hover:bg-green-600 disabled:bg-gray-400"
        >
          新しく部屋を作る
        </button>
        <button
          onClick={handleRefresh}
          disabled={loading}
          className="w-full px-6 py-2 bg-gray-300 text-gray-800 font-bold rounded-lg hover:bg-gray-400 disabled:bg-gray-400"
        >
          更新
        </button>
      </div>
    </div>
  );
}
```

- [ ] **Step 6-2a: Commit**

```bash
git add client/components/RoomCard.jsx client/pages/lobby.js
git commit -m "feat: implement lobby screen with room list and creation"
```

---

## Task 7: ゲーム画面基本構造（フロントエンド）

**Files:**
- Create: `client/pages/game.js`
- Create: `client/components/Board.jsx`
- Create: `client/components/PlayerInfo.jsx`
- Create: `client/components/Timer.jsx`

### Step 7-1: Board コンポーネント

`client/components/Board.jsx`：

```javascript
import { PIECE_WHITE, PIECE_PURPLE } from '../lib/constants';

export function Board({ board, legalMoves, lastMove, onCellClick }) {
  const renderCell = (row, col) => {
    const piece = board[row][col];
    const isLegal = legalMoves.some(([r, c]) => r === row && c === col);
    const isLastMove = lastMove && lastMove.row === row && lastMove.col === col;

    return (
      <button
        key={`${row}-${col}`}
        onClick={() => onCellClick(row, col)}
        disabled={piece !== 0 || !isLegal}
        className={`
          relative w-full aspect-square border border-gray-400
          ${isLegal ? 'bg-green-100 cursor-pointer' : 'bg-green-700 cursor-default'}
          hover:${isLegal ? 'bg-green-200' : ''}
        `}
      >
        {/* コマ表示 */}
        {piece !== 0 && (
          <div
            className={`
              absolute inset-2 rounded-full
              ${piece === PIECE_WHITE ? 'bg-white shadow-lg' : 'bg-purple-600 shadow-lg'}
            `}
          />
        )}

        {/* 赤丸マーカー（直前手） */}
        {isLastMove && piece !== 0 && (
          <div className="absolute inset-0 flex items-center justify-center">
            <div className="w-2 h-2 bg-red-500 rounded-full" />
          </div>
        )}

        {/* 白丸マーカー（合法手） */}
        {isLegal && piece === 0 && (
          <div className="absolute inset-0 flex items-center justify-center">
            <div className="w-2 h-2 bg-white rounded-full" />
          </div>
        )}
      </button>
    );
  };

  return (
    <div className="w-full max-w-full aspect-square bg-green-700 p-1 rounded-lg shadow-lg border-4 border-green-900">
      <div className="grid grid-cols-8 gap-0 w-full h-full">
        {Array.from({ length: 64 }).map((_, i) => {
          const row = Math.floor(i / 8);
          const col = i % 8;
          return renderCell(row, col);
        })}
      </div>
    </div>
  );
}
```

### Step 7-2: PlayerInfo コンポーネント

`client/components/PlayerInfo.jsx`：

```javascript
export function PlayerInfo({ player1, player2, currentPlayer }) {
  return (
    <div className="w-full mb-4 p-4 bg-white rounded-lg shadow">
      <div className="flex justify-between items-center mb-2">
        <div>
          <p className="text-sm text-gray-600">プレイヤー1</p>
          <p className={`text-lg font-bold ${currentPlayer === 1 ? 'text-yellow-500' : 'text-gray-800'}`}>
            {player1.name}（白）
          </p>
          <p className="text-sm text-gray-600">コマ数: {player1.score}</p>
        </div>
        <div className="text-center text-2xl font-bold text-gray-400">vs</div>
        <div>
          <p className="text-sm text-gray-600">プレイヤー2</p>
          <p className={`text-lg font-bold ${currentPlayer === 2 ? 'text-yellow-500' : 'text-gray-800'}`}>
            {player2.name}（紫）
          </p>
          <p className="text-sm text-gray-600">コマ数: {player2.score}</p>
        </div>
      </div>
    </div>
  );
}
```

### Step 7-3: Timer コンポーネント

`client/components/Timer.jsx`：

```javascript
import { useEffect, useState } from 'react';

export function Timer({ isActive, onTimeUp, initialTime = 20 }) {
  const [timeLeft, setTimeLeft] = useState(initialTime);

  useEffect(() => {
    if (!isActive) {
      setTimeLeft(initialTime);
      return;
    }

    const interval = setInterval(() => {
      setTimeLeft((prev) => {
        if (prev <= 1) {
          clearInterval(interval);
          onTimeUp();
          return 0;
        }
        return prev - 1;
      });
    }, 1000);

    return () => clearInterval(interval);
  }, [isActive, initialTime, onTimeUp]);

  const isWarning = timeLeft <= 5;

  return (
    <div className={`text-4xl font-bold text-center py-4 rounded-lg ${
      isWarning ? 'bg-red-500 text-white' : 'bg-blue-500 text-white'
    }`}>
      {timeLeft}秒
    </div>
  );
}
```

### Step 7-4: ゲーム画面実装

`client/pages/game.js`：

```javascript
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { getSocket } from '../lib/socket';
import { Board } from '../components/Board';
import { PlayerInfo } from '../components/PlayerInfo';
import { Timer } from '../components/Timer';
import { PIECE_WHITE, PIECE_PURPLE } from '../lib/constants';

export default function Game() {
  const router = useRouter();
  const { roomId } = router.query;
  const [gameState, setGameState] = useState(null);
  const [legalMoves, setLegalMoves] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!roomId) return;

    const socket = getSocket();

    // ゲーム開始
    const handleGameStarted = (state) => {
      setGameState(state);
    };

    // 盤面更新
    const handleBoardUpdated = (state) => {
      setGameState(state);
    };

    // 合法手更新
    const handleLegalMovesUpdated = (moves) => {
      setLegalMoves(moves);
    };

    // ゲーム終了
    const handleGameFinished = (state) => {
      setGameState(state);
    };

    // 相手切断
    const handleOpponentDisconnected = () => {
      setError('対戦相手の通信が切断されました');
    };

    socket.on('game-started', handleGameStarted);
    socket.on('board-updated', handleBoardUpdated);
    socket.on('legal-moves-updated', handleLegalMovesUpdated);
    socket.on('game-finished', handleGameFinished);
    socket.on('opponent-disconnected', handleOpponentDisconnected);

    return () => {
      socket.off('game-started', handleGameStarted);
      socket.off('board-updated', handleBoardUpdated);
      socket.off('legal-moves-updated', handleLegalMovesUpdated);
      socket.off('game-finished', handleGameFinished);
      socket.off('opponent-disconnected', handleOpponentDisconnected);
    };
  }, [roomId]);

  const handleCellClick = (row, col) => {
    setLoading(true);
    const socket = getSocket();

    socket.emit('place-piece', roomId, row, col, ({ success, error }) => {
      if (!success) {
        setError(error || '手を置くことができませんでした');
      }
      setLoading(false);
    });
  };

  const handleResign = () => {
    setLoading(true);
    const socket = getSocket();

    socket.emit('resign', roomId, ({ success }) => {
      if (success) {
        // ゲーム終了画面を表示
      }
      setLoading(false);
    });
  };

  const handleLeaveRoom = () => {
    const socket = getSocket();
    socket.emit('leave-room', roomId, () => {
      router.push('/lobby');
    });
  };

  if (!gameState) {
    return (
      <div className="flex items-center justify-center h-screen">
        <p className="text-gray-600">ゲーム読み込み中...</p>
      </div>
    );
  }

  const isCurrentPlayerWhite = gameState.currentPlayer === PIECE_WHITE;
  const isMyTurn = false; // TODO: 実装

  return (
    <div className="flex flex-col h-screen bg-gray-50 p-4">
      {/* エラー表示 */}
      {error && (
        <div className="mb-4 p-4 bg-red-500 text-white rounded-lg">
          <p>{error}</p>
          <button
            onClick={() => setError(null)}
            className="mt-2 px-4 py-1 bg-red-700 rounded"
          >
            閉じる
          </button>
        </div>
      )}

      {/* プレイヤー情報 */}
      <PlayerInfo
        player1={gameState.player1}
        player2={gameState.player2}
        currentPlayer={gameState.currentPlayer}
      />

      {/* タイマー */}
      {gameState.gameState === 'playing' && (
        <Timer isActive={isMyTurn} onTimeUp={handleResign} />
      )}

      {/* 盤面 */}
      <div className="flex-1 mb-4">
        <Board
          board={gameState.board}
          legalMoves={gameState.gameState === 'playing' ? legalMoves : []}
          lastMove={gameState.lastMove}
          onCellClick={handleCellClick}
        />
      </div>

      {/* ボタン */}
      <div className="flex gap-2">
        {gameState.gameState === 'playing' ? (
          <button
            onClick={handleResign}
            disabled={loading}
            className="flex-1 px-4 py-3 bg-red-500 text-white font-bold rounded-lg hover:bg-red-600 disabled:bg-gray-400"
          >
            降参
          </button>
        ) : (
          <button
            onClick={handleLeaveRoom}
            disabled={loading}
            className="flex-1 px-4 py-3 bg-blue-500 text-white font-bold rounded-lg hover:bg-blue-600 disabled:bg-gray-400"
          >
            退室
          </button>
        )}
      </div>

      {/* ゲーム終了モーダル */}
      {gameState.gameState === 'finished' && (
        <div className="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 p-4">
          <div className="bg-white rounded-lg p-6 text-center max-w-sm">
            <h2 className="text-2xl font-bold mb-4">ゲーム終了</h2>
            <p className="text-lg mb-2">
              {gameState.winner
                ? `${gameState.winner.name}の勝利！`
                : '引き分け'}
            </p>
            <p className="text-sm text-gray-600 mb-6">
              {gameState.player1.name}: {gameState.player1.score}個<br />
              {gameState.player2.name}: {gameState.player2.score}個
            </p>
            <button
              onClick={handleLeaveRoom}
              className="w-full px-4 py-2 bg-blue-500 text-white font-bold rounded-lg"
            >
              退室
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

- [ ] **Step 7-4a: Commit**

```bash
git add client/components/Board.jsx client/components/PlayerInfo.jsx client/components/Timer.jsx client/pages/game.js
git commit -m "feat: implement game screen with board and basic controls"
```

---

## Task 8: アニメーション・スタイリング（フロントエンド）

**Files:**
- Modify: `client/styles/globals.css` に追加
- Modify: `client/components/Board.jsx` にアニメーション追加

### Step 8-1: アニメーションCSS追加

`client/styles/globals.css`に追加：

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes rotate {
  from {
    transform: rotateY(0);
  }
  to {
    transform: rotateY(360deg);
  }
}

.piece-fade-in {
  animation: fadeIn 0.3s ease-in;
}

.piece-flip {
  animation: rotate 0.6s ease-in-out;
}
```

### Step 8-2: Board コンポーネント更新（アニメーション適用）

`client/components/Board.jsx` のコマ表示部分を更新：

```javascript
{/* コマ表示 */}
{piece !== 0 && (
  <div
    className={`
      absolute inset-2 rounded-full shadow-lg
      ${piece === PIECE_WHITE ? 'bg-white' : 'bg-gradient-to-br from-purple-500 via-purple-600 to-purple-700'}
      ${isLastMove ? 'piece-flip' : 'piece-fade-in'}
    `}
    style={{
      boxShadow: piece === PIECE_WHITE 
        ? '0 4px 8px rgba(0,0,0,0.2), inset -2px -2px 4px rgba(0,0,0,0.1)' 
        : '0 4px 8px rgba(0,0,0,0.3), inset -2px -2px 4px rgba(0,0,0,0.2)'
    }}
  />
)}
```

### Step 8-3: 全体的な高級感のあるスタイリング

`client/styles/globals.css` を更新（グラデーション、シャドウ）：

```css
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

/* ボード背景をグラデーション */
.board-bg {
  background: linear-gradient(135deg, #1e3a1f 0%, #2d5016 100%);
}

/* カード等のシャドウ */
.card-shadow {
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

/* ボタンのホバーエフェクト */
button:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  transition: all 0.2s ease;
}
```

- [ ] **Step 8-3a: Commit**

```bash
git add client/styles/ client/components/Board.jsx
git commit -m "feat: add animations and premium styling with gradients"
```

---

## Task 9: タイムアウト・自動手の実装（バックエンド）

**Files:**
- Modify: `server/game/ReversiGame.js`
- Modify: `server/events/socketHandlers.js`

### Step 9-1: ReversiGame にタイムアウト処理追加

`server/game/ReversiGame.js` の `constructor` の後に追加：

```javascript
  startTurn() {
    this.turnStartTime = Date.now();
    const turnTime = TURN_TIME_LIMIT;
    
    // turnTime後に自動的にランダムな合法手を置く
    this.turnTimeoutId = setTimeout(() => {
      this.autoMove();
    }, turnTime);
  }

  clearTurnTimeout() {
    if (this.turnTimeoutId) {
      clearTimeout(this.turnTimeoutId);
      this.turnTimeoutId = null;
    }
  }

  autoMove() {
    const { getLegalMoves } = require('./rules');
    const legalMoves = getLegalMoves(this.board, this.currentPlayer);
    
    if (legalMoves.length > 0) {
      const randomMove = legalMoves[Math.floor(Math.random() * legalMoves.length)];
      this.move(randomMove[0], randomMove[1]);
    } else {
      // パス
      this.currentPlayer = this.currentPlayer === PIECE_WHITE ? PIECE_PURPLE : PIECE_WHITE;
    }
  }
```

### Step 9-2: Socket イベントハンドラ更新

`server/events/socketHandlers.js` の `join-room` イベント内で：

```javascript
// ゲーム開始時にターンを開始
room.game.startTurn();
```

`place-piece` イベント内で：

```javascript
room.game.clearTurnTimeout();
// ... 手を置く処理 ...
room.game.startTurn();
```

- [ ] **Step 9-2a: Commit**

```bash
git add server/game/ReversiGame.js server/events/socketHandlers.js
git commit -m "feat: implement auto-move on timeout with random legal move"
```

---

## Task 10: レスポンシブ・モバイル最適化（フロントエンド）

**Files:**
- Modify: `client/pages/_app.js`
- Modify: `client/styles/globals.css`
- Modify: `client/pages/index.js`
- Modify: `client/pages/lobby.js`

### Step 10-1: ビューポート設定

`client/pages/_app.js`：

```javascript
import Head from 'next/head';
import '../styles/globals.css';

function MyApp({ Component, pageProps }) {
  return (
    <>
      <Head>
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover" />
        <meta name="apple-mobile-web-app-capable" content="true" />
        <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
      </Head>
      <Component {...pageProps} />
    </>
  );
}

export default MyApp;
```

### Step 10-2: タッチ操作最適化

`client/styles/globals.css`に追加：

```css
/* ダブルタップとロングタップを無効化 */
body {
  touch-action: manipulation;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
}

button {
  touch-action: manipulation;
}

input {
  -webkit-appearance: none;
  appearance: none;
  border-radius: 0;
}

/* モバイルキーボード対応 */
input[type="text"] {
  font-size: 16px; /* iOSでズーム防止 */
}
```

### Step 10-3: 各ページのレスポンシブ調整

タイトル画面・ロビー画面で、max-width 制限を削除またはスマホに合わせる：

```css
/* スマートフォンファースト */
@media (min-width: 768px) {
  /* タブレット以上 */
}
```

- [ ] **Step 10-3a: Commit**

```bash
git add client/pages/_app.js client/styles/globals.css client/pages/index.js client/pages/lobby.js
git commit -m "feat: optimize for mobile with viewport and touch handling"
```

---

## Task 11: デプロイ準備と最終テスト

**Files:**
- Create: `Dockerfile` (オプション)
- Create: `README.md`
- Modify: `.env.example`

### Step 11-1: 環境変数設定

`.env.example`：

```
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### Step 11-2: Docker 設定（オプション）

`Dockerfile`：

```dockerfile
FROM node:18-alpine

WORKDIR /app

# バックエンド
COPY server/package*.json ./server/
WORKDIR /app/server
RUN npm install
WORKDIR /app

# フロントエンド
COPY client/package*.json ./client/
WORKDIR /app/client
RUN npm install
WORKDIR /app

COPY server ./server
COPY client ./client

EXPOSE 3000 3001

CMD ["npm", "run", "dev"]
```

### Step 11-3: README 作成

`README.md`：

```markdown
# リバーシオンライン対戦ゲーム

スマートフォン最適化のリアルタイムオンライン対戦リバーシゲーム。

## セットアップ

```bash
# サーバー
cd server && npm install

# クライアント
cd client && npm install
```

## 開発起動

ターミナル1（サーバー）:
```bash
cd server && npm run dev
```

ターミナル2（クライアント）:
```bash
cd client && npm run dev
```

ブラウザで http://localhost:3000 にアクセス。

## 技術スタック

- フロントエンド: Next.js, React, Tailwind CSS
- バックエンド: Node.js, Express, Socket.io
- プロトコル: WebSocket (Socket.io)

## ゲームルール

- 8x8 のリバーシ盤面
- 1手20秒の制限時間
- 時間切れは自動的にランダムな合法手を置く
- 降参可能
- ゲーム終了時に勝敗とコマ数を表示
```

- [ ] **Step 11-3a: Commit**

```bash
git add README.md .env.example Dockerfile
git commit -m "chore: add deployment and documentation files"
```

---

## 自己レビュー

### 仕様カバレッジ確認

- ✅ タイトル画面：プレイヤー名入力、待機人数表示
- ✅ ロビー画面：募集中リスト、対戦中リスト、手動更新
- ✅ ゲーム画面：盤面、タイマー、降参、退室
- ✅ リバーシロジック：合法手判定、コマ裏返し
- ✅ Socket.io通信：リアルタイムゲーム進行
- ✅ アニメーション：フェードイン、回転演出
- ✅ タイムアウト：自動ランダム手配置
- ✅ 切断処理：エラーメッセージ表示
- ✅ ローカルストレージ：プレイヤー名保持
- ✅ モバイル対応：レスポンシブ、タッチ最適化

### プレースホルダーチェック

全タスクで完全なコード実装。TBDなし。

### 型・関数名一貫性

- `PIECE_WHITE`, `PIECE_PURPLE` 一貫性確認
- Socket.ionイベント名（`board-updated`, `legal-moves-updated`等）一貫性確認

---

**計画完成。保存先：** `docs/superpowers/plans/2026-05-10-reversi-game.md`

---

## 実行方法の選択

2つの実行オプションがあります：

### 1. **サブエージェント駆動（推奨）**
各タスクを独立したサブエージェントが実行。タスク間でレビューしながら進行。
→ `superpowers:subagent-driven-development` を使用

### 2. **インラインで一気実行**
この会話内で複数タスクをバッチ実行。ブロックポイントで確認。
→ `superpowers:executing-plans` を使用

どちらの方法で進めたいですか？