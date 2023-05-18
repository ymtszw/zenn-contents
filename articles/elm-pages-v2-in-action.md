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

- elm-pages v2をそこそこいじってきた知見をまとめて紹介
- イケてると思うポイントを説明
- elm-pages v3で新たに取り組まれているところと現状の共有

---

## そもそもelm-pagesとは

[elm-pages](https://elm-pages.com/docs)

- 基本的には静的サイトジェネレータ(SSG)である
- 主要な部分はすべてElmで記述できる
