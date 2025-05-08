---
layout: post  
title: "스프링 부트에서 대용량 엑셀 업로드 검증 전략: 서비스 레이어 vs Argument Resolver"
author: "이영석"
categories: "기술블로그"
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2025/0425/BeanValidation.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`유효성 검증`, `Validation`, `Excel`, `Argument Resolver`, `@Valid`]
---

# 스프링 부트에서 대용량 엑셀 업로드 검증 전략: 서비스 레이어 vs Argument Resolver

## 유효성 검증의 중요성과 문제점

이번 프로젝트를 진행하던 중 다수의 고객 정보를 하나의 엑셀파일로 업로드해 고객을 생성하는 API를
개발하게 되었는데, 고객은 전화번호나 이메일 같은 Validation이 필요한 부분이 있어 유효성 검증을 어떻게 할지에 대해 고민을 하게 되었습니다. 그래서 오늘은 스프링 부트에서 엑셀 파일 업로드 시 발생하는 유효성 검증 문제와 해결 방법에 대해 이야기해보려고 합니다.

**유효성 검증**은 애플리케이션의 비즈니스 로직이 올바르게 동작하기 위해 데이터를 사전에 검증하는 필수적인 작업입니다. 그런데 전통적인 계층별 검증 방식은 몇 가지 문제를 가지고 있습니다:

1. **검증 로직의 분산**: 각 계층(Controller, Service, Domain)마다 검증 로직이 흩어져 있어 관리하기가 어렵습니다.
2. **코드 중복**: 같은 검증 로직을 여러 계층에서 반복해서 구현해야 하는 번거로움이 있습니다.
3. **유지보수 어려움**: 검증 규칙이 조금만 변경되어도 여러 계층을 모두 수정해야 하는 불편함이 있습니다.

## Bean Validation의 등장

이런 문제를 해결하기 위해 Java 진영에서는 **Bean Validation**이라는 표준 프레임워크를 제공합니다. 스프링 부트는 **Hibernate Validator**를 기본 구현체로 채택하여 도메인 객체 중심의 선언적 검증을 지원하고 있죠.  
일반적으로 가장 많이 사용하시는 Validation 방법일 거라고 생각합니다!

**Bean Validation 동작 방식**:

- **검증 프로세스**:
  - `@Valid` 어노테이션이 적용된 객체가 컨트롤러에 전달되면
  - Validator가 객체의 필드에 적용된 검증 어노테이션을 확인
  - 각 필드의 값이 검증 규칙을 만족하는지 확인
  - 검증 실패 시 `ConstraintViolationException` 발생

```java
public class CustomerDto {
    @NotBlank(message = "이름은 필수 항목")
    @Size(max = 50)
    private String name;

    @Pattern(regexp = "^01\\d-\\d{3,4}-\\d{4}$")
    private String contact;
}
```

**Bean Validation의 장점**:

1. **선언적 검증**: 어노테이션만으로 검증 규칙을 명시적으로 선언할 수 있어서 편리합니다.
2. **코드 재사용**: 검증 로직을 도메인 모델에 정의하면 여러 계층에서 재사용할 수 있습니다.
3. **표준화**: JSR-380 표준을 따르므로 다른 Java 애플리케이션과의 호환성이 보장됩니다.
4. **확장성**: 커스텀 검증 어노테이션을 만들어 복잡한 검증 규칙도 구현할 수 있습니다.

**Bean Validation의 단점**:

1. **런타임 검증**: 컴파일 타임이 아닌 런타임에 검증이 이루어져 오류 발견이 늦을 수 있습니다.
2. **복잡한 검증 제한**: 복잡한 비즈니스 로직이나 여러 필드 간의 관계 검증은 어려울 수 있습니다.
3. **성능 오버헤드**: 대량의 데이터를 검증할 때 성능 저하가 발생할 수 있습니다.
4. **커스텀 메시지 관리**: 검증 실패 메시지를 관리하기 위한 별도의 리소스 번들 관리가 필요할 수 있습니다.

## 엑셀 업로드 검증의 특수성

