# 命名規則・コミット規約 — EC-CUBE

---

## コミットメッセージ

### 形式
```
<type>: <日本語メッセージ>
```

### タイプ一覧
| タイプ | 用途 |
|--------|------|
| `add:` | 新機能追加 |
| `update:` | 既存機能の改善 |
| `fix:` | バグ修正 |
| `remove:` | 機能・コード削除 |
| `chore:` | ビルド・ツール変更 |
| `docs:` | ドキュメント更新 |
| `refactor:` | リファクタリング |
| `test:` | テスト追加・修正 |

### ルール
- メッセージは日本語で簡潔に
- 複数修正の場合はカッコで列挙: `fix: 3件の不具合を修正（カート, 送料計算, 会員登録）`

---

## ファイル命名

| 種類 | パターン | 例 |
|------|---------|-----|
| Controller | `<名前>Controller.php` | `app/Customize/Controller/SampleController.php` |
| Entity | 単数形 PascalCase | `app/Customize/Entity/Sample.php` |
| Repository | `<Entity>Repository.php` | `app/Customize/Repository/SampleRepository.php` |
| FormType | `<名前>Type.php` | `app/Customize/Form/Type/SampleType.php` |
| Twig | snake_case | `app/template/default/Sample/index.twig` |

---

## 変数・関数命名
- クラス: PascalCase / メソッド・変数: camelCase / 定数: UPPER_SNAKE_CASE
- DB カラム: snake_case

---

## コミット前チェック
（lint・テストのコマンドが確定したらここに記載）
