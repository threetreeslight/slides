## What is the best way to collect log on kubernates?

@threetreeslight

on shinjuku.rb #64

---

# whoami

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

@threetreeslight / VPoE at Repro

最近はイベントおじさん

---

# 突然ですが

---

## kubernatesはいいぞ（小並感）

---

そんなReproは

# ECS

---

## 🤗
自身のblogをk8sで運用した経験からのお話です

---

# 閑話休題

---

## 悩ましいcontainer log

container logは集約したいけど。。。

1. どこに集約(logging backend)する?
1. そのためのlog driver何にする？
1. log aggerate先が落ちても大丈夫？
1. node単位で集約してから送る？
1. どんなlogを集める？

etc...

---

## k8sの方針を知る

---

## logging architecutre

![](https://d33wubrfki0l68.cloudfront.net/59b1aae2adcfe4f06270b99a2789012ed64bec1f/4d0ad/images/docs/user-guide/logging/logging-node-level.png)

1. container内のapplicationはstdout, stderrにログを吐く
1. nodeの `/var/log`, `/var/log/docker` にlogが集約される
1. logrotateで世代管理

ref. [kubernates - logging at the node level](https://kubernetes.io/docs/concepts/cluster-administration/logging/#logging-at-the-node-level)

---

## Cluster-level logging

cluster-levelのlogを集約する方法は標準で用意されていない。

そのため、自前でlog aggregateする必要がある。

（よく考えられているなぁ 🤔）

---

## Approaches

1. 各nodeにlog aggregateするagentいれて集約先に送る
1. podにlogging用side car containerつけて送る
1. applicationのsidecar contaienrから集約先に送る

c.f. [kubernates - Cluster-level logging architectures](https://kubernetes.io/docs/concepts/cluster-administration/logging/#cluster-level-logging-architectures)

---

## Using a node logging agent

![](https://d33wubrfki0l68.cloudfront.net/2585cf9757d316b9030cf36d6a4e6b8ea7eedf5a/1509f/images/docs/user-guide/logging/logging-with-node-agent.png)

nodeにlogging agent を置いて、集約先(logging backend) に送るという最も一般的な方法。

---

## Pros/Cons

- Pros
  - logging agentの生き死にapplicationが影響を受けない
  - logging agentのupdateが楽
- Cons
  - applicationはstdout, stderrでしかlogを転送できない
  - node logging agentが詰ったら上手くスケールさせる必要がある

---

### Using a sidecar container with the logging agent

![](https://d33wubrfki0l68.cloudfront.net/c51467e219320fdd46ab1acb40867b79a58d37af/b5414/images/docs/user-guide/logging/logging-with-streaming-sidecar.png)

Streaming sidecar containerをつけてログストリームを分割する

---

## Pros/Cons

- Pros
  - stderr, stdoutへの書き込みをサポートしていないミドルウェアを使った時に使える
  - lostしてはいけないlogのbufferingやretryの制御を行いたい
- Cons
  - pod終了時の振る舞いに気をつける必要がある

他にもProsがありそう? 🤔

---

### Using a sidecar container with the logging agent

![](https://d33wubrfki0l68.cloudfront.net/d55c404912a21223392e7d1a5a1741bda283f3df/c0397/images/docs/user-guide/logging/logging-with-sidecar-agent.png)

sidecar container から直接 logging backend にlogを送る

---

## Pros/Cons

- Pros
  - node logging agentの負荷を気にする必要がない
  - node のscale out/inに影響を受けない
- Cons
  - kubeletのlogs機能を使ったlogの監視をすることが出来ない
  - logging backendが落ちたときのhandleを考慮する必要がある

---

### Exposing logs directly from the application

![](https://d33wubrfki0l68.cloudfront.net/0b4444914e56a3049a54c16b44f1a6619c0b198e/260e4/images/docs/user-guide/logging/logging-from-application.png)

applicationからlogging backendに直接送る男らしいやり方

---

## Pros/Cons

- Pros
  - 構成としてシンプル
  - node logging agentの負荷を気にする必要がない
  - node のscale out/inに影響を受けない
  - podの終了に際し、コンテナ間の終了順序を意識する必要がない
- Cons
  - kubeletのlogs機能を使ったlogの監視をすることが出来ない
  - logging backendが落ちたときのhandleを考慮する必要がある
  - buffering, retryなどを実装する必要がある

---

## 落ちない超強いlogging backendがあればシンプルな構成にできるがそんなものはない！

---

## 12 Factors App

> アプリケーションはログファイルに書き込んだり管理しようとするべきではない。
> 代わりに、それぞれの実行中のプロセスはイベントストリームをstdout（標準出力）にバッファリングせずに書きだす。

c.f. [The Twelve-Factors App - XI. log](https://12factor.net/ja/logs)

---

## Accepted Approach

こんな感じ？ 🤔

- Using a node logging agent Approachを採用
  - GKEのdefaultもコレ
- 必要に応じてsidecar agentを採用する
  - Applicationの機能として、stdout, stderrへの出力が貧弱なときに中間約として利用する
  - node logging agentの負荷に大きく影響を与えるログがあるときに直接logging backendに送る

---

# Logs

---

## target logs

収集すべきログはこんな感じ？

1. application log
  1. container log
1. system component log
  1. node(host) system log
  1. docker daemon log
  1. k8s(kubelet, api, etcd) log

---

# いっぱいある 😇

---

# 見ていく 👀

1. どんなログ
1. どこに置かれているのか

---

# container log

---

## k8s log location

> On machines with systemd,
> 
> - the kubelet and container runtime write to journald.
> - If systemd is not present, they write to .log files in the /var/log directory.

つまり、 `/var/log` をnode logging agent containerで収集しておけば良いので楽ちん！

c.f. [kubernates - Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

---

## `/var/log/containers`

container logが上記のpathにフラットに配置されている。

これは実態ではなくsymlinkであり、kubeletがこの手のバイパス作業を実施している。

```sh
# format
# pod_name + revision + namespace + container_name + container sha1
blog-84bd9f6d5b-9f6kt_default_blog-e3e2ad507585302aa3d77cc3670ffd3b86263bbff896ec489ddb42eb1c7f214e.log -> /var/log/pods/b5765594-98b2-11e8-8483-42010a8a0017/blog_0.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_fluentd-gcp-6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/fluentd-gcp_11.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_fluentd-gcp-8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/fluentd-gcp_10.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_prometheus-to-sd-exporter-2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/prometheus-to-sd-exporter_0.log
```

---

## `/var/log/pods`

container logのsymlink先。pod単位のlogが配置されている。

こうするメリットはなんだろう？ 🤔

```sh
# ls -la /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140
fluentd-gcp_10.log -> /var/lib/docker/containers/8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07/8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07-json.log
fluentd-gcp_11.log -> /var/lib/docker/containers/6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655/6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655-json.log
prometheus-to-sd-exporter_0.log -> /var/lib/docker/containers/2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234/2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234-json.log
```

---

### `/var/lib/docker/containers/`

docker標準のlocal strage配置場所。この中に各種設定情報も置かれている。
この中のlogだけをsymlinkで引っ張っている。

c.f. [docker - About storage drivers](https://docs.docker.com/storage/storagedriver/)

```sh
# /var/lib/docker/containers
071b3c094a4332044997fb425aa95348d324ff6fc5f02d3311cd9a88ce999216
09af55d72e42330f1903cab2fb42f274bc2c5ee24149d72e69356e1faca64879
0aa68a6f97640043365d080b64c557379271f5ef1730763537772e3ecd695659
17834a541c748564ebb79772cafd8a6858f095fb46334778550d84e33ab51627
2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234
40730ac5da23aa22dcfc2baac15efd31dc82445c168d9230506930e5575e840d
556d1b7d52f70e4e0f17e4a0021a493048cb34aae1b52036799ca31f2def4b3a

# /var/lib/docker/containers/2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234
2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234-json.log
checkpoints
config.v2.json
hostconfig.json

# tail 2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234-json.log
{"log":"I0811 03:42:05.658378       1 main.go:83] GCE config: \u0026{Project:threetreeslight Zone:us-west1-a Cluster:blog-cluster Instance:gke-blog-cluster-pool-1-767a6361-7t2r.c.threetreeslight.internal MetricsPrefix:container.googleapis.com/internal/addons}\n","stream":"stderr","time":"2018-08-11T03:42:05.658698744Z"}
{"log":"I0811 03:42:05.658464       1 main.go:115] Running prometheus-to-sd, monitored target is fluentd localhost:31337\n","stream":"stderr","time":"2018-08-11T03:42:05.658801097Z"}
```

---

# system component log

---

## default GKE node logging agent collect

```sh
/var/log/salt/minion
/var/log/startupscript.log
/var/log/docker.log
/var/log/etcd.log
/var/log/kubelet.log
/var/log/kube-proxy.log
/var/log/kube-apiserver.log
/var/log/kube-controller-manager.log
/var/log/kube-scheduler.log
/var/log/rescheduler.log
/var/log/glbc.log
/var/log/cluster-autoscaler.log
```

---

## `/var/log/salt/minion`

もしや、GKEはnodeのprovisioningにsaltstackを使うの? 🤔

GEKのnodeの中を見ても存在しなかった。

c.f.

- [slatstack](https://saltstack.com/about/)
  - イベント駆動のオーケストレーションツール
- [kubernates - Configuring Kubernetes with Salt](https://kubernetes.io/docs/setup/salt/)

---

## `/var/log/startupscript.log`

cloud init みたいなやつのlog。素のGKEでは利用されない。

c.f. [google cloud - 起動スクリプトの実行](https://cloud.google.com/compute/docs/startupscript?hl=ja)

---

## kubernates logs

[kubernates - kubernates components](https://kubernetes.io/docs/concepts/overview/components/)

- `/var/log/kubelet.log`
  - nodeの制御やcontainer起動などを行うcontainer. ecs agentみたいなやつ。
- `/var/log/kube-proxy.log`
  - dynamic port mappingを実現しているやつ。設定はkube apiを利用して行われる
- `/var/log/kube-apiserver.log`
  - 様々な操作はすべてkube apiserverを経由して処理される
- `/var/log/kube-controller-manager.log`
  - node, replication, endoint, service account のcontroller log
- `/var/log/kube-scheduler.log`

---

## `/var/log/kube-proxy.log`

serviceにおけるpodの起動・終了で以下のようなlogが流れます

```
I0603 15:38:32.507621       1 proxier.go:345] Removing service port "default/monitoring:prometheus"
I0603 15:38:32.507679       1 proxier.go:345] Removing service port "default/monitoring:grafana"
I0603 15:40:53.472870       1 proxier.go:329] Adding new service port "default/monitor:prometheus" at 10.59.249.194:9090/TCP
I0603 15:40:53.472930       1 proxier.go:329] Adding new service port "default/monitor:grafana" at 10.59.249.194:3000/TCP
I0603 15:40:53.493729       1 proxier.go:1769] Opened local port "nodePort for default/monitor:grafana" (:30110/tcp)
I0603 15:40:53.493840       1 proxier.go:1769] Opened local port "nodePort for default/monitor:prometheus" (:30980/tcp)
```

---

## `/var/log/etcd.log`

kubernatesの設定を伝搬するやつ

- [coreos/etcd](https://github.com/coreos/etcd)

---

## `/var/log/rescheduler.log`

これなに？ 🤔

---

## `/var/log/glbc.log`

GCE Load-Balancer Controller (GLBC) のlog.

GKEでingress使う場合はデフォルトでgoogleのloadbalancing機能が使われる。

- [kubernates - cluster-loadbalancing/glbc](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/cluster-loadbalancing/glbc)

---

## `/var/log/cluster-autoscaler.log`
k8s clusterのauto scalingに関わるログが出力される（はず）

- [kubernates - Cluster Autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler?hl=ja)

---

# いっぱいある 😇

---

## まとめ

- application <-> logging agentとのやりとりは直接pushせず、fileを使ってなるべく疎結合にする
- kubernatesのlog収集には Using a node logging agent Approachをやろう
- 監視対象は `/var/log` をみよう

それだけで大丈夫!

---

## 参考資料

- [Fluentd Blog - Unified Logging Layer: Turning Data into Action](https://www.fluentd.org/blog/unified-logging-layer)
- [fluentd - Buffer Plugin Overview](https://docs.fluentd.org/v1.0/articles/buffer-plugin-overview)
- [The Twelve-Factors App - XI. log](https://12factor.net/ja/logs)
- [kubernates - logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [kubernates - Configuring Kubernetes with Salt](https://kubernetes.io/docs/setup/salt/)
- [slatstack](https://saltstack.com/about/)
- [infoQ - LyftがPuppetからSaltStackにリプレース](https://www.infoq.com/jp/news/2014/09/lyft-moves-to-saltstack)
- [google cloud - Puppet、Chef、Salt、Ansible での Compute Engine の管理](https://cloud.google.com/solutions/google-compute-engine-management-puppet-chef-salt-ansible?hl=ja#salt)
- [coreos/etcd](https://github.com/coreos/etcd)
- [kubernates - cluster-loadbalancing/glbc](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/cluster-loadbalancing/glbc)
- [kubernates - Cluster Autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler?hl=ja)
