# 採番の規則の聞き方

`convert_manual` の `allocation` を、利用者に聞いて決める。DMC・ICN・PMC は writer が付けるので、この Skill が値を作ることはない。聞くのは「規則」と「前回の台帳」だけ。

## 聞く項目

| 引数 | 内容 | 聞き方 |
|---|---|---|
| `modelIdentCode` | モデル識別コード（製品・システムの識別。英数字 2〜14 文字） | 「プロジェクトで登録済みのモデル識別コードは何か」 |
| `systemDiffCode` | システム差異コード（英数字 1〜4 文字） | 「製品に派生がある場合、どの差異コードか。無ければプロジェクトの既定の値」 |
| `rules.sns` | SNS（`systemCode`、`subSystemCode`、`subSubSystemCode`、`assyCode`） | 「このマニュアルが扱う機器は、プロジェクトの SNS のどこに当たるか」。BREX が SNS を定義していれば、brex の `lookup_sns` で意味を確かめられる |
| `rules.infoCodeByKind` | 種類ごとの情報コードと変形（`infoCode`、`infoCodeVariant`） | 下の「情報コード」 |
| `rules.itemLocationCode` | 品目位置コード（英数字 1 文字） | 「プロジェクトの規則の値は何か」 |
| `icn.cageCode` | ICN に入る CAGE コード（英数字 5 文字） | 「ICN を発行する企業の CAGE コード」。`responsiblePartnerCompany` と `originator` を省くと、この値が入る |
| `icn.next` | 次に使う ICN の連番 | 初回は `1`。続きは前回の応答の `icnNext` |
| `ledger` | 前回の台帳 | 下の「台帳」 |
| `brexDmRef` | 従う BREX の DMC | 省略すると、`S1000D_SPEC_DIR` にある既定 BREX。既定 BREX が見つからないときは `brexDmRef` ごと省かれ、BREX は照合されない（`brex.state: not-run`）。プロジェクト専用の BREX があれば、その DMC を聞く |

`rules.disassyCode` を指定すると分解コードを連番にしない（衝突は入力の誤りとして返る）。省略した既定のままでよい。

## 情報コード

> [!IMPORTANT]
> **情報コードの既定は無い。** 省略は `input-error`（`/allocation/rules/infoCodeByKind/<種類>`）で返る。情報コードの意味はプロジェクトが決めるので、**必ず利用者に聞く**。この Skill が「だいたいこの値」と推測して渡してはならない。

渡す必要がある種類は、変換するマニュアルの Topic の種類で決まる。変換する Topic（`topics` で絞ったときはその Topic）にある種類の分だけ聞く。

| マニュアルの内容 | 作られる DM の種類 | 必要な `infoCodeByKind` |
|---|---|---|
| `description` の Topic、用語集 | `descript` | `descript` |
| `procedure` の Topic | `proced` | `proced` |
| `screenFlow` の Topic | プロセス 1 件と画面ごとの手順 | `process` と `proced` |
| `troubleshooting` の Topic | `fault` | `fault` |
| パブリケーションモジュール | `pm` | 不要（PMC は情報コードを持たない） |

聞き方の例:

> このマニュアルには説明・手順・画面遷移・故障探求の Topic があるので、descript、proced、process、fault の 4 つの情報コード（と変形）が要ります。プロジェクトで登録済みのコードを教えてください。

### 試作の見本

利用者が「試作で動かしたい」と言ったときだけ、次の値を見本として示してよい（券売機の例で使っている）。

| 種類 | `infoCode` | `infoCodeVariant` |
|---|---|---|
| descript | `040` | `A` |
| proced | `520` | `A` |
| process | `130` | `A` |
| fault | `411` | `A` |

> [!WARNING]
> 上の値は**試作用で意味を持たない**。実納品では、プロジェクトで登録したコードを使う。見本を示すときは、この一文を必ず添える。

試作の他の値（券売機の例）: モデル識別コード `KENBAIKI`、システム差異コード `A`、SNS は `10` / `2` / `0` / `00`、品目位置コード `D`、CAGE コード `K0001`。これも試作用。

## 台帳（`ledger`）

台帳は、sourceId から DMC・PMC への対応と、素材の id から ICN への対応の写し。writer は保管しないので、呼び出す側で持つ。

- **保存先の提案**: マニュアルの JSON の隣の `<manual>.ledger.json`（例: `kenbaiki.manual.ledger.json`）。応答の `ledger` をそのまま JSON で保存する。保存は、利用者に確認してから
- **次の変換で渡す**: 保存した内容を `allocation.ledger` に、応答の `icnNext` を `allocation.icn.next` に渡す
- **渡さないと何が起きるか**: 台帳が無いと、writer は CSDB 内の既存の DM から次の番号を数える。同じ SNS・情報コードの DM が既にあれば、**同じ Topic でも分解コードが進んで別の DMC になる**。画面の手順 DM のように同じ情報コードの DM が複数あるときは、分解コードの連番（`00`、`01`、`02`…）で区別される
- 台帳が無くて CSDB にも前回の DM が無いときは、新規として番号が振られる。初回は台帳が空で正しい

版（`issueInfo`）は別の話。CSDB に同じ DMC の DM があると、既定（`issuePolicy: "increment-issue"`）では発行番号が上がる。版を上げたくないときは `issuePolicy` を `keep` にするか、`increment-inwork` にするかを利用者に聞く。
