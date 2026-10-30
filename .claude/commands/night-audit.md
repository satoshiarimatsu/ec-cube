---
allowed-tools: Bash(composer validate*), Bash(composer audit*), Bash(composer outdated*), Bash(php -v*), Bash(php -l *), Bash(xargs -0 -n1 php -l*), Bash(php bin/console lint:*), Bash(php bin/console debug:router*), Bash(php bin/console doctrine:schema:validate*), Bash(php bin/console doctrine:migrations:status*), Bash(php bin/console doctrine:migrations:list*), Bash(vendor/bin/phpstan *), Bash(vendor/bin/php-cs-fixer fix --dry-run*), Bash(vendor/bin/phpunit*), Bash(wc *), Bash(find *), Bash(git log*), Bash(git diff*), Bash(git status*), Bash(git ls-files*), Bash(ls *), Bash(head *), Bash(tail *), Bash(grep *), Read, Grep, Glob, Task, Write, Edit
description: 寝ている間に自動実行するナイト監査 — 構文・静的解析・Lint・テスト・罠チェック・シミュレーションを全自動で実行し結果をMDに記録
---

# ナイト監査スキル（Night Audit）— EC-CUBE 4 / PHP 版

あなたは品質監査のエキスパートです。ユーザーが離席している間に、コードベース全体を自動で検査し、問題をすべて `docs/night-audit-report.md` に記録してください。

**最重要ルール: コードの修正は一切行わない。検出と記録のみ。**

**DB を変える操作は禁止**（CLAUDE.md 絶対ルール13）。`doctrine:migrations:migrate`・`doctrine:schema:update --force`・`doctrine:fixtures:load`・`eccube:install` などは実行しない。DB は「読む」コマンドだけを使う。対象はローカル開発 DB のみで、本番 DB には当てない。

---

## 出力ファイル

メインレポート: `docs/night-audit-report.md`

各フェーズ完了ごとにファイルを上書き保存すること（途中でPCが落ちても途中結果が残るように）。

## レポートフォーマット

```markdown
# ナイト監査レポート
実行日時: YYYY-MM-DD HH:MM (JST)
実行ブランチ: <branch name>
最終更新フェーズ: Phase X（進行中 or 完了）

## 実行環境
| 項目 | 結果 |
|------|------|
| PHP | 8.x.x |
| EC-CUBE | 4.x.x（composer.lock の ec-cube/ec-cube、または src/Eccube/Common/Constant.php の VERSION） |
| DB 接続 | OK / NG（NG なら DB 系チェックはスキップ） |

## スキップしたチェック
| チェック | 理由 |
|----------|------|

## サマリー
| 深刻度 | 件数 |
|--------|------|
| CRITICAL | X |
| HIGH | X |
| MEDIUM | X |
| LOW | X |
| INFO | X |

## Phase 1: 構文・静的解析・Lint・テスト
...
## Phase 2: ...
（以下フェーズごとに追記）
```

### 深刻度の定義

| 深刻度 | 基準 | 例 |
|--------|------|----|
| **CRITICAL** | 本番で即障害。データ損失・セキュリティ穴 | 管理機能が admin ルート外に公開、SQLインジェクション、CSRF なしの削除処理 |
| **HIGH** | 本番で機能不全。ユーザーが購入・運営できない | 構文エラー、DI コンテナが壊れている、テスト失敗、Entity とスキーマの不一致 |
| **MEDIUM** | 特定条件で問題発生。回避策あり | タイムゾーン表示ズレ、在庫の同時購入、例外の握りつぶし |
| **LOW** | 品質低下だが機能は正常 | 50行超え関数・500行超えクラス、TODO 残留、型宣言漏れ |
| **INFO** | 推奨事項・参考情報 | パフォーマンス改善候補、リファクタ候補 |

### 検査対象の範囲

- **主対象**: `app/Customize/`、`app/Plugin/`（自作プラグインのみ）、`app/template/`、`app/config/eccube/`、`tests/` のうち独自テスト
- **本体 `src/Eccube/`**: 中身の品質は監査しない。**変更されていないか**だけを Phase 3-1 で確認する
- `vendor/`・`var/`・`html/bundle/` は対象外

---

## 実行手順

### Phase 0: 前提確認

リポジトリのルート（`/workspaces/ec-cube`）で実行する。