![엑셀 검증이 어려운 이유](https://github.com/Kernel360/blog-image/blob/main/2025/0425/BeanValidation.png?raw=true)

그런데 엑셀 파일은 `MultipartFile`을 통해 바이너리 데이터를 수신하므로, 컨트롤러 계층에서 도메인 DTO로의 자동 변환이 불가능합니다. 이로 인해 발생하는 문제점이 있습니다:

1. **Bean Validation 적용 불가**: `MultipartFile`은 도메인 DTO가 아니므로 `@Valid` 어노테이션을 사용할 수 없습니다.
2. **검증 로직 중복**: 서비스 레이어에서 수동으로 검증 로직을 구현해야 합니다.

## 해결 전략

### 1. 서비스 레이어 검증 구현

```java
@Service
public class ExcelService {
    public List processExcel(MultipartFile file) {
        List dtos = parseExcel(file);
        dtos.forEach(dto -> {
            if (dto.getName() == null) {
                throw new InvalidDataException("Name required");
            }
        });
        return dtos;
    }
}
```

첫번째 방법으로는 서비스 Layer에서 파일 파싱, 유효성 검증을 처리하는 방법이 있습니다.  
이 방법은 유효성 처리를 직관적으로 수행할 수 있게 해 빠르게 구현이 가능하고, 서비스별로 특화된 검증로직을 구현하기가 편하다는 장점이 있습니다.  
하지만 이미 DTO에 같은 검증 로직이 존재하기 때문에 중복된 코드가 발생한다는 단점 역시 존재합니다.

**장점**:

- 빠른 구현 가능
- 엑셀 특화 검증 로직 추가 용이

**단점**:

- 도메인 검증 로직과 중복되므로 Bean Validation의 장점이 사라짐
- 유지보수 비용 증가

### 2. 커스텀 Argument Resolver

**Argument Resolver란?**

- Spring MVC의 핸들러 메서드 파라미터를 처리하는 컴포넌트입니다.
- 컨트롤러 메서드가 호출되기 전에 요청 데이터를 원하는 형태로 변환해줍니다.
- `HandlerMethodArgumentResolver` 인터페이스를 구현하여 커스텀 로직을 추가할 수 있습니다.

**동작 방식**:

1. **지원 여부 확인**

   ```java
   @Override
   public boolean supportsParameter(MethodParameter parameter) {
       return parameter.hasParameterAnnotation(ExcelValid.class);
   }
   ```

   - 파라미터에 특정 어노테이션이 있는지 확인합니다.
   - 지원하는 파라미터 타입인지 검사합니다.

2. **데이터 변환 및 검증**

   ```java
   @Override
   public Object resolveArgument(...) {
       // 1. 엑셀 파일 파싱
       List dtos = excelParser.parse(file);

       // 2. Bean Validation 적용
       Set> violations = validator.validate(dtos);

       // 3. 검증 실패 시 예외 발생
       if (!violations.isEmpty()) {
           throw new ConstraintViolationException(violations);
       }

       return dtos;
   }
   ```

   - 요청 데이터를 원하는 형태로 변환합니다.
   - 변환된 데이터에 대한 검증을 수행합니다.
   - 검증된 데이터를 컨트롤러로 전달합니다.

3. **Spring MVC 통합**
   - `WebMvcConfigurer`를 통해 Argument Resolver를 등록합니다.
   - 컨트롤러 메서드 호출 전에 자동으로 실행됩니다.
   - 변환된 데이터를 컨트롤러 파라미터로 주입합니다.

**서비스 레이어 검증의 단점 해결 방식(장점)**:

1. **검증 로직 중복 제거**

   - 도메인 모델의 `@Valid` 어노테이션을 그대로 활용합니다.
   - 서비스 레이어에서 별도의 검증 로직을 구현할 필요가 없습니다.
   - 검증 규칙 변경 시 도메인 모델만 수정하면 됩니다.

2. **관심사 분리**

   - 파일 파싱과 검증 로직을 Argument Resolver로 분리합니다.
   - 서비스 레이어는 순수한 비즈니스 로직에만 집중할 수 있습니다.
   - 코드의 가독성과 유지보수성이 향상됩니다.

3. **일관된 검증 처리**
   - 모든 엑셀 업로드 API에서 동일한 검증 로직을 적용합니다.
   - 검증 실패 시 일관된 예외 처리가 가능합니다.
   - API 응답 형식을 표준화할 수 있습니다.

**단점**:

- 구현 복잡도 증가
- Spring MVC 내부 동작 이해 필요

## 결론

두 방식 모두 장단점이 존재하므로, 프로젝트의 규모와 요구사항에 따라 적절한 방식을 선택하는 것이 중요합니다. 저는 초기 개발 단계에서는 서비스 레이어 검증으로 빠르게 구현하고, 안정화 단계에서는 Argument Resolver 방식으로 리팩토링을 진행했는데 좋은 접근 방법이라고 생각합니다!
