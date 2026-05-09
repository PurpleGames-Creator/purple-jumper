# Claude Code 引き継ぎメモ（Purple Park）

作成日：2026-04-23  
前任：Cowork上のClaude（Opus 4.7）  
後任：ローカル環境のClaude Code

---

## 🎯 まず読んでほしい1段落

このリポジトリは Hayato さん（営業職、Web制作勉強中）の個人趣味サイト「Purple Park」（自作ゲームのまとめサイト）です。直前まで Cowork上のClaudeと一緒に「Supabase RLS引き締め・README刷新・アニメーション追加」の3つの作業を完了し、**ローカルには変更が反映されているがGitHubにはまだpushされていない状態** です。最初のミッションは、変更内容を確認して、commit & push して本番反映することです。

---

## 1. Hayatoさんについて

- **背景**：1985年6月生まれの営業職。AI/IT知識は学習中。
- **学習中スキル**：Web制作（Next.js / TypeScript）、ゲーム開発（JavaScript / Canvas）
- **使用ツール**：Claude / GitHub / Supabase / Vercel / VS Code
- **好む説明スタイル**：
  - 結論を先に、理由を後に
  - カジュアル・フレンドリーな日本語
  - 「この画面のここをクリック」レベルの具体的な操作手順
  - 難しい比喩は避ける
  - 絵文字は控えめに（ユーザーが使ったときだけ呼応）
- **アニメーションの好み**：大企業サイト風の15種（フェードイン/パララックス/スプリットテキスト/カウンターアップ/ブラー→シャープ etc.）を好む。使い過ぎないバランスも重視。

---

## 2. このプロジェクト（Purple Park）の概要

- **正体**：自作ゲーム3本（Purple Jumper / Purple Biker / Purple Diver）+ LINEスタンプ等をまとめたシングルページサイト
- **公開URL**：https://purple-park.vercel.app/
- **GitHubリポジトリ**：https://github.com/PurpleGames-Creator/purple-games （**private**）
- **ローカルパス**（推定）：`C:\purple-games\` あたり（要確認）
- **技術スタック**：
  - Next.js 16（App Router）
  - TypeScript
  - Tailwind CSS v4
  - framer-motion
  - Supabase（PostgreSQL）
  - Vercel（自動デプロイ：master push → 本番反映）

### 5つのタブ構成（小ネタ：内部IDと表示名が一致してない）

| 表示名 | 内部ID | 中身 |
|---|---|---|
| ゲーム | `games` | 自作ゲーム3本 + 今日のランキング |
| SNS | `topics` | SNSアカウント紹介 |
| LINEスタンプ | `hardware` | LINEスタンプ販売リンク |
| 会社概要 | `characters` | ヒーロー画像 + アコーディオン |
| サポート | `support` | サポート窓口 |

---

## 3. 🚨 直前のCoworkセッションでやった変更（未push）

3ファイル変更されています。**まずこれらをcommit & pushするのが第一ミッション** です。

### 変更ファイル一覧

| ファイル | 変更内容 |
|---|---|
| `README.md` | Next.js初期テンプレ → Purple Park専用に全面書き換え |
| `app/components/game-today-ranking.tsx` | スコア表示を `CountUpScore` コンポーネント化（カウンターアップ） |
| `app/page.tsx` | `HeroGameTitles` をスプリットテキスト＋ブラー→シャープ演出に改良 |

### 推奨commitメッセージ

```
RLS引き締め＋README刷新＋カウンターアップ＆スプリットテキスト追加

