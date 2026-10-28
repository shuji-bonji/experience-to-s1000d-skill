---
name: experience-to-s1000d
description: 体験型マニュアル（experiential-manual-model の JSON。Topic・画面・ホットスポットを持つ）を、S1000D の CSDB（データモジュール・パブリケーションモジュール・ICN）に変換する手順。s1000d-writer-mcp の convert_manual で採番・組み立て・検証を行い、残った修正は候補の中から利用者に選んでもらい、s1000d-verify-mcp の verify_csdb で CSDB 全体を確かめて、Conversion Report を返す。ユーザーが「この体験型マニュアルを S1000D にして」「manual.json を DM に変換して」「券売機のマニュアルを CSDB に出力して」「体験型マニュアルから S1000D のデータモジュールを作りたい」「マニュアルの JSON を CSDB に入れて検証まで」などに言及したら、単発の convert_manual 呼び出しで済ませず必ずこの Skill を使う。XML は書かず、S1000D への準拠は判定しない。
---

# experience-to-s1000d — 体験型マニュアルから CSDB への変換

体験型マニュアル（experiential-manual-model の JSON）を、S1000D の CSDB に変換する手順書。
自前の変換・検証の処理は持たない。変換は s1000d-writer-mcp、検証は s1000d-verify-mcp の応答を根拠にする。

共通の原則:

1. **XML を書かない。** XML は writer が組み立てる。この手順に、LLM が XML を書く工程は無い
2. **判定はサーバーが下し、Skill は説明する。** XSD・BREX・参照の判定、修正の候補は MCP の応答をそのまま根拠にする。応答を読んで利用者に説明し、次の操作を決めるだけで、判定を上書きしない
3. **候補の外の値を使わない。** 修正は `fixes[]` の候補から選ぶ。`confidence: none` のときと、`valueForm` が `range` / `pattern` のときは利用者に聞く
4. **書き込む前に確かめる。** `write: true` は、利用者が書き込み先を確認してからにする
5. **検証状態を必ず報告に書く。** `schema-not-found` と `not-run` を「合格」と書かない。S1000D への準拠は判定しない
6. **前提の MCP が無いときは止まる。** 必須の MCP が接続されていなければ、その旨と接続の方法を伝えて止まる。任意の MCP が無いときは、報告の該当項目に「未実施（ツール未接続）」と書く

> [!NOTE]
> この Skill は ASD とは無関係の個人プロジェクトの一部。S1000D® は ASD の登録商標。S1000D の仕様書・XSD・既定 BREX は同梱されておらず、利用者が入手して `S1000D_SPEC_DIR` で指定する。

## 前提 MCP

| MCP | 必須/任意 | 最低版 | 役割 |
|---|---|---|---|
| s1000d-writer-mcp | **必須** | 0.1.1 | `convert_manual`（採番・組み立て・検証・書き込み）、`apply_fixes`（選んだ修正の適用） |
| s1000d-verify-mcp | **必須** | 0.6.0 | `verify_csdb`（CSDB 全体の検証）、`check_references`（参照の内訳） |
| s1000d-brex-mcp | 任意 | 0.4.0 | `list_notations`（図の形式を BREX が許すか）、`lookup_value`（機密区分などの許可値） |
| s1000d-reader-mcp | 任意 | 0.7.0 | `read_data_module`（変換結果を構造として読んで見せる） |

writer か verify が未接続なら、変換は成立しない。次を伝えて停止する。

- `/plugin install s1000d-writer-mcp@shuji-bonji` と `/plugin install s1000d-verify-mcp@shuji-bonji`
- または `npx @shuji-bonji/s1000d-writer-mcp` と `npx @shuji-bonji/s1000d-verify-mcp`
- XSD 検証には、利用者が s1000d.org から入手した配布物の展開先を `S1000D_SPEC_DIR` に指定する必要がある（Claude Desktop の plugin では環境変数が渡らないので `~/.config/s1000d-stack/config.json` の `specDir` に書く）

## 手順

### Phase 0 — 入力と出力先の確認

次を利用者に確かめる。推測で埋めない。

