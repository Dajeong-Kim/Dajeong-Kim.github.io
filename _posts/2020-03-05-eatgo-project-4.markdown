---
layout: post
title: "스프링부트-day4"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day4 (부제:레이어 분리)
---

<h2>레이어 분리</h2>
이전 수업까지는 ui layer/domain layer 까지 나누도록 학습하였다.<br>
이번에는 application layer를 추가한다.
 
* interfaces layer - controller
* application layer - service
* domain layer - domain, repository

 
> application layer인 Service를 추가해 보자

```
@Service
public class RestaurantService {
    @Autowired
    RestaurantRepository restaurantRepository;
    //...
}
```

> @Service

Service어노테이션을 붙여준다.
비즈니스 로직이나 repository layer를 호출하는 곳에서 사용

```
public class RestaurantServiceTest {

    private RestaurantService restaurantService;
    private RestaurantRepository restaurantRepository;
    private MenuItemRepository menuItemRepository;

    @BeforeEach
    //@Before //모든 test를 수행하기 전에 항상 수행
    public void setUp(){
        restaurantRepository = new RestaurantRepositoryImpl();
        menuItemRepository = new MenuItemRepositoryImpl();
        restaurantService = new RestaurantService(restaurantRepository, menuItemRepository);
    }

    @Test
    public void getRestaurants(){
        List<Restaurant> restaurants = restaurantService.getRestaurants();
        Restaurant restaurant = restaurants.get(0);
        assertThat(restaurant.getId(), is(1004L));
    }

    @Test
    public void getRestaurant(){
        Restaurant restaurant = restaurantService.getRestaurant(1004L);
        assertThat(restaurant.getId(), is(1004L));
    }
}
```

Service도 테스트를 만들어준다.

> @Before

@Before 어노테이션을 붙이면 테스트를 수행하기전에 먼저 수행한다고 한다.<br>

그런데 restaurantService을 못받아오는 에러가 발생..<br>

> @BeforeEach
>  
[해결법](https://stackoverflow.com/questions/47383225/before-beforeclass-seems-not-working-objects-indicates-on-null)을 참고하여 @BeforeEach로 수정해주었다. 
<hr>
<br><br><br>

<h3>인텔리제이 단축키</h3>

- 한줄 주석 : ctrl + /
- 여러줄 주석 : ctrl + shift + /