---
layout: post
title: "Mockito, HTTPie 사용하기, REST API 추가"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day5 (가짜객체/REST 추가)
---

<h2>가짜객체</h2>
테스트를 위해 Mock Object를 사용할 수 있다.<br>
가짜객체란 역할을 똑같이 해내는 대역을 의미한다.<br>
자바에서는 Mockito라는 라이브러리를 사용하여 테스트가 가능하다.
<br><br>

> @Mock / @MockBean 

해당 어노테이션을 사용하면 해당 객체를 Mock 객체로 사용가능하다.

![mockito](../../images/20200307/mockito.png){: width="300"} 

<s>로고가 귀여워서 가져와봤다</s>

```java
@MockBean
private RestaurantService restaurantService;
```

위와 같은 모양으로 바꿔주었다.

---
<br><br>
<h2>HTTPie 사용</h2>

이전 강좌에서는 목록/상세까지 학습하였다.<br>
오늘은 추가에 대해 알아본다.

> POST /restaurants

추가시에는 post로 요청한다.<br>
성공시에는 http status는 201을 받는다.<br>
또한 만들어낸 정보를 header location에 담아 보내준다.<br>


> HTTPie
 
[HTTPie](https://httpie.org/)는 python 기반의 유틸리티로 http 개발 및 디버깅에 편리하다.<br>
기존에는 postman만 사용해 보았는데,<br>
HTTPie를 사용해보니 직관적이고 사용하기 편하다.
<br><br>
 
파이썬이 설치되어 있지 않아서 깔아주고..파이썬의 pip를 먼저 설치해주었다.
 

> $pip install --upgrade pip setuptools<br>
  $pip install --upgrade httpie

나는 윈도우니까 cmd에서 위의 명령어를 실행해주면

> http GET localhost:8080/restaurants

터미널에서 이런 식으로 사용가능하다.

<br><br>
<h2>REST 추가</h2>

```java
    @PostMapping("/restaurants")
    public ResponseEntity<?> create(@RequestBody Restaurant resource) throws URISyntaxException {
        String name = resource.getName();
        String address = resource.getAddress();

        Restaurant restaurant = new Restaurant(name, address);
        restaurantService.addRestaurant(restaurant);

        URI location = new URI("/restaurants/"+restaurant.getId());
        return ResponseEntity.created(location).body("{}");
    }
```

RestaurantController에 create()를 추가해주었다.<br>
아직은 제대로된 코드는 아니고 추가하는 로직을 만들어주었다.


```java
    @Test
    public void create() throws Exception{
        //Restaurant restaurant = new Restaurant("SamGyeopSal", "Seoul");
        mvc.perform(post("/restaurants")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"BeRyong\",\"address\":\"Busan\"}"))
                .andExpect(status().isCreated())
                .andExpect(header().string("location", "/restaurants/1234"))
                .andExpect(content().string("{}"));

        //verify(restaurantService).addRestaurant(restaurant);
        verify(restaurantService).addRestaurant(any()); //아무 객체나 통과
    }

```

RestaurantControllerTest에서 테스트는 기존 목록/상세와 약간 달라졌다.

- json타입으로 받을 것 이므로 contentType(MediaType.APPLICATION_JSON)을 지정
- http status도 기존 status.isOk()에서 status().isCreated()로 체크
- header location에 만들어낸 레스토랑의 주소를 보내줌
- 목록/상세에서는 given()메서드로 유효한지 확인했다면 verify()를 사용하여 체크 
 
<br><br><br>

그럼 HTTPie를 이용해 가게를 추가해보자

![post 요청](../../images/20200307/image1.png)

삼겹살집이 잘 들어가는 것을 확인하였다.

![목록](../../images/20200307/image2.png)

목록도 잘 나온다.


<h2>...</h2>
HTTPie를 사용하니까 직관적이고 편하다.<br>
포스트맨도 좋지만 간단한 테스트하기엔 HTTPie가 편한 것 같다.<br>
TDD기반의 개발을 하기로 마음을 먹었지만서도.. 손에 잘 안붙는다 노력해야지..<br>

다음시간에는 드디어 JPA에 대해 배운다.


 