| 項目 | 引数 | 確かめること |
|---|---|---|
| マニュアルの JSON | `manualPath` | 絶対パス。相対の素材パス（`Asset.source.path`）は、このファイルの場所から解決される |
| 出力先の CSDB | `csdbDirs` | 1 件目が書き込み先。**無いと、参照を解決できず、書き込みもできない**（応答に `No CSDB folders` の注記が付く） |
| 機密区分 | `securityClassification` | **必須。** マニュアルに持ち主が無いので利用者に聞く。値は解釈されず、そのまま書かれる。プロジェクトの BREX が許す値は brex の `lookup_value`（`target: "securityClassification"`）で読める |
| 言語 | `language` | 省略するとマニュアルの `language`（`ja` は `ja-JP`）。地域が必要なら BCP 47 で渡す |
| 責任企業・作成者 | `responsiblePartnerCompany`、`originator` | 省略すると ICN の CAGE コード |
| 素材の置き場所 | `assetBaseDir` | 図の原本（PNG など）。無いと ICN の参照が解決できない |
| 一部だけ変換するか | `topics` | Topic の id の配列。省略すると全部（用語集の DM は全部のときだけ作られる） |

マニュアルの JSON の検査（`validateManual` 相当）は `convert_manual` が行う。この Skill は行わない。

### Phase 1 — 採番の規則を決める

**[references/allocation.md](references/allocation.md) を読み、そこにある聞き方で利用者に聞く。** モデル識別コード・システム差異コード・SNS・種類ごとの情報コード・CAGE コード・品目位置コードを決める。

- **情報コードの既定は無い。** 省略は入力の誤り（`input-error`）として返る。意味はプロジェクトが決めるので、利用者が登録したコードを聞く
- 試作なら券売機の見本の値を示してよい。ただし「試作用の値で意味を持たない。実納品では登録したコードを使う」と必ず添える
- 渡す情報コードの種類は、マニュアルの Topic の種類で決まる（allocation.md の対応表）

### Phase 2 — 台帳を決める

`ledger` は、sourceId（Topic や画面の id）から DMC への対応と、素材から ICN への対応。writer は状態を持たないので、保管は呼び出す側（この Skill）の仕事になる。

- 前回の変換の `ledger` があれば `allocation.ledger` に渡す。**渡さないと、同じ Topic でも分解コードが進んで別の DMC になる**（既存の DM と衝突したように見える）
- 前回の ICN の続きは `allocation.icn.next` に、前回の応答の `icnNext` を渡す
- 初回は空でよい。変換後に、応答の `ledger` を `<manual>.ledger.json`（マニュアルの隣）に保存するよう提案する（書くのは利用者の確認の後）

### Phase 3 — 書かずに変換する

`convert_manual` を `write: false`、`includeXml: false` で呼ぶ。引数は Phase 0〜2 で決めたもの。`csdbDirs` は渡す。

`input-error` が返ったら `error.path` と `error.detail` を利用者に伝え、直して呼び直す（例: `/allocation/rules/infoCodeByKind/fault` は故障探求の情報コードが無い）。何も書かれていない。

### Phase 4 — 結果を読んで報告する

応答のこれらを読む。

| 項目 | 読むこと |
|---|---|
| `dataModules[]` | `role`（topic / screen / glossary）、`dmCode`、`fileName`、`verification`、`fixes[]`、`byConfidence` |
| `publicationModule` | PM の `pmCode`、`verification` |
| `summary` | `schema` / `brex` / `references` の状態の集計、`remainingFixes`、`failedToBuild` |
| `assets[]` | 素材と ICN の対応。**`sourceExists: false` の素材は ICN の参照が解決できない** |
| `notRepresented[]` | 表せなかったもの。**[references/not-represented.md](references/not-represented.md) に従って利用者に伝える** |
| `_notes` | 補った値（地域、`すべて` の適用性、未知の形式の NOTATION など）。省かず確かめる |

報告は [references/report.md](references/report.md) の雛形に沿う。ここでの報告は途中経過で、この時点では何も書いていない。

検証状態の書き方（原則 5）:

| 応答 | 書くこと | 書いてはいけないこと |
|---|---|---|
| `schema.state: valid` | 「XSD の検証で違反なし」 | 「S1000D 準拠」 |
| `schema.state: schema-not-found` | 「XSD が見つからず、XSD 検証は行われていない」。`S1000D_SPEC_DIR` の設定を案内 | 「合格」「問題なし」 |
| `brex.state: passed` | 「機械で検査できる BREX のルールで違反なし。`manualChecks` は人の確認が要る」 | 「BREX 準拠」 |
| `brex.state: incomplete` / `not-run` | 「BREX は未確認」 | 「合格」 |
| `references.state: unresolved` | 内訳を確かめて書く（次の注意） | 一括で「参照切れ」「問題なし」 |

