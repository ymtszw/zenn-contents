---
title: "Erlang/ElixirでSendGridのWebhook署名検証する"
emoji: "🔐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - erlang
  - elixir
  - sendgrid
  - webhook
  - ecdsa
published: true
publication_name: siiibo_tech
---

以前書いたクライアント認証とはまた毛色が違うものの、Erlangの[public_key](https://www.erlang.org/doc/man/public_key.html)を利用するTips第２段。

https://zenn.dev/ymtszw/articles/erlang-elixir-hackney-client-auth-20210327

## SendGridのWebhook署名を検証したい

SendGridを使って配信されるメールの関連イベントはWebhookで受け取ることができますが、偽イベントを受け取らないようきちんと署名検証をしたいところ。

https://docs.sendgrid.com/for-developers/tracking-events/getting-started-event-webhook-security-features

採用されている方式はECDSA（楕円曲線デジタル署名アルゴリズム）で、公式SDKが提供されているのは:

- [C#](https://github.com/sendgrid/sendgrid-csharp/tree/master/src/SendGrid/Helpers/EventWebhook)
- [Go](https://github.com/sendgrid/sendgrid-go/tree/master/helpers/eventwebhook)
- [Java](https://github.com/sendgrid/sendgrid-java/tree/master/src/main/java/com/sendgrid/helpers/eventwebhook)
- [Node.js](https://github.com/sendgrid/sendgrid-nodejs/tree/master/packages/eventwebhook)
- [PHP](https://github.com/sendgrid/sendgrid-php/tree/master/lib/eventwebhook)
- [Python](https://github.com/sendgrid/sendgrid-python/tree/master/sendgrid/helpers/eventwebhook)
- [Ruby](https://github.com/sendgrid/sendgrid-ruby/tree/master/lib/sendgrid/helpers/eventwebhook)

例のごとく我々（？）の使いたい言語がないので、SDKの実装を参考に実装してみます。

実はこの記事はほぼ

https://fumieval.hatenablog.com/entry/2022/06/24/184435

でHaskellでやってる内容のErlang/Elixir版です。

## 公開鍵を入手して使える状態にする

ドキュメントの通り、SendGridの管理画面にログイン→"Mail Settings"から"Event Webhook"を作成して、"Signature Verification"を有効化し、"Verification Key"をコピーしてきます。

```text:Verification Key
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAfSmpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g==
```

:::message
↑の例は[公式SDKのユニットテスト](https://github.com/sendgrid/sendgrid-nodejs/blob/main/packages/eventwebhook/src/eventwebhook.spec.js)で使われているサンプル公開鍵です
:::

### 鍵の正体

公式解説やSDKの実装を読み解くと分かる通り、このVerification Key文字列はいわゆる"One-Line PEM"形式の公開鍵です。

PEM形式の話は[以前の記事](https://zenn.dev/ymtszw/articles/erlang-elixir-hackney-client-auth-20210327)でも書きましたが、One-Line PEMは、

```text
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAfS
mpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g==
-----END PUBLIC KEY-----
```

のような一般的なPEM形式の鍵を、改行を取り除いて一行にし、ヘッダーとフッターを取り除いたものです。

複数行の証明書（公開鍵）ファイルは環境変数経由での利用などが面倒なので、この形式に一定の利便性があるのはパッと見うなずけます。
一方、そもそもこの形式が正式に標準化されたものかというとそうではないようです。

https://serverfault.com/questions/466683/can-an-ssl-certificate-be-on-a-single-line-in-a-file-no-line-breaks

https://stackoverflow.com/questions/59943474/can-i-remove-all-break-lines-except-the-first-and-last-one-from-jwt-key

後者の回答を引用すると、

> **It depends.**
>
> Theoretically, the answer is **no** \- you can't *always* remove the line breaks. This is simply because [RFC 1421](https://www.rfc-editor.org/rfc/rfc1421) defines situations in which you must include the line breaks

> Practically, however, the answer is generally **yes**: a number of common implementations are quite lenient when parsing objects in PEM format and allow for excessively long text lengths.

ということで、RFC 1421まで遡るとOne-Line PEMは非標準だが、多くの実装はこれを許容しているということのようです。

### Erlangのpublic_keyでOne-Line PEMをdecodeする

この前置きで想像できたかと思いますが、Erlangの`public_key:pem_decode/1`は**One-Line PEM非対応**です。

（以下、コード例はElixirのiex環境）

```elixir
iex> olpk = "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAfSmpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g=="
"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAfSmpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g=="
iex> :public_key.pem_decode(olpk)
[] # エントリが認識されない
```

ということで選択肢としては、

- １行64文字に区切ってヘッダーとフッターを人為的に加え、標準のPEM形式に再構成する
- Base64デコードしてDER形式に変換するところまでやって、あとはDERを扱う関数で処理する

が考えられます。

:::message
[PEMとDERの関係も以前の記事参照](https://zenn.dev/ymtszw/articles/erlang-elixir-hackney-client-auth-20210327#%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E8%A8%BC%E6%98%8E%E6%9B%B8%E3%81%AE%E5%BD%A2%E5%BC%8F)
:::

今回は後者でやってみます。早速コード例：

#### PEM => Base64デコード => DER

```elixir
iex> decoded_pem = Base.decode64!(olpk)
<<48, 89, 48, 19, 6, 7, 42, 134, 72, 206, 61, 2, 1, 6, 8, 42, 134, 72, 206, 61,
  3, 1, 7, 3, 66, 0, 4, 243, 116, 248, 59, 249, 252, 226, 42, 45, 34, 242, 22,
  226, 103, 65, 129, 15, 251, 116, 7, 210, 154, 154, ...>>
```

#### DER => ASN.1エントリ名を指定してデコード => SubjectPublicKeyInfo

```elixir
iex> {:SubjectPublicKeyInfo, alg_id, public_key} = :public_key.der_decode(:SubjectPublicKeyInfo, decoded_pem)
{:SubjectPublicKeyInfo,
 {:AlgorithmIdentifier, {1, 2, 840, 10045, 2, 1},
  <<6, 8, 42, 134, 72, 206, 61, 3, 1, 7>>},
 <<4, 243, 116, 248, 59, 249, 252, 226, 42, 45, 34, 242, 22, 226, 103, 65, 129,
   15, 251, 116, 7, 210, 154, 154, 136, 51, 201, 5, 246, 99, 117, 126, 90, 85,
   41, 45, 198, 70, 167, 186, 218, 12, 62, 217, 243, 77, 69, ...>>}
iex> {:AlgorithmIdentifier, _ec_point_alg_id, params} = alg_id
{:AlgorithmIdentifier, {1, 2, 840, 10045, 2, 1},
 <<6, 8, 42, 134, 72, 206, 61, 3, 1, 7>>}
```

#### Algorithm Identifierのパラメータをデコード => EcpkParameters

```elixir
iex> ecc_params = :public_key.der_decode(:EcpkParameters, params)
{:namedCurve, {1, 2, 840, 10045, 3, 1, 7}}
```

#### :public_key.verify/4に投入するRecord形式を構築

```elixir
iex> pk = {{:ECPoint, public_key}, ecc_params}
{{:ECPoint,
  <<4, 243, 116, 248, 59, 249, 252, 226, 42, 45, 34, 242, 22, 226, 103, 65, 129,
    15, 251, 116, 7, 210, 154, 154, 136, 51, 201, 5, 246, 99, 117, 126, 90, 85,
    41, 45, 198, 70, 167, 186, 218, 12, 62, 217, 243, 77, 69, ...>>},
 {:namedCurve, {1, 2, 840, 10045, 3, 1, 7}}}
```

これで準備完了です。

## 実際に署名検証してみる

### Fixture整備

公式SDKのテストで使われているFixtureをさらに拝借してきます。

```elixir
iex> signature = "MEUCIGHQVtGj+Y3LkG9fLcxf3qfI10QysgDWmMOVmxG0u6ZUAiEAyBiXDWzM+uOe5W0JuG+luQAbPIqHh89M15TluLtEZtM="
"MEUCIGHQVtGj+Y3LkG9fLcxf3qfI10QysgDWmMOVmxG0u6ZUAiEAyBiXDWzM+uOe5W0JuG+luQAbPIqHh89M15TluLtEZtM="
iex> decoded_signature = Base.decode64!(signature)
<<48, 69, 2, 32, 97, 208, 86, 209, 163, 249, 141, 203, 144, 111, 95, 45, 204,
  95, 222, 167, 200, 215, 68, 50, 178, 0, 214, 152, 195, 149, 155, 17, 180, 187,
  166, 84, 2, 33, 0, 200, 24, 151, 13, 108, 204, 250, 227, 158, 229, 109, ...>>
iex> timestamp = "1600112502"
"1600112502"
iex> body = [
...>     %{
...>       "email" => "hello@world.com",
...>       "event" => "dropped",
...>       "reason" => "Bounced Address",
...>       "sg_event_id" => "ZHJvcC0xMDk5NDkxOS1MUnpYbF9OSFN0T0doUTRrb2ZTbV9BLTA",
...>       "sg_message_id" => "LRzXl_NHStOGhQ4kofSm_A.filterdrecv-p3mdw1-756b745b58-kmzbl-18-5F5FC76C-9.0",
...>       "smtp-id" => "<LRzXl_NHStOGhQ4kofSm_A@ismtpd0039p1iad1.sendgrid.net>",
...>       "timestamp" => 1_600_112_492
...>     }
...>   ]
[
  %{
    "email" => "hello@world.com",
    "event" => "dropped",
    "reason" => "Bounced Address",
    "sg_event_id" => "ZHJvcC0xMDk5NDkxOS1MUnpYbF9OSFN0T0doUTRrb2ZTbV9BLTA",
    "sg_message_id" => "LRzXl_NHStOGhQ4kofSm_A.filterdrecv-p3mdw1-756b745b58-kmzbl-18-5F5FC76C-9.0",
    "smtp-id" => "<LRzXl_NHStOGhQ4kofSm_A@ismtpd0039p1iad1.sendgrid.net>",
    "timestamp" => 1600112492
  }
]
iex> json = Jason.encode!(body) <> "\r\n"
"[{\"email\":\"hello@world.com\",\"event\":\"dropped\",\"reason\":\"Bounced Address\",\"sg_event_id\":\"ZHJvcC0xMDk5NDkxOS1MUnpYbF9OSFN0T0doUTRrb2ZTbV9BLTA\",\"sg_message_id\":\"LRzXl_NHStOGhQ4kofSm_A.filterdrecv-p3mdw1-756b745b58-kmzbl-18-5F5FC76C-9.0\",\"smtp-id\":\"<LRzXl_NHStOGhQ4kofSm_A@ismtpd0039p1iad1.sendgrid.net>\",\"timestamp\":1600112492}]\r\n"
iex> payload = timestamp <> json
"1600112502[{\"email\":\"hello@world.com\",\"event\":\"dropped\",\"reason\":\"Bounced Address\",\"sg_event_id\":\"ZHJvcC0xMDk5NDkxOS1MUnpYbF9OSFN0T0doUTRrb2ZTbV9BLTA\",\"sg_message_id\":\"LRzXl_NHStOGhQ4kofSm_A.filterdrecv-p3mdw1-756b745b58-kmzbl-18-5F5FC76C-9.0\",\"smtp-id\":\"<LRzXl_NHStOGhQ4kofSm_A@ismtpd0039p1iad1.sendgrid.net>\",\"timestamp\":1600112492}]\r\n"
```

### 署名検証本番

一通り準備できましたので、最後に`:public_key.verify/4`で検証。

```elixir
iex> :public_key.verify(payload, :sha256, decoded_signature, pk)
true
```

🎉 **！！完！！** 🎉

## いろんな紆余曲折

ここまでの書き方だとなんの問題もなく署名検証まで実装できたかのようですが、実はいろんなつまづきがありました。

### One-Line PEM

公式SDKを最初から参考にしていたので、発行される公開鍵がPEM形式であること（Base64文字列であること）は明らかだったのですが、他の言語のOpenSSL系ライブラリでは（言語標準・非標準問わず）すんなり読み込めているようだったので、**Erlangの`public_key:pem_decode/1`で鍵が読み込めない**のにそこそこ時間を費やしました。

あとで、標準PEM形式を復元するというパターンも試してみたのですが、はっきり言ってこっちのほうがその後の流れは楽です。

```elixir
iex> pem = """
...> -----BEGIN PUBLIC KEY-----
...> MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAf
...> SmpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g==
...> -----END PUBLIC KEY-----
...> """
"-----BEGIN PUBLIC KEY-----\nMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE83T4O/n84iotIvIW4mdBgQ/7dAf\nSmpqIM8kF9mN1flpVKS3GRqe62gw+2fNNRaINXvVpiglSI8eNEc6wEA3F+g==\n-----END PUBLIC KEY-----\n"

iex> [pk] = :public_key.pem_decode(pem) |> Enum.map(&:public_key.pem_entry_decode/1)
[
  {{:ECPoint,
    <<4, 243, 116, 248, 59, 249, 252, 226, 42, 45, 34, 242, 22, 226, 103, 65,
      129, 15, 251, 116, 7, 210, 154, 154, 136, 51, 201, 5, 246, 99, 117, 126,
      90, 85, 41, 45, 198, 70, 167, 186, 218, 12, 62, 217, 243, 77, ...>>},
   {:namedCurve, {1, 2, 840, 10045, 3, 1, 7}}}
]
```

これで準備完了です。

:::message
試してみたところ、厳密にはErlangの`public_key:pem_decode/1`が対応していないのは"One-Line"のほうではなく、「ヘッダー・フッターを省略した」PEM形式のようです。
改行しないのはそのまま、ヘッダーとフッターを補うだけでも読み込めるようになります。
:::

### Named Curve

SendGridのECDSAは楕円曲線として`secp256r1`（＝`prime256v1`）を使っていて、これを明示指定すれば当然署名検証できたのですが、初め公開鍵からこの名前付き曲線情報を抽出する方法にたどり着けませんでした。

これもOne-Line PEMを自前デコードしているゆえの障壁なのですが……（↑の標準PEM形式に戻すやり方だと、見て分かる通りPEMデコードした時点で`namedCurve`エントリが得られています）。

とりあえずいいか、と実装を区切って、申し送り事項として呟いたところ、前掲のHaskell版の記事を書かれた[@fumieval](https://twitter.com/fumieval)さんが、「曲線を表すOIDもちゃんとパラメータとして含まれてるはずだ」と示唆してくださったので、再調査して解決できました。（ありがとうございます！）

https://twitter.com/gada_twt/status/1689975183510319104

しかし、`public_key:pem_decode/1`の実装を丁寧になぞっておけば最初からたどり着けたはずだったので、悔しさはある。

## おわりに

とはいえまた一つ`public_key`モジュール周りの取り扱いに習熟できる結果となり、僥倖でした。記事も一つ書けたし。

Erlang/Elixirでもこのように、標準ライブラリで非対称鍵アルゴリズム各種をきちんと扱えるので、落ち着いてやっていきましょう。
