# Claude Code に最初に送るメッセージ（コピペ用）

下のグレー枠の中身を、Claude Code 起動直後にそのままコピペして送ってください。  
（ファイルパスの部分だけ、実際のフォルダ名に合わせて書き換えてください）

---

## 📋 コピペ用テンプレート（短め版）

```
はじめまして、Hayatoです。
Cowork上のClaudeから引き継ぎでこちらに移ってきました。

まず、このフォルダにある「Claude_Code_引き継ぎメモ.md」を読んでください。
（場所：プロジェクトの1つ上の階層、または C:\...\Claude（Purple Park）\ にあります）

読み終わったら、現在の git の状態（git status と git diff）を確認して、
変更内容を要約して教えてください。
そのあと、commit & push の作業に進めるか相談したいです。

私はWeb制作勉強中で、コマンドの意味は丁寧に説明してもらえると助かります。
```

---

## 📋 コピペ用テンプレート（長め版・引き継ぎメモを開けない場合）

引き継ぎメモのファイルを Claude Code がうまく開けないときは、こちらをそのまま送ってください。

```
はじめまして、Hayatoです。Cowork上のClaudeから引き継ぎで来ました。
状況を簡単に共有します：

【プロジェクト】
- 自作ゲームをまとめた個人サイト「Purple Park」
- リポジトリ名：purple-games（GitHub: PurpleGames-Creator/purple-games）
- 公開URL：https://purple-park.vercel.app/
- 技術：Next.js 16 + TypeScript + Tailwind v4 + framer-motion + Supabase + Vercel

【直前にやった変更（未push）】
3ファイルが変更済み、まだGitHubに送ってません：
1. README.md → Next.js初期テンプレからPurple Park専用に書き換え
2. app/components/game-today-ranking.tsx → スコアにカウンターアップ追加
3. app/page.tsx → ヒーロータイトルにスプリットテキスト＋ブラー演出追加

加えてSupabase側でmigration適用済み（DBの話なのでファイルdiffには出ません）：
- 3つのスコアテーブル（scores/biker_scores/diver_scores）の
  INSERTポリシーをWITH CHECK付き（score 0〜100万、nickname 1〜30文字）に強化

【最初にやってほしいこと】
1. git status と git diff で変更を確認
2. 変更内容を私に説明
3. OKならcommit & push（Vercelが自動デプロイしてくれます）

【私について】
営業職、Web制作勉強中。コマンドの意味は丁寧に説明してもらえると助かります。
カジュアルな日本語でOK、結論を先に言うスタイルが好きです。
```

---

## 補足：引き継ぎメモの場所

完全な引き継ぎメモは下記にあります（こちらの方が情報量豊富）：

```
C:\Users\（あなたのユーザー名）\（mountパス）\Claude（Purple Park）\Claude_Code_引き継ぎメモ.md
```

または Cowork の「選択中のフォルダ」から `Claude（Purple Park）` を開けば、その中に入ってます。

Claude Code を起動したターミナルで、`cat "../Claude_Code_引き継ぎメモ.md"` などで読めるはずです（プロジェクトの1つ上にある場合）。
