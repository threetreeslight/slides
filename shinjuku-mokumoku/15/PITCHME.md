## What is service mesh?

@threetreeslight

on shinjuku mokumoku programming #15

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

## [Distributed System?](https://en.wikipedia.org/wiki/Distributed_computing)

ネットワーク上の異なるコンピュータ上にコンポーネントが配置されているシステム

e.g. cloud系のmanaged serviceを使ったsystem

---

## 牧歌的な分散システム

![](http://philcalcado.com/img/service-mesh/4.png)

シンプルなサービスであればこんなイメージで行けるが、考えるべきことは多い 🤔

---

### [Fallacies of distributed computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)

1. ネットワークは落ちない
1. 遅延はゼロ
1. 無限の帯域
1. セキュア
1. 変更されないトポロジ
1. 一人の管理者
1. 転送コストはゼロ
1. 均一なネットワーク

---

# 誤った考えには対策しなきゃいかん

---

## さらに求められること

- Rapid provisioning of compute resources
- Basic monitoring
- Rapid deployment
- Easy to provision storage
- Easy access to the edge
- Authentication/Authorisation
- Standardised RPC
- etc...

---

## これはやばい

---

## 対策としての

1. service discovery
1. circuit breakers

---

# Service Discovery

---

## What's Service Discovery

特定の要求を満たすサービスのインスタンスやコンテナを自動的に見つけるプロセスのこと

---

## Typically Service Discovery

以下のhealth checkによって実現される事が多い

1. DNS
1. load balancer

---

## more complex requirement

1. clientの負荷分散
1. staging, productionなどの異なる環境
1. 別リージョン、クラウドに散らばる

となると典型的な方法では苦しい、、、

---

# circuit brakers

---

## concern of remote request

- Remote Requestは、timeoutになるまでhangする
- その状態で多数のrequestがくるとresourceを食いつぶして死ぬかもしれない

ref [Martin Fowler - CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)

---

## 何度も同じことをする

すべてのサービスにおいて同じようなコードを同じように書きまくる必要がある

---

# こんな感じ

開発者はこれを意識しなければいけない

![](http://philcalcado.com/img/service-mesh/5.png)

---

# 上記を踏まえた昨今

---

サービス規模・人員規模がそれなりになってくると出てくること

- マイクロサービスいっぱい
- AWS, GCPの楽しいマルチクラウド

いわゆる distributed microservice architecture

---

## 開発者もDevOpsも死ぬ

- 可搬性・スケーラビリティを考えたマイクロサービス設計
- 必要とされている知識は利用するクラウドとサービス数だけある
- 監視のための各種サービスのlog, metricsの収集、そのフォーマットの共通化
- サービス間の**認証・認可によるアクセス制御**

---

## 特に認証認可は、、、

上記の問題を解決するために言語向けSDKを提供していた。が

- サービスごとにリリースサイクルは異なる
- SDK提供する言語でしか効果がない

結果、SDKが揃わない。開発者はこれを意識しなければいけない

---

## しょうがないから、、、

serviceと分離し、sidecar でいく

![](http://philcalcado.com/img/service-mesh/6-a.png)

---

## そこでIstio

> provides a uniform way to secure, connect, and monitor microservices.

1. 勝手にサイドカー立てる
1. 上記の問題を解決するための設定を外部から注入

---

## ref

- [SOTA - Service meshとは何か](https://deeeet.com/writing/2018/05/22/service-mesh/)
- [Phil Calçado - Pattern: Service Mesh](http://philcalcado.com/2017/08/03/pattern_service_mesh.html)
- [The mechanics of deploying Envoy at Lyft](https://schd.ws/hosted_files/kccncna17/87/Kubecon%2012%252F17.pdf)
- [Orilly - seeking SRE](http://shop.oreilly.com/product/0636920063964.do)
- [Introducing Istio Service Mesh for Microservices](https://developers.redhat.com/books/introducing-istio-service-mesh-microservices/)
- [Martin Fowler - CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)
