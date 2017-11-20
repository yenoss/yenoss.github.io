---
layout: post
title: '[nginx] HTTPAuth with Nginx'
tags: [nginx]
---
nginx, httpAuth

<br>
## 1. 왜.

* 인증을 서비스에서 자체 구축하긴하지만, 경우에 따라 간편하게 nginx에서 특정 웹에 접근 할때 간단히 id/pw를 받아 볼 수 있다.



<br>
## 2. 그래서 무엇인가.

* 간략히 내부 어드민등을 간편하게 id/pw인증 시스템을 구축할 수 있다. 



<br>
## 3. 써보자.

### 1. ApacheUtils 설치
- `sudo apt-get install apache2-utils`
    + apache의 utils를 이용하여 해당 기능을 쉽게 구현할 수있다.

### 2. id/pw 만들기
- `sudo htpasswd -c /etc/nginx/.htpasswd {userId}`
    + id를 인풋으로 넣으면 password를 입력하라고 한다. 

- `cat /etc/nginx/.htpasswd`
	 + 해당 id&pw가 나오는 것을 확인할 수 있다.

    
### 3. nginx 설정
- nginx config에 아래와 같이 `auth_basic`, `auth_basic_user_file`를 입력해준다.

```
 server {
  listen       80;
  server_name  {domain};
  location / {
      root   /var/www/mywebsite.com;
      index  index.html index.htm;
      auth_basic "간단한 문구";                                
      auth_basic_user_file /etc/nginx/.htpasswd; //위치 
  }
}
```


### 4. nginx 재시작
- `sudo service nginx reload`
	+ reload를 하면 원하는 인증 창이 나온다.



<br><br>
## 4.마치며

* 어떠한 사람인지 인증하는 것은 매우 중요하다. 어떻게 사용하든 최소한의 인증을 어떻게 해야하는가는 고민은 선택이 아닌 필수이다.

## Ref.

* [nginx-auth](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx-on-ubuntu-14-04)