> [!NOTE]
> `references.state` が `unresolved` でも、原因が仕様書の章への `dmRef`（モデル識別コード `S1000D`）だけのことがある。`fixes[]` が空で `omitted.specificationReferences` が `unresolvedCount` と同じなら、残りは仕様書への参照だけで、CSDB に無いのが通常。内訳に迷ったら verify の `check_references` で確かめ、「仕様書への参照のみ未解決」と書く。

### Phase 5 — 残った修正を片付ける

`fixes[]` の各項目を `confidence` で分ける。

| `confidence` | 意味 | 扱い |
|---|---|---|
| `exact` | 候補が 1 つで値も決まっている | サーバーが自動で適用済み（`appliedFixes[]`）。報告に載せるだけ |
| `several` | 候補が複数 | 各候補の `value` と `description`、`source` を示して、利用者に選んでもらう |
| `none` | 候補を決められない | `problem` と `note` をそのまま伝えて、利用者に聞く。値を作らない |

候補の `valueForm` が `range` / `pattern` のときは値ではなく**制約**。範囲や正規表現を示し、合う値を利用者が決める。

**素材が無いことによる `none`（ICN のファイルが CSDB に無い）は、修正の問題ではない。** 原本を `assetBaseDir` に置いて Phase 3 からやり直すか、素材なしで先へ進むかを利用者に聞く。進むなら、報告の「未解決」に残す。

選んでもらった修正は `apply_fixes` で適用する。

- `selections[]` に、`fixes[]` の `location` と**候補 1 つ**（`action`、`target`、`value`）を写す。候補の外の値や、範囲に合わない値は `rejected[]` に理由付きで返る
- `apply_fixes` は**既存の文書**（`path` か `xml`）を受ける。まだ書いていない DM は、`topics` をその Topic だけにして `includeXml: true` で変換し直し、`xml` を渡す。書いた後なら `path` を渡す
- `action: "provide"` の候補は、文書ではなく製品の値（`product`）を与えるもの
- 適用後に再検証される。`verification` を読み直して報告する

### Phase 6 — 書き込む

1. 書き込み先（`csdbDirs` の 1 件目）、書く DM と PM の数、素材のコピーの有無を利用者に示し、**確認を得る**
2. `convert_manual` を `write: true` で呼ぶ。`overwrite` は既定の `false` のまま。`write-conflict` が返ったら、同じ名前のファイルがあるので、上書きするかを利用者に聞く
3. XSD に違反する DM は書かれない（`schema-invalid`）。`allowInvalid: true` は、利用者が明示したときだけ
4. 各 DM の `written`（書いたパス）と `summary.written`（件数）を控える
5. 応答の `ledger` と `icnNext` を保存する提案をする（Phase 2）

### Phase 7 — CSDB 全体を確かめる

`verify_csdb` を `csdbDirs` で呼ぶ。`totals`（`schemaValid`、`schemaInvalid`、`schemaNotFound`、`brexPassed`、`brexViolations`、`brexIncomplete`、`referencesUnresolved`、`unreadable`）を、Phase 4 の書き方で報告する。

- 件数が多いときは `offset` / `limit` で分けて読む。全件の詳細は `reportPath` で JSON に書き出せる
- 違反があれば、`suggest_fixes` でその DM の候補を得て Phase 5 に戻る
- reader があれば、`read_data_module`（`section: "outline"`）で変換結果を構造として見せられる。機密区分が付いているので、本文を引用する前に利用者に確かめる

### Phase 8 — Conversion Report

[references/report.md](references/report.md) の雛形で最終報告を出す。未接続の任意 MCP の項目には「未実施（ツール未接続）」と書く。

## やらないこと

- XML を書く、または DMC・ICN・BREX の参照を自分で作ること
- 情報コードの値を、利用者に聞かずに決めること
- 候補の外の値で修正すること
- `schema-not-found` と `not-run` を合格と書くこと
- `valid` と `passed` を「S1000D に準拠している」と書くこと
- ホットスポットの座標など、`notRepresented` に出たものを「変換できた」と書くこと
- ACT・CCT・PCT を作ること（writer は作らない。適用性は、マニュアルの `scopes` から DM ごとの表示文を作るだけ）
