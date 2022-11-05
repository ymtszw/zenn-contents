---
title: "NimbleParsecの解説とちょっとした利用例"
emoji: "💧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Elixir
  - Parsec
  - Parser
published: true
publication_name: siiibo_tech
marp: true
theme: gaia
class: invert
paginate: true
---

<!-- _class: [lead, invert] -->

## NimbleParsecとちょっとした利用例

By [ymtszw](https://zenn.dev/ymtszw) @ tokyo.ex#21 (2022/11/05) [^1]

[^1]: この発表資料はZennで公開しつつ、[Marp](https://marp.app/)スライドとして作成しているので水平線がいっぱい入っています

---

## NimbleParsecとは

- [dashbitco/nimble_parsec](https://github.com/dashbitco/nimble_parsec)
- 元々Plataformatec（Joséの所属していたRuby/Elixir開発コンサル。Nubankに買収された）で開発され、新会社のDashbitに移管されたパーサーコンビネータライブラリ
- "Nimble"シリーズライブラリの一つ

> `NimbleParsec` is a simple and fast library for text-based parser combinators.

---

## パーサーコンビネータ？

- [Parser combinator @ Wikipedia](https://en.wikipedia.org/wiki/Parser_combinator)
- "Parser"というデータ型を入出力のインターフェイスとし、それらを関数で組み合わせて目的とする文書の解釈器を組み立てる手法
  - ないしその関数群のこともいう
- ゆえに**パーサーコンビネータ**

---

## "Parser"

- Parserの正体は**関数**
- 文字列を受け取り、いずれかを返す:
  - 成功した場合は読み取り結果であるデータと、読み取り完了した位置
  - 失敗した場合はその事実、ライブラリによってはより具体的な失敗内容

---

## パーサーコンビネータの例

- [HaskellのParsec](https://wiki.haskell.org/Parsec)
- [Rustのnom](https://github.com/Geal/nom)
- [Elmのparser](https://github.com/elm/parser)

---

## Re: NimbleParsec

- 名前の通りParsecのElixir版
  - [HexDocs]
- すべてが**バイナリパターンマッチを使った関数**にコンパイルされるので、Erlang VMの様々な最適化の恩恵に与れる
- 加えて記述したいmoduleで`import`するだけでよく、`use`を利用していないので、**実行時はライブラリへの依存が一切ない**

[HexDocs]: https://hexdocs.pm/nimble_parsec/NimbleParsec.html

---

## コード例（READMEより）

<!-- style: pre{font-size: 60%;} -->

```elixir
defmodule MyParser do
  import NimbleParsec

  date =
    integer(4)
    |> ignore(string("-"))
    |> integer(2)
    |> ignore(string("-"))
    |> integer(2)

  time =
    integer(2)
    |> ignore(string(":"))
    |> integer(2)
    |> ignore(string(":"))
    |> integer(2)
    |> optional(string("Z"))

  defparsec :datetime, date |> ignore(string("T")) |> concat(time), debug: true
end

MyParser.datetime("2010-04-17T14:12:34Z")
#=> {:ok, [2010, 4, 17, 14, 12, 34, "Z"], "", %{}, 1, 21}
```

---

## コード例の解説1: 基本

- 最終的に、`defparsec/3`マクロで、完成したパーサに実際に文書を流し込んで使うためのエントリポイント関数が生成される
  - コード例では`datetime/1`関数
- 組み合わせるパーサ群はmoduleトップレベルに記述していく
  - `date`, `time`
- 文字列を「よくあるデータ型」として解釈するパーサはビルトイン
  - [`integer/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#integer/2), [`string/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#string/2)

---

## コード例の解説2: 読み捨てとoptional

- 解釈した結果を「読み捨てたい」場合、パーサを[`ignore/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#ignore/2)で囲む
  - パーサーコンビネータでは非常によくある要求
    - 「**人間の目にはあったほうがいいが、コンピュータには不要なもの**」はたいていignoreすることになる
  - コード例ではISO8061形式の文字列を解釈しているので、`-`や`:`などの区切り記号はすべてignore対象
- 「あってもなくてもいい」記述は[`optional/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#optional/2)で囲む

---

## コード例の解説3: 組み合わせ

- パーサの組み合わせはElixirらしくpipelineで！
- このとき、すべてのビルトインパーサや`ignore/2`などのユーティリティは、
  - 単体で呼び出されると、対象文字列を最初から読み始める（暗黙の[`empty/0`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#empty/0)パーサが第1引数のデフォルト）
  - Pipelineで呼び出されると、直前のパーサの読み取り完了位置の次の文字から開始する

---

## コード例の解説3の補足: そもそもパーサの挙動

- パーサーコンビネータではパーサ（正体は関数）をパズルのように組み立てていく
- 部品であるパーサは「**文書の頭（あるいは直前のパーサが読み取り完了した位置の次）から文字列を読んでいき、解釈失敗するまで突き進む**」挙動
- したがって、文書の初めから終わりまで完全にルールに則っていれば成功するし、**どこか1箇所でもルール違反があると全体として失敗することになる**

---

## コード例の解説3の補足: OR構造

- とはいえ、文書記述のルール（目的とする文書の構文）には**分岐のような構造**や**曖昧さ**がよくある
  - 例えばMarkdownでは、見出し（`#`, `##`, ...）で書き始めてもいいが、いきなり本文を書き始めてもいい
  - 何なら他にもありとあらゆる有効な記法を好き勝手な順序で書いていい
- 「あるルールでは解釈できず失敗となるが、**その場合いきなり全体として失敗させず、同じ位置で別のルールを試したい**」という"OR"構造のサポートが必要

---

## コード例の解説3の補足: choice

- これは[`choice/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#choice/2)で実現できる

```elixir
# Markdown parserっぽいもの
choice([
  heading,
  unordered_list,
  ordered_list,
  blockquote,
  fenced_code_block,
  paragraph,
  ...
])
```

---

## コード例の解説4: 結果

- 解釈した結果はリストに積まれていく
- 複数のパーサを単純に連結する（結果リストも連結する）のは[`concat/2`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#concat/2)
- 最終的に以下のようなspecの関数が定義される

```elixir
  @spec run(binary(), keyword()) ::
          {:ok, [term()], rest, context, line, byte_offset}
          | {:error, reason, rest, context, line, byte_offset}
        when line: {pos_integer(), byte_offset},
             byte_offset: pos_integer(),
             rest: binary(),
             reason: String.t(),
             context: map()
```

---

## Wrap-up

- 一応、これくらい理解すればちょっとした文書のパーサをNimbleParsecで書き始められる！
- 比較的構文の大きな文書でも、自分の目的とする内容に限った部分的なサポートだけが必要なのであれば、NimbleParsecを使って自作するのはアリな選択肢
  - 正規表現よりも遥かにreadableなコードで、十分パフォーマンスの良いパーサが手に入る
- 例えばElixirであまり使われていない and/or ライブラリがないが、自分では使いたい文書形式など

---

## Siiiboでの利用例

- Siiiboでは、DBスキーマを設計するにあたり、**PlantUMLでまずER図を書いてコミットし、コードレビューする**というプロセスを採っている
  - 当初は画像として生成して見ることもあったが、開発が進んでスキーマが大規模化した今は画像生成はほぼしておらず、設計・レビューツールとしての位置づけに落ち着いた[^2]
- すると当然、「**PlantUMLで書いた内容からEctoのマイグレーションスクリプトや`Ecto.Schema` model moduleを自動生成したい**」という欲求が生じる

[^2]: もうSQLのDDL直接書いたほうが早くね？という説はある。実際Ectoマイグレーションの結果としての最新DDLもコミットしており、開発・テスト環境の高速立ち上げに利用しているし、チームも脱出口としてSQLを直接扱うことにそれなりに慣れている。レガシー味の多少あるプロセス

---

## PlantUML Parser

- そこで**ERDの文法のみ対応したPlantUML Parser**を自作した🎉
- Closed sourceなので、さわりだけ紹介すると以下のような構造:

```elixir
  uml =
    startuml
    |> repeat(
      choice([
        ignore(line_comment),
        title,
        rendering_option,
        legend_block,
        class_or_entity_block,
        association,
        ignore(unknown_command),
        ignore(newlines)
      ])
    )
    |> ignore(enduml)
```

---

## PlantUML Parser雑紹介

- 解説した`ignore/2`や`choice/2`がふんだんに使われる
- [`repeat/3`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#repeat/3)は読んで字の如くパーサを繰り返し適用するユーティリティだが、正規表現で言う`*`（zero or more）挙動なので、色々注意が必要
- 今となってはもうちょい良い書き方はたくさんありそう……[^3]

[^3]: 実装はもう3年くらい前

---

## おまけ解説: tag

- NimbleParsecでは解釈結果をリストに積んでいくので、ビルトインパーサを組み合わせたそのままでは成果物が扱いづらい
  - `[1, "something", 333_402, ...]`のような「要素の意味がわからないリスト」になってしまう
- そこで、パーサの部分部分としては、[`tag/3`](https://hexdocs.pm/nimble_parsec/NimbleParsec.html#tag/3)関数で結果リストを`{:tag, [...]}`の形式にタグ付けすることが多い

```elixir
# パース結果をこんなふうにできるので、後処理しやすい
[
  {:preamble, [1]},
  {:body, ["something"]},
  {:char_count, [333_402]},
  ...
]
```

---

## まとめ

Elixirで「パーサ書きたいんだが？」と思ったらNimbleParsecを思い出そう！

- [著者紹介](https://docs.google.com/presentation/d/e/2PACX-1vRuIA2ocDafLRJUn6nWScZmOq6YwpqXba7x5RG72yzT3X7FB-JcET33QMGsBidHsAdbnVF9KYCOa00R/embed?start=false&loop=false&delayms=3000&slide=id.g155be708576_0_55)（前回のtokyo.ex#20のスライド）
