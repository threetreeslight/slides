## Getting Started Istio

@threetreeslight

on shinjuku mokumoku programming #17

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

コード書く時間がないので

# 自前のサービス運用を通して知見を得る

---

### わたしのぶろぐ(月2万)

![](/shinjuku-mokumoku/17/architecture.png)

---

技術の無駄遣い感はんぱないけど

# `Istio` 導入じゃっ！

---

## `Istio` って何？

service mashを実現するサービスだよ

service meshが何かについては以下のブログ見てもらうと分かりやすいかもしれません 😇
[threetreeslight - Why do we need service mesh?](https://threetreeslight.com/posts/2018/09/22/why-do-we-need-service-mesh/)

---

# 早速使っていく

---

## Install

Istioはinstallに必要なkitがいい感じにまとまっているので助かる。それだけsystem componentが多いということも示している用に感じる。

curlでもってきて、 `istioctl` だけpath通しておく

```bash
curl -L https://git.io/getLatestIstio | sh -
cp ./istio-1.0.2/bin/istioctl /usr/local/bin
```

---

## Manual OR Helm?

Istioはいくつかの方法がある

1. helm (kubernates用のpakcage managerだと思ってもらえればOK)
1. kubectlでapplyしていく

Istio公式見解としえはhelmを使って構築されることを期待しているようだ。

---

## `Helm` でGo

ちなみにhelmでinstallできるcomponentsは以下の通り。

https://github.com/istio/istio/blob/master/install/kubernetes/helm/istio/Chart.yaml

```yaml
keywords:
  - istio
  - security
  - sidecarInjectorWebhook
  - mixer
  - pilot
  - galley
```

---

## Monitoring

他にもinstallationとして、prometheus, grafana, servicegraph, tracingなど様々なchartが用意されている。幸せ。

prometheusとgrafanaのchartを参考にできるのはありがたい.

---

## Tracing

tracing chartに含まれているのが `Zipkin` ではなく `jaeger` (https://www.jaegertracing.io/) だった。

jaegerがデファクトなのかな？

---

## ちなみに

公式ドキュメントを見ていてCustom resource definisionを手動で入れてしまい、helmで入らんと嘆く人が多い。

- [Helm delete does not clean the custom resource definitions](https://github.com/istio/istio/issues/7688)
- [Tutorial Helm note update](https://github.com/IBM-Bluemix-Docs/containers/issues/2275)

```sh
# Install Istio’s Custom Resource Definitions via kubectl apply, and wait a few seconds for the CRDs to be committed in the kube-apiserver:
$ kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```

---

istioはkubernates上で利用するcustom resource definisionがあり、helm 2.10.0より前だと作成できないので手動でそのresourceをapplyする。

# もちろん私も踏んだ。

---

## 意気揚々にvalidate

あるぇ、、、prometheus serviceが立ってる、、、

```sh
% kubectl get svc -n istio-system
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                                                                                                                   AGE
istio-citadel              ClusterIP      10.59.252.14    <none>         8060/TCP,9093/TCP                                                                                                         1h
istio-egressgateway        ClusterIP      10.59.252.179   <none>         80/TCP,443/TCP                                                                                                            1h
istio-galley               ClusterIP      10.59.247.233   <none>         443/TCP,9093/TCP                                                                                                          1h
istio-ingressgateway       LoadBalancer   10.59.255.51    35.197.96.12   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:32227/TCP,8060:30881/TCP,853:32278/TCP,15030:31463/TCP,15031:32115/TCP   1h
istio-pilot                ClusterIP      10.59.245.42    <none>         15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                     1h
istio-policy               ClusterIP      10.59.246.86    <none>         9091/TCP,15004/TCP,9093/TCP                                                                                               1h
istio-sidecar-injector     ClusterIP      10.59.249.91    <none>         443/TCP                                                                                                                   1h
istio-statsd-prom-bridge   ClusterIP      10.59.245.16    <none>         9102/TCP,9125/UDP                                                                                                         1h
istio-telemetry            ClusterIP      10.59.252.157   <none>         9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                     1h
prometheus                 ClusterIP      10.59.252.174   <none>         9090/TCP                                                                                                                  1h
```

---


## mixierの依存っぽい

componentsのenable,disableを制御できるようなので以下のコマンドで再構築すると良い。

```sh
% helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
--set prometheus.enabled=false
```

よく `sidecarInjectorWebhook` の制御はこれで行うらしい。気持わかる

---

## Deploying

というわけで、自前のサービスのnsを整理して、development環境から `istio-injection=enabled` flagを付与していく

```sh
% kubectl label ns staging istio-injection=enabled
% kubectl get ns --show-labels
NAME           STATUS    AGE       LABELS
default        Active    146d      <none>
development    Active    2m        istio-injection=enabled
istio-system   Active    2h        name=istio-system
kube-public    Active    146d      <none>
kube-system    Active    146d      <none>
production     Active    2m        <none>
staging        Active    2m        <none>
```

---

# nsガチャガチャいじったら権限とかいろいろぶっ壊れた <- イマココ

---

# 以下余談

---

## What's CRD?

ubernatesにはcustom resourceという概念がある。
あたらしく独自リソースを定義することで、汎用的なリソースとして、kubectlやkubernatesのAPIを利用した操作であったり、権限管理を行うことができるようになる。

---

### Consider API aggregation if:

- Your API is Declarative.
- You want your new types to be readable and writable using kubectl.
- You want to view your new types in a Kubernetes UI, such as dashboard, alongside built-in types.
- You are developing a new API.
- You are willing to accept the format restriction that Kubernetes puts on REST resource paths, such as API Groups and Namespaces. (See the API Overview.)
- Your resources are naturally scoped to a cluster or to namespaces of a cluster.
- You want to reuse Kubernetes API support features.

---

### 典型的な宣言的なAPI

- 小さいオブジェクトやリソースから成り立っている
- アプリもしくはインフラから定義・設定されることがあり
- よく更新される
- 運用ために読み書きを行う
- 基本的な操作はCRUD
- 複数のオブジェクトをまたいだトランザクションが無い

---

### configmap + podをcustom resourceとして管理しちゃったほうが良いのかな？

と思ったら書いてあった

- config file formatがあるものだったらconfigmap
- podのプログラムの設定ファイルだったらconfigmapのが楽
- kubernetes apiより環境変数で定義しちゃいたくなるものであればconfigmap
- rolling updateを必要とするものだったらconfigmap

うーむ。








