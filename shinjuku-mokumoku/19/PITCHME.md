## もくもくしたtips

@threetreeslight

on shinjuku mokumoku programming #18

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

## 今日やること

- blogとgrafanaが時々downする事象を治す
- blog監視用grafanaがログインできなくなったのでなんとかする
- Istioをちゃんと可動させる

---

## できたこと

- 🙅‍♀️ blogとgrafanの外形監視だけが落ちる
- 🙅‍♀️ blog監視用grafanaがログインできなくなったのでなんとかする
- 🙅‍♀️ Istioをちゃんと可動させる

---

なので

# 小ネタを紹介

していく

---

## `status code 0`

外形監視で落ちているときのstatus codeは`0`

0 ? 公式にはなさそう。


> [stackoverflow - What does HTTP status code 0 mean](https://stackoverflow.com/questions/19858251/what-does-http-status-code-0-mean)
Unreachable

if we cannot reach your server the message will be delayed. We will tell the mail server to try again later.

---


## Update deployment with record option

> Note: You may specify the --record flag to write the command executed in the resource annotation kubernetes.io/change-cause. 
> It is useful for future instrospection, for example to see the commands executed in each Deployment revision.

[kubernetes - deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

## やってみた

何のコマンドによって変更されたかを知ることができ、Instropectionで便利そうだが、これ`apply -f` だけだったら意味ないよね？ 🤔

```yaml
metadata:
  annotations:
    ...
    kubernetes.io/change-cause: kubectl set image deployment blog blog=threetreeslight/blog:6f45fd600bae9e3be2c0f13aa28672f9e9600b55
      --namespace=app --record=true
    ...
```

---

## `nginx-blackbox-exporter`

nginxのstub_status page情報をれるようにしてみようとする

[nginx/nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)

---

### こんな感じの情報

exporter経由でstub_status pageをとると

```sh
% curl http://localhost:9113/metrics
# HELP nginx_connections_accepted Accepted client connections
# TYPE nginx_connections_accepted counter
nginx_connections_accepted 4
# HELP nginx_connections_active Active client connections
# TYPE nginx_connections_active gauge
nginx_connections_active 1
# HELP nginx_connections_handled Handled client connections
# TYPE nginx_connections_handled counter
nginx_connections_handled 4
# HELP nginx_connections_reading Connections where NGINX is reading the request header
# TYPE nginx_connections_reading gauge
nginx_connections_reading 0
# HELP nginx_connections_waiting Idle client connections
# TYPE nginx_connections_waiting gauge
nginx_connections_waiting 0
# HELP nginx_connections_writing Connections where NGINX is writing the response back to the client
# TYPE nginx_connections_writing gauge
nginx_connections_writing 1
# HELP nginx_http_requests_total Total http requests
# TYPE nginx_http_requests_total counter
nginx_http_requests_total 25
```

---

### service discoveryどうするの？

この手の特定のサービスだけのmetricを取得するときにservice disocveryを書くのか、staticに指定するのか？

prometheusのあるべきぞうがわからない 😇

てかこの情報だったらいらない

---

# 頑張る 😇
