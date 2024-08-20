---
layout: post  
title: 인터페이스를 활용하여 조건분기 개선하기
author: 이강민
categories: 기술세미나
banner: 
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [인터페이스를 활용하여 조건분기 개선하기, 기술세미나]
---




안녕하세요, Kernel360 크루 이강민 입니다. 오늘 포스팅에서는 인터페이스를 활용하여 조건분기를 개선하는 방법에 대해 이야기해보려고 합니다.

## 조건분기
조건분기는 아마도 우리가 코드를 작성하면서 가장 마주치는 친구들이죠.
가장 많이 사용하면서 가장 많이 실수하는 부분이기도 합니다. 너무 쉽지만 정말 잘 사용하고 있다라고 확신 할 수 있나요?


## 자주하는 실수
### if문의 무한 중첩
가령 우리는 여러 요구사항을 맞이 할 때 복잡한 분기를 맞이하고는 합니다. 
이러한 요구사항을 가장 쉽게 풀어 갈 수 있는 방법이 조건 분기인데요. 아마도 개발을 처음 시작하는 사람들은 아래와 같이 코드를 작성해본 경험이 있을 겁니다.
```java
if(조건식 || 조건식){
   //비즈니스 로직,,
   if(조건식  && 조건식){
   //비즈니스 로직,,
   
   }else if(조건식){
   //비즈니스 로직,,
   
   }else{
   //비즈니스 로직,,
   
   }
}
```

위와 같은 코드의 문제가  무엇일까요?
저렇게 처리한다면 리턴하는 값이 정확이 무엇인지? 그리고 조건에서 말하고자하는 것이 무엇인지? 알기 어렵습니다.
심지어 저런 코드에 확장이라도 하면 분기는 끊임없이 길어지거나 혹은 원하지 않은 다른 값이 나오기도 하겠지요.

### 개선하기 
가장 쉽게 리팩토링하는 방법은 조기리턴을 하는 겁니다. 

```java
  if(조건식){
  return;
  }
  if(조건식){
  return;
  }
  if(조건식){
  return;
  }
```
### 어디까지 있니, switch case
다음 코드를 보실까요?
```java
switch(type) {
    case type1 : 
            //로직….
    case type 2 : 
            //로직…
    case type 3: 
            //로직….
    case type 4 : 
            //로직…
}

```

무엇이 문제일까요?
코드만 봐서는 무엇이 문제인지 모릅니다. 사실 switch 문은 제가 코딩테스트 하면서 자주 쓰는 문법이기도 하고 사용하다보며는 조건식을 나눌 때 굉장히 편리합니다.
조건을 넘겨주면 사용되는 분기마다 실행되는 함수를 작성해주면 굉장히 파워풀한 로직이 완성되겠죠 .
그럼 무엇이 문제라는 걸까요?

