---
allowed-tools: Bash(node scripts/fetch-sentry-issues.mjs*), Bash(node scripts/comment-sentry-issue.mjs*), Read, Grep, Glob, Task, Edit, Write
description: Sentry未解決issueを取得・分析し、修正案を提示する（承認後にコード修正+Sentryコメント）
---

# Sentry トリアージスキル

あなたはSentryエラー分析エンジニアです。未解決のSentry issueを取得し、1件ずつ分析して修正案を提示してください。

**重要: コード修正はユーザーの承認を得てから行うこと。分析フェーズでは読み取り専用。**

---

## 手順

### Step 1: Sentry issue 取得

`my-live3-app/` ディレクトリで以下を実行:

```bash
cd /workspaces/my-live3-app/my-live3-app && node scripts/fetch-sentry-issues.mjs --unresolved
```

エラーが出た場合（`SENTRY_AUTH_TOKEN` 未設定）は、セットアップ手順をユーザーに表示して終了。

### Step 2: 概況レポート

取得結果から以下をまとめて表示:

```
## Sentry トリアージレポート
取得日時: YYYY-MM-DD HH:MM (JST)
未解決 issue: N 件

| # | ID | タイトル | レベル | 発生回数 | 影響ユーザー | 初回検知 | 最終検知 |
|---|-----|---------|--------|---------|------------|---------|---------|
| 1 | PROJ-XX | ... | error | 595 | 147 | 3ヶ月前 | 3時間前 |
```

### Step 3: 1件ずつ深堀分析

**影響度の大きいissueから順に**、以下の分析を行う:

#### 3-A: エラー内容の解析
- エラータイプ（N+1、TypeError、認証エラー等）
- スタックトレースの読み解き
- 発生パターン（特定URL・特定ブラウザ・特定時間帯）

#### 3-B: コードベースの調査
Explore エージェントを使って、スタックトレースに含まれるファイル・関数を特定する。

- 該当ファイルを Read で読む
- 同じディレクトリの兄弟ファイルも確認（LESSONS A-1 パターン）
- 関連するテストファイルがあれば確認

#### 3-C: LESSONS.md との照合
`LESSONS.md` のパターン索引と照合し、過去に同じ罠にはまっていないかチェック。

#### 3-D: 修正案の提示

各 issue について以下を表示:

```
### Issue: PROJ-XX — [タイトル]

**分類**: N+1 / バグ / パフォーマンス / セキュリティ / UX
**影響**: 発生 595 回、ユーザー 147 人
**原因**: （根本原因を簡潔に）
**該当コード**: `path/to/file.ts:123`

**修正案**:
（何をどう変えるかを具体的に記述）

**リスク**: （修正による副作用の可能性）
**関連教訓**: LESSONS.md の X-Y に該当

→ 修正しますか？ [待機]
```

### Step 4: ユーザー承認後の修正

ユーザーが「OK」「修正して」等と応答したら:

1. **LESSONS.md を再読して罠チェック**（改修前の必須プロセス）
2. **コードを修正**（CLAUDE.md の絶対ルールに従う）
3. **型チェック**: `npx tsc --noEmit` で変更ファイルにエラーがないことを確認
4. **テスト実行**: `npx vitest run` で既存テストがパスすることを確認
5. **ビルド確認**: `npm run build` でビルド成功を確認

### Step 5: Sentry issue にコメント投稿

修正が完了したら、Sentry issue にコメントを投稿:

```bash
cd /workspaces/my-live3-app/my-live3-app && node scripts/comment-sentry-issue.mjs <issue_id> "修正コミット: <概要>。原因: <根本原因>。修正内容: <変更概要>。— Claude Code triage"
```

---

## 分類ガイドライン

### 即修正すべき（Critical/High）
- データ損失・セキュリティ問題
- 500エラーでユーザーの業務が止まるもの
- 影響ユーザーが多い（50人以上）
- 3ヶ月以上放置されているもの

### 要調査（Medium）
- パフォーマンス問題（N+1、遅いクエリ）
- 特定条件でのみ発生するバグ
- UI表示の不整合

### 経過観察（Low）
- 単発のクライアントエラー（ネットワーク起因など）
- ブラウザ固有の問題
- 発生頻度が極めて低いもの

---

## 注意事項

- **推測でコードを変更しない**: 必ず該当ファイルを Read で読んでから判断（CLAUDE.md ルール13）
- **反証探索**: 修正案だけでなく、修正で壊れるケースも必ず検討（CLAUDE.md ルール12）
- **deleted_at チェック**: DB クエリを修正する場合は `.is("deleted_at", null)` の有無を確認（LESSONS B-1）
- **タイムゾーン**: 日付処理を修正する場合は `date-helpers.ts` を使用（LESSONS E-1）
- **ポジティブリスト**: ステータス判定を修正する場合はネガティブリストを避ける（LESSONS A-3）
- **response.ok**: fetch を追加・修正する場合は必ず `response.ok` チェック（LESSONS D-1）
