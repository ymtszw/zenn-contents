---
title: "Diffs from Elm Meetups 2019"
emoji: "💠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Elm
published: true
publication_name: siiibo_tech
marp: true
theme: gaia
class: invert
paginate: true
---

<!-- _class: [lead, invert] -->

## Diffs from Elm Meetups 2019

By [ymtszw](https://zenn.dev/ymtszw)
@[Elm Meeetup (第1回オンライン開催)](https://elm-jp.connpass.com/event/262458/) (2022/11/19) [^1]

[^1]: この発表資料はZennで公開しつつ、[Marp](https://marp.app/)スライドとして作成しているので水平線がいっぱい入っています

---

## 前回のElm-jp Meetupは…

- 2019年 (both @六本木, Fringe81 (現Unipos))
  - [Elm Meetup in Summer](https://elm-jp.connpass.com/event/140431/)
  - [Elm P1](https://elm-jp.connpass.com/event/156016/)
- その後コロナ禍となり、技術系イベント開催は減少・縮小…
  - [elm japan 2020](https://elmjapan.guupa.com/)中止😭
  - 小規模ハンズオンや勉強会・もくもく会などオンラインで

今回オンラインながらmeetup再開、ということで…

---

## 2019年後半以来のdiffを軽く紹介

- 主にElm言語・ライブラリ・ツールに関して
- グローバルコミュニティの動向とかも
- [Discourse投稿を大雑把にフィルター](https://discourse.elm-lang.org/search?expanded=true&q=after%3A2020-01-01%20min_views%3A2500%20order%3Alatest)しながら思い出しつつ書く

---

## 💠Elm, and its stability

![bg right:30% fit](/images/elm-0.19.1.png)

- Proudly Unchanged!
- しばらくはコミュニティが盛り上げ役です

---

- [Status Update - 3 Nov 2021](https://discourse.elm-lang.org/t/status-update-3-nov-2021/7870)
  - 当面の"API安定"が実質宣言されている

> If you like what you see in 0.19.1 now, that’s pretty much what Elm is going to be for a while!

- [なぜElmは0.19のままか、変化すること／しないこと](https://izumisy.work/entry/2021/11/13/181404) by [IzumiSy](https://twitter.com/sy_izumi)
- 最近Evanがゆったり取り組んでいる内容などもちょっとわかります

---

## 🌈[elm-spa](https://www.elm-spa.dev/) by [RyanNHG](https://twitter.com/rhg_dev)

- [Elm-spa: single page apps made easy](https://discourse.elm-lang.org/t/elm-spa-single-page-apps-made-easy/4950)
  - 2020/01/20
- ElmのTEAはある意味もう"普及した"概念
- とはいえ実際アプリを作るにあたっては"ボイラープレート"が多い
- それを解消しようとする"**フレームワーク**"の一つ
- File-based routing, authentication handling, shared state, zero config...

---

## 🌈[elm-land](https://elm.land/) also by [RyanNHG](https://twitter.com/rhg_dev)

- [Hello, world! | Elm Land](https://elm.land/news/hello-world.html)
  - 2022/10/04
- ちょっと先回りしてこちらを紹介

> A (not yet) production-ready framework for building Elm applications. Build your next app with confidence, step by step.

- elm-spaと同じRyanさんによるさらなる展開

---

## 📚[elm-pages (v2)](https://github.com/dillonkearns/elm-pages) by [Dillonkearns](https://twitter.com/dillontkearns)

- [Introducing elm-pages v2!](https://discourse.elm-lang.org/t/introducing-elm-pages-v2/7608)
  - 2021/08/21
- Elmによる**静的サイトジェネレーター**
  - と言いつつ動的なclient-side scriptingもElmで可能
- File-based routing, static generation, SEO made easy...
- **個人的推し！** [elm-pages・ヘッドレスCMS事始め](https://ymtszw.github.io/articles/elm-pages-and-headless-cms/)
- 現在絶賛v3準備中

---

## 🚀[Lamdera v1](https://lamdera.com/) by [supermario](https://twitter.com/realmario)

- [Lamdera: A year in review](https://discourse.elm-lang.org/t/lamdera-a-year-in-review-v1-0-0-paid-plans-carbon-mission-and-more/7584)
  - 2021/07/21
- 「サーバ側もElm appとして書いちゃえ！何なら両側を一つのアプリにしちゃえ！」「は！？」
  - 驚きのElm compiler forkなプロジェクト
- [Evergreen migration](https://dashboard.lamdera.app/docs/evergreen)によってデプロイ時に明示的・型安全に旧Modelから新Modelへの移行する手続きを定義するのが堅牢
- elm-pages v3で内部的に使っているぞ

---

## 🚀[elm-optimize-level-2](https://github.com/mdgriffith/elm-optimize-level-2) by [mdgriffith](https://twitter.com/mech_elephant)

- [Announcing Elm Optimize Level 2!](https://discourse.elm-lang.org/t/announcing-elm-optimize-level-2/6192)
  - 2020/08/20
- 次のElmバージョンではこんな最適化ができるかも！を試すコミュニティ実験場
- elm-pagesなどですでに内部的に使われている
- `elm`コマンドのdrop-in replacementとして使えるので、とりあえず試してみるのもあり

---

## 📦[@lydell/elm](https://github.com/lydell/compiler) by [lydell](https://twitter.com/SimonLydell)

- [Help test the new npm elm package!](https://discourse.elm-lang.org/t/help-test-the-new-npm-elm-package/8761)
  - 2022/11/15
- とりあえず試してみると言えば今週出たてのコレ
- Elmが安定している間、世の中ではArm64が急速に利用拡大（M1/M2 Mac, ラズパイなどのSBC, AWSのGraviton2...）
- `elm`もネイティブバイナリを～～～と思ってたところに颯爽登場
  - 大きな問題がなければ早晩公式npmパッケージにmergeされるのでぜひ動作報告を

---

## ⌛[elm-watch](https://github.com/lydell/elm-watch) also by [lydell](https://twitter.com/SimonLydell)

- [Introducing elm-watch: elm make in watch mode. Fast and reliable](https://discourse.elm-lang.org/t/introducing-elm-watch-elm-make-in-watch-mode-fast-and-reliable/8653)
  - 2022/09/09
- Lydellさんといえばこっちも最近のリリース
- Elmにはwatcher(ファイル変更監視&HMR)実装がいくつかあるが、Elm専用にground-upで書かれた最新版
  - debug/optimizeビルドの切り替えやエラー表示など、Elmに寄り添っている
- 実プロジェクトに組み込むパターンをぜひ試行しよう

---

## 👮[ElmLS](https://github.com/elm-tooling/elm-language-server) by [razze](https://twitter.com/razzee)

- [Elm language server and a new VSCode Plugin](https://discourse.elm-lang.org/t/elm-language-server-and-a-new-vscode-plugin/3891)
  - 2019/06/19
- [Another ElmLS and VSCode client update (1.0.0)](https://discourse.elm-lang.org/t/another-elmls-and-vscode-client-update-1-0-0/5894)
  - 2020/06/20
- [Tree-sitter](https://tree-sitter.github.io/tree-sitter/)の[Elm言語用実装](https://github.com/elm-tooling/tree-sitter-elm)をベースとしたElmのLanguage Server
  - 2019年以前はAtomの[elmjutsu](https://github.com/halohalospecial/atom-elmjutsu)があったが、ほとんどの機能は移植済み
- 今や欠かせない。貢献 and/or スポンサーしよう！

---

## 🔍[elm-review](https://github.com/jfmengels/elm-review) by [jfmengels](https://twitter.com/jfmengels)

- [Announcing elm-review](https://discourse.elm-lang.org/t/announcing-elm-review/4401)
  - 2019/09/19
- [Announcing elm-review v2](https://discourse.elm-lang.org/t/announcing-elm-review-v2/5475)
  - 2020/08/20
- **Elmで書ける、Elmのlinter**
  - 非常にフレンドリーなエラーメッセージとautofixを提供できる
- Autofixで大規模なコード生成を仕掛けることもできる
  - ボイラープレートコード生成器として使ってます @ Siiibo

---

## How To Catch Up

- このようにコミュニティの動きは熱い！どう追っていけば？
- [公式Slack](https://elm-lang.org/community/slack)は今も健在
  - Arm64バイナリの議論はつい先週くらいに`#core-coordination`チャンネルで活発にかわされていた
  - ただ今もFreeプランなので時々不満が
- Dillonさん運営のコミュニティDiscord: [Incremental Elm](https://incrementalelm.com/chat/)が活発
  - 新規プロジェクトのアイデア議論などが多い

---

- [Discorse](https://discourse.elm-lang.org/)もやっぱり重要
  - ElmのDiscourseは常時watchするのにちょうどいい流量
- [Elm Weekly](https://www.elmweekly.nl/)
  - ニュースレター。キュレーションしてくれているので最初はこれだけ追うのも結構良さそう
  - 短いので英語苦手でも大丈夫
- [Elm radio](https://elm-radio.com/)
  - ラジオ。これも粒度的にはElm Weeklyに近い
- もちろん今回紹介した開発者の皆さんをフォローするのおすすめ

---

## 紹介しきれなかったもの

- [Gren](https://gren-lang.org/news/220530_first_release/) by [Robin](https://twitter.com/robheghan)
  - Elm forkのgeneral-purpose言語
- [elm-codegen](https://github.com/mdgriffith/elm-codegen) by [mdgriffith](https://github.com/mech_elephant)
  - Elmで書ける、Elmのコード生成器
- [elm-test-rs](https://discourse.elm-lang.org/t/announcing-beta-of-a-new-tests-runner-elm-test-rs/6667) by [mattpiz](https://twitter.com/mattpiz)
  - Rust製elm-test runner. "Fast and portable executable to run your Elm tests"
- ...and more!

---

## まとめ

頑張ればもっと書けそうだ。あとは読者への課題とします。😎

紹介したもの、してなかったもの、なんでも触ってみよう！

- [著者紹介](https://docs.google.com/presentation/d/e/2PACX-1vRuIA2ocDafLRJUn6nWScZmOq6YwpqXba7x5RG72yzT3X7FB-JcET33QMGsBidHsAdbnVF9KYCOa00R/embed?start=false&loop=false&delayms=3000&slide=id.g155be708576_0_55)
- 私もTwitterで気になったのを紹介してることが多いので、[良かったらフォローください！](https://twitter.com/gada_twt)
  - でもTwitter滅亡するかもしれないし、[elm-jp Discord](https://discord.com/invite/4j2MxCg)でもうちょい積極的に紹介するようにしようかな
