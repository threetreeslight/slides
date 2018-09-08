## Issue and Update ssl certificate automatically on GCP

@threetreeslight

on shinjuku mokumoku programming #13

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

## やりたいこと

(証明書の更新忘れがちなので)

GEKで運用しているblog([threetreeslight.com](https://threetreeslight.com))やその監視のssl証明書を自動更新したい

---

## 自動更新の方法

GKE上での自動更新の方法は大きく２つ存在する

1. kubernatesのecosystemにのっかり [cert-manager](https://github.com/jetstack/cert-manager/) を使った自動更新
1. certbotで発行したcertificateをGCPにuploadして利用

---

## 今回はcertbotを利用

理由は至ってチキン 😅

1. [helm]() (kubernates用package manager) に慣れていない
1. 自前で構築するnginxベースのingressにまだ慣れていない

---

## Certbotによる証明書発行

SSL証明書を発行する方法はいろいろあるが、今回はdocker使って楽にやりたかったので DNS Plugins 方法で行う

1. Apache, Webroot, Nginx
1. Standalone
1. DNS Plugins
1. Manual

cf. https://certbot.eff.org/docs/using.html

---

## dockerを使った更新方法

DNSはroute53を使っているので以下の段取り。楽ちん

1. route53 record更新policyとその権限を持つcertbot userの作成
1. certbot/dns-route53 でcertificateを発行
1. 発行したcertificateをGCPに登録
1. ingressの設定を更新してapply

---

## AWS IAM policy&user

とりあえずhostedzoneの参照権限とrecordの変更権限が有れば良い

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:GetChange"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/YOURHOSTEDZONEID"
            ]
        }
    ]
}
```

---

# Issue certificate

こんな感じでえいやっと投げる

```sh
mkdir -p ./certificate/(etc,lib,log)

docker pull certbot/dns-route53

docker run -it --rm \
-v $PWD/certificate/etc:/etc/letsencrypt -v $PWD/certificate/lib:/var/lib/letsencrypt -v $PWD/certificate/log:/var/log/letsencrypt \
-e "AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID" -e "AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY" \
certbot/dns-route53 certonly \
-n --agree-tos --email YOUR_EMAIL_ADDRESS \
--server https://acme-v02.api.letsencrypt.org/directory \
-d threetreeslight.com -d "*.threetreeslight.com"
```

---

## OKできた

```
./certificate/etc/live
└── threetreeslight.com
    ├── README
    ├── cert.pem -> ../../archive/threetreeslight.com/cert1.pem
    ├── chain.pem -> ../../archive/threetreeslight.com/chain1.pem
    ├── fullchain.pem -> ../../archive/threetreeslight.com/fullchain1.pem
    └── privkey.pem -> ../../archive/threetreeslight.com/privkey1.pem
```

---

# 実は

threeetreeslightと `e` が1個多くtypoっていたことで、hostedzoneが見つからず 1h 詰ったことは内緒 🤡

---

## Regsiter ssl certificate to gcp

```sh
# update
gcloud compute ssl-certificates create threetreeslight-com-2018-09-08 \
--certificate $PWD/certificate/etc/live/threetreeslight.com/fullchain.pem \
--private-key $PWD/certificate/etc/live/threetreeslight.com/privkey.pem
```

---

## Update certificate

```diff
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blog
  annotations:
      kubernetes.io/ingress.allow-http: "false"
      kubernetes.io/ingress.global-static-ip-name: blog-gke
+      ingress.gcp.kubernetes.io/pre-shared-cert: threetreeslight-com-2018-09-08
```

```sh
kubectl apply -f ./kubernates/ingress.yaml
```

---

## いい感じ 💪

```sh
curl -s -v -X --tlsv1.2 https://threetreeslight.com
* Rebuilt URL to: https://threetreeslight.com/
*   Trying 35.201.67.24...
* TCP_NODELAY set
* Connected to threetreeslight.com (35.201.67.24) port 443 (#0)
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate: threetreeslight.com
* Server certificate: Let's Encrypt Authority X3
* Server certificate: DST Root CA X3
```

---

## 自動更新

上記のプロセスを定期的に回せば良い感じ。

job kickerは、kubernates jobでもいいしciでもよさそう。

---

## circleci scheduled build

circleciでbuild作業を行っているので、circleciのscheduled triggerを使う。

Cron step syntax (for example, */1, */20) はサポートされていないので気をつける。

---

## 雑にこんな感じで

```yaml
version: 2
jobs:
  ...
  issue-and-update-certificate:
    docker:
      - image: google/cloud-sdk:latest
    environment:
      GOOGLE_PROJECT_ID: "threetreeslight"
      GOOGLE_COMPUTE_ZONE: "us-west1-a"
      EMAIL: "me@threetreeslight.com"
    steps:
      - run:
          name: Issue certificate
          command: |
            docker pull certbot/dns-route53

            docker run -it --rm \
            -v $PWD/certificate/etc:/etc/letsencrypt -v $PWD/certificate/lib:/var/lib/letsencrypt -v $PWD/certificate/log:/var/log/letsencrypt \
            -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
            certbot/dns-route53 certonly \
            -n --agree-tos --email $EMAIL \
            --server https://acme-v02.api.letsencrypt.org/directory \
            -d threetreeslight.com -d "*.threetreeslight.com"
      - run:
          name: Store Service Account
          command: echo $GCLOUD_SERVICE_KEY > ${HOME}/gcloud-service-key.json
      - run:
          name: Register and Update SSL Certificate
          command: |
            # auth and setup gcp
            gcloud auth activate-service-account --key-file=${HOME}/gcloud-service-key.json
            gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
            gcloud --quiet config set compute/zone ${GOOGLE_COMPUTE_ZONE}

            # regsiter new certificate
            CERT_NAME=threetreeslight-com-$(date +%Y-%m-%d-%H:%M:%S)
            gcloud compute ssl-certificates create $CERT_NAME \
            --certificate $PWD/certificate/etc/live/threetreeslight.com/fullchain.pem \
            --private-key $PWD/certificate/etc/live/threetreeslight.com/privkey.pem

            gcloud compute ssl-certificates list 

            # Update ingress certificate
            sed -i -e "s/pre-shared-cert:.*/pre-shared-cert: $CERT_NAME/" ./kubernates/ingress.yaml
            kubectl apply -f ./kubernates/ingress.yaml

            # commit update diff
            git add ./kubernates/ingress.yaml
            git checkout -b update-certificate
            git push origin update-certificate

workflows:
  ...
  update-certificate:
    triggers:
      - schedule:
          cron: "0 0 10 2,4,6,8,10,12 *"
          filters:
            branches:
              only:
                - master
    jobs:
      - issue-and-update-certificate
```

---

## references

- [certbot - running with docker](https://certbot.eff.org/docs/install.html#running-with-docker)
- [certbot - certbot-dns-route53](https://certbot-dns-route53.readthedocs.io/en/latest/)
- [circleci - triggers](https://circleci.com/docs/2.0/triggers/#scheduled-builds)

