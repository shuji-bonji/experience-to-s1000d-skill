# `notRepresented` の読み方と伝え方

`convert_manual` の応答の `notRepresented[]` は、**マニュアルにあるが、下書きの形（データモジュール）では表せず、DM に入らなかったもの**。黙って捨てずに、`what`（何か）、`count`（件数）、`where`（マニュアルの JSON の中の位置）で返される。

利用者への伝え方の原則:

- 「変換できた」ではなく「**変換した DM には入っていない**」と書く
- 件数と `where` を示す。`where` はマニュアルの JSON の位置なので、利用者が元を見に行ける
- 失われる情報が、DM を読む人にとって何を意味するかを一言で添える。判断（直すか、受け入れるか）は利用者に委ねる

## 出るものと伝え方

`what` の文は writer が返すもの。下の表の「意味」は、利用者に説明するための言い換え。

| `what`（一部） | 意味 | 伝え方 |
|---|---|---|
| Hotspot shape and coords | ホットスポットの形と座標。DM には画面上の位置を表す要素が無い。ホットスポットの操作と遷移先は、画面の手順の文として入る | 「押す場所の座標は DM に入らない。手順の文には『何を押すか』と遷移先が入っている。クリックできる画面は、体験型マニュアルの JSON 側にだけ残る」 |
| a transition back to a screen already in the flow | すでに通った画面へ戻る遷移 | 「戻る遷移は、その画面の手順の文にだけ書かれ、プロセスの流れには入らない」 |
| Topic.summary of a non-description topic | 説明以外の Topic の要約 | 「手順・画面遷移・故障探求の Topic の要約は DM に入らない。必要なら説明 DM に書き足す」 |
| Asset.display | 閲覧用の表示コピー（軽い画像など）。DM には原本だけが入る | 「表示用のコピーは DM に入らない。原本が ICN になる」 |
| scopeRefs on a block, step or screen below the topic | Topic より下（ブロック・手順・画面）の適用範囲。適用性は DM 単位 | 「適用範囲の絞り込みは DM 単位までで、手順ごとの適用性は入らない。手順ごとに分けたいなら Topic を分ける」 |
| Step.next | 手順の飛び先の指定。DM では手順の並びが順序になる | 「手順の飛び先は入らない。DM では並びの順に進む」 |
| Step.targetRef | 手順が指す対象 | 「手順の対象の指定は入らない」 |
| question / answers on a step of a procedure | 手順の中の問いと答え。分岐は故障探求（troubleshooting）で表す | 「手順の中の分岐は入らない。分岐が要るなら故障探求の Topic にする」 |
| Step.next on a troubleshooting step | 故障探求の手順の飛び先（答えで表す） | 「飛び先は答えの `next` で表す」 |
| Transition.condition on an answer | 答えに付いた条件。答えの文が条件になる | 「答えの条件は入らない。答えの文が条件になる」 |
| transition to something that is not a screen of this topic | 同じ Topic の画面ではない遷移先 | 「遷移先が同じ Topic に無いので、リンクを作れなかった」 |
| reference to an asset that is not in Manual.assets | 素材の一覧に無い素材への参照 | 「参照先の素材がマニュアルに登録されていない。JSON を直す」 |
| Manual.description | マニュアル全体の説明。パブリケーションモジュールには説明が無い | 「全体の説明は PM に入らない。説明 DM に書くか、受け入れる」 |

上に無い `what` が出たら、`what` と `where` をそのまま利用者に示す。意味を推測して書かない。

## 同じ応答の `_notes` で出る、表せなかったことに近い注記

`notRepresented` ではないが、同じように利用者へ伝える。

| `_notes` の内容 | 伝え方 |
|---|---|
| `language: "ja" has no region` | 地域が無いので `ja-JP` を使った。違うなら `language` を渡す |
| `data modules without scopeRefs get the display text` | 適用範囲の無い DM に、表示文 `すべて` を入れた（機械で判定できる条件は無い）。`defaultApplic` で変えられる |
| `no known public identifier for ".xxx"` | 図・3D の形式の公開識別子が分からず、NOTATION の宣言に仮の値を入れた。`assetFormats` で、プロジェクトが使う識別子を渡す。BREX が許す形式かは、brex の `list_notations` で確かめられる |
| `N original file(s) were not found on disk` | 素材の原本が見つからず、ICN の参照が解決できない。原本を置くか、受け入れる |

## 作られないもの

- **ACT・CCT・PCT（適用性の表）は writer が作らない。** DM の適用性は、マニュアルの `scopes` から作った表示文（と、あれば条件）で、適用性リポジトリは既存のものを前提にする。製品ごとの適用性の判定が要るなら、利用者が用意した ACT・CCT・PCT を CSDB に置く
- **DML・DDN も作らない。** 一覧は、CSDB 側の運用で作る
- **図の中身は作らない。** 素材は ICN を採番して参照を入れ、原本をコピーするだけ
