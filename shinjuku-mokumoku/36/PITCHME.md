### Which Static Site Generator & Hosting Service
## is best?

@threetreeslight

on shinjuku mokumoku programming #36

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

VP of Engineering at [Repro](https://repro.io)

実態は全社の人事オジサン😇

---

## 今日やること

- static site generator何を使うか？
- どこでhostするか？

---

## why

静的サイトを２つ作る需要が出てきた

1. 海外のイケイケHR記事を翻訳して日本に広めるサイト
1. ReproのCultureやstyleが伝わるサイト

---

## 要件

1. custom domain, ssl対応が可能
1. コンテンツ表示は早いほうが良い
1. blogとして扱うのに手間がかからない
1. 非エンジニアが触って編集できる(git client使って編集ぐらいが限界)
1. 何なら閲覧者からcontributeしてもらう
1. 個人的に**wordpressはもう触りたくない**

---

### About
# Static Site Generator

---

## Candidates

ここらへんを検討。ざっくりと

- [Nuxt](https://nuxtjs.org/)
  - vuejs + vue-router
- [Gatsby](https://www.gatsbyjs.org/)
  - Reactjs + GraphQL
- [vuepass](https://vuepress.vuejs.org/)
  - vuejs + vue-router
- [Hugo](https://gohugo.io/)
  - go

---

## Nuxt

- バギーである、blog libraryはあるが、ちゃんとSEO対策されたblogとして作り込むには自前で頑張る部分が結構でるだろう
- ある程度自由度高く作りたい場合はこれを使いたい。なぜならsingle file componentの利点を使えるのは嬉しい。
- [repro.io](https://repro.io)のサービスサイトはnuxtで作ってたりしてるし

---

## Gatsby

- バギーである、blog libraryはあるが、ちゃんとSEO対策されたblogとして作り込むには自前で頑張る部分が結構でるだろう
- GraphQLいるのか？と思ったけど、[contentful](https://www.contentful.com/)を使ってdataをhostingするのであれば何かとよさそう
- JSXをガンガン書いていくスタイル。嫌いじゃない。
- 個人的にReact書いたことないので触っててナルホド感多く楽しい。

---

## vuepass

- technical documentを書くことに特化したstatic site generatorだからnuxtより良いぜという紹介。
- 実際ソレ用の機能がほぼデフォルトで同梱されて居ることも良い。
- Docsify / DocuteよりもSEO friendlyであるとのこと。
- しかし触ってみると、何かしっくりこないなぁという雰囲気を醸し出す。

---

## Hugo

- おなじみのstatic site generator。高速さとテーマの豊富さが売り
- plugin機構は無く、とにかく詰め込まれている。
- 自分もよく使っているので勘所も分かっており何かと使いやすい。

---

## 結論

まだ迷っている。

1. とにかく構築コストがかかるのがテーマ。この点、楽になりたい。
1. そうするとGatsbyとNuxtは楽しい。初期構築コストが低くない問題がある
1. もう少し各種ツールを触り込み **楽しさ** && **楽さ** で意思決定していく

---

### About
# Hosting

---

## Ideas

1. firebase hosting
1. Netlify
1. Google LB + Google Cloud Storage
1. Cloudfront + S3
1. Github Pages
1. surge.sh

---

## GCS OR S3

- 構築経験ある身からすると、static siteのhostingにおいて学習以外でこの環境を構築する必要はあまりない。

---

## Github Pages

- Custom Domain & SSLも対応しているし何かと便利。
- だが、困ったらこれだが、GithubPagesってCDNで配布されていないので、チョッパヤにするのであれば不安が残る

---

## Firebase Hosting

- CDNで巻いてくれるタダのhostingとかヤバイ
- custom domain + sslも対応している

---

## Netlify

1. あんま使ったことないけど楽と聞く
1. github pages と同レベルの感覚

---

## surge

- 正直わからん。詳しい人いたら良いところおしえて

---

## 結論

firebase hostingが最強すぎるので一旦これを使う

ただしcontentfulなどを使う場合はcdnの恩恵の偉大さは減るので、ぶっちゃけどれでやっても一緒

---

# 検討を進める

