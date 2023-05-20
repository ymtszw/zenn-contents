---
title: "elm-pages in Action"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - elm
  - ssg
published: false
marp: true
theme: gaia
class: invert
paginate: true
---

<!-- _class: [lead, invert] -->

## elm-pages in Action

By [ymtszw](https://zenn.dev/ymtszw)
@[Elm-jp 2023](https://elm-jp.connpass.com/event/280401/) (2023/05/20) [^1]

[^1]: この発表資料はZennで公開しつつ、[Marp](https://marp.app/)スライドとして作成しているので水平線がいっぱい入っています

---

## 前回？までのあらすじ

- 昨年、[elm-pages (v2)とHeadless CMSに入門し、好印象を抱いた](https://ymtszw.github.io/articles/elm-pages-and-headless-cms)
  - せっかくなので[personal website](https://ymtszw.github.io/)として成果物を整備し、その後継続的に記事更新したり、サイト改良したりしている
  - [ソース](https://github.com/ymtszw/ymtszw.github.io)
- 昨年後半の[Elm Meetup (online)でコミュニティ製フレームワークの一つとして紹介](https://zenn.dev/siiibo_tech/articles/elm-meetup-202211?redirected=1#%F0%9F%93%9Aelm-pages-(v2)-by-dillonkearns)

---

## 今回は…

- elm-pages v2の概念的なポイントを紹介
- elm-pages v3で新たに取り組まれているところと現状の共有

---

## そもそもelm-pagesとは

[elm-pages](https://elm-pages.com/docs)

- 基本的には静的サイトジェネレータ(SSG)である
- 主要な部分はすべてElmで記述できる
- 以下、v2のAPIの概念的なポイントを紹介

---

## Build-timeとRun-time, 2つのElm App実行

- SSGである以上、事前にコンテンツを静的なHTMLとして生成し、配信したい
- しかし、Elmは**クライアントでの実行時にコンテンツが動的生成される**SPAに特化した言語だったはず…

=> elm-pagesは、**2つのタイミングでのElm App実行**によってこのギャップを埋める

---

## Build-time App

- `elm-pages build`を実行したとき、elm-pages CLIはまずボイラープレートコード生成を行う
  - elm-pagesはFile-based Routingを採用
  - ファイル名(=モジュール名)規則を元に対応するroutingコード等が生成され、root appの`main`関数に結線するあたりは自動

```elm
module Page.Index exposing (..)
```

```elm
module Page.Articles.ArticleId_ exposing (..)
```

---

- 生成されたElm Appは**実際にpre-rendering機構によって実行され**、結果としてのHTMLがrouteごとに得られる
- これらのHTMLがSSGとしての**第一の**成果物
  - 適当なWebサーバから配信すれば、クローラーフレンドリーなWebサイトとして機能する
  - JavaScriptが無効化されていても、prerenderされたwebサイトは正常に表示される

---

## Run-time App

- 一方、生成されたElm Appそれ自体も、**第二**の成果物となる
- 第二の成果物は、以下の要素から成る：
  - みなさんが想像する**Elm SPAそのもの**
  - routeごとに事前生成されたHTMLを表示後、上記のElm SPAをクライアントwebブラウザで実行し、以後の処理をSPAに移譲するためのグルーコード

---

- この構成によって、静的サイトの事前生成も、webブラウザでのサイト読み込み後の動作もいずれもElmで記述する、という世界観が成立する
- …さて御存知の通り、Elm 0.19のVDom実装にはいわゆる**hydration機能(VDomを元にSSRされた結果から、ブラウザ内でVDomを再構築する機能)がない**

=> それではどのようにして、「以後の処理をSPAに委譲」するのか？

---

## 決定論的rendering

- 予め結論を述べると、意外に特別なことはやってない
- `DOMContentLoaded`イベント契機で第二の成果物であるElm Appを起動しているだけ

```js
// 成果物HTMLで実質的に実行されるPromise
const appPromise = new Promise(function(a){
  document.addEventListener("DOMContentLoaded", () => {
    a(loadContentAndInitializeApp())
  })
});
```

---

1) Build-timeのpre-rendering
2) ブラウザ内でのclient-side rendering

=> この2つが(少なくともrender完了直後のフレームにおいて)**寸分違わず一致するならば**、第一の成果物由来のHTML表示そのままに、第二の成果物由来のSPAによる処理に間断なく移行できる、という理屈

---

- 「寸分違わず一致」させるために、elm-pagesはこれら**二種類のrendering結果が決定論的に定まる**よう設計されている
- 具体的には、
  - ページがrendering時に依存するデータの`DataSource`による抽象化
  - Flagsの分離
- （注: 多分ほかにも工夫はあるけど、概念的にポイントとなるのはこのへんだと理解している）

---

## `DataSource`

- **Build-timeに解決**されるデータ
  - 由来はCMS等の外部APIでもいいし、ローカルファイルでもいい
- elm-pagesの各ページモジュールは、`init`時には基本的に`DataSource`にのみ依存できる
- ポイントは、解決された`DataSource`の内容は、**JSONとしてシリアライズされてHTML(第一の成果物)に埋め込まれる**こと

---

- Run-timeに実行されたElm App(第二の成果物)も、Build-timeから持ち越された`DataSource`をそのまま読み込んでページモジュールを`init`する
- これによって両者の評価結果が一致する仕組み
  - たとえば、CMS APIの返す値がbuild-timeとrun-timeで変化していても影響を受けない

---

## Flagsの分離

- 通常、Elm App起動時のもう一つの依存データはFlags
- elm-pagesでは、

```elm
type Flags
    = BrowserFlags Json.Decode.Value
    | PreRenderFlags
```

- このように2つのElm Appが受け取るFlagsが分離される

---

- 見て分かる通り、**pre-render時にはFlagsは存在できない！**
  - 例えば、Elm App起動時のブラウザの画面サイズや現在時刻を取得する、といったことはpre-render時にはできないよう型で保証されている
- 2つのElm Appは、どちらの起動タイミングでも最初はpre-render状態(Flagsなし)で起動する
- Webブラウザ上で起動したとき、`BrowserFlags Json.Decode.Value`がElm App起動後の処理で遅れて利用可能となる

---

## elm-pages v2のポイントまとめ

- Built-timeとRun-time, 同じElm Appを2つのタイミングで実行
- かつ両者の結果が一致するよう設計することで、できないはずだったSSG→SPA委譲を可能にする
- そのための工夫
  - `DataSource`
  - Flagsの分離

---

## elm-pages v3では何が起きようとしているか？

- v2までで、いわゆるJAMStackにおけるSSG→SPAという部分についてはElmだけで実現可能になっている
  - 事前にページ生成して検索インデックスに乗るようにし、ページ表示後はCSRで動的コンテンツ提供
- が、御存知の通りこの構成には**中間地点**が欲しくなる
  - 特に事前に生成したいページが大量にあったり、頻繁に更新されるようなwebサイト/webサービス
  - 最たるものはECサイトや、ユーザ生成コンテンツからなるサービスなど

---

- その中間地点を満たすのが**SSRの諸形態**
  - 特にNext.jsの**ISG; Incremental Static Generation**や**ISR; Incremental Static Regeneration**, **On-demand ISR**あたり
  - 参考
    - [Next.jsのOn-demand ISRで、ビルド不要の高速配信を実現する | microCMSブログ](https://blog.microcms.io/on-demand-isr/)
    - [【Next.js】On-Demand ISRが安定版になったよ！ | マイナビエンジニアブログ](https://engineerblog.mynavi.jp/technology/on-demand_isr_introduction/)

---

- アプローチ
  - ISG: **リクエストがあったとき**に初めて静的生成を行い、それをエッジキャッシュに載せる
  - ISR: `stale-while-revalidate`戦略で、キャッシュデータを高速に表示しつつ再生成も行う
  - On-Demand ISR: 更に、イベントドリブンな適時再生成にも対応して、stale表示を最小化する
- elm-pages v3では、端的に言うとISG相当のアプローチに対応しようとしている
  - [Server-rendered route](https://github.com/dillonkearns/elm-pages-v3-beta/blob/master/docs/FAQ.md#is-elm-pages-full-stack-can-i-use-it-for-pure-static-sites-without-a-server-can-i-use-it-for-server-rendered-pages-with-dynamic-user-content)
  - リファレンス実装はNetlify Functionsを使用
  - Adapter層を導入することで、他のプラットフォームにも対応

---

## v3 status

- ほかにも、webサイトで頻出の要求であるフォーム送信機能もサポートするため、設計・実装中
- 更に、`DataSource`等の内部実装を効率化するためにLamdera compilerを採用
- ...and more!

=> という状況のため、v3-betaとそのstarter repoはまだまだ流動的。一方v2のAPIはそこそこ **「枯れている」と言っていいくらいに整っている**ので、23年5月現在ならv2でまずはデビューしてみてください

---

## あとがき

- elm-pagesの中心概念はどっかで自分の言葉で書いておきたいと思ったのでこのような内容になった
- ホントはタイトルの"in action"にもっと寄せて、
  - v2でのパフォーマンスチューニングとか、
  - 自前サイトでの具体的なコードの話とか、
  - いくつか裏技小技とか、
  - その辺も書きたいのだが、また別の機会に
