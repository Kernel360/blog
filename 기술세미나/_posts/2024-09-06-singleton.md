---
layout: post  
title: `싱글톤 패턴`
author: `이선우`
categories: `기술세미나`
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`스프링`,`자바`,`싱글톤`]
---

# 싱글톤 패턴이란
> 싱글톤 패턴은 특정 클래스의 인스턴스를 1개만 생성되는 것을 보장하는 디자인 패턴입니다. 즉, 생성자를 통해서 여러 번 호출이 되더라도 인스턴스를 새로 생성하지 않고 최초 호출 시에 만들어두었던 인스턴스를 재활용하는 패턴입니다.

`싱글톤 패턴을 사용하면 메모리 사용을 최적화하고, 객체 생성 비용을 절감할 수 있습니다.`

```java
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();

        MemberService memberService1 = appConfig.memberService();
        MemberService memberService2 = appConfig.memberService();

        System.out.println("memberService1: " + memberService1);
        System.out.println("memberService2: " + memberService2);

        assertThat(memberService1).isNotSameAs(memberService2);
    }
```
위의 코드는 AppConfig 클래스를 이용해 MemberService 객체를 두 번 호출을 한 후 memberService1과 memberService2의 참조값을 비교하는 테스트 코드입니다. 참조값을 비교해보면 다른 것을 확인할 수 있고, 이는 매번 객체를 호출할 때마다 새로운 인스턴스가 만들어지고, 이는 메모리 낭비로 이어질 수 있습니다.

위의 문제점을 해결하기 위해 싱글톤 패턴을 구현해보고 테스트를 다시 한 번 해보았습니다.

``` java
package springproject.basicspring.singleton;

// 싱글톤 패턴을 구현하는 방법은 여러가지가 있고, 이 코드에서 사용한 방법은 객체를 미리 생성해두는 가장 단순하고 안전한 방법이다.
public class SingleTonService {

    //1. static 영역에 객체를 딱 1개만 생성해둔다.
    private static final SingleTonService instance = new SingleTonService();

    //2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
    public static SingleTonService getInstance() {
        return instance;
    }

    //3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 만든다.
    private SingleTonService() {
    }

    public void login() {
        System.out.println("싱글톤 객체 로직 호출");
    }

}
```

```java
    @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest() {
        SingleTonService singleTonService1 = SingleTonService.getInstance();
        SingleTonService singleTonService2 = SingleTonService.getInstance();

        System.out.println("singleTonService1: " + singleTonService1);
        System.out.println("singleTonService2: " + singleTonService2);

        assertThat(singleTonService1).isSameAs(singleTonService2);
    }
    
  ```
위의 코드들은 싱글톤 패턴을 자바로 구현을 한 코드입니다.
객체의 인스턴스를 static을 사용해 static 영역에 미리 생성해 올려주고, 객체 인스턴스가 필요할 때는 getInstance() 메서드를 통해서만 접근할 수 있게 해 항상 동일한 인스턴스를 반환하게 해줍니다. 그 후 객체의 생성자를 private로 선언하여 외부에서 new 키워드를 사용해 인스턴스를 생성하지 못하도록 해주었습니다.

테스트 코드에서 하나의 인스턴스만 생성하는지 확인해보면 두 번의 getInstance() 호출이 동일한 객체를 반환하는 것을 확인할 수 있습니다. 이처럼 싱글톤 패턴은 객체를 하나만 생성하여 여러 곳에서 공유할 수 있도록 해줍니다.

하지만 이 패턴에도 몇 가지 단점이 있습니다. 우선 getInstance()와 같은 메서들르 직접 구현해야 하며 클라이언트가 구체 클래스에 의존하게 되어 DIP와 OCP를 위반할 수 있습니다. 그로 인해 코드의 유연성이 떨어지고 안티패턴이라고도 불리기도 합니다.
그러나 스프링 프레임워크를 사용하면 이러한 단점들을 해결하고 싱글톤을 사용할 수 있습니다.

스프링 컨테이너는 @Configuration 어노테이션이 붙은 설정 파일을 통해 객체 간의 의존관계를 설정하고 관리합니다. 이를 통해 애플리케이션 내에서 Bean을 싱글톤으로 관리할 수 있습니다. 그로 인해 직접 싱글톤 패턴을 구현할 필요 없이 객체를 효율적으로 관리할 수 있습니다.

