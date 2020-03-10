---
layout: post
title: "validation/예외처리"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day8 (validation/예외처리)
date: 2020-03-10T01:00:00Z
---
 
# validation

Spring에서 `@Valid` 어노테이션을 사용하면 쉽게 유효성을 검사할 수 있다.

RestaurantController의 create 부분을 보자.

```java
    @PostMapping("/restaurants")
    public ResponseEntity<?> create(@Valid @RequestBody Restaurant resource) throws URISyntaxException {
        ...
    }
```

생성할 때 받아오는 파라미터 앞에 `@Valid`를 붙여 주었다.  
이제 resource가 유효하지 않으면 에러를 반환하게 된다.

아직은 유효성체크에 대한 설정을 하지 않아 문제가 없다.

```java
    public class Restaurant {
        ...
        @NotEmpty
        private String name;
    
        @NotEmpty
        private String address;
        ...
    }
```

Restaurant 도메인 객체에 유효성에 대한 설정을 해주었다.  
변수 위에 `@NotEmpty`을 붙여주면 해당 변수는 Not Null 속성을 갖는다.

이제 생성할 때 name이나 address가 들어오지 않는다면 오류를 발생 할 것이다.

validation에는 더 많은 어노테이션이 있는데,  
- `@Min(value)`/`@Max(value)` : 값의 최소/최대 값을 설정 
- `@Pattern(regex=, flag=)` : 정규식을 만족해야함   
- `@Size(min=value, max=value)` : 문자열이나 배열이 지정된 값 사이

등 이 있다. 필요에 따라 사용하자.


# 예외 처리

위의 유효성 체크를 해주었다면, 이제 유효하지 않은 값이 들어온 경우 처리를 해보자.

`@ControllerAdvice`라는 어노테이션을 사용하여 예외를 잡아 줄 수 있다.

우선 예외를 발생 해 보자. 가게의 detail을 조회하는 서비스 부분이다. 
```java
    //RestaurantService.java
    public Restaurant getRestaurant(Long id){
        Restaurant restaurant = restaurantRepository.findById(id)
                .orElseThrow(() -> new RestaurantNotFoundException(id));
        List<MenuItem> menuItems = menuItemRepository.findAllByRestaurantId(id);
        restaurant.setMenuItems(menuItems);
        return restaurant;
    }
```
restaurantRepository에서 id를 이용해서 상세를 얻어온다.  
이때 사용자가 존재하지 않는 id를 요청한 경우의 예외처리를 해보자.

기존에는 `orElse(null)` 해주어 상세가 없는 경우 null을 리턴해주었다.  
이번에는 `orElseThrow(() -> new RestaurantNotFoundException(id))`으로 바꿔 주었다.

이렇게 되면 상세가 없는 경우 RestaurantNotFoundException라는 커스텀 Exception이 발생하도록 하였다. 

```java
public class RestaurantNotFoundException extends RuntimeException {

    public RestaurantNotFoundException(long id) {
        super("Could not find restaurant " + id );
    }
}

```

RestaurantNotFoundException는 단순히 RuntimeException 상속받고 메세지만 설정해 주었다.

이제 `@ControllerAdvice`을 사용하여 예외를 catch 해보자.   

```java
@ControllerAdvice
public class RestaurantErrorAdvice {

    @ResponseBody
    @ResponseStatus(HttpStatus.NOT_FOUND)
    @ExceptionHandler(RestaurantNotFoundException.class)
    public String handleNotFound() {
        return "{}";
    }
}
```

RestaurantErrorAdvice 라는 이름의 클래스를 만들고 `@ControllerAdvice` 어노테이션를 붙여 주었다.
  
`@ExceptionHandler` 를 통해 RestaurantNotFoundException이 발생한 경우 처리하도록 하였다.

존재하지 않는 상세를 요청한 경우에 대한 예외이므로 {}값을 리턴해주도록 하였다.

`@ControllerAdvice`을 사용하면 예외에 대한 처리를 쉽게 할 수 있다.