- Supabase RLS: scores/biker_scores/diver_scoresにWITH CHECK追加
- README.md: Next.jsテンプレ→Purple Park専用ドキュメント
- game-today-ranking.tsx: CountUpScoreコンポーネント追加
- page.tsx: HeroGameTitlesを文字単位アニメ＋ブラーに改良
```

※ Supabase RLS変更は **DB側のmigrationなのでコード変更には現れません**（ファイルにdiffなし）。コミットメッセージにメモとして残すだけでOK。

---

## 4. Supabase まわり（DB側で完了済み）

### プロジェクト情報

- **プロジェクト名**：Purple Games
- **プロジェクトID**：`hefayilffszrczxhnpii`
- **リージョン**：ap-northeast-1（東京）
- **Postgres**：17

### 適用済みmigration（前任Coworkセッションで実行済み）

`tighten_insert_policies_with_check` という名前で適用。3テーブル（scores / biker_scores / diver_scores）の「Allow public insert」を「Allow safe insert」に置き換え、以下の WITH CHECK を追加：

```sql
WITH CHECK (
  score >= 0 AND score <= 1000000
  AND char_length(trim(nickname)) BETWEEN 1 AND 30
)
```

→ Advisor警告は0件まで減少済み。今後 RLS をさらに強化したい場合の方向性は「Edge Function 経由でレート制限」など。

### ゲームとテーブルの対応

| ゲーム | テーブル | スコア単位 |
|---|---|---|
| Purple Jumper | `scores` | 段 |
| Purple Biker | `biker_scores` | m |
| Purple Diver | `diver_scores` | m |

⚠️ Jumperのテーブル名だけ `scores`（プレフィックスなし）。歴史的経緯。直すと既存3万件のデータが影響を受けるので触らない方針。

---

## 5. Vercel まわり

- **プロジェクト名（Vercel上）**：`purple-games`（GitHubリポジトリ名と同じ）
- **ドメイン**：`purple-park.vercel.app`
- **GitHub連携**：master ブランチ push で自動デプロイ
- **直近20件デプロイすべて READY**（健全）
- ちなみに：ゲーム3本（jumper/biker/diver）もVercelに登録済みだが、本番公開はGitHub Pages（github.io）側を使ってる模様

---

## 6. リポジトリ構成（重要ファイル）

```
purple-games/
├── app/
│   ├── page.tsx                  ← トップページ本体（983行・全タブの中身）
│   ├── layout.tsx                ← フォント設定
│   ├── globals.css
│   ├── components/
│   │   ├── game-character-replay.tsx
│   │   ├── game-today-ranking.tsx     ← 今回CountUpScore追加
│   │   └── top-info-accordion.tsx
│   └── api/rankings/[game]/route.ts   ← Supabaseクエリ用API
├── lib/
│   ├── ranking.ts                ← JST日時計算・順位ロジック
│   └── supabase/server.ts        ← Supabase接続（サーバー側）
├── public/                       ← 画像
├── docs/SUPABASE_RANKING.md      ← Supabase設定手順
├── .env.local                    ← 環境変数（GitHub除外）
└── package.json
```

---

## 7. 環境変数

`.env.local` に必要：

```
SUPABASE_URL=https://hefayilffszrczxhnpii.supabase.co
SUPABASE_ANON_KEY=（Supabaseダッシュボードから取得した anon key）
```

Vercel側にも同じ環境変数が登録済み。`NEXT_PUBLIC_*` プレフィックスはあえて使わない（anon keyをブラウザに載せない方針）。

---

## 8. 今後の候補タスク（Hayatoさんが「次これ」と言ったら）

### 🟡 中優先度

- **C. Supabase anon key の整理**：ゲーム3本のリポジトリ（purple-jumper/biker/diver）に `supabase-config.js` でキー直書き＆コミット済み。anon keyなので致命的ではないが整理したい。
- **E. page.tsx の分割リファクタ**：983行を5タブ別ファイルに切り出し。新機能を足しやすくなる。

### 🟢 楽しみ系

- **F. 新機能**：4本目のゲーム追加 / 「お知らせ」タブ追加 / 問い合わせフォーム追加 など
- **アニメーション追加候補**：未使用の Lottie / パララックス / スクロールトリガー など

---

## 9. ハマりやすい注意点

### 命名の3層構造

| 層 | 名前 |
|---|---|
| ローカルフォルダ名（旧） | `purple-games` |
| GitHubリポジトリ名 | `purple-games` |
| Vercelプロジェクト名 | `purple-games` |
| 表示名・ドメイン | **`purple-park`** |

→ コードや会話では「Purple Park」と呼ぶが、リポジトリ操作では `purple-games` を使う。混乱しないように。

### タブの内部IDと表示名のズレ

- `characters` = 会社概要タブ（キャラクター画面ではない）
- `topics` = SNSタブ（トピックではない）
- `hardware` = LINEスタンプタブ（ハードウェアではない）

→ 過去の名残。改修時に直す候補だが、急ぎではない。

### Jumperテーブル名

- 他は `biker_scores` `diver_scores` と統一されてるが、Jumper だけ `scores`。
- `jumper_scores` への改名は **既存3万件のデータ影響あり** なので非推奨。

---

## 10. 関連リンク

- 本サイト：https://purple-park.vercel.app/
- GitHubリポジトリ：https://github.com/PurpleGames-Creator/purple-games
- Purple Jumper：https://purplegames-creator.github.io/purple-jumper/
- Purple Biker：https://purplegames-creator.github.io/purple-biker/
- Purple Diver：https://purplegames-creator.github.io/purple-diver/
- Supabaseダッシュボード：https://supabase.com/dashboard/project/hefayilffszrczxhnpii
- Vercelダッシュボード：https://vercel.com/gaki042046-2138s-projects/purple-games

---

## 11. 最初にやってほしいこと（チェックリスト）

- [ ] このメモを読んで状況把握
- [ ] `git status` で変更ファイルを確認（README.md / page.tsx / game-today-ranking.tsx の3つあるはず）
- [ ] `git diff` で変更内容をざっと確認
- [ ] Hayatoさんに「pushしていいですか？」と確認
- [ ] OKなら commit & push
- [ ] Vercelの自動デプロイが走るので、1〜2分後に https://purple-park.vercel.app/ を開いて反映確認
- [ ] 「次に何をしますか？」と聞く（候補は上記セクション8）

---

おつかれさまです、Claude Code！Hayatoさんは丁寧で優しい方なので、わからないことは遠慮なく聞いてあげてください。
