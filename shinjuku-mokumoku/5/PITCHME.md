## slack alert prettfy and nginx ltsv

shinjuku mokumoku programming #5
@threetreeslight

---

## 今日やりたかったこと

いつものblogのやつ

- [done] blogの監視をしている prometheus で alert recover 通知をslackにいい感じにする
- [done] blogの ngix のhttp2対応, logのLTSV化 (h2oのほうが面白いかも？)
- [WIP] prometheus + GEKにおいてnodeのmetricいい感じのが取れないのか、node_exporter入れるべきか調査

---

## slack alert prettfy

---

## やらなければいけないこと

1. resolve時のalarmを飛ばす
1. 状態をred, greenで現状わかりやすくする

![](https://threetreeslight.com/images/blog/2018/07/2018-07-01-gke-prometheus-slack.png)

---

## まずresolved 通知

なるほど、send_resolvedを変えればよい。

```yaml
send_resolved: true
```

---

## これだけか

---

## 小ネタ: default false

ちなみに、webhook, pagerduty, OpsGeneis, VictorOps,PushHover がdefault true でしたなんで？

https://github.com/prometheus/alertmanager/blob/master/config/notifiers.go#L27

```go
	// DefaultSlackConfig defines default values for Slack configurations.
	DefaultSlackConfig = SlackConfig{
		NotifierConfig: NotifierConfig{
			VSendResolved: false,
		},
...
```

---

## なんで???

commit messageを読む

> brian-brazil committed on Jan 6, 2016
>
> Don't send resolved to Slack by default.
> Slack is a general chat system, it has no notion of resolved messages. Default it to false to avoid spamming people as we do with all other such systems.

---

## お気持ちわからなくないが

主観ですね

---

## prometheusからの通知内容をカスタマイズ

alert.rules.yml

```
    annotations:
      summary: 'Endpoint {{ $labels.instance }} down'
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been has been down for more than 10 seconds.'
```

---

## alermanagerのslack通知フォーマットカスタマイズ

alertmanager.yml

```yaml
receivers:
- name: slack
  slack_configs:
  - send_resolved: true
    channel: '#blog'
    color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'
    footer: '{{ template "slack.default.footer" . }}'
    icon_url: 'https://pbs.twimg.com/profile_images/588945677599780865/mrhc1gSh_400x400.png'
    pretext: '{{ template "slack.default.pretext" . }}'
    short_fields: false
    title: '{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }} {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
    text: "{{ range .Alerts }}{{ .Annotations.description }}{{ end }}"
    title_link: '{{ template "slack.default.titlelink" . }}'
    username: 'prometheus'
```

---

## 何度も試験するのがだるいので

簡単な再起動スクリプトを書く

monitor_restart.sh

```sh
#!/bin/sh

kubectl create configmap alertmanager-yml --from-file=prometheus/alertmanager.yml --dry-run -o yaml | kubectl replace configmap alertmanager-yml -f -
kubectl create configmap prometheus-yml --from-file=prometheus/prometheus.yml --dry-run -o yaml | kubectl replace configmap prometheus-yml -f -
kubectl create configmap alert-rules-yml --from-file=prometheus/alert.rules.yml --dry-run -o yaml | kubectl replace configmap alert-rules-yml -f -
kubectl create configmap grafana-ini --from-file=grafana/grafana.ini --dry-run -o yaml | kubectl replace configmap grafana-ini -f -
kubectl create configmap grafana-datasource --from-file=grafana/datasource.yaml --dry-run -o yaml | kubectl replace configmap grafana-datasource -f -

kubectl scale --replicas=0 deployment/monitor

while [ $(kubectl get pods | grep monitor | wc -l) = 1 ]; do
  kubectl get pods
  sleep 1
done

kubectl scale --replicas=1 deployment/monitor
sleep 10

while [ $(kubectl get pods | grep monitor | grep Running | wc -l) = 1 ]; do
  kubectl get pods
  sleep 1
done

echo "restarted"
exit 0
```
---

## これでいける!!111

---

## demo

```sh
% kubectl scale --replicas=0 deployment/blog
deployment.extensions "blog" scaled

% kubectl scale --replicas=1 deployment/blog
deployment.extensions "blog" scaled
```

---

## 動いてよかった 😇

---

## nginx logのLTSV化

---

## の前に以下をなんとかしなければならない

1. GCP load balancerから後ろはhttpでアクセスしている
1. アクセス元がhttpだったらhttpsにredirectしておくようにした

```
    # redirect http to https
    if ($http_x_forwarded_proto = "http") {
        rewrite  ^/(.*)$  https://$host/$1 redirect;
    }
```

---

```
＿人人人人人人人＿
＞　localで邪魔　＜
￣Y^Y^Y^Y^Y^Y^Y^￣
```

---

## 解決案

1. dockerfile事分けちゃう
1. 環境変数を使って分ける
  1. nginx perl module
  1. nginx lua module

---

## ぐぬぬ。。。

configurationとapplicationがくっついているの良くない、、、

---

## そういうときのentrykit

> Entrypoint tools for elegant containers.
> Entrykit takes common tasks you might put in an entrypoint start script and lets you quickly set them up in your Dockerfile. It's sort of like an init process, but we don't believe in heavyweight init systems inside containers. Instead, Entrykit takes care of practical tasks for building minimal containers.

---

## 今回はetrykitを使って

docker側で制御することで、nginxはgenelicな形を保てるようにする

1. 環境変数を使って制御する
1. container起動時に環境変数を元にtemplateからrenderingする

---

## nginx.conf.tmpl

```diff
+    {{ if var "DEBUG" }}
    # redirect http to https
    if ($http_x_forwarded_proto = "http") {
        rewrite  ^/(.*)$  https://$host/$1 redirect;
    }
+    {{ end }}
```

---

## entrykitを使ったbuild

```diff
+ ENV ENTRYKIT_VERSION 0.4.0
+ RUN apk add --no-cache --virtual build-dependencies curl tar \
+   && curl -SLo entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz https://github.com/progrium/entrykit/releases/download/v${ENTRYKIT_VERSION}/entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
+   && tar xvzf entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
+   && rm entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
+   && apk del --purge build-dependencies \
+   && mv entrykit /bin/entrykit \
+   && chmod +x /bin/entrykit \
+   && entrykit --symlink

COPY --from=build /site/public /usr/share/nginx/html
COPY ./nginx/blog.conf.tmpl /etc/nginx/conf.d/default.conf.tmpl
COPY ./nginx/nginx.conf.tmpl /etc/nginx/nginx.conf.tmpl

+ ENTRYPOINT [ \
+   "render", "/etc/nginx/conf.d/default.conf", "--", \
+   "render", "/etc/nginx/nginx.conf", "--" \
+   ]
```

---
## 幸せ 😇

---

## LTSV化する

---

## What ltsv

http://ltsv.org/

> Labeled Tab-separated Values (LTSV) format is a variant of Tab-separated Values (TSV). 
> ...

つまり、パースしやすい!!!

---

## こんなのを

```sh
# nginx.conf
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

---

## こんな感じに

```diff
+    log_format ltsv "time:$time_local"
                    "\thost:$remote_addr"
                    "\tforwardedfor:$http_x_forwarded_for"
                    "\treq:$request"
                    "\tstatus:$status"
                    "\tsize:$body_bytes_sent"
                    "\treferer:$http_referer"
                    "\tua:$http_user_agent"
                    "\treqtime:$request_time"
                    "\tcache:$upstream_http_x_cache"
                    "\truntime:$upstream_http_x_runtime"
                    "\tvhost:$host";

-    access_log    /dev/stdout ltsv;
+    access_log    /dev/stdout;
```

---

## するとlogが

before

```sh
curl http://localhost:8888
nginx_1              | 172.19.0.1 - - [07/Jul/2018:07:26:55 +0000] "GET / HTTP/1.1" 200 26354 "-" "curl/7.54.0"
```

after

```sh
curl http://localhost:8888
time:07/Jul/2018:07:31:38 +0000  host:172.19.0.1 forwardedfor:-  req:GET / HTTP/1.1      status:200      size:26354referer:-        ua:curl/7.54.0  reqtime:0.000   cache:- runtime:-       vhost:localhost
```

---

## まとめ

- slackが通知の見栄えが良いと心が落ち着く
- docker使う上でentrykitは鉄板感ある
- 次はこのlogを自前log shipperに送り込んでゴニョる

---

## 参考
- https://prometheus.io/docs/alerting/configuration/
- https://github.com/prometheus/alertmanager/blob/master/template/default.tmpl


