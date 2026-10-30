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
- [x] (2026-10-30) investigate-feedback / resolve-feedback / triage-sentry を削除（`scripts/*.mjs`・Supabase・Sentry 前提で ec-cube では動かないため）
- [x] (2026-10-30) night-audit を PHP 向け（composer / phpstan / php-cs-fixer / phpunit / bin/console）に書き換え
- [ ] night-audit を EC-CUBE 導入後に実際に1回回し、コマンド・パスが通るか確認（MS1 の lint/テスト手順確立と合わせて）
  - 2026-10-30: 新アプリは EC-CUBE を使わないと決定 → EC-CUBE 4 前提の night-audit・CLAUDE.md は技術スタック確定後に作り直す
- [x] (2026-10-30) 旧サイト（自社 EC）の資料置き場 `legacy/` を作成し `.gitignore` で除外（public リポジトリのため）
- [ ] CLAUDE.md の技術スタック・ディレクトリ構成を実態に合わせて確定（新アプリの技術選定後。EC-CUBE 4 前提の記述・ルールを全面見直し）

## MS0.5: 旧サイト調査・移行範囲の決定

> 新アプリは EC-CUBE を使わず、自社サイト（EC-CUBE 製・大幅カスタマイズ）の**社内管理画面側の機能**を移す。詳細メモは `legacy/notes.md`（Git 管理外）。

- [x] (2026-10-30) 管理画面のスクショで全メニュー・申請系・受注系を棚卸し
- [x] (2026-10-30) 移す範囲の方向性を決定: 管理画面側 / 最初は受注・出荷 / 主な機能 = 受注入力・POS・申請系・請求管理
- [x] (2026-10-30) 申請系は「申請書の項目・承認ルートを画面で作れる」形で作る、過去の申請は全件移す、と決定
- [ ] 質問14: 受注 CSV（受注CSV登録 → 通常）の出どころを確認（取り込み担当の事務に聞く。実 CSV か列名を `legacy/` に）
- [ ] 請求管理の画面を棚卸し（スクショ）
- [ ] 制作会社にソース一式・DB 定義（スキーマのみ）・運用情報を依頼し、`legacy/` に受け取る
- [ ] 旧サイトのデータを新アプリでどう扱うか決める（旧 DB を直接読む / 同期）
- [ ] 新アプリの技術スタック（フレームワーク・DB）を決定
- [ ] `docs/legacy-inventory.md` 相当の機能一覧を作り、移行する/しないを確定（public に出してよい粒度に注意）

## MS1: 環境構築

- [ ] 開発環境（Codespace / Docker）で EC-CUBE を起動できる
- [ ] DB 初期化・管理画面ログイン確認
- [ ] lint / テストの実行手順を確立し CLAUDE.md に記載

## MS2: （未定）

- [ ] 
