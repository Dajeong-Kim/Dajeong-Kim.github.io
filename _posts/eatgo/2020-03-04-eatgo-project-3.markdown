---
layout: post
title: "REST API의 목록/상세 및 DI"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day3 (REST목록/상세, DI)
---

<h3>REST API</h3>
> REST API의 CRUD

- GET - READ
- POST - INSERT
- PUT/PATCH - UPDATE
- DELETE - DELETE

<h3>레스토랑 목록</h3>
```
    @GetMapping("/restaurants")
    public List<Restaurant> list(){
        List<Restaurant> restaurants = restaurantRepository.findAll();
        return restaurants;
    }
```
레스토랑의 목록은 GET /restaurants으로 요청한다.<br>
간단한 CRUD는 흔히 접하는 형식이고 테스트 위주로 작성해보면<br>
```
    @Autowired
    private MockMvc mvc;

    @SpyBean(RestaurantRepositoryImpl.class)
    private RestaurantRepository restaurantRepository;

    @Test
    public void list() throws Exception {
        mvc.perform(get("/restaurants"))
                .andExpect(status().isOk())
                .andExpect(content().string(
                        containsString("\"name\":\"Bob Zip\"")
                ))
                .andExpect(content().string(
                        containsString("\"id\":1004")
                ));
    }
```
위와 같은 코드로 테스트를 할 수 있다.<br>

> MockMvc

웹 어플리케이션을 서버에 배포하지 않고 스프링 MVC 동작을 재현할 수 있다.
 
>@SpyBean

컨트롤러에 직접 원하는 객체를 주입가능

> perform

 MockMvc의 메서드로 get/post/put/delete 에 대한 요청 진행 가능 
 
> .andExpect(status().isOk())

status가 200이며

> .andExpect(content().string(containsString(내용)))

response body에 내용이 있는지 확인

<h3>레스토랑 상세</h3>
```
    @GetMapping("restaurants/{id}")
    public Restaurant detail(@PathVariable("id") Long id){
        Restaurant restaurant = restaurantRepository.findById(id);
        return restaurant;
    }
```
id에 따라 해당 요청을 하기 위해 @PathVariable를 사용

```
    @Test
    public void detail() throws Exception {
        mvc.perform(get("/restaurants/1004"))
                .andExpect(status().isOk())
                .andExpect(content().string(
                        containsString("\"name\":\"Bob Zip\"")
                ))
                .andExpect(content().string(
                        containsString("\"id\":1004")
                ));

        mvc.perform(get("/restaurants/2020"))
                .andExpect(status().isOk())
                .andExpect(content().string(
                        containsString("\"name\":\"Cyber Food\"")
                ))
                .andExpect(content().string(
                        containsString("\"id\":2020")
                ));
    }
```

> 상세도 목록과 동일한 방법으로 테스트


```
//RestaurantRepository.java 
public interface RestaurantRepository {
    List<Restaurant> findAll();

    Restaurant findById(Long id);
}

//RestaurantRepositoryImpl.java
@Component
public class RestaurantRepositoryImpl implements RestaurantRepository {
    private List<Restaurant> restaurants = new ArrayList<>();
    
    public RestaurantRepositoryImpl(){
        restaurants.add(new Restaurant(1004L, "Bob Zip", "Seoul"));
        restaurants.add(new Restaurant(2020L, "Cyber Food", "Seoul"));
    }

    @Override
    public List<Restaurant> findAll() {
        return restaurants;
    }

    @Override
    public Restaurant findById(Long id) {
        Restaurant restaurant = restaurants.stream()
                .filter(r -> r.getId().equals(id))
                .findFirst()
                .orElse(null);
        return restaurant;
    }
}
```

> 아직 db연동을 안해서 지저분하지만..<br> 
> interface로 repository를 만들고<br>
> repositoryImpl에서 repository를 구현해준다.<br>

---
<br><br>
<h3>Dependency Injection</h3>
>의존성 주입

* 의존성 -> 둘 이상의 객체가 협력한다.<br>
* 주입 -> 외부에서 객체를 생성하여 넣어준다.
* 위의 코드에서는 Controller는 Repository에 의존한다.

> 왜 쓰는 것일까?

* 사용하는 객체를 다양하게 변경가능하다.
* 강하게 연결되어있는 객체를 유연하도록 해준다.