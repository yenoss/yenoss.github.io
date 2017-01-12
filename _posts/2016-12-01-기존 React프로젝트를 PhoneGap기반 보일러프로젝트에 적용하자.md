---
layout: post
title:  "기존 React프로젝트를 PhoneGap기반 보일러프로젝트에 적용하자."
date:   2016-12-01 13:37:00
categories: phoneGap/React
---

##기존 React 에서  PhoneGap

<br>
pedaler 프로젝트같은경우는 React.js로만 프로젝트를 진행할 목적이었다. 
그러나 내부 stoarge, gelocation 등을 보다 빠르게 적용하고자 기존웹에서 phoneGap으로 감싸기로 결정이 되었다. 
이에 따라 기존 React로 제공되는 프로젝트를 phoneGap React 보일러 프로젝트에 어떻게 붙인가에 대한 경험을 공유한다.
<br>

####0.phoneGap 보일러 프로젝트를 다운받는다.

https://github.com/kjda/ReactJs-Phonegap

폰갭에서 제공해주는 기본 보일러프로젝트다. 
먼저 해당 깃 래파짓토리를 다운받고 How to use를 통해 기본 프로젝트가 잘 돌아가는지 확인하는게 우선이다.
<br>


####1.보일러 react 소스를 기존 소스로 바꿔준다.

보일러 프로젝트 소스를  기존프로젝트 소스로 바꾼다.
react src 와 www(즉 public)에 있는 소스를 바꿔줘야겠다.

해당 리액트 소스는 보일러 프로젝트에서

~~~
app>src          
~~~

의 해당하는 곳에 옮겨주면되고

www등의 public 폴더는

~~~
app>phonegap-app>www
~~~

의 해당하는 곳에 옮겨주면 된다. 

정말 손쉽다.
<br>

####2.package.json 통합
react를 이용하는 유저라면 매우 당연하게 npm 설치 환경설정인 package.json을 가지고 있을 것이다.
기존 보일러 프로젝트는 react 구버전을 사용하기도하고.아마도 사용자와 완전히 같은 경우일 수가 거의 없을것이다. 
내 package.json/dependacies 보며  기존 보일러 프로젝트를 비교하며 내꺼에서 버전이 다른것 혹은 추가해야할 부분들을 수정한다. 
#####그러나.
몇가지 기본 보일러프로젝트에서 실행을위한 dependacies 가 있는데 그런것들은 추가 수정을 삼가해주는것이좋다. 아래와 같은것들이다.

* gulp 관련 (통합빌드)
* bower 

필자는 보일러프로젝트와 비교해서 버전이 다른것 그리고 bable-core,loader,css-loader등을 추가하였다. 이는 지극기 커스텀적인것임으로 설명을 패스한다.

그리고 수정된 configure 파일로 npm install을 다시해줘야할 것이다.
(깔끔히 설치를 위해 nodemodule을 삭제하고 깨끗하게 설치하는 것을 추천한다.)

<br>

####3.webpack.config를 수정한다.

보일러 프로젝트와 비교하여 기존 webpack.config와 다른점을 추가한다. 이 역시 2번의 과정처럼 추가를 핳면되는데

필자는 jsx를 사용하지 않고 js를 사용하며 babel기반으로 로드를 하였기에 해당 module loader를 기존것 대신하여 바꿔주었다. 또한 css른 src파일 내부에서 로더를 해야했음으로 css로더를 추가하였다.

필자의 로더는 아래와 같다(지극히 커스텀적이지만 누군가에겐 도움이..)

~~~
        loaders: [
           {
                test: /\.js$/,
                loader: 'babel',
                exclude: /node_modules/,
                query: {
                    cacheDirectory: true,
                    presets: ['es2015', 'react']
                }
            },
            {
                    test: /\.css$/,
                    loader: 'style!css-loader?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]'
                }
        ]
~~~~        
   
또한 필자는 jquery관련 plugin을 사용하고 있었는데 해당 플러그인이 작동하지 않는 이슈가있었다.(정확히는 해당 플러그인이 폴더내부에 있음에도 찾지못하였다)
원인은 보일러프로제트에서  webpack.config에 jquery plugin때문에 어떠한 중복, 혹은 이슈가 있어 실행되지 않았던것이다. 
plugins의 jquery관련 부분을 제거하니 기존 jquery plugin이 잘 작동하였다.

**모든 경우를 완료하고 how to use 의 빌드/run 과정을 깔끔하게 실행해보면된다.


##쓰고보니 결론은 간단하다. 
##당연히 맞춰줘야할 라이브러리 버전 및 config파일을 제대로 보지 아니하였음.



