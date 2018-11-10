## Istio Traffic Management

@threetreeslight

on shinjuku mokumoku programming #21

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

最近は人事部長やEventおじさんと揶揄されます 😇

---

## 最近はtypingおじさんとまで 😇

@div[left-50]
![](/shinjuku-mokumoku/21/repro-2018-11-09-type-1.png)
![](/shinjuku-mokumoku/21/repro-2018-11-09-type-3.png)
@divend

@div[left-50]
![](/shinjuku-mokumoku/21/repro-2018-11-09-type-2.png)
@divend

---

# やっていきっ 💪

---

## 今日やること

- Istio pilotのresouceが多いので、そこらへんの調整
- Istioの設定、および監視の設定

---

## できたこと

- 🙅‍♀️Istio pilotのresouceが多いので、そこらへんの調整
- 🙅‍♀️Istioの設定、および監視の設定

---

## なにをしていたか

GKEを`1.9.7-gke.5` -> `1.9.7-gke.7` にupgradeしたらいろいろ死んだ！

なのでIstioのtraffic managementについて学び直す

---

## What is Istio?

kubernetes上にservice meshを実現するフレームワークです

---

### Istio traffic management can ...

@div[left-80]
![](https://istio.io/docs/concepts/traffic-management/TrafficManagementOverview.svg)
@divend

---

## ここが嬉しい

1. compornent間の通信を宣言的に書ける
1. canary releaseが簡単にできる
1. trafficをmirreringできる
1. latencyを仮想的に実現できる
1. circit braker機能を宣言的に書ける

---

## それが実現できるということは

それなりに設定を書く必要がある

1. 定義したpodは基本的にenvoy (sidecar proxy server) にしかアクセスできない
1. 基本的にenvoyは外部へも通信を許可しない（どこ向けのアクセスか把握できないため？）
1. トラフィックルーティングを定義することで通信が可能となる

---

kubernetesのservice, ingressの考えを

# 一旦捨てる

必要がある

---

## Custom Resource

新たに登場するresourceたち

1. DestinationRule
1. VirtualService
1. Gateway
1. ServiceEntry
1. DestinationPolicy

---

## `DestinationRule`

- 通信先として仮想的なhost名を定義する
- subsetsを更に定義でき、content(userなど)やversion(canary)など通信先を細かく制御できる

---

## `VirtualService`

- 公開する通信先を定義する

---

## `Gateway`
- Istio ingressへの通信をどこにproxyするか決める

---
## `ServiceEntry`

- 外部への通信について設定を記述する

---

## `DestinationPolicy`

- 通信先ごとのcircit braker機能を記述

---

遊んでみて大体イメージが付いたので

# 頑張っていく 💪
