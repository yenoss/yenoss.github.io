---
layout: post
title: 'hashicorp vault란? 01-secret/kv'
tags: [hashicorp,vault,key sharing,secret-engine,한글]
background: '/img/posts/bg_java.jpg'

---

hashicorp,vault,key sharing,secret-engine,한글

<br>

## 1. 왜.

- vault를 잘 사용해 보기 위해서
- service를 만들면서 관리되어야할 정보들을 어떻게 관리해야 '잘' '안전하게' '그나마 편하게'  관리되어질 수 있을까?



<br>

## 2. 그래서 무엇인가.

<br>

<img src= "https://www.fyber.com/engineering/wp-content/uploads/2019/11/hash1.png" width="500" height="250">

<br>



> hashicorp vault란 무엇인가? 

* vault  페이지메인을 가면 아래와 같이 이야기가 나온다.

  * Manage Secrets and Portect Sensitive Data
  *  tight하게 컨트롤 되어야하는 token, passowrd, certificate, encryption key등을 `안전하게 저장하는 저장소`라고 이야기한다.

* 몇 가지 특징이 있는데,

  * centrally store

    * 중앙에서 store, access, deploy secrets등을 해주기때문에 여러 어플리케이션과 infrastructrue에서 쉽게 사용가능 하다. 
    * 개발시 많은 secret들이 쓰인다. aws secret, db  정보 등이 있을것이고, 이것들은 여러 infra 환경, 개발/prod등등 다양하게 다르고 변경될 수 있다. 
    *  요소들을 `한곳에서 관리`해준다면? > good

  * Authorization, Authentication

    * 중앙에서 관리가 되는 것 까지는 좋은데, 당연히 
      * 각각의 환경, 혹은 서비스마다 다른 값을 가지고,
      * 원하는 혹은 접근 가능한 데이터가 다를 것이기에
    *  각 secret에 접근할 수 있는 `권한`과 `인증`이 필요하다. 
    * vault는 접근 가능 policy를 만들어 임의의 guest key와 같은 token을 발행해 특정 token을 기반으로 데이터에 접근할 수 있도록 해준다. 
    * 특히, AWS,Github, Googl cloud 등의 다양한 인증 정보로 사용자를 만들어 policy를 제공해 줄 수 있다.([Auth methods](https://www.vaultproject.io/docs/auth)라 한다
    * 또한 모든 발급되어지는 것들에 대해 `ttl`이 있고, 언제든 `revoking`할 수 있게 지원해준다. 

  * key sharing

    * vault는 최초 서버가 실행되면  5개의 key를 주고 서버를 sealling해둔다. sealling된 상태는 데이터 넣고 빼는(결국 encrypt,decryption) api자체가 불가하다. 
    * unseal을 해야하는데 그러기위해선 5개의 key중에 3개의 key를 입력해야한다. 이러한 방식은 [shamir 비밀키 공유 알고리즘](https://en.wikipedia.org/wiki/Shamir's_Secret_Sharing)을 이용하며, 5개중 최소 3개가 있어야 master key를 만들수 있게된다.
    * materkey가 있어야 비로소 encryption key를 만들 수 있다. 
    * 더 해서, keyt rolling도 지원해준다. `key rolling`을 통해 사용자는 특정 주기로 새로운 마스터키를 생성해 사용함으로 보다 안전한 운영이 가능하다.

  * certification issuing

    * SSL을 위한  cert를 발급해준다.  특정 secret engine으로 제공해주며 ,  필요한 정보를 저장하고 있어  최종적으로 사용자는 몇 가지 설정만 해두면 아주 손쉽게 cert를 발급받을 수 있다. 

      

<br>

### 3. 써보자

* 아래 사용 예시는  [vault get started](https://learn.hashicorp.com/vault/getting-started/install) 를 보고 따라해 보았다. 
* tutorial안에 있는 텍스트 해석 + 필자의 자유로운 해석 이라고 보면된다.

<br><br>

## Starting the server

- vault는 client/server 로 동작. server는 datastorage, backend와 통신.
- 모든 동작은 vault cli로 통제 가능.



<br>

## Starting the dev server

- dev server는 built in에 미리 conf가 설정되있다. 강한 시큐어는 아니다. 로컬에서 적당히 테스트.
- fork되지 않아 foreground에서 동작.
- dev는 in memory, no TLS 그래서 key, root access key가 개봉된 상태로 바로 유저에게 보여진다. 이를 잘 간직해두도록하자.

1. `export VAULT_ADDR='[<http://127.0.0.1:8200>](<http://127.0.0.1:8200/>)'`  실행, config로 vault client가 server와 통신하기 위해 사용될것임.
2. unseal key를 어딘가에 저장. 어떻게 저장될지는 걱정 ㄴㄴ, 일단 저장해둔다. 어짜피 dev모드라 unsel key가 크게 의미가 없다..  허나 production 레벨로 vault가 restart등의 액션이 일어나면 이 키를 이용해 다시 서버를 열어줘야하기 때문에 의미가있다.
3. Root Token을 `VAULT_DEV_ROOT_TOKEN_ID` 로 저장해둔다.

```jsx
//최종 env 저장형태
env | grep VAULT 
VAULT_ADDR=http://127.0.0.1:8200
VAULT_DEV_ROOT_TOKEN_ID=s.bOGKkztDunY9OcTT4js5MPjW
```

<br>

## Verify the Server is Running

- `vault status` 로  상태확인 하려했으나 에러.

```jsx
Error checking seal status: Get <https://127.0.0.1:8200/v1/sys/seal-status:> http: server gave HTTP response to HTTPS client
```

- 특정 버전 이후부터 default address client가 https를 사용 하여 address flag 사용. ([link1](https://github.com/hashicorp/vault/issues/1147))
- 위에 env가 잘 설정되었다면 아래와 같이 --address를 입력해 주지 않아도된다..

```jsx
//vault status --address="<http://127.0.0.1:8200>"

Key             Value
---             -----
Seal Type       shamir  // key 분배 알고리즘(<https://learn.hashicorp.com/img/vault-shamir-secret-sharing.svg>)
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.3.4
Cluster Name    vault-cluster-6258aaba
Cluster ID      8d6af2ac-c0b9-0890-568e-95ae04836997
HA Enabled      false
```

- 갯수등의 문제가 있으면 리스타트를 해봐야된다. 문제는 아마 전 가이드를 잘 따르지 못 해서일것이다.



<br>

## Your first secret

- cli를 통해 http api로 vault를 사용해볼것이다.
- secret은 vault의 backend storage에 저장될 것이다. dev server는 memory라 memory에 저장될것임. production에서는 consul과 같은 곳에 저장하면 된다.
- vault는 storage driver 가기전에 암호화 핸들링을 한다.  이 말은 곧 백앤드가  vault없이 복호화 하지 못 한다는 것이다.



<br>

## Writing a secret

- ```
  vault kv
  ```

   로 저장해보자.

  - `vault kv put secret/hello foo=world`

```jsx
// vault kv put secret/hello foo=world
Key              Value
---              -----
created_time     2020-04-07T02:26:56.656645Z
deletion_time    n/a
destroyed        false
version          1
```

- foo=word를  secret/hello라는 path에 썼다. 뒤에 이야기가 나오겠지만 `prefix`인  secret이 중요하다. 이 위치가 임의의 secret을 read / write 한다.
- 데이터를 쪼개서 여러개 넣을 수 있다.
  - `vault kv put secret/hello foo=world excited=yes`

```jsx
// vault kv put secret/hello foo=world excited=yes
Key              Value
---              -----
created_time     2020-04-07T02:29:34.140561Z
deletion_time    n/a
destroyed        false
version          2
```

- 지금은 key=vaule로 넣었지만 shell history등으로 감지될 수 있으니 파일로 관리하는게 좋다.



<br>

## Getting a Secret

- get으로 가져온다.
  - `vault kv get secret/hello`

```jsx
// vault kv get secret/hello
====== Metadata ======
Key              Value
---              -----
created_time     2020-04-07T02:29:34.140561Z
deletion_time    n/a
destroyed        false
version          2

===== Data =====
Key        Value
---        -----
excited    yes
foo        world
```

- vault는 storage로부터 암호화해 값을 리턴해 준다.
- white sapce는 [awk](https://zzsza.github.io/development/2017/12/20/linux-6/)(공백, 라인등으로 이쁘게 데이터를 가공해주는 좋은 명령어)등으로 가져올때 시스템상에서 쉽게 사용하기위해 의도적으로 넣어준것이다.

```jsx
//vault kv get secret/hello | awk '{print $1}'
======
Key
---
created_time
deletion_time
destroyed
version

=====
Key
---
excited
foo

//vault kv get secret/hello | awk '{print $2}'
Metadata
Value
-----
2020-04-07T02:29:34.140561Z
n/a
false
2

Data
Value
-----
yes
world
```

- 많은 시스템들은 secrets에 time-limit을 가진다. `lease_id`가 하나의 lease에 고유한 값을 가지고  `lease_duration` 가 valid 한 시간을 가진다.
- 값을 확인할 수 있다.

```jsx
godres ~/key $ vault kv get -field=excited secret/hello
yes
```



<br>

## Deleting a secret

- 저장된 값을 제거할 수 있다.
  - `vault kv delete secret/hello`

```jsx
//vault kv delete secret/hello
Success! Data deleted (if it existed) at: secret/hello

//vault kv get secret/hello
====== Metadata ======
Key              Value
---              -----
created_time     2020-04-07T02:29:34.140561Z
deletion_time    2020-04-07T02:39:22.994973Z
destroyed        false
version          2
```



<br>

## Secret Engines

- 위에서  `secret/` prefix로 put을 등록했었다. 그렇지 않다면?

```jsx
// vault kv put foo/bar a=b
Error making API request.

URL: GET <http://127.0.0.1:8200/v1/sys/internal/ui/mounts/foo/bar>
Code: 403. Errors:

* preflight capability check returned 403, please ensure client's policies grant access to path "foo/bar/"
```

- prefix 는 vault에게 라우팅해야되는 engine을 알려주는 것이다.
- request가 vault에게 오면  path를 매칭시켜 일치하는 엔진에게 전달한다.  보이는것처럼 파일시스템에서 따왔다.
- defualt [key/value version2 secret engine](https://www.vaultproject.io/docs/secrets/kv/kv-v2/)을 가진다. 이 엔진은 key/value raw data를 저장한다.
- 당연히 다양한  [secret engine](https://www.vaultproject.io/docs/secrets)을 제공한다.



<br>

## Enable a Secretes Engine

- engine의 각 path는 철저히 `isolated`되어있어 서로 `path끼리 이야기할 수 없다`.

```jsx
// vault secrets enable -path=kv kv
Success! Enabled the kv secrets engine at: kv/
```

- secret engine에 kv패스를 추가한다.

```jsx
// vault secrets enable kv
Error enabling: Error making API request.

URL: POST <http://127.0.0.1:8200/v1/sys/mounts/kv>
Code: 400. Errors:

* path is already in use at kv/
```

- kv가 추가되어 이미 있다는 에러가 뜬다.
- 실제 추가 됬는지 보려면  secrets list를 보면된다.
  - `vault secrets list`

```jsx
// vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_7bfffbf6    per-token private secret storage
identity/     identity     identity_f6977784     identity store
kv/           kv           kv_ac183fa7           n/a
secret/       kv           kv_e2117b1e           key/value secret storage
sys/          system       system_21f99e6c       system endpoints used for control, policy and debugging
```

- 4가지의 secret engine들이 vault server에 존재한다.
- 각  path에 맡게 값을 저장할 수 있다.
- `sys/` path는 vault core system에 사용되 뉴비에게는 의미없다.
- path를 kv아래 자유롭게 추가 변경할 수 있다.

```jsx
// vault kv put kv/hello target=world
Success! Data written to: kv/hello
// vault kv get kv/hello
===== Data =====
Key       Value
---       -----
target    world

//  vault kv put kv/hello2 target=world
Success! Data written to: kv/hello2
//  vault kv list kv/
Keys
----
hello
hello2
```

<br>



## Disable a secrets Engine

- secrets engine을 끌수있다. 이때 모든 secrets는 revoke된다. 관련 configuration도 역시 remove된다.
  - `vault secrets disable kv/`

```jsx
// vault secrets disable kv/
// vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_7bfffbf6    per-token private secret storage
identity/     identity     identity_f6977784     identity store
secret/       kv           kv_e2117b1e           key/value secret storage
sys/          system       system_21f99e6c       system endpoints used for control, policy and debugging
```



<br>

## What is a Secret Engine?

- secret engine의 포인트는?
  - vault는 virtual file system과 같이 컨트롤 될수 있다. read/write/delete/list등의 동작은 secretengine과 일치해야하며 secret engine이 어떻게 반응해야되는지 동작한다.
  - 이것은 vault가 phsycial system, databases에서 이터페이싱 된다는 것을 의미한다. 또한 aws iam, sql등에서도 마찬가지의 인터페이스를 가짐으로 모두와 상호작용 할 수 있다.

<br><br>

### 4.마치며

* 아주아주아주  궁금했던 내용 중 하나였다. 도대체가  어디엔가 서비스 관리를 위한 비밀키들이 어디인가에  나도 모르게(?) 안전하게 저장 되어야 할텐데 그곳은 어디였나.  
* 마냥 파일로 나름 안전하다 생각되는 특정위치에 저장해두고  ttl 등의 단순한 로직으로 관리하는것이 다였는데  vault를 쓰면  분산환경에서 아주 안전하게 관리가 가능 할 수 있다. 
* 예전에 네트워크 보안 교수님이 암호화 vs 편리함 은 항상 trade off 되는 관계라고 하셨는데 (유저에게도 마찬가지..) vault는 이를 적절하게 잘 맞추어준, 개발자가 최대한 편리할 수 있게 기능을 최대한 빼서 제공해주는 멋진 모듈인거같다.(대충 많은것을 해준다는 얘기..)
* hashicorp는 보면 볼수록 매력적인 회사다. 다루는 분야들의 공통점을 명확히 모르겠는데  무튼 재밌는 프로젝트를 많이 서비스 한다. 

<br>

## Ref

- [hashicorp vault](https://www.vaultproject.io/)
- [vault get started](https://learn.hashicorp.com/vault/getting-started/install)