---
layout: post
title: "메뉴 추가/수정/삭제"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day8 (메뉴 추가/수정/삭제)
date: 2020-03-10T02:00:00Z
---

# MenuItem 추가/수정/삭제
이제 Restaurant의 Sub객체인 MenuItem 추가/수정/삭제을 할 것이다.  
MenuItem은 여러개 업데이트를 위해 bulk update를 한다.

> PATCH /restaurants/{id}/menuitems

http 요청은 위의 형태로 추가/수정/삭제를 할 것이다. 

성공하면 http Status는 200이다.

save와 deleteById를 이용한다.

- save - 모두 저장
- deleteById - 해당 메뉴 삭제

# 메뉴 추가
```json
[
  {
    "name": "Kimchi"
  },
  {
    "name": "Gukbob"
  }
]
```

메뉴를 추가하기 위해 위와 같은 menuitems.json파일을 만들어 주었다.

```java
@RestController
public class MenuItemController {

    @Autowired
    private MenuItemService menuItemService;
    
    @PatchMapping("restaurants/{restaurantId}/menuitems")
    public String bulkUpdate(
            @PathVariable("restaurantId") long restaurantId,
            @RequestBody List<MenuItem> menuItems
    ) {
        menuItemService.bulkUpdate(restaurantId, menuItems);
        return "";
    }
}
```

MenuItemController는 위와 같이 만들어주었다.  
url을 patch로 매핑해주고 bulkUpdate()를 수행하도록 해주었다.

```java
    // MenuItemService.java
    public void bulkUpdate(long restaurantId, List<MenuItem> menuItems) {
        for (MenuItem menuItem : menuItems){
            update(restaurantId, menuItem);
        }
    }

    private void update(long restaurantId, MenuItem menuItem) {
        menuItem.setRestaurantId(restaurantId);
        menuItemRepository.save(menuItem);
    }
```

```java
// MenuItemRepository.java
List<MenuItem> findAllByRestaurantId(Long restaurantId);
```

> http PATCH localhost:8080/restaurants/1/menuitems < menuitems.json

이제 이렇게 해주면 메뉴를 추가할 수 있다.

그런데 실행을 할 때마다 계속 메뉴가 추가가 된다. 수정이나 삭제도 해보자.  

# 메뉴 수정

```json
[
  {
    "id": 2,
    "name": "Samgyepsal"
  },
  {
    "id": 3,
    "name": "Jabchae"
  }
]
```
메뉴 수정을 위해 menuitems.json을 수정했다. 

생성된 메뉴의 id를 지정해서 요청하면 해당 메뉴를 수정할 수 있다.

# 메뉴 삭제
 
```json
[
  {
    "id": 2,
    "destroy": true
  }
]
```
이번엔 메뉴 삭제를 위해 menuitems.json을 수정했다.  
destroy라는 값을 두어 삭제를 해보자.

```java
    //MenuItem.java
    @Transient
    @JsonInclude(JsonInclude.Include.NON_DEFAULT)
    private boolean destroy;
```
MenuItem.java에 destroy를 추가해 주었다.
`@Transient` 어노테이션을 달아서 DB에 넣지 않을 값으로 지정해 주었다. 

이제 destroy=true로 넘어오면 해당 id의 메뉴를 삭제해보자.

```java
    // MenuItemService.java
    public void bulkUpdate(long restaurantId, List<MenuItem> menuItems) {
        for (MenuItem menuItem : menuItems){
            update(restaurantId, menuItem);
        }
    }

    private void update(long restaurantId, MenuItem menuItem) {
        if(menuItem.isDestroy()){
            menuItemRepository.deleteById(menuItem.getId());
            return;
        }
        menuItem.setRestaurantId(restaurantId);
        menuItemRepository.save(menuItem);
    }
```

menuItem.isDestroy()일때 메뉴를 삭제하도록 하였다.

```java
// MenuItemRepository.java
void deleteById(Long id);
```

MenuItemRepository.java에 deleteById를  추가하였다.

이제 메뉴에 대한 추가/수정/삭제를 할 수 있다.  
json 데이터로 값을 넘겨주어 번거롭게 느껴질 수 있으나, 프론트단을 만들어주면 쉽게 처리된다. 


# MenuItemService 테스트

```java
    //MenuItemServiceTest.java
    @Test
    public void bulkUpdate() {
        List<MenuItem> menuItems = new ArrayList<>();

        menuItems.add(MenuItem.builder().name("Samgyeopsal").build());
        menuItems.add(MenuItem.builder().id(1001L).name("Jabchae").build());
        menuItems.add(MenuItem.builder().id(1002L).destroy(true).build());

        menuItemService.bulkUpdate(1L, menuItems);

        verify(menuItemRepository, times(2)).save(any());
        verify(menuItemRepository, times(1)).deleteById(1002L);
    }
```

MenuItemService의 테스트를 해보자. menuItems에 3개의 메뉴를 만들었다. 

- 첫번째는 Samgyeopsal메뉴를 추가
- 두번째는 1001번 id를 갖는 메뉴를 Jabchae로 수정
- 세번째는 1002번 id를 갖는 메뉴를 삭제

이렇게 설정하고 bulkUpdate()를 수행하면 

2개에 대해서는 저장하고, 1개는 삭제되는 것을 확인 할 수있다.

times()를 사용하여 호출되는 횟수도 체크가 가능하다.