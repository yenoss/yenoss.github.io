---
layout: post
title: 2016-10-20-[ubuntu/git]ssh키로 git에 접근해보자.
---

##ssh키로 git에 접근해보자.

<br></br>
깃을 기본 클론만하여 https로 접근하도록 remote하였다면 당신은 push등의 깃 명령어를 사용할때 매번 깃헙 등의 id/pw를 입력했어야 했을것이다. 

이제 ssh 키를 이용하여 간단하게 ssh passphrase(일종의 비번)로 접근가능하도록 하자.
(물론 passPhrase를 설정 안하게 할수도있다.)
<br></br>

####0.등록된 키를 확인한다.

명령어를 통해 현재 등록되있는 키를 확인한다.
id_rsa.pub, id_rsa 등이 존재하지 않는다면 기존 등록된 키가 없다고 보면된다.
~~~
ls -al ~/.ssh
~~~
<br></br>


####1.RSA방식으로 키를 생성한다.
명령어를 통해 키를 생성한다.

~~~
ssh-keygent -t rsa -C "나의이메일"
~~~

위명령어를 통하면 아래와같이 몇가지 옵션으로 키설정을 해준다.

1. 키를 어디에 저장할것인지. 우린 디폴트위치인 .ssh에 생성할것이다. 그러므로 그냥 enter로 넘어가준다.
2. 키를 사용할때 passphrase설정을 할것인지를 묻는다. 내가 키를 사용할때 마다 패스워드를 입력하기싫다면 이부분역시 그냥 enter로 넘어가면된다.
3. 최종 저장된위치를 보여주고 키에대한 정보를 보여주며 키생성이 끝나게된다.

~~~
Generating public/private rsa key pair.

Enter file in which to save the key (/home/yenos/.ssh/id_rsa):

Created directory '/home/yenos/.ssh'.

Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in /home(디렉토리)
Your public key has been saved in /home(디렉토리)

The key fingerprint is:
SHA256:5fiL*@#@$@$(($) cpg0504@gmail.com

The key's randomart image is:
+---[RSA 2048]----+
|          .=. .o |
~~
|        .oo=+=o +|
+----[SHA256]-----+

~~~

<br></br>

####2.Git제공 사이트에 SSH키를 등록한다.
당신이 github,gitlab등 깃제공사이트를 사용한다면 사이트내에 나의프로필에서 ssh 키설정을 해준다.

키설정을 위에 정한 디렉토리에서 해당 ssh키를 가져온다. 

만약 당신이 enter로 넘어가 default 디ㄹ게토리로 설정하였다면 

~~~
ls -al ~/.ssh 
~~~

내의 **id_rsa.pub** 파일을 복사하면된다.

**복사한 값을 가지고 github를 예를 들어 ssh key를 등록해보자**

~~~
User>Setting>ssh and GPGKeys 
~~~

위의 위치에도착하면 **NewSSHKey** 라는 것을 눌러  복사한 값을 붙여넣어준다. 

이렇게되면 언제든 sshkey를 이용해 간편히 깃을 사용할 수 있게된다.
<br></br>

####3.기존에 있던 Remote를 제거하고 ssl접근으로 origin을 수정한다.

만일 당신이 필자처럼 id/pw기반으로 작업을 하고 있었다면, remote repository 접근 방법을 https=> ssl로 바꿔줘야한다.

그러기 위해 기존것을 삭제하고, 다시 remote 추가하는 방향으로 나아가야한다.

먼저 git Remote를 확인한다.

~~~
git remote -v
~~~

그렇담 현재 내가 remote 되어있는 방식이 https로 되어있을것이다. 예를들면

~~~
origin	https://github.com/(상세레파짓토리)(fetch)
origin	https://github.com/(상세레파짓토리)(push)
~~~

이런식으로 ..  이제 이 origin이라 명명된 연결을 끊고 ssl로 접근하도록 한다.

~~~
git remote remove origin
git remote add origin git@github.com:/(상세레파짓토리)
~~~

이렇게 새로운 remote연결을 달아주면 끝!

앞으로 git으로 push등을 할때 passphrsa만을 입력하는것으로 작동이 될것이다.




