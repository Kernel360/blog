---
layout: post
title: '커스텀 예외 처리'
author: "박소은"
categories: "기술세미나"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [예외, 에러, 자바]
---


# 📗 시작하며


이 글은 **커스텀 예외 처리**를 어떻게 할지 고민하며 점진적으로 개선해나가는 과정을 담고 있습니다. **처음 커스텀 예외 클래스를 작성**하시는 분부터, **더 좋은 예외 처리**를 위해 고민하고 계시는 분들 모두에게 도움이 되는 글을 작성하고자 했습니다!


## 🙋‍♂️커스텀 예외를 왜 만들어야 하나요?

로또 번호는 `1`~`45` 사이의 숫자입니다. `validateLottoNumber()` 메서드는 이를 확인하고 범위를 벗어난 숫자에 대해 `IllegalArgumentException`을 던집니다.

```java
public class LottoNumber implements Comparable<LottoNumber> {

    private final int lottoNumber;

    public LottoNumber(int lottoNumber) {
        validateLottoNumber(lottoNumber);
        this.lottoNumber = lottoNumber;
    }

    private void validateLottoNumber(int lottoNumber) {
        if(lottoNumber < 1 || lottoNumber > 45) {
            throw new IllegalArgumentException(lottoNumber);
        }
    }
}
```
메서드 파라미터에 잘못된 값이 입력되면 아래와 같이 에러가 발생합니다.

```
Exception in thread "main" java.lang.IllegalArgumentException
	at lotto.domain.lotto.LottoNumber.validateLottoNumber(LottoNumber.java:18)
	at lotto.domain.lotto.LottoNumber.<init>(LottoNumber.java:12)
    /* 생략 */
```

> ❓🙋‍이러한 에러 처리 방식에는 어떠한 **문제점**들이 있을까요?

1. 먼저, **예외의 원인을 분명하게 파악하기 어렵습니다. **
   에러 메시지를 통해 validateLottoNumber 메서드에 문제가 발생했다는 것은 알 수 있습니다. 하지만 정확히 해당 메서드에 무슨 문제가 발생했는지 알 수 없습니다.

2. 또한, **어떤 값이 문제를 일으켰는지 알기 어렵습니다.** 개발자는 로그를 통해 에러의 상세 내용을 확인할 수 있어야 합니다. 사용자가 잘못된 값을 입력했다면, 이를 로그로 남겨 개발자가 확인할 수 있어야 합니다.

단순히 `IllegalArgumentException`을 던지고, 아무런 메시지도 남기지 않는다면 예외가 발생할 때마다 디버깅을 반복적으로 해야겠죠!

위와 같은 이유로, **커스텀 예외**를 만들고 **예외가 발생한 원인을 구체적**으로 남기는 것을 선호합니다.

<br>

## 🌱 어떻게 만드나요?

아주 간단합니다! 이번 3주차 미션의 요구사항에는 IllegalArgumentException을 던지는 것으로 되어 있었습니다. 그래서 CustomException 클래스를 만들고, IllegalArgumentException을 상속받도록 구현하면 됩니다.

```java
public class InvalidLottoNumberException extends IllegalArgumentException{

    private static final String ERROR_MESSAGE = "[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.";
    
    public InvalidLottoNumberException() {
        super(ERROR_MESSAGE);
    }
}
```

이제 잘못된 값이 전달되면 아래와 같이 직접 정의한 메시지를 함께 확인할 수 있습니다.

```
Exception in thread "main" lotto.common.exception.InvalidLottoNumberException: [ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.
	at lotto.domain.lotto.LottoNumber.validateLottoNumber(LottoNumber.java:18)
	at lotto.domain.lotto.LottoNumber.<init>(LottoNumber.java:12)
	at
    /* 생략 */
```

> ❓🙋‍ super()는 왜 하는거에요?

`super(ERROR_MESSAGE)`를 통해 부모 생성자가 호출됩니다. IllegalArgumentException를 잠깐 살펴보면 아래와 같은 생성자가 있습니다.

```java
public IllegalArgumentException(String s) {
	super(s);
}
```

여기서도 부모 생성자를 호출하는데요, 이를 타고 들어가다 보면 최상위 Throwable이 나옵니다.


```java
public Throwable(String message) {
    fillInStackTrace();
	detailMessage = message;
}
```   

`fillInStackTrace()`는 예외가 발생한 시점의 스택 트레이스를 캡처합니다. 이를 통해 예외가 발생한 위치와 호출 경로를 추적할 수 있습니다.

`detailMessage`는 `getMessage()` 메서드를 통해 외부에서 조회할 수 있습니다. 따라서, InvalidLottoNumberException에서 전달한 메시지는 최상위 Throwable 클래스의 `detailMessage` 필드에 저장됩니다.


