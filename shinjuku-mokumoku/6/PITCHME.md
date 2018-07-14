## How to cluster-level logging aggregate on kubernates?

shinjuku mokumoku programming #6

@threetreeslight on 2018-07-14

---

### 今日やりたいこと

自分の運用するblogがGEKで動いている。このログを分析に活かせるようなpipelineを作りたい

1. k8s(主にGKE)でのlog driverまわりやlog集約の調査
1. 用意したほうが自由度高そうであれば自前のlog shipperを動作させるか考える

---

## 図にすると

```sh
nginx prometheus grafana
  |       |         |
  -------------------
         |||
   ココらへんどうしよう？
          |
  -------------------
  |       |         |
bigquery gcs  papertrail/stackdriver
          |
     spark(data proc)
```

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

### How to GKE -> stackdriver?

GKEのデフォルトでは、logがstackdriverに送られている。

こんな感じ

```text
I  GET 200 10.31 KB null Go-http-client/1.1 https://grafana.threetreeslight.com/login GET 200 10.31 KB null Go-http-client/1.1 
I  GET 200 26.02 KB null Go-http-client/1.1 https://threetreeslight.com/ GET 200 26.02 KB null Go-http-client/1.1 
I  GET 200 10.31 KB null Go-http-client/1.1 https://grafana.threetreeslight.com/login GET 200 10.31 KB null Go-http-client/1.1 
I  GET 200 26.02 KB null Go-http-client/1.1 https://threetreeslight.com/ GET 200 26.02 KB null Go-http-client/1.1 
```

---

### 構成を確認する

```sh
% kubectl get deployment,ds,pod --namespace=kube-system
NAME                                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/event-exporter-v0.1.8   1         1         1            1           55d
deployment.extensions/heapster-v1.4.3         1         1         1            1           55d
deployment.extensions/kube-dns                1         1         1            1           55d
deployment.extensions/kube-dns-autoscaler     1         1         1            1           55d
deployment.extensions/kubernetes-dashboard    1         1         1            1           55d
deployment.extensions/l7-default-backend      1         1         1            1           55d

NAME                                      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                              AGE
daemonset.extensions/fluentd-gcp-v2.0.9   1         1         1         1            1           beta.kubernetes.io/fluentd-ds-ready=true   55d

NAME                                                   READY     STATUS    RESTARTS   AGE
pod/event-exporter-v0.1.8-599c8775b7-qthr8             2/2       Running   0          41d
pod/fluentd-gcp-v2.0.9-8mm85                           2/2       Running   0          41d
pod/heapster-v1.4.3-f9f9ddd55-n8cnf                    3/3       Running   0          41d
pod/kube-dns-778977457c-68gsl                          3/3       Running   0          41d
pod/kube-dns-autoscaler-7db47cb9b7-p2f8p               1/1       Running   0          41d
pod/kube-proxy-gke-blog-cluster-pool-1-767a6361-7t2r   1/1       Running   0          41d
pod/kubernetes-dashboard-6bb875b5bc-t8r4n              1/1       Running   0          41d
pod/l7-default-backend-6497bcdb4d-8s5lq                1/1       Running   0          41d
```

---

# fluetndがdaemonsetで動いているんですね

---

### GEK logging with fluentd

どのような設定かも確認するしていく

---

### daemonset

https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/fluentd-gcp/fluentd-gcp-ds.yaml

```yaml
      containers:
      - name: fluentd-gcp
        image: gcr.io/stackdriver-agents/stackdriver-logging-agent:{{ fluentd_gcp_version }}
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
```

---

### fluentd conf

https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/fluentd-gcp/fluentd-gcp-configmap.yaml

container input

```yaml
  containers.input.conf: |-
    ...
      @type tail
      path /var/log/containers/*.log
```

---

### sysmte input

