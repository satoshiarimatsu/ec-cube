# PROGRESS.md - EC-CUBE 開発進捗

> **ルール**: 完了=`[x] (YYYY-MM-DD)`。削除しない。新規タスクは追記してから着手。

---

## MS0: 初期設定

- [x] (2026-10-27) `shiono-a11y/my-live3-app` から `.claude/commands/`（6コマンド）をコピー
- [x] (2026-10-27) CLAUDE.md / LESSONS.md / PROGRESS.md / docs/ / PR テンプレートを my-live3-app と同じ構成で作成
- [x] (2026-10-27) `.vscode/settings.json` でターミナルの選択時コピー・右クリックコピー/貼り付けを有効化
- [ ] `setup/initial-config` を push して PR 作成（ユーザーが手動実行）
- [ ] investigate-feedback / resolve-feedback / triage-sentry の扱いを決める（`scripts/*.mjs` 前提で現状動かない → 削除 or ec-cube 向けに書き換え）
- [ ] night-audit を PHP 向け（composer / phpunit 等）に書き換え
- [ ] CLAUDE.md の技術スタック・ディレクトリ構成を実態に合わせて確定

## MS1: 環境構築

- [ ] 開発環境（Codespace / Docker）で EC-CUBE を起動できる
- [ ] DB 初期化・管理画面ログイン確認
- [ ] lint / テストの実行手順を確立し CLAUDE.md に記載

## MS2: （未定）

- [ ] 
