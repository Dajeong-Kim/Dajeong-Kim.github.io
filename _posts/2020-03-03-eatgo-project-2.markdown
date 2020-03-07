---
layout: post
title: "TDD기반의 프로젝트 만들어보자"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day2 (부제:TDD)
---

<h3>TDD</h3>
> TDD란 Test Driven Development, 테스트 주도 개발

- Red : 실패
- Green : 성공
- Refactoring : 리팩토링

TDD란 위의 Red, Green, Refactoring 이 순환하는 구조이다.
올바르게 작동하는 코드를 작성하여 리팩토링한다.

Java에서는 junit을 사용하여 TDD를 한다.

```
public class Restaurant {
    public String getName(){
        return "Bob Zip";
    }
}
```
위과 같은 Restaurant의 도메인을 만들었다.
테스트를 수행해보자.

> 해당 클래스에서 오른쪽 버튼을 누르고 Go To > Test > Create New Test
 
```
    @Test
    public void creation(){
        Restaurant restaurant = new Restaurant();
        assertThat(restaurant.getName(), is("Bob Zip"));
    }
```

>Test 클래스를 작성하고 <br>
>assertThat을 통해 restaurant의 이름이 "Bob Zip"인지 확인한다.

<br><br><br>

    

<h2>..</h2>
> 실무를 하면서도 TDD의 중요성은 많이 느꼈었다.<br>
> 바쁘다는 핑계로 중요한 부분 이외에는 테스트기반의 프로그램을 만들지 못했던 것 같다.<br>
> 반성하며.. 앞으로는 열심히 하자.