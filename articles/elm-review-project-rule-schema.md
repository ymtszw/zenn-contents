---
title: "Elmのlintツールelm-reviewのProjectRuleSchemaを使ってボイラープレート生成する"
emoji: "✏"
type: "tech"
topics:
  - elm
  - linter
  - codegen
published: true
publication_name: siiibo_tech
marp: true
theme: gaia
class: invert
paginate: true
---

<!-- _class: [lead, invert] -->

## Elmのlintツールelm-reviewのProjectRuleSchemaを使ってボイラープレート生成する

By [ymtszw](https://zenn.dev/ymtszw)
@[Elm-jp Online #3](https://elm-jp.connpass.com/event/288224/) (2023/07/22) [^1]

[^1]: この発表資料はZennで公開しつつ、[Marp](https://marp.app/)スライドとして作成しているので水平線がいっぱい入っています

---

## [jfmengels/elm-review](https://github.com/jfmengels/elm-review)

- 一言でいうと「Elmでルールを書けるElmのlintツール」
- Previous studies:
  - [elm-reviewを使ってモジュールの依存関係に制限を付ける](https://qiita.com/misoton665/items/be113badc7dfeb2ee4f1) by [misoton665](https://twitter.com/misoton665)
  - [elm-review悪用でコード自動生成](https://zenn.dev/miyamoen/scraps/55c5a34398cf3f) by [miyamo](https://twitter.com/miyamo_madoka)

---

## コード生成ツールとして

- 今日紹介するのもコード生成ツールとしての活用
- 特に[elm-review悪用でコード自動生成](https://zenn.dev/miyamoen/scraps/55c5a34398cf3f)を改良した記録です

---

## elm-reviewの構造

- [`Rule`](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Rule#Rule)型のデータでlintルールを表現する
  - かいつまんで言うと「**プロジェクトまたはモジュールを評価する関数**」
- `Rule`を実行した結果、[`Error`](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Rule#Error)が1つ以上あればlint警告となる
- `Error`は[`Fix`](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Fix#Fix)を提供しても良い

---

## [`ProjectRuleSchema`](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Rule#ProjectRuleSchema)と[`ModuleRuleSchema`](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Rule#ModuleRuleSchema)

- ほぼ読んで字のごとく、
  - プロジェクト全体を評価するルールか、
  - モジュール（Elmの場合＝ファイル）を単体で評価するルールか

---

## ボイラープレート生成 (1)

- Elmの文脈でよく言われる「ボイラープレート」
  - Client-side routingのあるアプリで、個別のページモジュールを`Main`モジュールに結線する
  - `Model`, `Msg`, `init`, `update`, `subscription`, (`view`)からなる、いわゆる「コンポーネント」的モジュール同士を結線する
- といった「**モジュール間の結線**」を指すことが多い（体感）

→つまり**複数のモジュールの情報が必要となる**。したがって、

---

## ボイラープレート生成 (2)

- `ProjectRuleSchema`を使ってプロジェクト全体から情報を集め、
- **未結線のモジュールが見つかったら**`Error`とし、
- `Fix`を提供して結線する実装を生成しよう！

…となる。 [^2]

[^2]: elm-spaなどの「フレームワーク」は、そういった結線機構をdevサーバが提供している

---

## 実際のコード例

以下、Siiibo証券で実際に使っているコードをもとに紹介していきます。 [^3]
（elm-review 2.13.1現在）

```elm
rule : Rule
rule =
    Rule.newProjectRuleSchema "GenerateBoilerplate" initialContext
        |> Rule.withModuleVisitor moduleVisitor
        |> Rule.withModuleContextUsingContextCreator contextCreator
        |> Rule.withFinalProjectEvaluation finalEvaluation
        |> Rule.providesFixesForProjectRule
        |> Rule.fromProjectRuleSchema
```

[^3]: 一部省略しているので動かすためにはある程度手を入れる必要あり

---

## `ProjectRuleSchema`の構造 (1)

```elm
    Rule.newProjectRuleSchema "GenerateBoilerplate" initialContext
```

- `ProjectRuleSchema`を使うよ、という宣言と`Rule`の命名。
- `initialContext`では、プロジェクト全体から必要な情報を集めて格納するデータ置き場(`ProjectContext`)を初期化する

```elm
type State
    = Visiting
    | Visited { key : ModuleKey, implementedPages : PageFileDict, endOfDeclarationsRow : Int }

initialContext =
    { state = Visiting, currentPages = Dict.empty }
```

---

## `ProjectRuleSchema`の構造 (2)

```elm
        |> Rule.withModuleVisitor moduleVisitor
```

- プロジェクト内の個別のモジュール（ファイル）を評価するのが`moduleVisitor`

```elm
moduleVisitor =
    Rule.withModuleDefinitionVisitor accumulateCurrentPageFile
        >> Rule.withImportVisitor accumulatePageFileFromPreviousImport
        >> Rule.withDeclarationListVisitor annotatePageFilesFromPreviousImplementation
```

- 内部的には`ModuleRuleSchema`が使われており、`withXXXXXXVisitor`系関数はそこに評価器を加えていく（Builderパターン）

---

## `ProjectRuleSchema`の構造 (3)

```elm
        |> Rule.withModuleContextUsingContextCreator contextCreator
```

- `moduleVisitor`が利用する`ModuleContext`の初期化と、`moduleVisitor`が評価されたあとに`ProjectContext`に結果を反映する処理を定義

```elm
contextCreator =
    { fromProjectToModule = Rule.initContextCreator fromProjectToModule |> Rule.withModuleKey |> Rule.withModuleName
    , fromModuleToProject = Rule.initContextCreator fromModuleToProject
    , foldProjectContexts = foldProjectContexts
    }
```

---

## ハマりどころ

- このあたりが`ProjectRuleSchema`の難しいところ。**全体構造がデカい**
- ドキュメントに`ProjectRuleSchema`で[どのようにプロジェクトが走査されていくか説明](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/Review-Rule#newProjectRuleSchema)があるので、困ったらそこに立ち返ろう
- 逆に言うと、走査の仕組みを初めに大まかに把握しておかないと、デカい型定義リストを目の前にしてハマることになる

---

## `moduleVisitor`の中身

- [elm-syntax](https://package.elm-lang.org/packages/stil4m/elm-syntax/latest/)で各モジュールのソースコードがASTとして評価可能な形で関数に渡ってくる
- 関数記述自体はそれに対する愚直なパターンマッチで実装します
- 詳細は割愛（発表時には投影するかも）

---

## `fromModuleToProject`と`foldProjectContexts`

- ２つの関数で、`moduleVisitor`が各モジュールを評価した結果を`ProjectContext`に蓄積する
- （恐らく）並列処理を可能にするため、プロジェクト内のファイルは**順不同で`moduleVisitor`に評価される**という前提がある
- 評価結果を順不同に`ProjectContext`に合流させて支障なきよう、
  - `foldProjectContexts`は順番に依存しないよう実装
  - `ProjectContext`自体、`Dict`や`Set`などの挿入順序に依存しないデータ構造を活用

---

## `ProjectRuleSchema`の構造 (4)

```elm
        |> Rule.withFinalProjectEvaluation finalEvaluation
```

- すべてのプロジェクト内モジュールを評価し終わり、最終評価に必要な情報を`ProjectContext`に蓄積し終わったあと、`finalEvaluation`で`Error`の有無を判定する
- ここでやりたいボイラープレート生成なら、
  - `ProjectContext`の`currentPages`に現在のページモジュールの一覧が蓄積されているので、
  - それを元に必要なコードを`elm-syntax`のAPIを使って書き出すことになる

---

<!-- _class: [lead, invert] -->

## Tips & Tricks

---

### 漸進的なFix

- 実はここまでの内容は[elm-review悪用でコード自動生成](https://zenn.dev/miyamoen/scraps/55c5a34398cf3f)をちょっと詳しくしただけ
- 当時は**毎回のfix時に必ず生成先ファイル全体を書き換えていた**

![w:800 以前の実装](/images/previous-generator.png)

---

- １年くらいこの実装だったが、**自動生成をdevサーバで継続的に実行したくなった**
- すると、ファイルを一旦空にして全体を書き換えるのは一瞬コンパイルが通らないコードになって不都合
- ということで必要な箇所だけ書き換える・書き換える必要がなければno-op、という**漸進的なFixにしたい**

---

### 直前の実装から情報を集める

- `ProjectContext`にあった`state`がここで意味を持つ
- `state = Visited { ... }`状態になった場合、ペイロードの`implementedPages`に**評価直前のファイルで実装済みだった関数情報**が入っている仕組み

---

### `finalEvaluation`で"diffをとる"

- すると`currentPages`と`implementedPages`で差分があるか、"diffをとる"実装が可能になる
- Diffが何らか存在したら（＝ボイラープレートに更新が必要になったら）`Error`としてfixを提供、そうでなければ何もしない
- （時間あったら投影でコード紹介）

---

### フォーマットされたコードの生成

- `finalEvaluation`でfixを提供する際、[elm-syntax-dslのElm.Pretty](https://package.elm-lang.org/packages/the-sett/elm-syntax-dsl/latest/Elm.Pretty)が活用できる
- その名の通りprettyな（＝elm-format済みの）コードが生成できる

---

### elm-reviewによる差分検知の仕組み

- 実はフォーマット済みのコードをfixで生成するのには意味がある
- elm-review CLI自身も内部的にelm-formatを呼び出すようになっており、fixが提供したコードはelm-reviewが最終的に整形する
- 2.13.1現在、「Ruleのfixが提供したコード」と「それをelm-reviewが最終的に整形したコード」に差分があると、**elm-review CLIは「fixあり」と報告し続けてしまう**
- Fix時点で整形済みのコードを吐くようにすればきれいにfixが完了する、という理屈

---

<!-- _class: [lead, invert] -->

## elm-review 2.13.1でできないこと

---

### 前回実行との差分検知

- [直前の実装から情報を集める](#直前の実装から情報を集める)でやったようなことは、多くのコンパイラやツールでは「**前回実行時のコンテキスト情報を記録ファイルに書き出しておき、次回実行時に読み出す**」という形で実現されることが多い
  - いわゆるmanifest（目録）ファイル
- この機構が今のelm-reviewにはないので、実際のソースコードから情報を再構成した、という事情
  - とはいえmanifestファイルも万能ではなくて、例えば別のメンバーがコード生成をしたときのmanifestが手元マシンになければ、結局実装からの再構成は必要

---

### 任意ファイルの読み出し

- manifestファイル機構を自前で実装できないのはこれができないからでもある
- 今はREADME.mdファイル、elm.jsonファイル、Elmファイルにしかvisitorを定義できない
- （無理やりやるなら、README.mdに何らかデータを仕込めば悪用できそうではあるが）

---

### ファイルの新規作成

- そもそもelm-reviewはあくまでlinterが出自
- なので既存ファイルを評価して警告・fixすることはできても、全く新しいファイルを作り出すことはできない
- プロジェクトルールでコード生成する場面ではほしいこともあるのだが…
- それもあってか、純粋な「**Elmで書けるElmのコード生成ツール**」としては最近[mdgriffith/elm-codegen](https://github.com/mdgriffith/elm-codegen)も出てきてます

---

## あとがき

- Siiibo証券のフロントエンド開発で日常的に使っているelm-reviewの`ProjectRuleSchema`によるコード生成を紹介した
- 表現力が高いので複雑ではあるものの、一度分かればいろいろ応用できます✌
- [著者紹介](https://docs.google.com/presentation/d/e/2PACX-1vRuIA2ocDafLRJUn6nWScZmOq6YwpqXba7x5RG72yzT3X7FB-JcET33QMGsBidHsAdbnVF9KYCOa00R/embed?start=false&loop=false&delayms=3000&slide=id.g155be708576_0_55)（tokyo.ex#20のスライド）

---
