---
layout: post
title:  "Hashicorp Vault에 대하여"
date:   2021-08-19 20:20:10 +0900
---
Hashicorp Vault를 설치해서 사용해 봤습니다.

<span style="color:green"><sup>*설치 방법은 따로 작성할 예정입니다.</sup></span>

Hashicorp의 Vault는 Getting Started가 잘 구성되어 있었습니다.

[https://learn.hashicorp.com/collections/vault/getting-started](https://learn.hashicorp.com/collections/vault/getting-started)

영문으로 읽기 귀찮아지는 시점에 한글 블로그를 검색해 봤습니다.

[https://lejewk.github.io/vault-get-started/](https://lejewk.github.io/vault-get-started)

서버 한 곳에 vault server를 설치, 실행했습니다.

[http://10.xxx.xx.35:8200](http://10.xxx.xx.35:8200)

접속하시면 Token을 물어볼텐데, Token은 별도로 공유하겠습니다.

접속이 되면, 몇가지 key value를 보실 수 있을 겁니다.

그럼 바로 어떤건지 이해가 되실 겁니다.

web 페이지에서 내용을 확인하셨다면, 터미널에서 아래와 같이 vault client툴을 사용해 보세요.

```bash
$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
$ sudo apt-get update && sudo apt-get install vault
```

```bash
$ vault login
Token (will be hidden):
```

```bash
$ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cooooo/       kv
.
.
.
.
```

```bash
$ vault kv get -format=json cooooo/configure | jq .
{
  "request_id":
  "lease_id":,
  "lease_duration":
  "renewable":
  "data": {
    "access":
       .
       .
       .
  },
  "warnings":
}
```

```bash
$ vault kv get -field=access cooooo/configure
A*********************
```