```bash
php -v
ls composer.json composer.lock bin/console vendor/autoload.php 2>&1
ls phpstan.neon* .php-cs-fixer*.php phpunit.xml* codeception.yml 2>&1
php bin/console doctrine:migrations:status 2>&1 | head -30
```

- `composer.json` か `bin/console` が無い → EC-CUBE 未導入。Phase 1・5・6 の実行系をスキップし、grep 系（Phase 2〜4）だけを行う
- `vendor/` が無い → `composer install` は**実行しない**（依存を変える操作のため）。実行系をスキップし「composer install が必要」と記録
- 設定ファイル（phpstan.neon 等）が無いツールはスキップし、「スキップしたチェック」に理由を書く
- `doctrine:migrations:status` が接続エラー → DB 接続 NG。DB 系チェック（1-6, 6-4）をスキップ

**スキップは失敗ではない。** ただし「何を検査していないか」がレポートで必ず分かるようにする。

---

### Phase 1: 基盤チェック（構文・静的解析・Lint・テスト）

#### 1-1. composer.json の妥当性
```bash
composer validate --no-check-publish 2>&1
```
- エラーは HIGH、警告は LOW

#### 1-2. PHP 構文チェック
```bash
find app/Customize app/Plugin -name '*.php' -print0 2>/dev/null | xargs -0 -n1 php -l 2>&1 | grep -v '^No syntax errors'
```
- 構文エラーは全件 HIGH

#### 1-3. PHPStan
```bash
vendor/bin/phpstan analyse --no-progress --error-format=raw --memory-limit=1G 2>&1
```
- 設定ファイル（phpstan.neon / phpstan.neon.dist）の paths・level に従う。**設定は変更しない**
- エラーを全件記録（ファイル:行、メッセージ）。深刻度: HIGH（型不一致・未定義メソッド）／ MEDIUM（その他）
- 実行したレベルをレポートに書く

#### 1-4. PHP-CS-Fixer（差分の確認のみ）
```bash
vendor/bin/php-cs-fixer fix --dry-run --diff 2>&1
```
- **`--dry-run` 必須**（付けないとファイルが書き換わる）
- 違反ファイル数とルール別件数を集計。深刻度: LOW

#### 1-5. Symfony の Lint
```bash
php bin/console lint:container 2>&1
php bin/console lint:yaml app/config 2>&1
php bin/console lint:twig app/template app/Customize app/Plugin 2>&1
```
- `lint:container` の失敗は CRITICAL（サイト全体が 500 になる）
- YAML / Twig の構文エラーは HIGH

#### 1-6. Entity とスキーマの整合（読み取りのみ・DB 接続 OK のとき）
```bash
php bin/console doctrine:schema:validate 2>&1
```
- Mapping の失敗は HIGH
- Database の不一致は、Phase 6-4 で原因（未適用マイグレーション／マイグレーション漏れ）を切り分けてから深刻度を決める

#### 1-7. テスト
```bash
vendor/bin/phpunit 2>&1 | tail -80
```
- 本体のテスト（tests/Eccube）まで含むと長時間かかる。phpunit.xml に独自テストのスイートがあればそれを優先し、全件は時間が許せば実行
- 2分を超えそうなら Bash の `timeout` を延ばす（最大600000ms）か `run_in_background` で実行する
- テストが DB を書き換える構成（テスト用 DB でない）なら実行せず、「スキップしたチェック」に記録
- 失敗テストを全件記録（テスト名、ファイル名、エラーメッセージ）。新規失敗は HIGH
- 既知の失敗は LESSONS.md / PROGRESS.md に記載があれば「既知」とマーク

**→ Phase 1 完了後、レポートファイルに書き出して保存**

---

### Phase 2: セキュリティ監査

#### 2-1. ハードコードされたシークレット
以下を `app/`・`html/`・`tests/` で検索:
- `password' => '`, `'secret'`, `api_key`, `apiKey`, `token = '`, `Bearer ` + 固定文字列
- `.env` がコミットされていないか（`git ls-files | grep -E '(^|/)\.env($|\.local)'`）。`.env.dist` 等の雛形は対象外
- `APP_DEBUG=1` / `APP_ENV=dev` が本番向け設定に入っていないか
- 深刻度: CRITICAL

