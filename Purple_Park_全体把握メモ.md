# Purple Park（まとめサイト）全体把握メモ

作成日：2026-04-22  
解析対象：`Claude（Purple Park）/purple park/`（GitHub: PurpleGames-Creator/purple-games）

---

## まずひと言で

**シングルページ（1ページ完結）型の自作ゲームポートフォリオサイト**です。  
5つのタブを切り替えて中身が変わる構成で、framer-motionでアニメーションをふんだんに使ってます。すごくよく作り込まれていて、初学者の作品とは思えない品質です。

---

## 1. 技術スタック（使っている材料）

| 役割 | 何を使っているか | ひとことで |
|---|---|---|
| フレームワーク | Next.js 16（App Router） | Reactの上にある人気フレームワーク |
| 言語 | TypeScript | JavaScriptに型を足したやつ |
| デザイン | Tailwind CSS v4 | クラス名でスタイルを書ける |
| アニメーション | framer-motion | フェード/スライドが気持ちいいやつ |
| アイコン | lucide-react | キレイなアイコン集 |
| データベース | Supabase | ランキング保存に使用 |
| ホスティング | Vercel | purple-park.vercel.app で公開 |
| フォント | M PLUS Rounded 1c / Cormorant Garamond / Noto Serif JP | 3種類使い分け |

---

## 2. 5つのタブの構成

ヘッダーのナビにある5つのタブを切り替えて、コンテンツが入れ替わる構成です。

| 表示名 | 内部ID | 中身 |
|---|---|---|
| ゲーム | `games` | 自作ゲーム3本（カード式・キャラアニメ・今日のランキング付き） |
| SNS | `topics` | SNSアカウント2つ（イッショウ／ハナエ） |
| LINEスタンプ | `hardware` | LINEスタンプ2種類（ハナエ／パプ太郎） |
| 会社概要 | `characters` | ヒーロー画像 + アコーディオン情報 |
| サポート | `support` | 「おれのサポートか？」のメッセージ |

⚠️ **小ネタ：内部IDが表示名と一致してない**ので、コードを読むときに混乱します（例：`characters` が「会社概要タブ」、`topics` が「SNSタブ」、`hardware` が「LINEスタンプタブ」）。今後の改修時、いつかリネームしたほうが分かりやすくなります（急ぎではない）。

---

## 3. ファイル構成（重要なファイルだけ）

```
purple park/
├── app/
│   ├── page.tsx              ← トップページ本体（983行・全タブの中身が入ってる）
│   ├── layout.tsx            ← フォント設定とHTMLの外側
│   ├── globals.css           ← 全体に効くCSS
│   ├── components/
│   │   ├── game-character-replay.tsx   ← ゲームキャラのアニメ（再生ボタン付き）
│   │   ├── game-today-ranking.tsx      ← 今日のランキング表示
│   │   └── top-info-accordion.tsx      ← 会社概要タブのアコーディオン
│   └── api/rankings/[game]/route.ts    ← ランキング取得API
├── lib/
│   ├── ranking.ts            ← JST日時計算・順位計算ロジック
│   └── supabase/server.ts    ← Supabase接続（サーバー側）
├── public/                   ← 画像（hero.png, jumper.png, hanae.jpg など）
├── docs/SUPABASE_RANKING.md  ← Supabase設定の手順
├── .env.local                ← ローカル用の環境変数（GitHubには上げない設定）
└── package.json              ← 依存ライブラリ一覧
```

**ポイント：** ロジック（lib/）とUI（app/）が分かれていて、コンポーネントも役割ごとに分割されています。Purple Diverの構成より一段階キレイです。

---

## 4. ランキング機能の仕組み

ゲームタブ各カードに「今日のランキング」が表示される仕組みは、3層構造になってます。

```
ブラウザ（GameTodayRanking コンポーネント）
   ↓ /api/rankings/jumper を叩く
Vercel上のNext.jsサーバー（route.ts）
   ↓ Supabaseに「今日のJSTスコア」をクエリ
Supabase（テーブル: scores / biker_scores / diver_scores）
```