<br>


# 🐉 커스텀 예외 잘 만들기

이제부터 앞서 만든 커스텀 예외를 하나씩 고쳐보도록 하겠습니다!


## 1️⃣ 동적 메시지를 제공한다.

```
"[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다."
```
해당 에러 메시지로는 **어떤 로또 번호가 예외를 발생시켰는지 알기 어렵**습니다.
`-1`이었을 수도 있고, `Integer.MAX_VALUE` 정말 큰 수였을 수도 있죠.

개발자는 이슈가 생겼을 때 **로그로 문제를 쉽게 파악**할 수 있어야 합니다. **어떤 파라미터가 문제를 일으킨건지 빠르게 확인**할 수 있어야 하는데요.

이때, **동적 메시지를 활용**할 수 있습니다. 즉, **예외를 발생시킨 값**을 메시지에 함께 제공하면 예외를 발생시킨 상황에 대한 파악을 더 빠르게 할 수 있습니다.

```java
public class InvalidLottoNumberException extends IllegalArgumentException{

private static final String ERROR_MESSAGE = "[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.";

    public InvalidLottoNumberException(int invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber + ")");
    }
}
```
커스텀 예외 클래스의 생성자에서 `int` 타입의 `invalidLottoNumber`를 전달받고 있습니다. 예외를 던지는 쪽에서는 아래와 같이 값을 전달합니다.

```java
private void validateLottoNumber(int lottoNumber) {
	if(lottoNumber < 1 || lottoNumber > 45) {
    	throw new InvalidLottoNumberException(lottoNumber);
	}
}
```

**예외를 발생시킨 값**을 에러 메시지를 통해 확인하고, **구체적인 상황을 파악**하기가 쉬워졌습니다!

```
[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다. (잘못된 로또 번호 : 90)
```

<br>

## 2️⃣ 예외 체이닝을 걸어준다.


예외 클래스를 만들땐 **꼭 기존 예외랑 체이닝**을 시켜줘야 합니다.

```java
public LottoNumber readBonusNumber() {
	int bonusNumber;
    String input = Console.readLine();
    try {
    	bonusNumber = Integer.parseInt(input);
    } catch (NumberFormatException e) {
    	throw new InvalidLottoNumberException(input);
    }
    return new LottoNumber(bonusNumber);
}
```
보너스 번호를 사용자에게 입력받고, 숫자가 아니라면 InvalidLottoNumberException을 던지고 있습니다. `String` 변수를 `int` 타입으로 변환하는 과정에서 숫자가 아니라면 NumberFormatException이 발생합니다. 이를 `try-catch`로 잡아 커스텀 예외를 다시 던지고 있습니다.

이제 잘못된 보너스 번호를 입력해보겠습니다.

```
Exception in thread "main" lotto.common.exception.InvalidLottoNumberException: [ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다. (잘못된 로또 번호 : invalid)
	at lotto.interfaces.input.InputHandler.readBonusNumber(InputHandler.java:53)
	at lotto.interfaces.lotto.LottoController.getWinningLotto(LottoController.java:62)
	at lotto.interfaces.lotto.LottoController.lottoGameStart(LottoController.java:33)
	at lotto.Application.main(Application.java:10)
```


보시다시피 **스택 트레이스에 NumberFormatException에 대한 정보가 어디에도 없습니다.** 원천 예외가 있을 때는, 이를 cause에 담아야 스택 트레이스에 남습니다. 최상위 Throwable에서는 아래와 같이 cause를 받고 있습니다.

```java
public Throwable(String message, Throwable cause) {
    fillInStackTrace();
    detailMessage = message;
	this.cause = cause;
}
```
여기에 **cause를 전달**하기 위해 커스텀 예외 클래스를 아래와 같이 수정할 수 있습니다.

```java
public class InvalidLottoNumberException extends IllegalArgumentException{

    private static final String ERROR_MESSAGE = "[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.";
    
    /* 생략 */
    
    // 1번: 원천 예외가 없을 때
    public InvalidLottoNumberException(String invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")");
    }

	// 2번: 원천 예외가 있을 때
    public InvalidLottoNumberException(String invalidLottoNumber, Exception e) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")", e);
    }
}
```
원천 예외를 받아, 이를 부모 생성자로 넘기고 있습니다.

**원천 예외가 없을 때**는 위쪽에 있는 생성자가 호출되고, **NumberFormatException와 같이 원천 예외가 있을 때**는 아래쪽에 있는 생성자가 호출됩니다.

따라서 커스텀 예외 클래스를 만들 때 위와 같이 **두 가지 생성자를 모두 만들어야** 합니다.

