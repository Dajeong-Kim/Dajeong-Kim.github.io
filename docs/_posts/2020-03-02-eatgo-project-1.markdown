---
layout: post
title: "스프링부트-day1"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day1 (부제:hello world)
---

<h3>들어가기 전에..</h3>
>패스트 캠퍼스 > 자바 웹 마스터 > 스프링 부트 프로젝트에 대한 기록

<h3>강의 내용</h3>
* 주제: 오늘 뭐 먹지?
* 어떻게 만들 것인가?
* 도메인: Restaurant, Menu Item, Review, Food&Beverage, Reservation 등..
* 시스템 아키텍쳐: 홈페이지-> Web, Mobile
3-tier Architecture: Presentation, Business, Data Source
1. Presentation: front-end, html/css/js
2. Business: back-end, Rest-API
3. Data-source: DBMS

<hr/>
<br><br><br>

<h2>1. Step.1</h2>
> [스프링 사이트](https://spring.io/)에서 Spring Initializr를 다운받기.

![스프링 사이트](../../assets/media/20200302/image1.PNG)
> Spring Initializr로 이동해준다.

![Spring Initializr1](../../assets/media/20200302/image2.PNG)
> Gradle을 선택
 Group과 Artifact를 입력

![Spring Initializr2](../../assets/media/20200302/image3.PNG)
> Spring Web, SpringBoot DevTools, lombok을 추가

<br><br><br>
<h2>1. Step.2</h2>
> step.1에서 생성한 프로젝트를 열어준다.

![Spring Initializr2](../../assets/media/20200302/image4.PNG)

> @SpringBootApplication 어노테이션이 붙은 EatgoApplication이 생성되었다.
> run을 한 뒤 localhost:8080에 접속해보면..

![Spring Initializr2](../../assets/media/20200302/image5.PNG)

> 에러가 발생한다

```
@RestController
public class WelcomeController {
    @GetMapping("/")
    public String hello(){
        return "Hello, World!";
    }
}
```
> WelcomeController를 만들어서 매핑을 시켜주면..

![Spring Settings](../../assets/media/20200302/image6.PNG)

> 잘 나오는 것을 확인할 수 있다.<br><br>
> 프로그램을 수정한 뒤 EatgoApplication을 run하지않고<br>
>EatgoApplicationTest를 run해주어도 수정 사항이 반영된다.

<br><br>

<h3>마치며..</h3>
> 중요한 내용은 아직 들어가지도 않았는데 해주어야 할 작업이 많았다. <br>
> 작심삼일로 끝나지 않도록 꾸준히 포스팅하자
