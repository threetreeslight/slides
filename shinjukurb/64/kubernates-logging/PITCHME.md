## What is the best way to collect log on kubernates?

@threetreeslight

on shinjuku.rb #64

---

### logging architecutre on kubernates

![](https://d33wubrfki0l68.cloudfront.net/59b1aae2adcfe4f06270b99a2789012ed64bec1f/4d0ad/images/docs/user-guide/logging/logging-node-level.png)

stdout, stderrに吐はれてnodeにそのログがたまる。`/var/log`, `/var/log/docker` 配下にlogが蓄積し、logrotateして管理している感じ。

---

### Cluster-level logging

defaultでcluster-levelのlogを集約したりする方法は用意されていない。そのため、自前でlog aggregateする必要がある。

---

### Approaches

kubernatesには、log集約のapproachが綺麗にまとまっていたので良いですね！

1. 各nodeにagentいれて走らせる
1. podにlogging用side car containerをつけて集約
1. applicationからlog serviceに直接送る

docker logdriverで直接送るようにすると接続先がdownしたときにどうにもならなくなるから推奨していないのだろう :thinking_face:

---

### Using a node logging agent

![](https://d33wubrfki0l68.cloudfront.net/2585cf9757d316b9030cf36d6a4e6b8ea7eedf5a/1509f/images/docs/user-guide/logging/logging-with-node-agent.png)

最も一般的な方法。稼働しているapplicationに一切変更することなく、log aggregateすることができる。
そのかわり、stdout, stderrでしかエラーを出力できない。

---

### Using a sidecar container with the logging agent

Streaming sidecar container

![](https://d33wubrfki0l68.cloudfront.net/c51467e219320fdd46ab1acb40867b79a58d37af/b5414/images/docs/user-guide/logging/logging-with-streaming-sidecar.png)

sidecar container を使うことで、ログストリームを分けることができる。

---

### 考えられる使い所

applicationでデータを直接送るのではなく、fluentdなりをsidecar containerとして立ててbufferingやretryを制御したいとき良い。

---

### Using a sidecar container with the logging agent: Sidecar container with a logging agent

![](https://d33wubrfki0l68.cloudfront.net/d55c404912a21223392e7d1a5a1741bda283f3df/c0397/images/docs/user-guide/logging/logging-with-sidecar-agent.png)

---

### 考えられる使い所

- ロストしてはいけない重要なデータストリームを分ける。
- こうすることで、負荷とかそこらへんがhandleしやすくて良い。
- nodeのscalein/outが激しいのでnode上のログを見ておくのが辛い

---

### Exposing logs directly from the application

![](https://d33wubrfki0l68.cloudfront.net/0b4444914e56a3049a54c16b44f1a6619c0b198e/260e4/images/docs/user-guide/logging/logging-from-application.png)

applicationとstickeyになるのであまりおすすめしないアプローチ。


---

### とはいえ、いくつかメリットもある？

1. node levelのscalein/outに対して気遣いが不要となる
1. podのscale out/inに対して sidecar containerの終了に係る気遣いが不要となる

リアルタイムにすべてのデータを受け付けられ、落ちないlogging backendがあるのであれば、この方法もありだと思います。

---

### 12factor's appに従うと

1. Using a node logging agent アプローチを採用する
1. applicationからログのparse, buffering, retry処理を切り離したいのであれば、Streaming sidecar container を一緒に採用する

---
## 収集すべきLogはなにか？

1. node(host)で発生したlog
1. docker daemon log
1. kubernates(kubelet, api, etcd) log
1. container log

あたりかな？ 🤔

そしてそれはどこにあるのか？見ていく

---

## k8s におけるlogの集積

> they write to .log files in the `/var/log` directory.
>
> --[kubernates - Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

おなじみの `/ver/log` 

---

# container log

---

## `/var/log/containers`

kubeletによってrenameされ、 上記pathにcontainer logがflatに配置されている。

これは実態ではなくsymlinkであって、fluentdはここのファイルを監視すれば良いので楽。

```sh
# format
# pod_name + revision + namespace + container_name + container sha1 + lotated_log_name
blog-84bd9f6d5b-9f6kt_default_blog-e3e2ad507585302aa3d77cc3670ffd3b86263bbff896ec489ddb42eb1c7f214e.log -> /var/log/pods/b5765594-98b2-11e8-8483-42010a8a0017/blog_0.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_fluentd-gcp-6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/fluentd-gcp_11.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_fluentd-gcp-8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/fluentd-gcp_10.log
fluentd-gcp-v2.0.9-5kd4h_kube-system_prometheus-to-sd-exporter-2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234.log -> /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140/prometheus-to-sd-exporter_0.log
```

---

## `/var/log/pods`

pod単位のlogが配置されている。

podにおけるcontainer突然死にも対応できる。
podの情報をfilenameに付加しやすくしていて良い。

```sh
# ls -la /var/log/pods/82a88873-9d18-11e8-b3c9-42010a8a0140
lrwxrwxrwx  1 root root  165 Aug 11 05:21 fluentd-gcp_10.log -> /var/lib/docker/containers/8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07/8b951e9978fae5781b55007de8f24a5b336dab9d6812855ffaa5e3f460340a07-json.log
lrwxrwxrwx  1 root root  165 Aug 11 05:33 fluentd-gcp_11.log -> /var/lib/docker/containers/6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655/6330ce359fb5c54012b85c1d6d57eb076578b15fbca1150f341e16f5f61f0655-json.log
lrwxrwxrwx  1 root root  165 Aug 11 03:42 prometheus-to-sd-exporter_0.log -> /var/lib/docker/containers/2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234/2cd439bb6d232379490dcb48f7a39e7c292bc6aba26a6249faf6d9a122697234-json.log
```

---

### `/var/lib/docker/containers/`

container logやcontainer設定が配置されている。

ここにlogの実態がある。

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

# kubernates log

---

## defaultで集めようとしていたlog

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

## journald で収集されている情報

1. [{ "_SYSTEMD_UNIT": "docker.service" }]
1. [{ "_SYSTEMD_UNIT": "kubelet.service" }]
1. [{ "_SYSTEMD_UNIT": "node-problem-detector.service" }]

---

# すんごいっぱいありますね 😇

---

## イマココ話

試しにcontainer logを持ってきて、期待する fluentd 設定に食わせたらreform errorとか悲しい

```sh
tail_1  | 2018-08-11 07:24:25 +0000 [warn]: #0 dump an error event: error_class=Fluent::Plugin::Parser::ParserError error="pattern not match with data '2014/09/25 21:15:03 Got request with path wombat\n'" location=nil tag="reform.var.log.sandbox.sample.log" time=2014-09-25 21:15:03.499185026 +0000 record={"log"=>"2014/09/25 21:15:03 Got request with path wombat\n", "stream"=>"stderr"}
tail_1  | 2014-09-25 21:15:03.499185026 +0000 raw.kubernetes.: {"log":"2014/09/25 21:15:03 Got request with path wombat\n","stream":"stderr"}
```

---

## ついでに

じゃんじゃんfluent pluginや設定を試せるsandbox環境欲しかったのでいい感じのも作った

https://github.com/threetreeslight/fluentd-sandbox

---

## ちょっとだけ今度やるイベントの宣伝

よければぜひ✨

- モバイルアプリ開発者向けRepro Tech Meetupやります！モバイル開発からFlutterまで色々お話します！
- 今月の shinjuku.rb で kubernates + fluentd の構成についてちょろっと話します

---

## ref

- [Fluentd Blog - Unified Logging Layer: Turning Data into Action](https://www.fluentd.org/blog/unified-logging-layer)
- [fluentd - Buffer Plugin Overview](https://docs.fluentd.org/v1.0/articles/buffer-plugin-overview)