### ゲームとテーブルの対応

| ゲーム | URL内のID | Supabaseテーブル | スコア単位 |
|---|---|---|---|
| Purple Jumper | `jumper` | `scores` | 段 |
| Purple Biker | `biker` | `biker_scores` | m |
| Purple Diver | `diver` | `diver_scores` | m |

⚠️ **発見：Purple Jumper のテーブル名だけ `scores`（プレフィックスなし）になってます。** 統一するなら `jumper_scores` のほうが綺麗ですが、過去のデータが入っている可能性があるので今は触らないほうが安全です。

### 環境変数

`.env.local`（ローカル）に以下が入っている前提：

```
SUPABASE_URL=...
SUPABASE_ANON_KEY=...
```

Vercelにも同じ環境変数を登録しておく必要があります（次のステップで確認します）。

---

## 5. 使われているアニメーション一覧

Hayatoさんのプリファレンスに書いてある15種のうち、Purple Parkでは以下が使われています：

| プリファレンスNo. | アニメーション | 使用箇所 |
|---|---|---|
| 1 | フェードイン | タブ切替時、画像の登場 |
| 7 | ホバーエフェクト | START ボタン（hover で明るく） |
| 9 | スライドイン | キャラクター（左/右/上から）、画像の登場 |
| 11 | リビールアニメーション | ヒーロータイトル「誰よりも 高く、深く、遠くへ」の遅延出現 |
| 14 | 微細アニメーション | ボタンタップ時の `scale: 0.97` |

**未使用：** パララックス、スムーススクロール、スプリットテキスト、マスク、カウンターアップ、ブラー→シャープ、スクロールスナップ、Lottie

→ 今後、改善案として **カウンターアップ（ランキングのスコア表示）** や **スプリットテキスト（タイトルの一文字ずつ出現）** を足すと、より「大企業サイトっぽい」印象を出せそうです。

---

## 6. 良い点・気になる点

### 👍 良い点

1. **コンポーネント分割がキレイ**：ロジックとUIが分離されている
2. **JST対応がしっかりしている**：日本時間の「今日」を正確に判定する関数あり
3. **同名ユーザー1人1スコア**処理：ランキングが荒れない
4. **anon key をサーバー側のみで使う設計**：環境変数の置き場所まで気を使っている
5. **ドキュメント（docs/SUPABASE_RANKING.md）あり**：将来の自分が助かる

### 🤔 気になる点（致命的じゃないけど、いつか手を入れたい）

1. **page.tsx が 983行と巨大**：5つのタブ全部が1ファイルにある。タブごとにコンポーネント分割すると保守が楽
2. **タブの内部IDと表示名が一致しない**：`characters` = ホーム、`topics` = SNS など
3. **Purple Jumper のテーブル名だけ `scores`**：統一したいが、過去データに影響するため要検討
4. **README.md がNext.jsの初期テンプレートのまま**：Purple Park 専用の説明に書き換えると親切
5. **画像の最適化**：`<img>` タグを使っているが、Next.jsの `<Image>` コンポーネントを使うともっと速くなる

---

## 7. 次にやるとしたら（提案）

優先度順：

1. **Vercel・Supabaseの接続状況確認**（次のタスク #4）
2. **Supabase RLS（書き込み防止）の設定確認**（セキュリティ）
3. **新機能 or 改善**：たとえばカウンターアップでスコアを派手に表示、トップタブにLottieでアイコン演出、などの「大企業サイト風」アップグレード
4. **page.tsx の分割リファクタ**：タブごとに別ファイルに切り出して保守性向上

どれから手をつけたいか、教えてください！

---

## 8. 公開URL（おさらい）

- 本サイト：https://purple-park.vercel.app/
- Purple Jumper：https://purplegames-creator.github.io/purple-jumper/
- Purple Biker：https://purplegames-creator.github.io/purple-biker/
- Purple Diver：https://purplegames-creator.github.io/purple-diver/
- LINEスタンプ（ハナエ）：https://line.me/S/sticker/33298055
- LINEスタンプ（パプ太郎）：https://line.me/S/sticker/33349503