당연히 길어지면 문제가 됩니다. 
![gif](https://github.com/user-attachments/assets/c41b2a14-05c1-4e6a-9595-ddd7cc109244)

#### Greawt Power Comes Great Responsibility
잘 사용하면 좋지만 잘 사용하지 못한다면 제품 자체가 망가질 수 있습니다. 
![다운로드](https://github.com/user-attachments/assets/df33a4df-ab60-4dfb-8249-b2cccba3a075)

그럼 어떻게 잘 다룰 수 있을까요? 

그건 바로 **인터페이스**입니다. 

## 인터페이스
먼저 인터페이스 사용의 교과서를 보실까요?
```java
public interface Shape {
	double area();
}


public class RectangleShape implements Shape{
	private double width;
	private double height;

	RectangleShape(double width, double height) {
		this.width = width;
		this.height = height;
	}

	@Override
	public double area() {
		return width * height;
	}
}

public class CircleShape implements Shape{
	private final double radius;

	public CircleShape(double radius) {
		this.radius = radius;
	}

	@Override
	public double area() {
		return radius * radius * Math.PI;
	}
}
```
인터페이스 Shape로 공통의 메소드를 통해 추상화를 해주고 각각의 모양 클래스들이 상속받아 필요한 메서드를 정의하고 있는 모습입니다.
이렇게 하면 장점이 인터페이스라는 공통인 타입이 존재하기 때문에 같이 모아서 사용할 수 있지요
*바로 여기서* 아이디어를 떠올라 같은 타입으로 모을 수 있다면 같은 자료구조에 담아 클라이언트의 요청에따라 특정 클래스를 반환할 수 있다는 말이 됩니다!

### 인터페이스로 조건 분기 개선하기 
```java

		switch (magicType) {
			case FIRE :
				name = "퐈이어";
				costMagicPoint = 2;
				attackPower = 20 + (int)(member.level * 0.5);
				costTechnicalPoint = 0;
				break;
			case LIGHTING:
				name = "라이트닝";
				costMagicPoint = 5 + (int)(member.level * 0.2);
				attackPower = 50 + (int)(member.agility * 1.5);
				costTechnicalPoint = 0;
				break;
			case HELLFIRE:
				name = "헬파이어";
				costMagicPoint = 10;
				attackPower = 200 + (int)(member.magicAttack * 0.5 + member.vitality * 2);
				costTechnicalPoint = 20 + (int)(member.level * 0.4);
				break;

			default:
				throw new IllegalAccessException();
		}
```
위 코드는 특정 마법을 요청하면 해당 마법의 특징들을 반환하는 조건분기입니다. 
만약 마법이 추가되거나 캐릭터마다 마법이 10개가 된다면 5개 캐릭터만 해도 50개이지요 
그럼 case 는 50가지나 됩니다. 후...

#### 인터페이스 정의하기 
인터페이스로 공통의 타입을 정의해볼까요?
```java

public interface Magic {
	String name();
	MagicPoint costMagicPoint();
	AttackPower attackPower();
	TechnicalPoint costTechnicalPoint();
}
```
case마다 작성했던 마법 관련 메소드들을 인터페이스에 정의한 모습입니다. 
앞으로 이 인터페이스를 구현체로 받은 클래스들은 저 메서드들을 무조건 정의해야되죠. 즉, 실수를 줄이는 겁니다. 

#### 마법 클래스들 구현체 정의하기
##### 불 마법 
```java
public class Fire implements Magic{
	private final Member member;

	public Fire(Member member) {
		this.member = member;
	}

	@Override
	public String name() {
		return "파이어";
	}

	@Override
	public MagicPoint costMagicPoint() {
		return new MagicPoint(2);
	}

	@Override
	public AttackPower attackPower() {
		final int value = 20 + (int)(member.level * 0.5);
		return new AttackPower(value);
	}

	@Override
	public TechnicalPoint costTechnicalPoint() {
		return new TechnicalPoint(0);
	}
}
```

##### 전기마법

```java
public class Lightning implements Magic{
	private final Member member;

	public Lightning(Member member) {
		this.member = member;
	}

	@Override
	public String name() {
		return "라이트닝";
	}

	@Override
	public MagicPoint costMagicPoint() {
		final int value = 5 + (int)(member.level * 0.2);
		return  new MagicPoint(value);
	}

	@Override
	public AttackPower attackPower() {
			final int value = 50 + (int)(member.agility * 1.5);
		return new AttackPower(value);
	}

	@Override
	public TechnicalPoint costTechnicalPoint() {
		return new TechnicalPoint(5);
	}
}
```

이렇게 2가지만 정의해보겠습니다. 앞으로 마법이 늘어나면 위와 같이 상속받아서 작성하면 됩니다. 
아, 해당 마법을 하드코딩하는 대신 enum으로 정의해볼까요?
```java

public enum MagicType {
	FIRE,
	LIGHTING,
}
```
이제 마법을 등록시켜볼까요?
```java
public class MagicManager {
	final Map<MagicType, Magic> magics = new HashMap<>();

	public MagicManager(Member member) {
		Fire fire = new Fire(member);
		Lightning lightning = new Lightning(member);
		HellFire hellFire = new HellFire(member);
		this.magics.put(MagicType.FIRE, fire);
		this.magics.put(MagicType.LIGHTING, lightning);
		this.magics.put(MagicType.HELLFIRE, hellFire);
	}

	void magicAttack(final MagicType magicType){
		final Magic magic = magics.get(magicType);
		showMagicName(magic);
		consumeMagicPoint(magic);
		consumeTechnicalPoints(magic);
		magicDamage(magic);
	}

	void showMagicName(final Magic magic){
		final String name = magic.name();
		System.out.println(name);
	}

	void consumeMagicPoint(final Magic magic){
		final MagicPoint costMagicPoints = magic.costMagicPoint();
		System.out.println(costMagicPoints.getMagicPoint() + "만큼 매직포인트 소모하였습니다.");
	}

	void consumeTechnicalPoints(final Magic magic){
		final TechnicalPoint costTechnicalPoints = magic.costTechnicalPoint();
		System.out.println(costTechnicalPoints.getTechnicalPoint() + "만큼 테크니컬 포인트를 소모하였습니다.");
	}

	void magicDamage(final Magic magic){
		final AttackPower attackPower = magic.attackPower();
		System.out.println(attackPower.getAttackPower()+"만큼 데미지!");
	}

}

```

각각의 반환하는 값들을 간단히 출력하고 있는 모습입니다. 
공통의 타입이 생겼기 때문에 위와 같이 추상화되어 사용할 수 가 있는 거지요 
그리고 공통의 타입으로 묶인 클래스들을 HashMap으로 작성한 모습입니다.

조건 분기로 나누어 정의되었던 메서드들이 분리되고 추상화된 모습을 확인 할 수 있습니다.
사용부분을 볼까요?
```java

public class Main {
	public static void main(String[] args) throws IllegalAccessException {
		Member member = new Member(10,20, 20,20);
		MagicManager magicManager = new MagicManager(member);

		magicManager.magicAttack(MagicType.FIRE);
		System.out.println("-----------------");
		magicManager.magicAttack(MagicType.LIGHTING);

	}
}

```

간단하게 사용되고 있는 모습입니다. 
조건분기가 없어졌죠?

이렇게 작성하면 좋은 점은 실수를 줄이고 코드의 가독성이 올라 생산성이 증가된다는 겁니다. 

## 정리
모든 조건 분기가 나쁘고 없애야하는 것은 아닙니다. 앞서 말씀드렸듯이 잘 사용하면 좋지만 우리 같은 주니어레벨은 쉽지 않죠.
그래서 위와 같은 전략패턴을 적용하여 최대한의 휴먼에러를 낮추는 것. 이것이 핵심입니다. 
결론은 조건분기는 인터페이스를 이용해 개선할 수도 있고 이를 위해서는 분기와 로직을 분리하고 한 곳에 모아두어 코드의 응집도를 높이는 관심사 분리가 뒷받침 되어야 한다는 말을 끝으로 마치겠습니다.


