## CiecleCIでもくもく会を<br>支える技術

@css[span-70 text-07 text-right](CircleCI ユーザーコミュニティミートアップ) on 2019-08-23

by [@threetreeslight](https://twitter.com/threetreeslight)

---

# こんにちわ！

---

@snap[north]
<br>
### whoami
@snapend

@snap[west span-70 text-08]
<ul>
<li> NTT系SIer (SE -> sales)
<li> 音楽系スタートアップ CTO
<li> メディア系スタートアップ 創業
<li> 国内EC 創業
<li> 越境系発送代行サービス 1号エンジニア
<li> Repro 創業兼CTO <br>&nbsp;&nbsp;-> VP of Engineering <br>&nbsp;&nbsp;-> VP of PeopleOps
</ul>
@snapend

@snap[east span-30 text-08 text-center]
[![](https://www.gravatar.com/avatar/0a918b7637fcfafeb06264db039552df?s=190)](https://twitter.com/threetreeslight)
<br>**Akira Miki**
<br> **[@threetreeslight](https://twitter.com/threetreeslight)**
@snapend

@snap[south span-80 text-09]
コードが書けるイベントおじさん @emoji[innocent]
<br><br>
@snapend

---

### 今日話すこと・話さないこと

- 話さないこと
  - ReproでのCircleCI 活用の詳細
- 話すこと
  - ReproとCircleCI利用どころ
    - **↑スポンサートークっぽいの一応大事↑**
  - CircleCIでイベント開催を自動化している話

---

#### 早速ですが
### スポンサートークいきます

---

# What's Repro?

---

## Customer Engagement Platform

![](/wantedly/2019-05-28-real-wantedly/repro-service.png)

---

### Web/App MA tool

- ## Right message
- ## To the right person
- ## At the right time

---

### T2D3

<div class='img-70'>
![](/wantedly/2019-05-28-real-wantedly/t2d3.png)
</div>

@css[text-05](cf. [Tech Crunch - The SaaS Adventure](https://techcrunch.com/2015/02/01/the-saas-travel-adventure/))

---

## Traffic

- 毎日10億を超えるイベントデータを処理
- AIでユーザーを自動セグメント
- 毎日2億近いプッシュなど施策を配信

---

### そんなReproでの
# CircleCI利用

---

## 色々使っています

- Kick to build Image on image build server
- Kick to test on EC2 cluster and collect artifacts
- Attach github label about risk
- Run plan and apply Terraform

---

## そんなハイトラフィックを楽しみにたい方へ


---?color=white

<div class='hiring'>
@snap[west]
![](/wantedly/2019-05-28-real-wantedly/stack-cassandra.png)
<br>
![](/wantedly/2019-05-28-real-wantedly/stack-presto.png)
@snapend

![](/wantedly/2019-05-28-real-wantedly/stack-ruby.png)
![](/wantedly/2019-05-28-real-wantedly/stack-golang.png)
<br>
## @color[#FF4E6A](We are hiring)
<br>
![](/wantedly/2019-05-28-real-wantedly/stack-kafka.png)
![](/wantedly/2019-05-28-real-wantedly/stack-scikit-learn.png)

@snap[east]
![](/wantedly/2019-05-28-real-wantedly/stack-fluentd.png)
<br>
![](/wantedly/2019-05-28-real-wantedly/stack-vuejs.png)
@snapend

@snap[south]
@snapend
</div>

---

# 宣伝
# 終了

---

### それでは本題
# 参ります

---

### イベント
## 継続は大変

1. 企画するのが大変
1. 作成するのが面倒
1. 運営に気を使う

---

## これを解決していく

---

### 企画するのが大変

もくもく会の内容を固定し、企画要素を削る

![](circleci/2019-08-23/shinjuku-mokumoku-banner.png)

---

### 作成するのが面倒

オーガナイザーが複数いる場合は特に見合う。

=> 自動化しましょう

---?code=circleci/2019-08-23/event_publisher.js&lang=js&title=Setup Config

@[16](3W後の開催日を指定)
@[18-24](イベントのメタ情報を突っ込む)
@[53-58](民度の担保を目的としたアンケートも作る)
@[66](イベント本文の位置を指定)

---?code=circleci/2019-08-23/connpass_event_creator.js&lang=js&title=Automatically Create event with Puppeteer

@[23-29](めちゃめちゃがんばる)
@[78-86](アニメーションに殺意を覚える)

---?code=circleci/2019-08-23/config.yml&lang=yml&title=Config circleci

@[7-8](puppeteer install orb作って)
@[96-104](nightly buildする)

---

@snap[north]
### 訪れる幸せ @emoji[heart_eyes]
@snapend

@snap[south-east]
![](circleci/2019-08-23/connpass-page.png)
@snapend
@snap[west]
![](circleci/2019-08-23/connpass-build.png)
@snapend

---

### ちなみに

- circleciと日本からではconnpass serverへのnetwork latencyが異なるので(致し方がない)sleepの微調整が必要

---

## 運営に気を使う

timetableにあわせ、いちいちslackのreminder設定したり連絡したりするの辛い。

=> firebase function + slash commandで凌ぐ

---?code=circleci/2019-08-23/prepare.js&lang=js&title=slack remind and command

@[15-19](開催回のchannel作って)
@[25-28](通知を設定)
@[34-38](諸注意も再連絡しつつ)
@[41-42](公開されていないSlack APIを使ってコマンドを叩き)
@[54-60](終了のリマインドまで入れる)

---?code=circleci/2019-08-23/deploy_functions.js&lang=js&title=deploy firebase

@[27-36](変更があるときだけfirebase functionsにdeployされるようにしておき)

---?code=circleci/2019-08-23/config.yml&lang=yaml&title=build and deploy

@[61-63](build & deployしていく)

---

@snap[north]
### 訪れる幸せ @emoji[heart_eyes]
@snapend

@snap[east]
![](circleci/2019-08-23/event-prepare-1.png)
@snapend
@snap[west]
![](circleci/2019-08-23/event-prepare-2.png)
@snapend

---

### ちなみに

- communityで使うfirebaseの登録クレカ、どうするか悩みますよね

---

## まとめ

- 誰やるにならないように常に自動化
  - puppeteerの可能性無限大
  - slackのslash commandの可能性無限大
- CIにまかせていく
  - sheduled workflow便利! orb便利!
  - network latencyには気をつける

@size[0.8em](時間見つけてイベント自動作成が外でもできるよう<br>package化していこう)