예외를 던지는 쪽에서는  아래와 같이 Exception을 함께 전달해주면 됩니다.


```java
public LottoNumber readBonusNumber() {
	int bonusNumber;
    String input = Console.readLine();
    try {
    	bonusNumber = Integer.parseInt(input);
    } catch (NumberFormatException e) {
    	throw new InvalidLottoNumberException(input, e);
    }
    return new LottoNumber(bonusNumber);
}
```

이제 잘못된 보너스 번호 입력 시 아래와 같이 NumberFormatException도 함께 확인할 수 있습니다.


```
Exception in thread "main" lotto.common.exception.InvalidLottoNumberException: [ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.(잘못된 로또 번호 : invalid)
	at lotto.interfaces.input.InputHandler.readBonusNumber(InputHandler.java:53)
	at lotto.interfaces.lotto.LottoController.getWinningLotto(LottoController.java:62)
	at lotto.interfaces.lotto.LottoController.lottoGameStart(LottoController.java:33)
	at lotto.Application.main(Application.java:10)
Caused by: java.lang.NumberFormatException: For input string: "invalid"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:662)
	at java.base/java.lang.Integer.parseInt(Integer.java:778)
	at lotto.interfaces.input.InputHandler.readBonusNumber(InputHandler.java:51)
	... 3 more
```
**예외를 체이닝**하면 **예외의 원래 원인에 대한 자세한 정보를 제공**할 수 있으며,
**오류 발생 지점을 더 명확하게 파악**하는데 필수적입니다.


<br>

## 3️⃣ 정의한 예외 vs 예기치 못한 예외: 구분의 중요성


개발을 하다보면, 종종 예외 상황을 마주하게 됩니다. 이때 커스텀 예외를 하나씩 만들며 **예상치 못한 예외**들을 줄여나가는데요! **이 둘을 구분**하기 위해서 **커스텀 예외가 상속받는 공통적인 예외 클래스**를 추가적으로 하나 더 선언합니다.

바로 코드로 확인해보겠습니다!


```java
public class LottoException extends IllegalArgumentException {

    private static final String ERROR_MESSAGE_HEADER = "[ERROR] ";
    private final String errorMessage;

    public LottoException(String message) {
        super(ERROR_MESSAGE_HEADER + message);
        this.errorMessage = ERROR_MESSAGE_HEADER + message;
    }

    public LottoException(String message, Exception e) {
        super(ERROR_MESSAGE_HEADER + message, e);
        this.errorMessage = ERROR_MESSAGE_HEADER + message;
    }
}
```

요구사항에 맞춰 IllegalArgumentException을 상속받은 LottoException 클래스를 하나 만들었습니다. 이는 앞으로 정의할 **모든 커스텀 예외 클래스들이 상속받을 중추적인 예외 클래스**인데요!

각 커스텀 예외에서 공통적으로 필요한 errorMessage를 갖고 있습니다. 또, `[ERROR]`를 메시지에 추가해주는 작업도 이쪽에서 수행합니다.

그럼, 다시 InvalidLottoNumberException을 수정해보겠습니다.

```java
public class InvalidLottoNumberException extends LottoException{

    private static final String ERROR_MESSAGE = "로또 번호는 1부터 45 사이의 숫자여야 합니다.";
    
    public InvalidLottoNumberException(int invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")");
    }

    public InvalidLottoNumberException(int invalidLottoNumber, Exception e) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")", e);
    }

    public InvalidLottoNumberException(String invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")");
    }

    public InvalidLottoNumberException(String invalidLottoNumber, Exception e) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")", e);
    }
}
```
커스텀 예외 클래스가 상속받는 부모 클래스가 IllegalArgumentException에서 LottoException으로 변경되었습니다!

이렇게 했을 때의 장점은, **개발자가 정의한 예외와 그렇지 못한 예외를 구분할 수 있다**는 점인데요!

예외를 처리하는 곳을 한 번 살펴보겠습니다.

```java
private LottoMoney getLottoMoney() {
	while (true) {
    	try {
        	return inputHandler.readPurchaseAmount();
        } catch (LottoException e) {
        	System.out.println(e.getMessage());
    	} catch (Exception e) {
        	System.out.println("예상치 못한 예외가 발생했습니다.");
	}
}
```  

먼저 try-catch 블록에서 LottoException을 잡아 이를 처리합니다. 만약 LottoException이 아닌 다른 예외가 발생할 경우에는 하위 catch 블록이 실행되어 별도의 메시지를 출력하도록 했습니다.


> ❓🙋‍♂️ 이렇게 했을 때 **장점**이 무엇인가요?


