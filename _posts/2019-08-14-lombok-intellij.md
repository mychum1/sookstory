---
title: intellij에서 lombok 실행하기
categories:
- java
excerpt: |
  면접 중에 JPA의 쓰기 지연에 대해 이야기 하다가 OSIV에 대한 얘기가 나왔습니다. 물론 이게 뭔지 모르는 사람이기 때문에 정리해보는걸로.
feature_text: |
  ## lombok 실행하기
  lombok은 의존성 추가만으로는 안된다.
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

lombok은 플러그인이 필요하다.
====
  
새로운 프로젝트를 인수인계받기로했다. 한참을 보고있는데 lombok을 발견했다. 물론 예전부터 봐오던 거지만 난 게터세터 쓰는데 불편함도 없고.. 원하는 기능을
구현하는데도 굳이 필요없는 부분이기도 해서 쓰지 않았던 라이브러리인데 회사에서 써야한다면 써야지 하고 코드를 받아 실행시키려고 했는데 빨간줄이 덕지덕지 떴다.
에러는 게터세터를 구현하지않아서였다. 그렇담 롬복 문제인데 의존성도 다 추가돼서 라이브러리도 잘 인스톨되어있었다. 뭐가문젠가? 
찾아보니, 롬복은 의존성 추가 말고도 설치를 해주어야한다고 한다.  
최종적으로 두 단계를 실행해야한다.
 
1. 지연 로딩
----

intellij에서는 플러그인을 설치해주면된다. file > settings > plugin > lombok 을 install 해주면된다.
[롬복 플러그인 설치](https://github.com/mychum1/learning_ref/blob/master/com.ksko.learning-ref/imgs/lombok.png?raw=true)  


2. 어노테이션 enable
----
  
어노테이션 프로세서도 활성화를 시켜주어야한다.
[어노테이션 프로세서 활성화 ](https://github.com/mychum1/learning_ref/blob/master/com.ksko.learning-ref/imgs/annotaion_processor.png?raw=true)  
