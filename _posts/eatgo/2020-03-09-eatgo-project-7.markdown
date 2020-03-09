---
layout: post
title: "lombok 사용하기"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day7 (lombok)
---
 
# lombok

lombok은 domain 클래스(dto, vo)의 setter/getter/constructor 등을   
lombok의 어노테이션을 사용하여 자동으로 생성해주는 자바 라이브러리이다.

lombok을 인텔리제이에서 사용해보자.

```
//build.gradle
dependencies {
	...
	compileOnly 'org.projectlombok:lombok'
	...
	annotationProcessor 'org.projectlombok:lombok'
	...
}
```
프로젝트를 생성할 때 lombok을 넣어주었기 때문에 build.gradle에 존재한다.

그리고 Settings > Plugins > Marketplace 에서 lombok을 install 해준다.

(Setting 바로가기 : Ctrl + Alt + S)

![lombok install](../../images/20200309/lombok.PNG){: width="600"}



# lombok을 이용하여 기존 소스 수정하기

lombok은 코드의 양을 현저하게 줄일 수 있다. 또한 개발자의 실수를 줄일 수 있다.

얼마나 짧아졌는지 확인해보면..

```java
//기존 Restaurant.java
@Entity
public class Restaurant {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String address;

    @Transient
    private List<MenuItem> menuItems = new ArrayList<>();

    public Restaurant(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public Restaurant(Long id, String name, String address){
        this.id = id;
        this.name = name;
        this.address = address;
    }

    public Restaurant() {
    }

    public void setId(long id) {
        this.id = id;
    }
    public  Long getId(){
        return id;
    }

    public String getName(){
        return name;
    }

    public String getAddress(){
        return address;
    }

    public String getInformation(){
        return name + " in " + address;
    }

    public void addMenuItem(MenuItem menuItem) {
        menuItems.add(menuItem);
    }

    public List<MenuItem> getMenuItems(){
        return menuItems;
    }

    public void setMenuItems(List<MenuItem> menuItems) {
        for(MenuItem menuItem : menuItems) {
            addMenuItem(menuItem);
        }
    }

    public void updateInformation(String name, String address){
        this.name = name;
        this.address = address;
    }
}

```
   

각종 setter, getter에 생성자도 많아 헷갈리고 @Entity를 사용하기위해 넣어준 기본생성자도 보기 안 좋다.

 
```java
//수정된 Restaurant.java
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Restaurant {
   @Id
   @GeneratedValue
   @Setter
   private Long id;

   private String name;

   private String address;

   @Transient
   private List<MenuItem> menuItems;

   public String getInformation(){
       return name + " in " + address;
   }

   public void setMenuItems(List<MenuItem> menuItems) {
       this.menuItems = new ArrayList<>(menuItems);
   }

   public void updateInformation(String name, String address){
       this.name = name;
       this.address = address;
   }
}
``` 

@Data 어노테이션을 붙이면 더 간단하게 setter/getter를 만들 수 있다.

여기서는 필요한 부분에만 setter를 따로 붙여주었다.

어노테이션이 많이 붙긴했지만 매우 짧아졌고 의미가 드러나도록 수정되었다.

어노테이션을 하나씩 보자
- @Getter : 변수들의 getter를 만들어준다.
- @Builder : builder 패턴으로 객체를 생성할 수 있게 해준다.
- @NoArgsConstructor : 기본생성자를 만들어준다.
- @AllArgsConstructor : 모든 변수를 갖는 생성자를 만들어준다.


# builder pattern

기존 소스들은 그렇게 많이 수정되지는 않는다.

인스턴스를 생성하는 부분을 builder pattern으로 수정할 것이다.

기존에는 생성자를 이용하여 인스턴스를 생성하였다.
```java
    Restaurant restaurant = new Restaurant(1L, "Samgyepsal", "Seoul"); 
```

builder pattern을 이용하여 생성하도록 수정하였다.

```java
    Restaurant restaurant = Restaurant.builder()
        .id(1001L)
        .name("Samgyepsal")
        .address("Seoul")
        .build();
``` 

기존 방식이 라인이 짧아 간단해 보이지만 argument가 늘어나면  
그에 맞는 생성자를 만들어 주어야하고 무엇을 의미하는지 알기가 어렵다.  

이제 무엇을 의미하는지 한눈에 확인 가능해졌다.    

    
# ...

- lombok의 어노테이션은 사용한 것 말고도 많이 있다. 필요에 따라 쓰면 될 것 이다.

- ~~(뻘짓)markdown 문법을 숙지하지 않아서 제목을 html 문법을 사용하고 있었다 h2...~~