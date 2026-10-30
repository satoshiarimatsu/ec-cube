---
name: iken
description: アプリ内フィードバック（GitHub Issue、label:feedback）を読み、再現・修正・返信・クローズまで行う。「フィードバック見て」「意見箱見て」「/iken」で起動（`feedback` は Claude Code の組み込みコマンドと衝突するため別名）。
---

# フィードバック対応

アプリの「💬 報告」「📮 取り込めなかったと報告」は `api/feedback.ts` 経由で GitHub Issue（label: `feedback` ＋ `parse-failed` / `bug` / `request`）になる。
このスキルは、その Issue を一覧 → 再現 → 修正 → 返信 → クローズ まで進める。

## 手順

1. **未対応の一覧**
   ```bash
   gh issue list --label feedback --state open --json number,title,labels,createdAt,body --limit 30
   ```
   0 件なら「未対応のフィードバックはありません」と報告して終了。

2. **Issue ごとに分類して対応**

   ### `parse-failed`（取り込み失敗）
   1. 本文先頭の `<!-- feedback:{...} -->` に kind / url / message / reporter が JSON で入っている。`解析情報` の JSON（route / pageTextLength / hasJsonLd / usedTools）も読む
   2. ローカルで再現する（Vercel を経由しない。Gemini を実際に呼ぶ）:
      ```bash
      node --env-file=.env scripts/parse-url.ts "<URL>"
      node --env-file=.env scripts/parse-url.ts "<URL>" --json   # 抽出内容を見たいとき
      ```
   3. 結果で切り分ける:
      | 結果 | 原因の見当 | 対処 |
      |---|---|---|
      | `text=0` かつ `route=tools` でも `empty=true` | ボット遮断 / JS 描画のみ / ログイン必須 | サイト別の取得方法（UA・API・JSON-LD の別経路）を `api/_lib/recipeParse.ts` に追加できるか調べる。無理なら「本文貼り付け／スクショ解析を使ってください」と返信 |
      | `text` は多いが `empty=true` or 材料・手順が極端に少ない | 抽出プロンプト / JSON-LD 解釈の問題 | `extractJsonLdRecipe` / `extractText` / `RECIPE_SPEC` を調整。他サイトが壊れないよう、代表 8 サイト（CLAUDE.md「エラーからの教訓」参照）で再確認 |
      | 動画サイト（YouTube / TikTok / リール） | テキストが無い | 対応不可。スクショ解析を案内して返信 |
      | ローカルでは成功する | 一時的なエラー / 本番の制限（関数タイムアウト等） | Vercel のログを確認（`npx vercel logs <deployment-url>`）。再発しなければその旨返信 |
   4. コードを直した場合: `npm run build` → `npm run lint` → 再現スクリプトで成功を確認 → コミット → push
   5. 返信してクローズ（日本語・利用者向けの言葉で。内部の実装名は書かない）:
      ```bash
      gh issue edit <番号> --add-label done
      gh issue comment <番号> --body "対応しました: ..."
      gh issue close <番号>
      ```
      対応できない場合は理由と代替手段を書いて、`gh issue close <番号> --reason "not planned"`。

   ### `bug`（不具合）
   - **オーナーの判断を待たずに直す**（利用者の合意事項）
   - 再現手順を本文から読み取り、コードを読んで原因を特定する
   - 直してビルド・lint → コミット・push → 返信 → label `done` を付けてクローズ
   - 設計変更や CLAUDE.md「必ず事前に相談するもの」に当たる場合だけ、原因と案を Issue にコメントしてオーナーに相談する

   ### `request`（機能の要望）／ `ui`（見た目・使いやすさ）
   - **オーナーがアプリの意見箱で「採用」にしたもの（label `approved`）だけ実装する**
   - `approved` が付いていない open の Issue には、**まだ実装しない**。代わりに「実装案・規模（小/中/大）・影響する画面」を 3〜5 行でコメントし、オーナーが判断しやすくする（既にコメント済みなら何もしない）
   - `on-hold`（保留）は飛ばす。クローズ済み（見送り／対応済み）も飛ばす
   - `approved` を実装したら、ビルド・lint → コミット・push → 利用者向けの言葉で返信 → label `done` を付けてクローズ
     ```bash
     gh issue edit <番号> --add-label done
     gh issue comment <番号> --body "対応しました: ..."
     gh issue close <番号>
     ```

   ### 状態の見方（label）
   | 状態 | label / state | 意味 |
   |---|---|---|
   | 未確認 | open、ラベルなし | オーナー未判断（要望・意見）／未着手（不具合・取り込み失敗） |
   | 採用 | open + `approved` | Claude が実装する |
   | 保留 | open + `on-hold` | 触らない |
   | 対応済み | closed + `done` | 完了 |
   | 見送り | closed（`done` なし） | オーナーが見送り |

3. **教訓を残す**
   新しい失敗パターン（特定サイトの挙動など）が分かったら、`CLAUDE.md` の「エラーからの教訓」に1行追記する。

4. **最後にまとめて報告**
   対応した Issue 番号・内容・残っている Issue を箇条書きで報告する。

## 注意
- `.env` は読めない。`node --env-file=.env` で渡す（内容は表示しない）
- Issue 本文には報告者のメールが入っている。コメントやコミットメッセージに転記しない
- ルール5（データ保護）は常に優先
