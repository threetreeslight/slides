# Update fleunt-gcp version

@threetreeslight

on shinjuku mokumoku programming #8

---

## 台風大変ですね

⛈ ⛈ ⛈

---

## 今日やること

GKE上のnode logging aggregator fluentdのversionup

fluentd-0.12.40 -> 1.12系 (0.14系でも)

---

## なぜ？

1. fluentdの remote_syslog output plugin が0.14系以上じゃないと TCP+TLS通信をサポートしていないから
1. 0.12系にbackportとかやってられない

---

## 掘っていく

fluentd-gcpなるimageはどこで管理されているのか？

---

## kubernates/kubernates

From [kubernetes/kubernetes - cluster/addons/fluentd-gcp/fluentd-gcp-image](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-gcp/fluentd-gcp-image)

> Collecting Docker Log Files with Fluentd and sending to GCP.
> The image was moved to the new location.

ほほぉ。

---

## kubernates/contrib

From [kubernetes/contrib - fluentd/fluentd-gcp-image](https://github.com/kubernetes/contrib/tree/master/fluentd/fluentd-gcp-image)

> This project has been moved to GoogleCloudPlatform/k8s-stackdriver:fluentd-gcp-image@master

ほほぉ。

---

## ここにある

[GoogleCloudPlatform/k8s-stackdriver - fluentd-gcp-image](https://github.com/GoogleCloudPlatform/k8s-stackdriver)

---

## 😇

もちろん最新の状態も 0.12系

```sh
source 'https://rubygems.org'

gem 'fluentd', '~>0.12.32'
gem 'fluent-plugin-record-reformer', '~>0.8.3'
gem 'fluent-plugin-systemd', '~>0.0.8'
gem 'fluent-plugin-google-cloud', '~>0.6.12'
gem 'fluent-plugin-detect-exceptions', '~>0.0.8'
gem 'fluent-plugin-prometheus', '~>0.2.1'
gem 'fluent-plugin-multi-format-parser', '~>0.1.1'
gem 'oj', '~>2.18.1'
```

---

## UpdateのPRもでているが

[Bumped fluentd from 0.12.32 to 0.14.25 #148](https://github.com/GoogleCloudPlatform/k8s-stackdriver/pull/148)

> Hi Yu,
> I'm interested in how have you tested this version of fluentd.
> Could you add more context to your PR.
> And maybe description which bugs have been fixed, or which features are introduced would be very helpful.
> Thanks

これは厳しい

---

## さて

1. GoogleCloudPlatform/k8s-stackdriver をforkしてupdateしつつ、そのimageをbaseにremote_syslog系pluginを入れる処理を書くか
1. fleuntd-gcp-imageの処理をバコッと持ってきてごにょごにょしちゃうか

---

## まっそんなことは言わずやっていき

[GoogleCloudPlatform/k8s-stackdriver - fluentd-gcp-image](https://github.com/GoogleCloudPlatform/k8s-stackdriver) をまるっと持ってきて


---

## dockerfile

```diff
FROM k8s.gcr.io/debian-base-amd64:0.3

COPY Gemfile /Gemfile

# 1. Install & configure dependencies.
# 2. Install fluentd via ruby.
# 3. Remove build dependencies.
# 4. Cleanup leftover caches & files.
RUN BUILD_DEPS="make gcc g++ libc6-dev ruby-dev libffi-dev" \
    && clean-install $BUILD_DEPS \
                     ca-certificates \
                     libjemalloc1 \
                     liblz4-1 \
                     ruby \
+                     curl \
    && echo 'gem: --no-document' >> /etc/gemrc \
    && gem install --file Gemfile \
    && apt-get purge -y --auto-remove \
                     -o APT::AutoRemove::RecommendsImportant=false \
                     $BUILD_DEPS \
+    && curl -sL -o /etc/papertrail-bundle.pem https://papertrailapp.com/tools/papertrail-bundle.pem \
    && rm -rf /tmp/* \
              /var/lib/apt/lists/* \
              /usr/lib/ruby/gems/*/cache/*.gem \
              /var/log/* \
              /var/tmp/*

# Copy the Fluentd configuration file for logging Docker container logs.
COPY fluent.conf /etc/fluent/fluent.conf
COPY run.sh /run.sh

# Expose prometheus metrics.
EXPOSE 80

ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.1

# Start Fluentd to pick up our config that watches Docker container logs.
CMD /run.sh $FLUENTD_ARGS
```

---

## Gemfile

一旦0.14系にあげる

```diff
source 'https://rubygems.org'

+ gem 'fluentd', '~>0.14.25'
gem 'fluent-plugin-record-reformer', '~>0.8.3'
gem 'fluent-plugin-systemd', '~>0.0.8'
gem 'fluent-plugin-google-cloud', '~>0.6.12'
gem 'fluent-plugin-detect-exceptions', '~>0.0.8'
gem 'fluent-plugin-prometheus', '~>0.2.1'
gem 'fluent-plugin-multi-format-parser', '~>0.1.1'
+ gem 'fluent-plugin-remote_syslog', '~> 1.0.0'
gem 'oj', '~>2.18.1'
```

---

## confは最新のバージョンに合わせて色々直す

そんなに大変でもなかった

---

## シュッ

imageをbuildし、configとimageをupdate!

```sh
timestamp=`date +%Y%m%d%H%M%S`

docker build -t threetreeslight/fluentd-gcp:$timestamp ./fluentd-gcp
docker tag threetreeslight/fluentd-gcp:$timestamp threetreeslight/fluentd-gcp:latest

echo "\nimage name is threetreeslight/fluentd-gcp:$timestamp\n"

docker push threetreeslight/fluentd-gcp:$timestamp
docker push threetreeslight/fluentd-gcp:latest

# udpate config
echo "\nUpdate fluentd-gcp config\n"
kubectl create configmap fluentd-gcp-config-v1.2.2 --from-file=fluentd-gcp/config/ --namespace kube-system --dry-run -o yaml | kubectl replace configmap fluentd-gcp-config-v1.2.2 -f -

echo "\nUpdate fluentd-gcp image to threetreeslight/fluentd-gcp:$timestamp\n"

kubectl set image ds/fluentd-gcp-v2.0.9 --namespace=kube-system fluentd-gcp=threetreeslight/fluentd-gcp:$timestamp
kubectl rollout status ds/fluentd-gcp-v2.0.9 --namespace=kube-system
```

---

## とどいているっぽい👀

![](/shinjuku-mokumoku/8/papertrail.png)

---

## しかし

もりもりエラー出ているのでなんとかしていく

```sh
# tyee error
fluentd-gcp	Jul 28, 2018, 4:57:22 PM	2018-07-28 07:57:22 +0000 [warn]: 'type' is deprecated parameter name. use '@type' instead.

# service discoveryで叩きすぎ
prometheus-to-sd-exporter	Jul 28, 2018, 4:57:17 PM	E0728 07:57:17.772231 1 stackdriver.go:58] Error while sending request to Stackdriver googleapi: Error 403: Stackdriver Monitoring API has not been used in project 763099284966 before or it is disabled. Enable it by visiting https://console.developers.google.com/apis/api/monitoring.googleapis.com/overview?project=763099284966 then retry. If you enabled this API recently, wait a few minutes for the action to propagate to our systems and retry., accessNotConfigured

# pattern not match with data
2018-07-28 06:57:47 +0000 [warn]: dump an error event: error_class=Fluent::Plugin::Parser::ParserError error="pattern not match with data '2018-07-28 06:57:43 +0000 [warn]: dump an error event: error_class=Fluent::Plugin::Parser::ParserError error=\"pattern not match with data '2018-07-28 06:57:34 +0000 [warn]: dump an error event: error_class=Fluent::Plugin::Parser::ParserError error=\\\"pattern not match with data 't=2018-07-24T08:10:30+0000 lvl=eror msg=\\\\\\\"Failed to start session\\\\\\\" logger=context error=\\\\\\\"open /var/lib/grafana/sessions/c/b/cb946940b9ec7ac4: no space left on device\\\\\\\"\\\\n'\\\" location=nil tag=\\\"reform.var.log.containers.monitor-5b7f555456-24dcs_default_grafana-17834a541c748564ebb79772cafd8a6858f095fb46334778550d84e33ab51627.log\\\" time=#<Fluent::EventTime:0x007f0ec9b48500 @sec=1532419830, @nsec=256019503> record={\\\"log\\\"=>\\\"t=2018
...
```

---

## やってて気になったこと

node logging agentのscaleってどうやっているんだろう 🤔

---

# 頑張っていく 💪

