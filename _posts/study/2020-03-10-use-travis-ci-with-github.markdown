---
layout: post
title: "github에 build passing badge 달기"
excerpt: "github에 travis-ci를 연동해보기 + 삽질"
---

![badge](../../images/20200310/badge.PNG)

↑↑↑ 깃헙에서 위에 있는 build passing 뱃지를 보게되었다.   
나도 readme.md에 추가해보자.

> Travis CI

[Travis CI](https://ko.wikipedia.org/wiki/Travis_CI)는 호스팅 지속적 통합(CI) 서비스의 하나로,   
깃허브에 호스팅되는 소프트웨어 프로젝트의 빌드, 테스트를 위해 사용된다.  


알아보니 github repository의 프로젝트를 빌드/테스트/배포까지도 되는 것 같다.  

<br><br>

# travis-ci 연동하기

우선 [https://travis-ci.org/](https://travis-ci.org/)에서 github 아이디로 로그인으로 연동을 해주면 된다.

그리고 repository를 선택하여 활성화 한다. 

![activate repository](../../images/20200310/activate_repository.PNG){:width="700"}

활성화를 했다면 루트 디렉토리에 .travis.yml파일을 만들어 준다.

```yaml
language: java
jdk:
  - openjdk8
script:
  - ./gradlew check
```

<br>

- 언어는 java / jdk는 openjdk8을 사용하도록 하였다.  
- ./gradlew check을 통해 build.gradle의 설정의 오류를 체크해준다.

<br>

![build success](../../images/20200310/build_success.PNG){:width="800"}
.travis.yml를 커밋하고 job log를 보면 빌드 성공!

이제 뱃지를 달아보자

![build success](../../images/20200310/badge_markdown.PNG){:width="800"}

프로젝트명 오른쪽 뱃지를 누르면 원하는 방식으로 뱃지를 얻을 수 있다.

나는 README.md에 넣을 것이므로 마크다운으로 선택하였다.

이제 원하는 곳에 넣어주면 끝.


<br><br><br><br>

# 삽질

![directory](../../images/20200310/directory.PNG){:width="200"}
프로젝트 디렉토리의 구조가 이렇게 생겼다.

원래는 기록용이기 때문에 /build, /gradle, gradlew 등을 커밋하지 않았다

그런데 이제는 빌드를 해야하기 때문에 올려주어 오류를 해결하였다.

<br><br>

```yaml
language: java
jdk:
  - openjdk8
script:
  - ./gradlew check
before_install:
  - chmod +x gradlew
```

로컬에서는 문제가 없었는데.. 올리면 ./gradlew: Permission denied 에러가 난다...  

서칭하여 `chmod +x gradlew` 을 추가해주었다

<br><br><br>

![build fail](../../images/20200310/build_fail.PNG){:width="400"}

~~참담한 빌드 fail의 현장..(밑에 더 많다는거..ㅎㅎ)~~