# ymtszw's Zenn Contents

## 執筆方法

- プレビューサーバ起動

```sh
npm ci
npm start
# http://localhost:8080 でプレビュー
# http://localhost:8081 でMarpスライドモード
```

- 新規記事追加

```sh
npx zenn new:article --slug <slug>
```

## Marpスライド

[Marp](https://marp.app/)を使っているので、Zenn記事と同時にMarpスライドであるようなMarkdownを作成できる。

Marpスライドとしては <http://localhost:8081> でプレビューできる。

[参考](https://ymtszw.github.io/articles/many-talks)
