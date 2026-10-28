# Conversion Report の雛形

変換の最終報告（Phase 8）。途中経過（Phase 4）は、同じ項目のうち分かっているものだけを出す。見出しは変えず、該当が無い項目は「なし」と書く。未接続の任意 MCP に頼る項目は「未実施（ツール未接続）」と書く。

````markdown
# Conversion Report

## 1. 入力
- マニュアル: <manualPath>（id: <Manual.id>、版: <version>、言語: <language>）
- 変換した Topic: <全部 / topics の一覧>
- 出力先の CSDB: <csdbDirs の 1 件目>（参照の探索先: <残り>）
- 機密区分: <値>（利用者の指定。値は解釈していない）

## 2. 採番の規則
- モデル識別コード / システム差異コード: <値>
- SNS: <systemCode>-<subSystemCode>-<subSubSystemCode>-<assyCode>
- 情報コード: descript <..> / proced <..> / process <..> / fault <..>（利用者が指定。試作の見本の値を使ったときは「試作用の値で意味を持たない」と書く）
- 品目位置コード / CAGE コード: <..> / <..>
- 台帳: <前回の台帳を使った / 新規>。保存先: <`<manual>.ledger.json` / 未保存>。次の ICN の連番: <icnNext>

## 3. 作ったもの
| 種類 | DMC / PMC | 題名 | Topic / 画面 | ファイル | 書き込み |
|---|---|---|---|---|---|
| descript | <dmCode> | <題名> | <topicId> | <fileName> | <written のパス / 未書き込み> |
| … | | | | | |
| pm | <pmCode> | | | <fileName> | |

- 素材（ICN）: <assetId → ICN と原本の有無（`sourceExists`）の一覧>

## 4. 検証状態
応答の `summary` と `verify_csdb` の `totals` から転記する。
| 工程 | 結果 | 備考 |
|---|---|---|
| XSD | valid <n> / invalid <n> / schema-not-found <n> / not-run <n> | schema-not-found と not-run は「検証していない」 |
| BREX | passed <n> / violations <n> / incomplete <n> / not-run <n> | `manualChecks` は人の確認が要る項目 |
| 参照 | resolved <n> / unresolved <n> | 内訳: 素材の原本なし <n>、仕様書への参照のみ <n>、その他 <n> |

- 検証した範囲: <変換直後の検証 / verify_csdb で CSDB 全体 / 両方>
- この報告は S1000D への準拠を判定していない。`valid` と `passed` は、XSD と機械で検査できる BREX のルールに誤りが無かったことだけを表す

## 5. 自動で直したもの（`appliedFixes[]`）
| DM | 場所 | 変更前 → 変更後 | 根拠（`source`） |
|---|---|---|---|

## 6. 利用者が選んだもの
| DM | 問題 | 候補 | 選んだ値 | `apply_fixes` の結果（`applied` / `rejected`） |
|---|---|---|---|---|

- 候補が無く、利用者に聞いて決めたもの（`confidence: none`）: <一覧 / なし>

## 7. 表せなかったもの（`notRepresented`）
| 内容 | 件数 | マニュアルの位置 | 利用者への説明 |
|---|---|---|---|
- `_notes` で補った値: <地域、既定の適用性表示文、仮の NOTATION など>

## 8. 未解決のまま残ったもの
- <問題と、残した理由（利用者の判断 / 素材なし / 候補なし）>

## 9. 任意の MCP
- brex（形式の確認）: <実施 / 未実施（ツール未接続）>
- reader（変換結果の読み出し）: <実施 / 未実施（ツール未接続）>

## 10. 書き込んだファイルと台帳
- 書き込んだファイル: <パスの一覧と件数（`summary.written`）>
- 台帳の保存先: <パス / 未保存>
````

## 書くときの注意

- 「検証済み」「合格」の語は、`valid` と `passed` の範囲を超えて使わない
- 自動で直したものと、利用者が選んだものを混ぜない（根拠が違う）
- 書き込んでいないときは、見出し 3 の「書き込み」を「未書き込み」にし、見出し 10 に「書き込みなし」と書く
