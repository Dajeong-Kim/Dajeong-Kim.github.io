---
layout: post
title: "JPA/프론트엔드(Node.js)/REST 수정"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day6 (JPA/프론트엔드(Node.js)/REST 수정)
---
 
<h2>JPA</h2>

기존 프로젝트까지는 도메인 객체들이 어플리케이션 메모리에서 관리가 되었다.<br>
그렇게되면 프로그램을 끄면 데이터가 사라진다.

하지만 데이터들은 영속성(Persistence)이 있어야 한다.<br>
그렇기 때문에..

> JPA (Java Persistence API)

Java 표준인 JPA를 사용할 것이다.

> Hibernate

JPA의 구현체 중 하나라고 하는데 이 부분은 나중에 다시 알아보자


> @Entity

@Entity 어노테이션을 사용하면 JPA로 사용가능하다.<br>
그렇게 되면 JPA에서 관리가 된다.

> Spring Data JPA

Spring Data JPA를 통해 쉽게 JPA를 활용할 수 있다.<br>
원래는 설정등이 복잡하다고 한다.

> H2 Database

지금부터는 H2 db를 사용할 것인데, H2는 In-memory 방식이다. <br>
그렇게 되면 영속성이 없는 것으로 여길 수 있지만 H2 설정을 해주면<br>
memory를 file로 저장가능하다.

<br><br>

---

<br><br>

<h2>H2 사용</h2>

```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'com.h2database:h2'
```
우선 build.gradle에 해당 코드를 추가해 주었다.

그러고나서 build > rebuild project를 해주었다.



<h2>JPA 사용</h2>
```
public interface RestaurantRepository extends CrudRepository<Restaurant, Long>
...
```
repository에서 CrudRepository를 상속받도록 수정해주었다.

```
@Entity
public class Restaurant {

    @Id
    @GeneratedValue
    private Long id;
    ...
```

도메인 객체에 @Entity 어노테이션을 붙여주고

id에는 @Id, @GeneratedValue 를 붙여서 자동적으로 증가하는 기본키로 설정해주었다.

<br><br>

----
<br>
<h2>Front-end 설정</h2>

이 프로젝트에서는 [Webpack](https://webpack.js.org/) 을 사용할 것이다.<br>
Webpack은 모듈 번들러라고 하는데, 자동으로 묶어 주는 역할을 하는 것 같다.

Webpack을 사용하기 전에 [Node.js](https://nodejs.org/ko/)를 설치해준다.

> $npm install --save-dev webpack webpack-cli webpack-dev-server
 
npm 명령어로 webpack을 install 해준다.

그런뒤 npm init을 해서 package.json을 생성해 준다.

![npm init](../../images/20200308/image1.png)

![npm init](../../images/20200308/image2.png)

eatgo-web란 이름의 패키지를 생성했다.

entry point로 설정하면 해당 js를 main.js로 호출이 가능하다.


```
"scripts": {
    "start": "webpack-dev-server --port 3000"
    ..
}
```

package.json에 start를 추가해 주었다. 

webpack-dev-server는 빠른 실시간 리로드를 할 수 있다고 한다.

웹서버의 포트는 3000으로 정해주었다.

```
(async () => {
    const url = 'http://localhost:8080/restaurants';
    const response = await fetch(url);
    const restaurants = await response.json();

    const element = document.getElementById('app');
    element.innerHTML = `
        ${restaurants.map(restaurant => `
            <p>
                ${restaurant.id}
                ${restaurant.name}
                ${restaurant.address}
            </p>
        `).join('')}
    `;
})();
``` 
index.js 이다.
어플리케이션 서버에 식당 리스트를 뿌려주도록 하였다. 

웹서버는 3000번 포트로 지정해주었고 어플리케이션 서버는 8080번 포트를 사용하기 때문에
데이터를 받아올 수 없다.

> CORS(cross-origin resource sharing)

도메인이나 포트가 다른 서버에 접근하기 때문이다.

> @CrossOrigin

서버단에서 @CrossOrigin 어노테이션을 사용하여 허용해준다.
```
@CrossOrigin
@RestController
public class RestaurantController {
    ...
}
```

![eatgo-web test](../../images/20200308/image3.png)

이제 데이터를 받아 올 수 있다.
<br>
<br>

-----

<h2>REST 수정</h2>

> PUT /restaurants/{id}
> PATCH /restaurants/{id} 

rest에서 수정은 PUT 또는 PATCH로 사용 가능하다

PUT과 PATCH의 차이를 찾아보니

- PUT : 전체 엔티티를 전달해 주어야한다.
- PATCH : 변경하고자 하는 속성만 넣어도 된다.

이 프로젝트에서는 PATCH를 사용할 것이다.

수정 성공시 HTTP Status는 200이다.

> @Transactional

데이터를 수정하면 db 수정하도록 어노테이션을 붙여주었다. 

```
    @Transactional
    public Restaurant updateRestaurant(long id, String name, String address) {
        Restaurant restaurant = restaurantRepository.findById(id).orElse(null);
        restaurant.setName(name);
        restaurant.setAddress(address);
        return restaurant;
    } 
```


<h2>...</h2>
JPA에 대해 아주 조금 학습하였다.


