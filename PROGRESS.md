# PROGRESS.md - EC-CUBE 開発進捗

> **ルール**: 完了=`[x] (YYYY-MM-DD)`。削除しない。新規タスクは追記してから着手。

---

## MS0: 初期設定

- [x] (2026-10-27) `shiono-a11y/my-live3-app` から `.claude/commands/`（6コマンド）をコピー
- [x] (2026-10-27) CLAUDE.md / LESSONS.md / PROGRESS.md / docs/ / PR テンプレートを my-live3-app と同じ構成で作成
- [x] (2026-10-27) `.vscode/settings.json` でターミナルの選択時コピー・右クリックコピー/貼り付けを有効化
- [x] (2026-10-30) `.claude/skills/` を導入: intent・ship（`shiono-a11y/sss-tool01`）、iken（`satoshiarimatsu/my-first-app`）。my-live3-app には skills なし。intent は EC-CUBE 向けに文言調整済み
- [ ] ship スキルを ec-cube 向けに書き換え（現状 Vercel・`my-app/scripts/ship-check.ts`・hooks・SST の LESSON 番号前提で動かない。デプロイ先・lint/テスト手順の確定後）
- [ ] iken スキルを ec-cube 向けに書き換え（現状 `api/feedback.ts`・レシピ解析前提。フィードバック受付の仕組みを決めてから）
- [x] (2026-10-30) `setup/initial-config` を push して PR 作成（#1）
- [ ] investigate-feedback / resolve-feedback / triage-sentry の扱いを決める（`scripts/*.mjs` 前提で現状動かない → 削除 or ec-cube 向けに書き換え）
- [x] (2026-10-30) night-audit を PHP 向け（composer / phpstan / php-cs-fixer / phpunit / bin/console）に書き換え
- [ ] night-audit を EC-CUBE 導入後に実際に1回回し、コマンド・パスが通るか確認（MS1 の lint/テスト手順確立と合わせて）
- [ ] CLAUDE.md の技術スタック・ディレクトリ構成を実態に合わせて確定

## MS1: 環境構築

- [ ] 開発環境（Codespace / Docker）で EC-CUBE を起動できる
- [ ] DB 初期化・管理画面ログイン確認
- [ ] lint / テストの実行手順を確立し CLAUDE.md に記載

## MS2: （未定）

- [ ] 
