# EC-CUBE - 開発ガイド

> **最初に必ず PROGRESS.md と LESSONS.md を読むこと。** 未完了の項目から再開。完了したら `[x]` + 日付に更新。存在しない作業は先に追記してから着手。

## チェックポイント
- セクション完了ごとにサマリー報告し、ユーザーの確認を取る
- セクション完了ごとに会話を区切り、新しい会話で PROGRESS.md を読み直して再開
- 設定ファイル（CLAUDE.md, PHPStan/PHP-CS-Fixer 設定, 環境変数）の変更時は必ずユーザーに報告

## 完了の定義（5条件すべて満たすこと）
1. 画面のボタン/フォームが Controller を実際に呼び出すこと
2. 処理結果が DB に反映されること
3. 成功時・失敗時のメッセージ（フラッシュメッセージ等）が出ること
4. 一覧画面があれば作成データが表示されること
5. 完了報告時に「何を操作すると何が起きるか」を手順で示すこと

## 技術スタック
<!-- 確定したら更新すること -->
| レイヤー | 技術 |
|---------|------|
| フレームワーク | EC-CUBE 4.x（Symfony） / PHP 8.x |
| テンプレート | Twig |
| DB | MySQL / PostgreSQL（Doctrine ORM） |
| テスト | PHPUnit / Codeception |
| デプロイ | （未定） |
| 監視 | （未定） |

## ディレクトリ構成
```
app/Customize/       → 独自カスタマイズ（Controller, Entity, Form, Repository …）
app/Plugin/          → プラグイン
app/template/        → テンプレート上書き（default / admin）
app/config/eccube/   → 設定ファイル
src/Eccube/          → 本体コード（原則直接編集しない）
html/                → 公開ディレクトリ
tests/               → テストファイル
```

## 絶対ルール
1. **本体を直接編集しない**: `src/Eccube/` は触らず、`app/Customize/` か プラグインで拡張する
2. **関数サイズ**: 1関数50行以内。クラスは500行以内を目安に分割
3. **型安全**: 引数・戻り値の型宣言を必ず書く。`@phpstan-ignore` での握りつぶし禁止
4. **タイムゾーン**: DB の保存形式と表示タイムゾーン（Asia/Tokyo）を混同しない
5. **エスケープ**: Twig の `|raw` は原則禁止。使う場合は理由をコメントで残す
6. **CSRF**: 状態を変更するフォーム・リクエストには必ず CSRF トークンを付与する
7. **TODO禁止**: TODO コメントを残さない。その場で実装するか PROGRESS.md に追記
8. **コミット前**: lint → テスト → キャッシュクリア後の動作確認が全パス必須
9. **マイグレーション**: Doctrine Migrations で管理。手作業の DDL は禁止
10. **仮説駆動**: バグ調査時は DB を先に確認し仮説を立ててから修正。推測でコードを変更しない
11. **反証探索**: 修正の正しさだけでなく壊れるケースも必ず探す。仮説が空振りしたら別仮説を3つ列挙
12. **実コード確認**: ファイル内容・DB 状態を推測しない。必ず読んでから判断する
13. **不可逆操作の確認**: DB マイグレーション・本番デプロイ・データ削除の前にユーザー確認必須

## 改修・バグ修正の必須プロセス
- **改修前**: `LESSONS.md` のパターン索引で同じ罠に陥っていないか確認する
- **改修後**: 新しいパターンを見つけたら `LESSONS.md` に反映する

## 過去のミスと教訓（致命的サマリ）
<!-- LESSONS.md で致命度「高」の教訓を 1 行ずつ追記する -->
- （まだなし）

## 詳細ドキュメント
- [LESSONS.md](LESSONS.md) — **過去のバグパターン・再発防止策（改修時必読）**
- [PROGRESS.md](PROGRESS.md) — 開発進捗
- [docs/PRD.md](docs/PRD.md) — プロダクト要件定義
- [docs/requirements.md](docs/requirements.md) — 全機能一覧 (F-xxx-nn)
- [docs/roadmap.md](docs/roadmap.md) — マイルストーン・ロードマップ
- [docs/db-design.md](docs/db-design.md) — テーブル設計
- [docs/conventions.md](docs/conventions.md) — 命名規則・コミット規約
- [docs/error-handling.md](docs/error-handling.md) — エラー処理方針
- [docs/coding-rules.md](docs/coding-rules.md) — コーディングルール詳細

## スキル自動起動ルール
以下のキーワード・意図を検出したら、対応するスキルを Skill ツールで即座に起動すること。

| トリガー（ユーザー発話の意図） | スキル名 |
|-------------------------------|---------|
| フィードバックを調査 / FB調査 / フィードバック分析 / 未処理フィードバック / フィードバック確認 / FBレポート | `investigate-feedback` |
| FBに返信 / フィードバック対応完了 / FB処理済み / フィードバックを閉じる / 対応済みにして | `resolve-feedback` |
| ナイト監査 / 夜間監査 / 寝てる間にチェック / 全体チェック / 品質監査 / night audit / 総点検 | `night-audit` |
| 引継ぎ / ハンドオフ / セッション終了 / 次のチャットへ / 閉じる準備 / handoff / 引き継ぎ | `handoff` |
| Sentry / エラー分析 / triage-sentry / Sentryチェック / 本番エラー / 障害チェック | `triage-sentry` |
| 穴チェック / 穴ある？ / 抜け漏れチェック / 修正の品質確認 / hole-check / 修正レビュー | `hole-check` |
| 作る前に整理 / 意図をまとめて / 要件をまとめて / intent（migration・社外に届く変更・新機能/画面の実装前は必須） | `intent` |