#### 2-2. 管理機能の公開範囲（認可）
```bash
php bin/console debug:router 2>&1
```
- `app/Customize/Controller/` と自作プラグインの Controller のルートを列挙する
- 管理機能（データの更新・削除・CSV 出力・会員情報の閲覧）のルートが `%eccube_admin_route%` 配下にあるか。admin 配下でないと管理者ログインで守られない
- マイページ系（会員のデータを扱う）で、ログイン会員と対象データの持ち主を照合しているか（URL の ID を書き換えて他人の受注を見られないか）
- 深刻度: CRITICAL（認可なしで他人のデータ・管理機能に到達）／ HIGH（照合が片方だけ）

#### 2-3. CSRF（CLAUDE.md 絶対ルール6）
- 状態を変える Controller のアクション（`methods={"POST"}`・`DELETE`・`PUT`、またはフォームの `handleRequest` で保存するもの）を列挙
- フォームを使わない処理は `$this->isTokenValid()` を呼んでいるか
- フォームを使う処理は FormType で `csrf_protection` を `false` にしていないか
- Twig で POST するリンク（`token-for-anchor`）に `csrf_token_for_anchor()` が付いているか
- GET リクエストでデータを変更していないか
- 深刻度: CRITICAL（削除・更新に CSRF なし）／ HIGH（その他の状態変更）

#### 2-4. SQL インジェクション
- `->query(`, `executeQuery(`, `executeStatement(`, `createQuery(`, `->where(`, `->andWhere(` に、文字列連結（`.` ）や `"...$var..."` で外部入力を埋め込んでいる箇所
- `setParameter()` を使わずに値を埋めている箇所
- 深刻度: CRITICAL

#### 2-5. XSS（CLAUDE.md 絶対ルール5）
- Twig の `|raw` の全使用箇所。理由コメント（`{# ... #}`）があるか、中身が外部入力（会員・商品名・問い合わせ本文など）由来でないか
- `autoescape false` ブロック
- PHP から HTML 文字列を組み立てて Twig に渡している箇所
- 深刻度: CRITICAL（外部入力を raw 出力）／ MEDIUM（理由コメントなし）／ INFO（理由あり・安全）

#### 2-6. ファイルアップロード
- アップロード処理で拡張子・MIME の**許可リスト**があるか、保存先が `html/` 配下で PHP として実行されうる場所でないか
- 深刻度: CRITICAL

**→ Phase 2 完了後、レポートファイルに追記して保存**

---

### Phase 3: CLAUDE.md 絶対ルール準拠チェック

#### 3-1. 本体の直接編集（ルール1）
```bash
git log --oneline --format='%h %ad %s' --date=short -- src/Eccube | head -30
git status --short -- src/Eccube
```
- EC-CUBE 本体の導入・バージョンアップ以外のコミットで `src/Eccube/` を変えているもの、未コミットの変更
- 深刻度: HIGH（バージョンアップで消える／衝突する）

#### 3-2. 関数・クラスのサイズ（ルール2）
- 50行を超えるメソッド、500行を超えるクラスファイルをリストアップ（対象は主対象の範囲のみ）
- 深刻度: LOW

#### 3-3. 型宣言（ルール3）
- 引数・戻り値・プロパティの型宣言が無いメソッド（コンストラクタの戻り値は除く）
- `declare(strict_types=1);` が無いファイル
- `@phpstan-ignore` / `@phpstan-ignore-line` / `@phpstan-ignore-next-line`、phpstan の baseline・`ignoreErrors` に独自コードが入っていないか
- 深刻度: MEDIUM（ignore による握りつぶし）／ LOW（型宣言漏れ）

#### 3-4. タイムゾーン（ルール4）
- `date(`, `time()`, `strtotime(`, `mktime(`, `new \DateTime()`, `new \DateTimeImmutable()`, `date_default_timezone_set(` を検索
- DB に保存する日時と表示用の日時を混同していないか。EC-CUBE は DB に UTC で保存し、表示時に `eccube_timezone`（Asia/Tokyo）へ変換する前提
- Twig で `|date` にタイムゾーンを渡さず、UTC のまま表示していないか
- 深刻度: MEDIUM

#### 3-5. TODO コメント残留（ルール7）
- `TODO`, `FIXME`, `HACK`, `XXX` を PHP / Twig / JS で全検索
- 深刻度: LOW

#### 3-6. 手作業の DDL（ルール9）
- マイグレーション（`app/DoctrineMigrations/` 等）以外にある `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE` と `.sql` ファイル
- 深刻度: MEDIUM

**→ Phase 3 完了後、レポートファイルに追記して保存**

