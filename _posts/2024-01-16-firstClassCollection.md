---
layout: post
title: 일급 컬렉션이란?
author: 박석희
categories: 기술세미나
banner:
  image: https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0116/thumb_first_class.png
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [java, 일급 컬렉션, 객체지향, 기술세미나]
---
안녕하세요, 지난번 [자바의 가비지 컬렉션](https://stonehee99.vercel.app/java-gc) 발표에 이어서 또 다시 기술 세미나를 맡아 시작하게 되었습니다.

특별히 지난번 가비지 컬렉션 발표에 이어서 또 다른 컬렉션에 대해 발표를 해보면 어떨까? 하는 생각에 (농담입니다)

객체지향에서 크루분들이 어려워하는 개념 중 하나인 `일급 컬렉션` 에 대해 발표를 진행하기로 하였답니다.

## 1. 일급 컬렉션(first-class-collection)이란?
![Untitled](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0116/Untitled.png)
`일급 컬렉션` 에 대한 개념은 `마틴 파울러` 의 책 `소트웍스 앤솔러지` 에서 처음으로 제시되었습니다.

### 규칙 8: 일급 콜렉션 사용

> 이 규칙의 적용은 간단하다. 콜렉션을 포함한 클래스는 반드시 다른 멤버 변수가 없어야 한다. 각 콜렉션은 그 자체로 포장이 되어있으므로 이제 콜렉션과 관련된 동작은 근거지가 마련된 셈이다. 필터가 이 새 클래스의 일부가 됨을 알 수 있다. 필터는 또한 스스로 함수 오브젝트가 될 수 있다. 또한 새 클래스는 두 그룹을 같이 묶는다든가 그룹의 각 원소에 규칙을 적용하는 등의 동작을 처리할 수 있다. 이는 인스턴스 변수에 대한 규칙의 확실한 확장이지만 그 자체를 위해서도 중요하다. 콜렉션은 실로 매우 유용한 원시 타입이다. 많은 동작이 있지만 후임 프로그래머나 유지보수 담당자에게 의미적 의도나 단초는 거의 없다.

위 내용을 정리해보자면, `일급 컬렉션` 이란.

- 컬렉션만을 멤버 변수로 갖는 클래스
- 컬렉션과 관련된 로직을 클래스 내에 캡슐화

를 뜻한다고 할 수 있겠습니다.

예시를 들어보겠습니다.

```java
class Product {
	private String name;
	private String category;
	// 생성자, getter, setter..
}
```
이러한 상품 객체가 있고
```java
List<Product> products = new ArrayList<>();
products.add(new Product("potato", "vegetables"));
products.add(new Product("robot", "toy"));
```
이러한 상품의 리스트 컬렉션이 있다고 가정을 해봅시다.

이를 아래와 같이 감싸는 것을 `일급 컬렉션` 으로 만들었다고 합니다.

```java
public class ProductCollection {
	private final List<Product> products;

	public ProductCollection(List<Product> products) {
		this.products = new ArrayList<>(products);
	}
}
```
이렇게 `일급 컬렉션`을 알아보았는데, 이 `일급 컬렉션`을 사용하면 어떠한 이점이 있는걸까요?

왜 `일급 컬렉션`을 사용해야 한다고 할까요?

## 2. 왜 일급 컬렉션인가?

### 2-1. 캡슐화와 관심사의 분리

```java
public class ProductCollection {
	private final List<Product> products;

	public ProductCollection(List<Product> products) {
		this.products = new ArrayList<>(products);
	}

	public ProductCollection filterByCategory(String category) {
		return new ProductCollection(
			products.stream()
				.filter(product -> product.getCategory().equals(category))
				.collect(Collectors.toList())
		);
	}
}
```

`일급 컬렉션`을 사용하면 컬렉션에 대한 모든 조작과 로직을 해당 컬렉션 클래스 내부에 구현할 수 있습니다.

이로써 로직이 분산되는 것을 방지하고, 관련 로직을 한 곳에서 관리할 수 있습니다.

또 클래스가 단 하나의 책임만을 가지게 됩니다. 이 클래스에서는 `products` 컬렉션을 관리하는 책임만을 가짐으로써 코드의 가독성과 유지보수성이 향상됩니다.

### 2-2. 일급 컬렉션의 불변성 보장


```java
public class TeamMembers {
	private final List<Member> members;
	
	public TeamMembers(List<Member> members) {
		this.members = new ArrayList<>(members);
	}

	public void addMember(Member member) {
		members.add(member);
	}

	public void removeMember(Member member) {
		members.remove(member);
	}

	public List<member> getMembers() {
		return Collections.unmodifiableList(Members);
	}
}
```

`일급 컬렉션`을 사용하면 불변성 보장을 더욱 용이하게 할 수 있어 외부에서의 컬렉션 상태 변경을 방지할 수 있습니다. 이를 통해 데이터의 안정성과 예측 가능성이 증가합니다. 또한 불변성이 보장되면 사이드 이펙트의 가능성이 줄어들며, 멀티 스레드 환경에서의 안정성이 향상됩니다.

이 코드에서 `TeamMembers` 클래스는 어떠한 팀의 멤버들의 리스트를 관리합니다. 여기서 멤버를 추가하거나 제거하는 메서드가 클래스 내부에 구현되어 있고, 일반적인 `List.add()` 와 같은 메서드를 통해 상태변경이 불가능합니다.

즉 이 클래스의 메서드를 통해서만 상태변경이 이루어질 수 있습니다.

`getMemebers()` 메서드는 불변 리스트를 반환하여 외부에서 컬렉션을 변경하는 것을 방지합니다.

이처럼 `일급 컬렉션`을 사용하면 컬렉션의 불변성을 내부 로직을 통해 더욱 강력하게 관리할 수 있습니다.

### 2-3. 비즈니스 규칙과 검증 로직의 중앙화

```java
public class StudentCollection {
	private final List<Student> students;

	public StudentCollection(List<Student> students) {
		this.students = new ArrayList<>();
		for (Student student : students) {
			addStudent(student);
		}
	}

	public void addStudent(Student student) {
		validateStudent(student);
		students.add(student);
	}

	private void validateStudent(Student student) {
		if (student.getAge() < 18) {
			throw new IllegalArguemntException("학생 중 18세 이상만 팀에 들어올 수 있습니다");
		}
	}
}
```

`일급 컬렉션`을 통해서 컬렉션에 적용되어야 할 비즈니스 규칙을 클래스 내부에 정의함으로써 일관된 규칙 적용이 가능합니다.

컬렉션에 추가되거나 변경되는 요소들에 대한 검증 로직을 `일급 컬렉션` 내부에 구현함으로써, 데이터의 정합성을 보다 체계적으로 관리할 수 있습니다.

`addStudent` 메서드에서 `validateStudent` 를 호출하여 학생의 나이가 18세 이상인지 검증합니다. 이렇게 하면 모든 학생 객체가 일관된 규칙을 따를 수 있겠죠?!

## 3. 일급 컬렉션을 사용할 때의 주의사항

### 오버엔지니어링의 위험

`일급 컬렉션` 을 사용할 때 주의해야 할 중요한 점 중 하나는 오버엔지니어링의 위험 입니다. 모든 컬렉션을 일급 컬렉션으로 만드는것이 항상 좋은 것은 아닙니다.

예시를 들어보겠습니다.

```java
public class SimpleDataCollection {
	private final List<Data> dataList;

	public SimpleDataCollection(List<Data> dataList) {
		this.dataList = dataList;
	}
}
```

이 예시에서 `SimpleDataCollection` 은 복잡한 로직 없이 데이터만을 저장합니다. 이런 경우 `일급 컬렉션`을 사용하는 것은 오버엔지니어링이 될 수 있습니다.

이처럼 `일급 컬렉션` 은 컬렉션에 복잡한 비즈니스 로직이나 규칙이 적용될 때 가장 유용합니다. 단순히 데이터를 저장하는 용도라면 굳이 `일급 컬렉션` 을 사용할 필요는 없겠죠.

따라서 `일급 컬렉션` 을 도입하기 전에 해당 컬렉션에 어떤 로직이나 규칙이 필요한지 고려해야 합니다. 컬렉션의 복잡성과 유지보수의 편의성을 균형있게 고려하는 것이 중요합니다.

## 4. 결론

![Untitled](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0116/frog.gif)

`일급 컬렉션` 을 오늘은 함께 알아보았는데요, 우리는 일급 컬렉션이 단순한 데이터의 모음 이상의 가치를 지닌다는 것을 배웠습니다.

이는 객체지향 프로그래밍의 근본적인 원칙인 캡슐화와 응집도를 강화하는데 큰 도움을 줍니다.

비즈니스 로직을 한 곳에 집중함으로써, 우리의 코드는 더욱 명확하고, 유지보수하기 쉬우며, 오류 발생가능성을 줄일 수 있습니다.

하지만 모든 상황에서 `일급 컬렉션` 이 해답은 아닙니다. 우리는 오버엔지니어링의 위험성을 항상 염두하고 일급 컬렉션이 꼭 필요한 상황인지를 항상 고려해야 합니다.

오늘 공유한 내용이 실제 프로젝트에서 여러분이 마주칠 문제를 해결하는데 도움이 되길 바랍니다.

## 5. 참고 문헌

[https://jojoldu.tistory.com/412](https://jojoldu.tistory.com/412)

[https://edu.nextstep.camp/c/9WPRB0ys](https://edu.nextstep.camp/c/9WPRB0ys)

