<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# 瞑想アプリ
気分・時間・ガイド量を選ぶと、AIが瞑想ガイドを作り、音声で読み上げるアプリ。個人開発のポートフォリオ。

## 技術スタック
- Next.js（App Router, src/, TypeScript, Tailwind, npm）
- Supabase（Auth, Postgres）。Next.jsからは @supabase/ssr を使う
- Vercel にデプロイ
- LLM と Text to Speech：未定（決まったらここに書く）

## 仕様の正本
- docs/db-schema.md：DB定義
- docs/screens.md：画面一覧
- 仕様と食い違う実装をする前に、必ず確認すること
- 仕様が足りなければ勝手に決めず、質問すること

## ルール
- 全テーブルに RLS を有効にする。user_id = auth.uid() の行だけ読み書きできるポリシーなしで、テーブルを作らない
- 入力値と AI の出力は Zod で検査してから保存する。DB の制約は使わない
- .env* は読まない、コミットしない。キーをクライアント側のコードに出さない
- 変更は小さく。1回の依頼で触るのは1つの機能まで

## コマンド
- 開発：npm run dev
- 検査：npm run lint
- 型：npx tsc --noEmit
- ビルド：npm run build