---

### Phase 4: 罠パターン総点検

LESSONS.md のパターン索引に載っている罠をコードで検索する。加えて、以下の PHP / Doctrine 共通の罠を検索する。

#### 4-1. 例外の握りつぶし
- 空の `catch` ブロック、`catch` でログだけ書いて成功扱いで続行している箇所
- `catch (\Exception $e)` で広く捕まえて、利用者に成功メッセージを出している箇所
- 深刻度: HIGH

#### 4-2. 保存漏れ・保存結果の未確認
- `persist(` / `remove(` の後に `flush()` が無い経路
- `flush()` 後に `addSuccess()`（成功フラッシュ）を出す前に、失敗時の `addError()` 経路があるか（完了の定義③）
- 深刻度: HIGH

#### 4-3. 緩い比較での判定
- `in_array(` の第3引数 `true` なし、`==` / `!=` で ID・ステータス・金額を比較している箇所
- secret・トークンを `===` で比較している箇所（`hash_equals()` を使うべき）
- `strpos(` / `str_contains(` で権限・ロールを判定している箇所
- 深刻度: HIGH（認可判定）／ MEDIUM（その他）

#### 4-4. 除外リスト方式（docs/coding-rules.md「許可リスト方式」）
- 「〜以外は許可」の配列（`excludeRoutes`, `skipStatuses` 等）で制御しているロジック
- 深刻度: LOW

#### 4-5. 本体を壊す拡張
- EventSubscriber / FormExtension で、本体の処理結果を黙って上書きしている箇所
- Entity Trait（`@EntityExtension`）で追加した列に、マイグレーションが無いもの
- 深刻度: MEDIUM

**→ Phase 4 完了後、レポートファイルに追記して保存**

---

### Phase 5: ビジネスロジック・シミュレーション

コードを読んで、以下のシナリオで問題が起きないかロジックを追跡する。
**実際に画面を操作したり DB を書き換えたりせず、コードリーディングによる机上シミュレーション。**

#### 5-1. 利用者別アクセスシミュレーション
以下の利用者それぞれが、独自追加した画面・処理にアクセスした場合のコードパスを追跡:
- 非会員（ゲスト購入含む）、会員、他の会員、管理者（権限設定 `Authority` 別）
- 確認項目:
  - 会員が他の会員の受注・お届け先・お気に入りを ID 書き換えで見られないか
  - 管理画面の URL を未ログイン・会員で直打ちしたときに弾かれるか
  - 権限の狭い管理者が、メニューに無い機能を URL 直打ちで使えないか
- 深刻度: CRITICAL（認可バイパス）／ HIGH（画面だけ隠して処理は通る）

#### 5-2. タイムゾーン・シミュレーション
- **JST 0:00〜8:59**（UTC では前日）— 受注日・出荷日・日別集計の日付境界
- **月末最終日 23:59** — 月次売上集計の境界
- **年末年始（12/31〜1/1）** — 年をまたぐ集計・クーポン期限
- **販売期間・クーポン期限の開始/終了時刻ちょうど**
- 確認項目: DB（UTC）と表示（Asia/Tokyo）の変換漏れ、検索条件の日付を UTC に変換しているか
- 深刻度: MEDIUM（表示ズレ）／ HIGH（集計・期限判定のズレ）

#### 5-3. 購入フロー・在庫シミュレーション
- 在庫1の商品を2人が同時に購入するケース（在庫がマイナスにならないか）
- 購入確認後にカートや価格・送料・税率が変わるケース（確認画面と確定金額の不一致）
- 決済失敗・ブラウザバック・二重送信のケース（受注の重複・在庫の二重引き当て）
- 表示側と計算側で金額・送料・税・ポイントを別実装していないか（docs/coding-rules.md 4）
- 深刻度: HIGH

#### 5-4. 大量データ・シミュレーション
- `findAll()`・ページネーションなしの一覧、CSV 出力でのメモリ全件読み込み
- ループ内で Repository を呼ぶ N+1、Twig 内での遅延ロードの連鎖
- 深刻度: MEDIUM（パフォーマンス）／ LOW（小規模データ）

#### 5-5. エラー時の UX シミュレーション
- 保存失敗・バリデーションエラー・外部 API 失敗時に、フラッシュメッセージやフォームエラーが出るか（完了の定義③）
- 500 エラー時に利用者向けのエラーページになるか、スタックトレースが出ないか
- 深刻度: MEDIUM