```yaml
  system.input.conf: |-
    ...
    # logfile
      path /var/log/startupscript.log
      path /var/log/docker.log
      path /var/log/etcd.log
      path /var/log/kubelet.log
      path /var/log/kube-proxy.log
      path /var/log/kube-apiserver.log
      path /var/log/kube-controller-manager.log
      path /var/log/kube-scheduler.log
      path /var/log/rescheduler.log
      path /var/log/glbc.log
      path /var/log/cluster-autoscaler.log

    # systemd
      filters [{ "_SYSTEMD_UNIT": "docker.service" }]
      pos_file /var/log/gcp-journald-docker.pos

      filters [{ "_SYSTEMD_UNIT": "{{ container_runtime }}.service" }]
      pos_file /var/log/gcp-journald-container-runtime.pos

      filters [{ "_SYSTEMD_UNIT": "kubelet.service" }]
      pos_file /var/log/gcp-journald-kubelet.pos

      filters [{ "_SYSTEMD_UNIT": "node-problem-detector.service" }]
      pos_file /var/log/gcp-journald-node-problem-detector.pos

      pos_file /var/log/gcp-journald.pos
```

---

### output

```yaml
  output.conf: |-
  ...
    <match {stderr,stdout}>
      @type google_cloud
```

---

### カスタマイズは？

設定変更やカスタマイズは、daemonsetやconfigをいじれば良い

- [GCP - Customizing Stackdriver Logs for Kubernetes Engine with Fluentd](https://cloud.google.com/solutions/customizing-stackdriver-logs-fluentd)
- [kubernates - Logging Using Stackdriver](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-stackdriver/)

---

### stackdriver -> gcs, bqすればよいのでは？

```sh
nginx prometheus grafana
  |       |         |
  -------------------
         |||
  node logging agent(fluentd)
         |||
      stack driver
          |
      cloud pubsub
          |
  -------------------
  |       |         |
bigquery gcs  papertrail/stackdriver
```

---

### こういうのもありかも？

```sh
nginx prometheus grafana
  |       |         |
  -------------------
         |||
  node logging agent(fluentd)
         |||
  logging aggregater(fluentd cluster)
          |
  -------------------
  |       |         |
bigquery gcs  papertrail/stackdriver
```

---

### そもそも

nodeごとにあるfleutnd設定いじるのって正しいの？
nodeにあるfluentdを触ると、配置されるpodによってnodeごとのfluentd負荷が変わっちゃうだろうけど、致し方ないのだろうか？

papertrailやdatadog loggingあたりにつなぎこむんであればnode logging agentをカスタマイズするのありな認識。

---

### とりあえずカスタマイズしてみる

logging serviceを停止する必要がある。

> Prerequisites.
> If you’re using GKE and Stackdriver Logging is enabled in your cluster, you cannot change its configuration, because it’s managed and supported by GKE. 

```sh
gcloud beta container clusters update --logging-service=none blog-cluster
```

dynamicに変えられるようになったのすごい. betaだけど。

---

### 設定

は以下のrepoの内容を使うことも可能

- [GoogleCloudPlatform/k8s-stackdriver](https://github.com/GoogleCloudPlatform/k8s-stackdriver)
- [GoogleCloudPlatform/container-engine-customize-fluentd](https://github.com/GoogleCloudPlatform/container-engine-customize-fluentd)
- [kubernetes/kubernetes - fluentd-gcp](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/fluentd-gcp/README.md)

すでにfluentd daemonsetが動いているので、設定を吐き出すことにする

---

### 既存設定を書き出す

```sh
# daemonset
% kubectl get ds --namespace kube-system | grep fluentd | awk '{ print $1 }'
fluentd-gcp-v2.0.9

% kubectl get ds fluentd-gcp-v2.0.9 --namespace kube-system -o yaml > fluentd/fluentd-gcp-ds.yaml
fluentd-gcp-v2.0.9

# fluentd conf
% kubectl get cm  --namespace kube-system | grep fluentd | awk '{ print $1 }'
fluentd-gcp-config-v1.2.2

kubectl get cm fluentd-gcp-config-v1.2.2 --namespace kube-system -o yaml > fluentd/fluentd-gcp-configmap.yaml
```

---

# ここからがんばること :innocent:

1. papertrailにつなぎこむ設定を書いていく
1. GCS, BQにはcloud pubsubを使って吐き出していく

---

## まとめ

1. Using a node logging agent アプローチを採用する
1. applicationからログのparse, buffering, retry処理を切り離したいのであれば、Streaming sidecar container も採用する
1. gcpのstackに乗るんであればstackdriver exportを使ってデータ加工することが望ましい
1. node logging agentの設定カスタマイズは趣味やpapertrailやdatadog loggingなど別基盤に流すときぐらいにしかつかわない

