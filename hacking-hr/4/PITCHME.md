## 僕が知りたいのは何をしたら良いのかだった

[@threetreeslight](https://twitter.com/threetreeslight) on Hacking HR! #4

---

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

[@threetreeslight](https://twitter.com/threetreeslight)
<br>
VP of Engineering at [Repro](https://repro.io)

最近は人事部長やEvent Organizer おじさんと揶揄されます 😇

---

# What's Repro?

---

# MA tool


![](/assets/images/repro/repro-service.png)

---

- 4桁万DAUのデータを処理・分析
- 数億のプッシュを毎日配信

---

グローバルで

## とりあえずReproいれとこ

を目指してます

---

# はじめます

---

## みなさまは応募者への対応管理は尽くされているので

---

## 今日話すこと・話さないこと

- 話すこと
  - 採用管理をしてやりたい・やりたかったこと
- 話さないこと
  - ATSを使ってapplicant対応の管理や日程調整する方法
  - 面談のプロセスやアテンドの管理

---

# 当時

---

## 採用業務ちゃんとやんなきゃ

![](/assets/images/meme-wtf.png)

---

## 管理を始める

talent poolの形成と応募者プロセスおよび対応を管理。

他のツールちょろっと見たけど、最終的には speadsheet + trello + slack を連携していい感じに

![](/hacking-hr/4/candidate-kanban.png)

---

## 思ったよりできた😇

---

## で、何したら良いんだっけ？

![](/assets/images/meme-wtf.png)

---

## そもそも管理とは

---

## 管理原則の父 Henri Fayol 曰く

> 管理とは、計画し、組織し、指揮し、調整し、統制するプロセス
>
> -- [Wikipedia - Management](https://en.wikipedia.org/wiki/Management)

---

そして

## 世のツール、なぜ計画を作るに資する情報出せないのか

---

## ほしいのは問題の発見

組織・指揮・調整・統制などのプロセスの最適化をする前にやることがある。

現状を監視・観測し分析し、問題を発見、そして何をするべきか計画を立てたい。

---

# やりましょう

---

スタートは

# Do not guess, Measure

にある

---

## やること

1. どの点を観測するか決める
1. Raw dataを貯めまくる
1. 負けパターン・損切りパターンを見出し
1. パターンを自動化・仕組み化
1. 各プロセスの目標値を決める

---

## これ、OODAでは？

![](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/OODA.Boyd.svg/2000px-OODA.Boyd.svg.png)

---

## どの点を観測するか決める

観測するべきKPIのtreeを考える。決定した数値については、他社ヒアリングやAgentなどを通してベンチマーク値を把握する。

面談通過率などATSで表示される数値はKPIではない。

---

e.g. KPI Treeの一部

- 採用数
  - オファー許諾率
  - オファー数
    - 採用に値する候補者からの応募数
      - 既存候補者リード数: 当時採用に値して当月動く可能性のある方
      - 既存候補者リード数: 成長ポテンシャルの高く、一定期間経過した方
      - 新規候補者リード数: 採用に値する当月に知り合った方

---

## Raw dataを貯めまくる

必要に応じて過去も掘り返しKPIの数値を出す。

問題のあるKPIへの問題仮説を考え、候補者の属性や面談内容から特徴量を抽出し分析する。

---

e.g. オファー許諾率の特徴量の一部

- オファーするまでの接触回数
- 提供した接触コンテンツごとのpositive/negative
- オファーしてから回答までのインターバル
- 初回面談からオファーするまでのインターバル
- オファーしている（するであろう）競合数

---

## 負けパターンの分析

問題となっているKPIと特徴量の相関を考え、負けパターン・損切りパターンを把握する。

勝ちパターンの再現は難しいため、何をすると負けるのかを把握するのが大事。

---

e.g. 許諾率の負けパターン

- A Positionは、オファーしてから回答までのインターバルがX日以内でないと内定許諾率がベンチマーク以下になる
- 接触時の平均Positive値がX以下の人は許諾率がベンチマークを下回る

---

## パターンを自動化・仕組み化

@color[red](ココやりきれてない)

- 見つけ出されたしきい値をもとに候補者ごとに自動でアラートを出す
- 候補者属性ごとの対応をパターン化する

---

## 各プロセスの目標値を決める

- 先月までのKPI Treeの数値をもとに、当月以降の行動目標数値にfeedbackするようにする

---

e.g. 翌月以降の目標数値を決定

先月までの改善施策の影響は大きい

- 何回オファー出せばよいのか
- 何回人と会わなければいけないのか

---

全て

# 😇 `speadsheet` 😇

アプリ作るかsalesforce add-on作るか悩んでる

---

### お気持ちの表明

![](/hacking-hr/4/tweet-salesforce.png)

---

## とはいえ悩ましいところ

- 面談などを通した定性情報の入力に際し、均一な判断基準を作るのが難しい
- 採用活動をする担当者が少ないとタレントプール作りと採用の波ができることを意識して活動する必要がある
- チームのリーダークラスの人間が状況に応じて自身の活動を判断することができない

---

# 頑張っていく 💪

---


## そんなデータドリブンな意思決定と積極的な行動をして生きていきたいあなたに

---

# 👋 We Are Hiring 👋

![](/assets/images/repro/repro-logo-colored.png)


---

# Appendix

---

## Reference

- [Linkedin - talent blog](https://business.linkedin.com/talent-solutions/blog)
- [CNBC - CAREER](https://www.cnbc.com/make-it/careers/)
- [GreenHouse - Structured Hiring 101: Your Blueprint for Success](https://resources.greenhouse.io/recruiting-ebooks/structured-hiring-101-4)
- [GreenHouse - 5 Recruiting Key Performance Indicators](https://resources.greenhouse.io/recruiting-ebooks/5-recruiting-key-performance-indicators-3)
- [Google - How we hire](https://careers.google.com/how-we-hire/)
- [Amazon - Interview at Amazon](https://www.amazon.jobs/en/landing_pages/interviewing-at-amazon)

---

## Globalのproductの筋の良さを感じる

- [salesforce](https://www.salesforce.com/jp/?ir=1)
- [Workable](https://www.workable.com/)
- [Greenhouse](http://www.greenhouse.io/recruiting)
- [Bamboo HR](https://www.bamboohr.com/)
- [Lever](https://www.lever.co/)
- [recruitee](https://recruitee.com/)
- [Hire by Google](https://hire.google.com)
