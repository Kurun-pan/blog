---
title: "How to use Hugo"
date: 2020-08-16T06:38:27+09:00
tags: ["hugo"]
bigimg: [{src: "/img/path.jpg", desc: ""}]
draft: false
comments: true
---

Github Pages + Hugoでブログを投稿するために最低限使うコマンドの備忘録です。
テンプレートとして利用しているテーマは[`Beautiful Hugo`](https://themes.gohugo.io/beautifulhugo/)です。

### 新しい記事を作成する

```
$ hugo new content/post/{適当な文字列。何でも良い}.md
```

### 作成した記事を確認する

```
$ hugo server -D
```

### リリース用データ (publicディレクトリ) を作成する

```
$ hugo
```

<br>

## 参考
-------

- [GitHub PagesとHugoでブログをつくった](https://uzimihsr.github.io/post/2019-08-07-create-blog-1/)
- [Hugoのテーマ一覧](https://themes.gohugo.io/)
