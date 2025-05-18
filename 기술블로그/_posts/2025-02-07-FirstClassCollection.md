---
layout: post  
title: "일급 컬랙션"
author: "김규영"
categories: "기술블로그"
banner:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["First Class Collection", "일급 컬렉션"]
---

### 일급 컬력션?
- 일급 컬랙션은 클래스에 Collection 외에 다른 필드가 없는 클래스를 의미합니다.
  ```java
  public class FirstClass {
    List<Integer> collection;
    
    public FirstClass(List<Integer> collection) {
        this.collection = collection;
    }
  }
  ```

### 일급 컬렉션을 쓰는 이유
- 일급 컬렉션을 사용함으로서 다음과 같은 이점을 얻을 수 있습니다.
  1. 비즈니스 종속적 구조
  2. 컬렉션의 불변을 보장
  3. 컬렉션이 이름을 부여
  4. 응집도 증가
  5. 명확한 책임 분리

#### 불변
- 일급 컬렉션을 사용하지 않는 경우 외부에서 컬렉션을 가져와 임의로 추가 삭제가 가능합니다.

```java
public class Order {
    private List<Item> items;

    public Order(List<Item> items) {
        this.items = items;
    }

    public List<Item> getItems() {
        return items;
    }
}

class Other {
  
  public void method() {
    //...
    List<Item> items = getItems();
    items.clear();
  }
}
```
- 외부에서 자유롭게 변경할 수 있는 컬렉션을 반환합니다.
- 위 구조도 일급 컬렉션이지만 이해를 돕기위한 장면입니다.
  - Order라는 클래스는 다른 필드가 추가될 가능성이 높은 이름인 것 같기도 합니다.



-
-
- 일급 컬렉션을 사용한 경우
```java
public class Items { 
    private final List<Item> items;

    public Items(List<Item> items) {
        this.items = new ArrayList<>(items);
    }

    public int totalPrice() {
        return items.stream().mapToInt(Item::getPrice).sum();
    }

    public List<Item> getItems() {
        return Collections.unmodifiableList(items);
    }
}
```

## 얻을 수 있는 이점
1. Items 클래스에서 컬렉션을 직접 관리하므로, 컬렉션 관련 로직이 한 곳에 집중됨 -> 비즈니스 종속적인 구조
2. getItems()에서 Collections.unmodifiableList()를 사용하여 외부에서 컬렉션을 변경할 수 없도록 방어적 처리 -> 불변 보장
3. Order 클래스가 List<Item>을 직접 다루지 않고 Items 클래스를 통해 다루므로 책임이 분리됨 -> 응집도 증가
4. 컬렉션에 이름 부여

### 마무리?
- 단순히 데이터를 담는 용도라면 클래스로 감싸지 않아도 아무런 문제가 없습니다.
  - 컬렉션에 비즈니스 로직이 필요하거나 검증이 필요한 경우에 일급 컬렉션을 사용했을 때 이점이 있습니다.
      - 예를 들면 조회된 데이터에 삭제 여부를 검증하거나 권한에 검증이 필요한 경우
- 일급 컬렉션은 불변을 유지해야 합니다.
  - 컬렉션을 클래스로 감싸놓고 조치 없이 그대로 갖다 준다면 아무런 의미가 없어요.
- 컬렉션을 감싸서 일급 컬렉션을 만드는게 중요한게 아니라 컬렉션에 필요한 로직의 응집도를 증가시키는게 중요합니다.
  - 비즈니스로직이나 검증이 필요하지 않은 컬력션을 감싸서 일급 컬렉션을 만드는것에 집착할 필요 없습니다.














