# Changelog

## 0.1.5 (2026-11-24)

### 変更

- `references/not-represented.md` に、`notRepresented[]` の `where` は最初の 5 件であることと、入らなかった数（`whereOmitted`。s1000d-writer-mcp 0.9.0 から）を「ほかに N 件」と伝えることを書きました。Grok Build の試用の所見 F-12 への対応です。手順は変わりません。

## 0.1.4 (2026-11-24)

### 変更

- 報告の「使用ツール」に、MCP ごとに応答の最後の `_server` から `<name> <version>（core <core>）` を書き写すことにしました。MCP の応答に `_server` が付く版（hub の DECISIONS の D-38、Issue [shuji-bonji/s1000d-stack#12](https://github.com/shuji-bonji/s1000d-stack/issues/12)）に合わせた変更です。`_server` の無い応答は「版の項目なし（D-38 より前の版）」と書きます。版の解釈と、npm の公開版との比較はしません。

## 0.1.3 (2026-11-24)

### 変更

- 参照の状態の注意を書き直しました。s1000d-core 0.16.0 の修正で、`verification.references` に `specificationReferences` が付く版（s1000d-verify-mcp 0.9.0・s1000d-writer-mcp 0.7.0 以降）では、仕様書への `dmRef` だけが残っても `references.state` が `resolved` になります。その版での書き方と、付かない古い版での読み替え（`fixes[]` が空で `omitted.specificationReferences` が `unresolvedCount` と同じ）の両方を書きました。

## 0.1.2 (2026-11-17)

### 修正

- 残った修正を片付ける手順に、候補に `position` があるときは `selections[]` にも写すことを書きました。`position` を受け取るのは s1000d-writer-mcp 0.4.0 以降で、0.3.1 まではその候補を適用できません。
- 前提 MCP の s1000d-writer-mcp の最低版を 0.4.0 に上げました。0.3.1 までは、条件付きで禁止する BREX ルールの違反に「この要素を取り除く」という候補を `exact` で返し、自動で適用することがありました（s1000d-core 0.12.0 で修正）。

## 0.1.1 (2026-10-29)

hub の E2E テスト（`e2e/`）で見つかった、Skill の記述と MCP の挙動の食い違いを直しました。手順の骨組みは変えていません。

### 修正

- 前提 MCP の最低版を verify 0.6.1・reader 0.7.1 に上げました。Phase 7 に、書いた直後のファイルが見えること（0.6.1 / 0.7.1 以降）と、それより古い版では見えないことがあり、その場合は MCP を起動し直すことを書きました。
- Phase 7 に、spec モードで `totals` を読むときの注意を書きました。`csdbDirs` に書き込み先だけを渡すと既定 BREX への参照が未解決として数えられることがあること（core 0.8.1 以降は解決済み）、`verify_csdb` に既定 BREX のフォルダーまで渡すと既定 BREX 自身も件数に入ること。
- `references/allocation.md` の `brexDmRef` に、既定 BREX が見つからないときは `brexDmRef` ごと省かれ BREX が照合されないことを書きました。

## 0.1.0 (未リリース)

- 初版。