### 😊 **개발자가 정의된 예외와 예상치 못한 예외**를 **분리하여 처리**할 수 있다.

```java
private LottoMoney getLottoMoney() {
	while (true) {
    	try {
        	return inputHandler.readPurchaseAmount();
        } catch (IllegalArgumentException e) {
        	System.out.println(e.getMessage());
    	} 
	}
}
```

위의 코드에서는 **개발자가 예상할 수 있는 예외**(커스텀 예외)와 **예상하지 못한 예외**가 **한곳에서 함께 처리**되고 있습니다.

개발자가 미리 정의한 커스텀 예외는 특정 에러 메시지를 출력하고, 사용자로부터 다시 입력을 받도록 설계되어 있습니다. 그러나 **예상치 못한 예외에 대해서는 다른 방식의 처리가 필요**할 수 있습니다. 예를 들어, 에러 메시지를 띄운 후 시스템을 종료하거나, 별도의 에러 처리 로직을 실행할 수도 있습니다.

이럴 때 중추적인 예외를 만들어 처리하게 만들면, **개발자가 정의된 예외와 예상치 못한 예외**를 **분리하여 처리**할 수 있습니다.

### 😊 예상치 못한 예외를 빠르게 파악하고, 그 수를 줄일 수 있다.

시스템에서 발생할 수 있는 예외를 미리 예측하고 대처하기 위해 **예상치 못한 예외를 줄이고, 커스텀 예외를 정의**하게 되는데요,

이때 try-catch에서 잡히지 못하는 예외는 모두 예상치 못한 예외로 간주되어, **미처 파악하지 못했던 예외 상황을 더 빠르게 구체화**하는 데 도움을 줍니다.



이러한 이유로, LottoException 처럼 중추적인 예외를 하나 선언하고, 각 커스텀 예외 클래스가 이를 상속받도록 합니다. 예외를 처리하는 쪽에서 이 둘을 구분하여 핸들링해주면 얻을 수 있는 이점을 말씀드렸습니다.





# 🌳 마치며

## Before & After

### 😈 Before

```
Exception in thread "main" java.lang.IllegalArgumentException
	at lotto.domain.lotto.LottoNumber.validateLottoNumber(LottoNumber.java:18)
	at lotto.domain.lotto.LottoNumber.<init>(LottoNumber.java:12)
    /* 생략 */
```


```java
private void validateLottoNumber(int lottoNumber) {
    if(lottoNumber < 1 || lottoNumber > 45) {
    	throw new IllegalArgumentException(lottoNumber);
	}
}
```

### 😁 After

```
Exception in thread "main" lotto.common.exception.InvalidLottoNumberException: [ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다. (잘못된 로또 번호 : invalid)
	at lotto.interfaces.input.InputHandler.readBonusNumber(InputHandler.java:53)
	at lotto.interfaces.lotto.LottoController.getWinningLotto(LottoController.java:62)
	at lotto.interfaces.lotto.LottoController.lottoGameStart(LottoController.java:33)
	at lotto.Application.main(Application.java:10)
Caused by: java.lang.NumberFormatException: For input string: "invalid"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:662)
	at java.base/java.lang.Integer.parseInt(Integer.java:778)
	at lotto.interfaces.input.InputHandler.readBonusNumber(InputHandler.java:51)
	... 3 more
```

```java
public class InvalidLottoNumberException extends LottoException{

    private static final String ERROR_MESSAGE = "[ERROR] 로또 번호는 1부터 45 사이의 숫자여야 합니다.";
    
    public InvalidLottoNumberException(int invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")");
    }

    public InvalidLottoNumberException(int invalidLottoNumber, Exception e) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")", e);
    }

    public InvalidLottoNumberException(String invalidLottoNumber) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")");
    }

    public InvalidLottoNumberException(String invalidLottoNumber, Exception e) {
        super(ERROR_MESSAGE + "(잘못된 로또 번호 : " + invalidLottoNumber+ ")", e);
    }
}
```
>🌙 **동적 메시지**를 제공한다. 
> <br>
> 🌙 **예외 체이닝**을 걸어준다.
> <br>
> 🌙 **정의한 예외**와 **예상치 못한 예외**를 **구분**한다.


이 세 가지를 적용해 커스텀 예외 클래스를 만들며 예외 처리 로직을 개선해보았습니다! **이슈가 생겼을 때 문제를 파악**하기 쉽게 하려면, **어떤 파라미터가 문제**를 일으킨 건지, **원천 예외**가 있다면 무엇인지를 메시지에 잘 적어주는 것이 필요하다고 생각합니다.

<br>

> 참고자료

예외 체이닝 관련
https://www.baeldung.com/java-chained-exceptions
