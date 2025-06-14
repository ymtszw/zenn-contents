---
title: "関数型言語を採用し、維持し、継続する"
emoji: ""
type: "tech"
topics:
  - erlang
  - elixir
  - elm
published: false
publication_name: siiibo_tech
marp: true
theme: gaia
class: invert
paginate: true
---

<!-- _class: [lead, invert] -->

## 関数型言語を採用し、維持し、継続する

By [松澤 有 (ymtszw)](https://zenn.dev/ymtszw) (Siiibo証券株式会社 CTO)
@[関数型まつり2025](https://2025.fp-matsuri.org/) (2025/06/14) [^1]

[^1]: この発表資料はZennで公開しつつ、[Marp](https://marp.app/)スライドとして作成しているので水平線がいっぱい入っています

---

### [Siiibo証券株式会社](https://siiibo.com/)

- 2019年創業、筆者はfounding engineer / CTO
- ↓の構成で社債専門の証券システムを作り上げてきた

![w:600 言語構成](/images/siiibo-languages.png)

---

### このセッションでは

- 実際に関数型言語を業務で採用し、維持し、継続するにあたって重視している価値観、手続き、手法などをざっくばらんに紹介します
- なんとか5年はやってこれた
- 次の5年もやっていきたいがためのやつ

---

### 前提

- 最新の開発組織規模
  - フルタイム - 5名（筆者含む）
  - 週3日程度 - 2名
  - 週2〜10時間程度 - 8名
  - 学生アルバイト - 3名
- 1日あたりの平均デプロイ（master push）回数
  - Copilotにお願いして雑に集計したら**4.94**だった

```sh
$ set start_date (git log master --reverse --pretty=format:'%ad' --date=short | head -1); set end_date (git log master --pretty=format:'%ad' --date=short | head -1); set days (math (date -j -f "%Y-%m-%d" $end_date "+%s") - (date -j -f "%Y-%m-%d" $start_date "+%s")); set days (math "ceil(($days/86400)+1)"); set commits (git rev-list --count master); math "$commits / $days"
4.941361
```

---

<!-- _class: [lead, invert] -->
## 狂気

---

### 「技術選定」

- CTOとかFounding Engineerが技術構成を選ぶにあたっての観点
  - まともな**モノ**を継続的に提供するに足る機能があるか
  - それを実際につくる**ヒト**を継続的に集められるか
- 加えて「スタートアップ」なら、
  - （上記2点を説得材料として）**カネ**を集められるか

<!-- カッコ付きのタイトル -->
<!-- 「まとも」とは？ -->
<!-- 既存組織内での採用でもある程度共通するはず -->

---

### 「メインストリーム」

- ElixirやElmを含む関数型言語の多くは「メインストリーム」とは目されていない[^要出典]
  - 10年前のEvan氏: ["Let's be Mainstream!"](https://www.youtube.com/watch?v=oYk8CKH7OhE)
- モノ・ヒト・カネの要素を自信を持って揃えるには弱い...

<!-- 同じくカッコ付き -->

---

### ...とか言ってられるか

- 良いと感じた技術があって、良いモノを作れると検証できたのであれば、あとは自分がやるかどうかでしかない
  - というか、勝手に機会が降ってくることがない
- ある種の狂気を必要とする点で、非メインストリームな技術採用はスタートアップ起業自体と似ている
  - その試み自体が仮説検証であり、失敗にもまた学びがあるが、結局は勇気と周りを巻き込んで走り続ける力

---

### 「支え」となるもの

- とはいえ「この技術がイイらしい、とりあえずやってみよう」は爆発しそう
  - そこまで無根拠なのは蛮勇
- 何が支えとなって一歩を踏み出せるのか？

**筆者の考えとしては**：

---

### その言語での「良いコード」に対する土地勘

- 本チャンの業務で採用する前に「**検証**」は必須
  - 言うまでもないこと？
  - 個人プロダクトでも、別業務内の軽いツール開発などでもいい
  - そこそこ大きなコードベースを作って勘所を押さえておくこと
  - 先行しているOSSのコードを読んで、先人の開拓した領域に触れておくこと
- 筆者の場合は...

---

#### Elixir

- そもそも前職でかなりElixirを業務で書いて自信をつけていた
  - [access-company/antikythera](https://github.com/access-company/antikythera) - Elixir製マルチテナントアプリケーションプラットフォーム
- 個人的にも業務内外でElixirをかなりの時間書いた
  - [blick](https://github.com/ymtszw/blick) - Antikythera上で動く勉強会スライド集積アプリ
  - [hipchat_elixir](https://github.com/ymtszw/hipchat_elixir) - 今は亡きHipChatのclientをSwaggerから自動生成
- 現職でも必要に応じてOSSリリースしている
  - [siiibo/assert_match](https://github.com/siiibo/assert_match) - Pipeline-friendlyなassertionマクロ

---

#### Elm

- こちらは業務採用には至らなかったがやはり相当書いたし、今も新しいことを試す場をいくつか持っている
  - [ymtszw.cc](https://ymtszw.cc/) - Elm & elm-pagesで書いてる個人ページ
  - [elm-xml-decode](https://github.com/ymtszw/elm-xml-decode) - XmlのApplicative-FunctorなDecoderパッケージ
  - [setem](https://github.com/ymtszw/setem) - 直接Elm製ではないが、Elmのコード生成をするツール

---

#### 主観は大事

- 世の中の採用事例とかを情報として集めておくのも（自分以外に対する）説得材料としては役に立つ
- ...が、究極的には**主観**だと思っている
  - この言語で5年10年戦えるか？実現したい要求に叶うか？飽きないか？
- プラス、「何らか行き詰まったときに捨てやすいか？」という観点もある
  - これは言語自体というよりアプリケーションアーキテクチャでそれなりに担保できる

<!-- 詳細書くかも -->

---

### コードレビューの技術

- 何らかの技術要素を採用して業務に挑もうとするとき、必然的にFounding Engineerはコードレビューをする立場
  - 自分がいいと思う技術であっても、それを「伝道」できなければあとが続かない
- 健全で有意義なコードレビューのやり取りがどのようなものか？という経験とその内面化

---
### コードレビューの技術
- 何をもって「できている」と言えるのかがそもそも難しい
  - ここでも主観が大事だし、ずっと研究が続く
  - マジで**環境に依存する**と思うので若いエンジニアには特に言っている
  - 筆者の個人的な経験としては前職（新卒入社）のチームが本当に素晴らしかった...そこで私のすべてが形作られた
- この話は直接次の章につながる

---

### 「ないものは作る」精神

- 「ライブラリや有名サービスのSDKがない」
  - 非メインストリーム言語においてツラミとしてよく挙がる
- これについては「ないときに自分で作りやすいか」「作ったものを腐らせずに維持しやすいか」まで含めて言語の評価点と捉えたい
  - Elixir/Elmでいうと、意外と典型的なニーズに応えるライブラリは揃っているのであまり困った経験はなし
  - とはいえこれは作ろうとしているサービスの特性にも左右されるので注意
- 後述するが、コミュニティ貢献へと直接つながる

---

### 狂気の章のまとめ

- 非メインストリーム技術採用は狂気ドリブンで機会を自ら作り出すもの
- 事前検証と伝道師としてのコードレビュー技術が支えになる
- ないものは作れ

---

<!-- _class: [lead, invert] -->
## コードレビュー / CI / Upkeep

ルーチン業務で重視する要素

---

### コードレビュー

- 「どういうコードが（チームにとって・製品にとって）良いコードか」の伝道事業
  - 伝「**道**」であることが大事
  - 感覚・信念として共有できる状態に持っていきたい
- よく引き合いに出す資料：
  - [How to Pull Request](https://medium.com/google-developer-experts/how-to-pull-request-d75ac81449a5) by Google Developer Experts
  - レビュイーとレビュワー、両方の視点で意識すると良いことがまとまっている

---

#### 永遠の事業

- **コードレビューに終わりはない**
  - 人は忘れる（レビューする側も、される側も）
  - 新しい人は入る
  - 一度言っただけで身につくことは誰であれ、ない
- 言葉を尽くして、繰り返し対話し続ける

---

#### 自動化する

- レビュー指摘項目のうち、自動的に強制できるものは自動化して機械にやらせる
  - Formatterで強制できるコードスタイル
  - Linterで強制できる観点

---

### 自動化する

- これがやりやすいか言語か、標準ツールが存在するか、というのは当該言語に対する大きな加点項目
  - 少なくともElixir/Elmでは長いことformatの議論をしてない
    - Nearly-zero-configなformatterが標準
    - 公式formatterのスタイルが好きじゃない、という状況はたまにあるが、考えることを減らせるメリットのほうが大きいので割り切っている
  - Linterの話も後述

---

### CI

- 自動化ツールの導入はCIとセット
- 人間は忘れるのだから、忘れない機械に何でもチェックさせる
  - ビルド、テスト、FormatterやLinter以外だと：
    - 破壊的変更（DB-Server間や、Server-Client間）の検知
    - 自動生成コードのコミット漏れの検知
- 後の章で更に触れる

---

### CD

- 作ったプロダクトを顧客に届ける方法が確立されていて迷いが少ないのも嬉しい
- 非メインストリームな技術にあっては、このあたりで詰まらないのは障壁をだいぶ下げる

---
#### CD: Elixir
- [mix release](https://hexdocs.pm/mix/1.18.4/Mix.Tasks.Release.html)によってパッケージ化し、あとは仮想サーバなりコンテナなりに置いて実行するだけ
  - 2020年のv1.10で導入されて大変やりやすくなった
  - DockerのMulti-stage buildによる軽量化とも相性が良い
- しかもreleaseスクリプトには、remote console機能が同梱されていて、これがAWSならECS Execと相性抜群
  - デプロイ環境でのデバッグに重宝する

---
#### CD: Elm
- Production build機能がコンパイラ同梱で、Dead-Code Eliminationを標準装備
- 基本的にはこれだけでも比較的小さい成果物が得られる
- 追加でMinifierをかけても良い
  - この辺はコミュニティ努力で知見が蓄積されているが、フロントエンド技術の進化に合わせてより良いツールに載せ替えることもあり
  - いささか周辺知識が必要なところ

---

### Upkeep

- 「維持費」の意、好きでよく使ってる
  - 個人的にはゲーム[Warcraft 3](https://classic.battle.net/war3/basics/upkeep.shtml)で覚えた
- 製品を作ってコードが育つとき、必然的に伴ってくる：
  - 依存ライブラリのバージョンアップ
    - Discontinuationにおける移行
  - 言語のバージョンアップ
  - その他諸々...

---

#### Upkeepしやすさ

- 恐怖症的にならずにこまめにやる
- バージョンアップ系作業がやりやすいのも言語の大きな加点項目
  - 依存関係解決が決定論的かつ高速にできる
  - 何ならDependabotなどに勝手にやらせられる範囲も広がる
- モダンな言語だとそんなに議論になることは少ないかもしれない

---

#### Upkeepしやすさ

- Elixir, Elmはいずれも比較的優秀
  - mix (Elixir)のdeps解決は速いしわかりやすい
    - 必要に応じて脱出ハッチ（override）も使える
  - Elmはそこに加えて**forced-semantic versioning**
    - 少なくともインターフェイス非互換はやる前に防げる

<!-- もちろん、インターフェイスを保って内部挙動に微差が導入されることはたまにあるが -->

---

### Elmの特記事項

- Elmはもう5年以上メジャーアップデートされていない

  > If you like what you see in 0.19.1 now, that’s pretty much what Elm is going to be for a while!
  > — [Status Update - 3 Nov 2021](https://discourse.elm-lang.org/t/status-update-3-nov-2021/7870)

- この「変わらなさ」を受け止められるかは、少なくともElmの採用に関しては大きな試練だろう
- 個人的には**Proudly Unchanged**とうそぶいている

---

### Elmの特記事項

- Elmの良いところは、Elm-runtimeで区切られた内部世界は純粋なので、その範囲では脆弱性の導入される余地がないこと
  - 少なくとも現行のアーキテクチャが確定したElm 0.18以来10年くらい経つが、筆者の認知している範囲ではこの前提は破られていない
- サプライチェーン攻撃に耐性があると言える

<!-- 一方で、たまに言われる「ランタイムエラーが発生しない」というのは流石に言い過ぎで、実際には発生する余地がある -->

---

### 関数型言語のルーチン業務を支える技術の章のまとめ

- 何はなくともコードレビュー、永遠の事業
- 自動化・デプロイ手法の観点なしでは今の技術選択は語れない
- 変わること・変わらないことどちらに対しても態度を明確化して品質維持に務める

---

<!-- _class: [lead, invert] -->
## ビルドとテストの高速化

血道をあげろ！

---

### コンパイル速度

- 関数型言語はAOTコンパイルが多い
  - ElixirもElmもそう
- プロダクトが育つとビルドやテストが遅くなりがち

---

### 高速化し続ける
- ユーザ側の努力でも高速化できる余地はあり、常に追求する
- 高速化に繋がりそうな更新情報の追跡も怠らない
- メンバーが増えるほどに、ローカル/CIいずれでもビルドやテストの実行時間短縮は乗算で大量の時間節約になる

---
### 高速化: mix test

- [mix test](https://hexdocs.pm/mix/Mix.Tasks.Test.html)はElixirの標準テスト実行経路
- 純粋なunit testも、背後のDBや外部サービス等の存在下で行うintegration testも書ける
- master merge前のCIではDBのみ存在する前提で記述しているmix testを必須としている
  - SMLテストサイズで言うところのMediumくらい
    - 参考: [ユニットテストってもう言わない！ CI/CD時代のテスト分類に最適なテストサイズという考え方](https://zenn.dev/koduki/articles/e0f8824adbe0e9)

---
### 高速化: mix test

- 該当するmix testの件数は最新のコードで4,500件ほどある
- かなり多くの割合がDB I/Oを含む
- これを少なくともApple Silicon MacBook系のローカル開発機では1分程度で実行できるよう維持している

![w:800 mix test実行時間](/images/siiibo-test-time.png)

---
### 高速化: mix test

- 実際にやっているのは、
  - テストの並列実行
  - 並列実行を可能にするための、
    - DBの複製
    - Test fixtureの独立化
    - Sandbox接続の利用
  - DB初期化を高速にするためのsnapshot活用

<!-- Sandbox接続は、DB接続直後にTxを開始し、その内部でのDB操作を接続終了時にすべてロールバックすることで、DB接続が続いている限り、そのプロセスにとっては実際にデータがDBに書き込まれたように振る舞うが、テスト終了時にはすべての操作がなかったことにされてクリーンな状態に戻るという仕組み -->

---
### 高速化: mix test

- 並列実行自体はmix testの標準機能
- DB接続のSandboxingも[Ecto](https://github.com/elixir-ecto/ecto)（ElixirのORM+Query Builder）の標準機能
- ただし、
  - module単位でしか並列実行できない
  - 自然体だと同じDBを使うので、sandboxingしていてもWRITE操作の内容によってはdeadlockする

---
### 高速化: mix test
- そこで、
  - テスト同士の相互依存をなくすために、必要なfixtureデータはsandbox内で独立して作成する
  - テストDBを実行並列度に応じて複製する
  - テストごとに利用DBをcheckout/checkinする機構を設ける
    - Ectoの[Dynamic Repo](https://hexdocs.pm/ecto/replicas-and-dynamic-repositories.html#dynamic-repositories)機能がこれを可能にしてくれる

---
### 高速化: mix test
- しかしEcto migration（よくあるパッチ方式のDB migration機能）を複製DB全てに適用するのは時間がかかる...
  - そこで、最新のDB schema snapshotを定期的に取得しておき、新規のテストDB作成時にはsnapshotから開始できるようにして高速化
- ここまで見てきたように、Ectoの諸機能は快適なテスト実行に大いに寄与しており、こいつがデファクト・スタンダードとして存在していることはElixirの大きな強み

---
### 高速化: elm make
- Elmのインクリメンタルコンパイルはデフォで比較的速い
  - これはDCEを有効化したprod buildでも同様
- が、コンパイル対象moduleが多いときはそこそこかかる
- しかも、**特定の実装パターンがあると、構成module数が増えたときにGC時間及びメモリ使用量が増えるという現象**に遭遇したことがあった

---
### 高速化: elm make
- 自分でやる精神のもと、社内で調査
- ここはメンバーの[@tsukimizake](https://github.com/tsukimizake)が大活躍
  - プロファイラーを有効化してコンパイラをビルド
  - プロファイラーオプションを調整してボトルネック調査
  - Excessive GCを引き起こしていた実装パターンを特定
  - 当該パターンを回避できるようにコード生成系を改修

---
### 高速化: elm make
- そもそも、このあたりをまるっと改修できるよう、コード生成系が整備されていたのも大きかった。この点は後述
- [elm-meetup](https://elm-jp.connpass.com/event/262458/)でも発表された
  - [tsukimizake/elm-compilation-time-slide](https://github.com/tsukimizake/elm-compilation-time-slide/?tab=readme-ov-file)
  - [tsukimizake/elm-compilation-time-slide の続き](https://github.com/tsukimizake/elm-compilation-slide-2/?tab=readme-ov-file)
- 得られた知見はフォーラムにも展開
  - [Improving Compilation time: Insights from Elm Compiler Internals - Show and Tell - Elm](https://discourse.elm-lang.org/t/improving-compilation-time-insights-from-elm-compiler-internals/9028)

---
### 高速化の章のまとめ

- AOTコンパイル言語、それもコンパイル時の機能（型検査含む）が豊富であれば、コードが育つほどに速度問題は不可避
- コンパイラの改善でほっといても速くなってくれることもあるが、基本的には自分でやる精神はここでも発揮する必要がある
- 速さと向き合え！

---

<!-- _class: [lead, invert] -->
## 静的型付けでない言語と型の話

---
### Elixirやってると思うこと

- **まあ型は欲しくなる**
  - 型があったらこのバグは起きなかった、みたいなことは定期的にある
- とはいえElmといっしょに開発し、多くのメンバーがなんだかんだElmも書いているという状況では、ありがたいことに多くの学びがある

---
### 直和による表現、パターンマッチによる分岐
- Elmは言語機能が少ない静的型付けの純粋関数型言語
  - 典型的な実装イディオムとして、直和型とパターンマッチが頻出する
  - 昔書いた記事: [「ADT, 直和・直積, State Machine」 - Qiita](https://qiita.com/ymtszw/items/dff02ad6350032688676)
- Elixirはその点、
  - 型の静的検査はない
  - 直和を表現する専用の言語機能はない

---
### 直和による表現、パターンマッチによる分岐
- が、Atomを使った「直和的な」データ表現はかなり頻出で（Erlang/Elixirの標準ライブラリ内でも！）、そのことに自覚的になると「自然な」実装パターンが見えてくる（主観）
  - 筆頭は`{:ok, value} | {:error, reason}`
- 「Elmだとそういえばこう書いてたな、これElixirでもやれないの？」はそれなりにやれる
  - 特に副作用の絡まない関数では表現を似せられて腹落ちすることが多い
  - `Result`型に対する諸APIはElixir側にも実装してある

---
### Pipeline-friendlyなassertion
- [elm-test](https://package.elm-lang.org/packages/elm-explorations/test/latest/Expect)では`Expect`系関数をpipelineで受けて書かれる

```elm
String.split " " "Betty Botter bought some butter"
    |> Expect.equal [ "Betty Botter", "bought", "some", "butter" ]
```

- 一方`mix test`では`assert`マクロを手続き的に並べて書くことが多いが、場合によってはElmからの影響もあってpipeで書きたくなる

---
### Pipeline-friendlyなassertion
- [siiibo/assert_match](https://github.com/siiibo/assert_match)はそのような日常的洞察から生まれたOSS
- 手前味噌ながらこの`assert_match`マクロは結構強力で、どのような構造に対してもパターンマッチとしてassertionを書けるので、
  - pipeで書けて気持ちいいし
  - Elixirの強力なパターンマッチ機構を活かせるし、
  - 「関数とは入力と出力なのだ」という意識が高まる
  - （主観）

---
### 天から型が降ってきた！

- そんなこんなで言語間比較による恩恵はある程度受けていた
- 関数型言語によく出てくる概念は、単一の言語だけやってるとイマイチ腹落ちせず、いくつか言語をまたぐと得心できることが多いという経験則があり、その意味でも有用
  - 個人的には、**Scalaで始まり、Elixirに入り、Haskellの手ほどきを受け、Elmで色々理解した**、という関数型キャリアパスを辿ってきた

---
### 天から型が降ってきた！
- とはいえ章冒頭の通り、Elixirに静的型検査がないことには定期的につまらなさを感じていたのだが...
- なんとElixirのコンパイラが静的型検査を始めるという大本営発表がなされたのだ！
  - [My Future with Elixir: set-theoretic types](https://elixir-lang.org/blog/2022/10/05/my-future-with-elixir-set-theoretic-types/) (2022/10/05)
  - [Gradual set-theoretic types](https://hexdocs.pm/elixir/gradual-set-theoretic-types.html)
  - [The Design Principles of the Elixir Type System](https://arxiv.org/abs/2306.06391)

---
### 天から型が降ってきた！
- Elixir v1.18で一部型検査機能がすでに有効化され、早速既存コードベースでもいくつかの潜在バグを発見・解消できた
- [Elixir v1.19](https://github.com/elixir-lang/elixir/releases/tag/v1.19.0-rc.0)でもさらに入る：
  - Type checking of protocol dispatch and implementations
  - Type checking and inference of anonymous functions
- ついでに大規模コードベースでのコンパイル速度改善施策も入る
- **Upkeepをこまめにやるのは、このような新施策をすぐ採用できる強みに直結**

---
### 静的型付けでない言語と型の話の章のまとめ

- 関数型言語のエッセンスを体得するには複数言語に触れたほうがいい
  - その意味でElixir/Elmのスタックは有用だったという振り返り
  - 両者の実装が自然と親和して、認知モデルを共有できる
- 型で考えていると型の方から（文字通り）やってくることもある

---

<!-- _class: [lead, invert] -->
## 情報収集 / コミュニティ還元
「アウトリーチ」

---
### 情報収集
- 非メインストリームな言語はほっといても情報が入ってくるわけではない
  - 規模にもよるが
- それでも狂気を発露して採用に踏み切ったのなら、**アクティブな情報収集**は必須

---
### 情報収集
- 言語公式フォーラム/ML/Slack/Discordのウォッチ
  - コミュニティがデカすぎないことを逆に活かして、ライブラリ開発者などがよく出入りしているトピックやチャンネルを購読しちゃう
  - 最新動向を嫌でも逃さなくなる
  - 社内にも展開する
- コンパイラや主要なライブラリのissue trackerも購読しちゃう
  - チーム内で抱えてる課題と似たような話題も出たりする
- X

---
### コミュニティ還元
- ウォッチしている経路での、質問者に対する回答
  - 聞いてるだけでなく回答もする
  - なにげに「自分が第一人者」「フロントランナー」である可能性を意識
  - 英語翻訳にはあまり困らない時代になった
- 国内外の勉強会参加
  - 何なら開催協力

---
### 採用の話
- ここまで書いてきた「やれることやってる」環境を維持し、アウトリーチもすることは、とりも直さず採用に直結
- 何しろ非メインストリームな技術要素は、それを業務でちょっと触ってみたいと思っても**会社の選択肢が少ない**
- 選択肢がほしい・見識を広げたい人にとっては良いマッチング機会

<!-- 幸運もあるが、これまで採用にはあまり困ったことがなく、何ならインバウンドで定期的に希望者が来てくれている -->

---
### アウトリーチの章まとめ
- やれることはやる、近道はない
- 人口の少なさは好機でもある
- 全世界を対象にすれば意外とコミュニティは広い

---

<!-- _class: [lead, invert] -->
## 新しい手法を取り入れる

---
### E2Eテスト
- E2Eテストやelm-reviewによるlint・コード自動生成
- AIコード支援など新しい道具の導入
- 当初はなかった手法を後から取り入れる柔軟性

---

## 覚悟

- 最終的には覚悟が必要
- 継続・維持のために大切なこと