```java
package springproject.basicspring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springproject.basicspring.discount.DiscountPolicy;
import springproject.basicspring.discount.FixDiscountPolicy;
import springproject.basicspring.discount.RateDiscountPolicy;
import springproject.basicspring.member.MemberRepository;
import springproject.basicspring.member.MemberService;
import springproject.basicspring.member.MemberServiceImpl;
import springproject.basicspring.member.MemoryMemberRepository;
import springproject.basicspring.order.OrderService;
import springproject.basicspring.order.OrderServiceImpl;

@Configuration
public class AppConfig {

    //@Bean memberService -> new MemoryMemberRepository()
    //@Bean orderService -> new MemoryMemberRepository()
    // -> 싱글톤이 깨지는 거 아닐까??

    //AppConfig.memberService
    //AppConfig.memberRepository
    //AppConfig.memberRepository
    //AppConfig.orderService
    //AppConfig.memberRepository

    //AppConfig.memberService
    //AppConfig.memberRepository
    //AppConfig.orderService

    @Bean
    public MemberService memberService() {
        System.out.println("AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
        System.out.println("AppConfig.memberRepository");

        return new MemoryMemberRepository();
    }

    // 스프링 컨테이너가 @Bean 메서드를 오버라이딩하여 싱글톤을 보장하는 프록시 메커니즘을 사용
    // 아래와 같이 static을 붙이면 static 메서드는 스프링 컨테이너에 의해 오버라이딩되어 관리될 수 없기 때문에
    // 싱글톤으로 관리되지 않는다.
//    @Bean
//    public static MemberRepository memberRepository() {
//        return new MemoryMemberRepository();
//    }

    @Bean
    public OrderService orderService() {
        System.out.println("AppConfig.orderService");

        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    @Bean
    public DiscountPolicy discountPolicy() {
//        return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }


}
```
위의 AppConfig 클래스 또한 스프링 설정 파일로 각 서비스 객체를 빈으로 등록해 사용합니다. 테스트 코드를 통해 MemberService 객체를 두번 호출하고 그 참조값을 비교해보면

```java
    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        // 참조값이 같은 것을 확인
        System.out.println("memberService1: " + memberService1);
        System.out.println("memberService2: " + memberService2);

        // memberService1 == memberService2
        assertThat(memberService1).isSameAs(memberService2);
    }
```
예상했던 바와 같이 두 객체의 참조값이 동일함을 확인할 수 있습니다. 이를 통해 스프링이 기본적으로 빈을 싱글톤으로 관리한다는 것을 알 수 있습니다.

스프링에서 싱글톤 패턴을 사용할 때는 주의할 점이 있습니다. 싱글톤 빈은 여러 클라이언트가 동일한 인스턴스를 공유하기 때문에 상태를 가지지 않도록 해야 합니다.

```java
package springproject.basicspring.singleton;

public class StatefulService {

    // statefulService
    private int price;

    public void order(String name, int price) {
        System.out.println("name = " + name + ", price = " + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

    // 무상태로 코드 짜는 법

//    public int order(String name, int price) {
//        System.out.println("name = " + name + ", price = " + price);
//        return price;
//
//    }



}
```

```java
package springproject.basicspring.singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

import static org.junit.jupiter.api.Assertions.*;

class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(StatefulService.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

//        int userAPrice = statefulService1.order("userA",10000);
//        int userBPrice = statefulService2.order("userB",20000);

        // ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);
        // ThreadB: B사용자 20000원 주문
        statefulService2.order("userB", 20000);

        // ThreadA: 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);
//        System.out.println("price = " + userAPrice);


        Assertions.assertThat(price).isEqualTo(20000);

    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }

}
```
StatefulService 클래스는 상태를 유지하는 필드 price를 가지고 있습니다. 이 상태를 가지는 필드 때문에 특정 클라이언트가 값을 변경하면 다른 클라이언트들에게도 영향을 미치게 됩니다. 테스트코드를 보시면 두 사용자가 각각 주문을 했을 때, 사용자 A의 주문 금액이 10000원이 아니라 20000원이 출력되는 것을 확인할 수 있습니다. 이는 상태를 가지는 싱글톤 빈의 문제점을 잘 보여주고 그로 인해 스프링 빈은 항상 상태를 가지지 않게 설계해야 합니다.

추가로 스프링 프레임워크를 사용해서 만든 모든 빈이 싱글톤으로 관리되는 것은 아닙니다.스프링 프레임워크에서의 빈의 기본 스코프는 싱글톤이지만, Scope 어노테이션을 사용하면 필요에 따라 빈의 스코프를 설정할 수 있습니다.
그리고 싱글톤 객체의 크기가 클 때에는 객체를 스코프 설정을 통해 싱글톤을 관리하지 않는게 더 메모리에 효율적일 수도 있습니다. 그 이유는 싱글톤 객체가 너무 많은 메모리를 차지하는 경우, 이 객체가 애플리케이션이 종료될 때까지 메모리를 점유하기 때문에 프로그램이 더 이상 필요하지 않은 메모리를 해제하지 않고 계속해서 점유하고 있는 메모리 릭이 발생할 수 있기 때문입니다.