**→ Phase 5 完了後、レポートファイルに追記して保存**

---

### Phase 6: コードヘルス・メトリクス

#### 6-1. 依存関係の健全性
```bash
composer audit 2>&1
composer outdated --direct 2>&1
```
- 脆弱性は深刻度に応じて CRITICAL / HIGH / MEDIUM
- 大きく古いパッケージ、EC-CUBE 本体のパッチ版の遅れは INFO
- `composer update` は**実行しない**

#### 6-2. 重複コードの検出
- 同じロジックが3箇所以上にコピーされている箇所（Controller・Service・Twig）
- 同じ QueryBuilder の組み立ての重複
- 深刻度: INFO

#### 6-3. テストカバレッジ概況
- テストが無い `app/Customize/` の Service・Repository・Controller
- 金額・在庫・権限に関わるのにテストが無いもの
- 深刻度: INFO（金額・在庫・権限は MEDIUM）

#### 6-4. マイグレーションと DB の食い違い（読み取りのみ・DB 接続 OK のとき）
```bash
php bin/console doctrine:migrations:status 2>&1
php bin/console doctrine:migrations:list 2>&1
```
- 未適用のマイグレーション → MEDIUM（ローカルが古いだけか、本番にも未適用かは区別できないので「ローカル DB では未適用」と書く）
- 1-6 で Database の不一致があり、それを埋めるマイグレーションが無い → HIGH（Entity だけ変えてマイグレーションを書き忘れている）
- `docs/db-design.md` に載っているのにマイグレーションも Entity も無いテーブル → MEDIUM（ドキュメントは適用の証拠にならない）
- **差分を埋める SQL を実行しない。** `doctrine:schema:update` は `--dump-sql` で見るだけでも使わない（`--force` との打ち間違いを防ぐため）

**→ Phase 6 完了後、レポートファイルに追記して保存**

---

### Phase 7: 最終サマリーと改善ロードマップ

すべてのフェーズの結果を集約し、以下を作成:

#### 7-1. 深刻度別サマリーテーブル（冒頭に配置）
全フェーズの合計件数を深刻度別に集計。

#### 7-2. 修正優先度ランキング（TOP 20）
深刻度・影響範囲・修正難易度を考慮して、修正すべき順にランキング。
各項目に以下を記載:
- **問題の概要**
- **影響範囲**: どの機能・どの利用者（購入者／会員／管理者）に影響するか
- **修正難易度**: 簡単 / 中程度 / 複雑
- **関連ファイル**: 修正が必要なファイルパス一覧
- **推奨修正方針**: 具体的にどう直すべきか（コードは書かない、方針のみ）
- **修正時のリスク**: 修正によって壊れる可能性がある箇所

#### 7-3. リスクマップ
機能領域ごとのリスク評価マトリクス:
```
| 機能領域 | CRITICAL | HIGH | MEDIUM | LOW | リスクレベル |
|----------|----------|------|--------|-----|------------|
| 認証・認可（会員/管理者） | X | X | X | X | 🔴/🟡/🟢 |
| 商品・在庫 | ... |
| カート・購入フロー | ... |
| 決済 | ... |
| 受注・出荷 | ... |
| 会員・マイページ | ... |
| メール | ... |
| 管理画面 | ... |
| プラグイン | ... |
```

#### 7-4. 次のアクション提案
改善開発に着手する際の推奨手順:
1. CRITICAL を先に全件対応
2. HIGH を機能領域ごとにまとめて対応
3. MEDIUM 以下は通常開発の中で順次対応

**→ 最終レポートを保存して完了**

---

## 実行上の注意

1. **各フェーズ完了ごとに必ずレポートファイルを保存する**（途中クラッシュ対策）
2. **並列実行を活用する**: Phase 2〜4 の grep 系チェックは Task エージェントを使って並列実行する
3. **タイムアウト対策**: PHPStan・テストが長い場合は適切な timeout を設定する
4. **誤検知の除外**: テストファイル内のパターンや、意図的な例外（コメントで明記されたもの）は別カウントする
5. **再実行可能**: 途中で停止した場合、「最終更新フェーズ」から再開できるようにする
6. **開発は一切しない**: コードの修正・ファイルの追加（レポート以外）は絶対に行わない。`composer install/update`・`cache:clear`・マイグレーション実行もしない
