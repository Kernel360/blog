---
layout: post  
title: 테스트코드에서의 유연한 Fixture 생성 (ObjectMother패턴 적용기)
author: 김민협
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0219/3.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [Java, ObjectMother, TestCode, TestFixture, Builder, Setter, Reflection]
---


## 개요

[현재 진행중인 프로젝트(클라이밍 커뮤니티 Orury)](https://github.com/Kernel360/f1-Orury-Backend)의 테스트코드를 작성하며, 테스트코드에서의 Fixture(테스트용 객체) 생성 중복에 대한 고민과 그 해결책들에 대해 생각해봤습니다.

우선, 아래와 같은 배경(팀적 컨벤션) 하에 이뤄진 과정임을 말씀드립니다.

1. Entity 코드에서는 @Setter의 사용을 금지함.
2. Dto는 record로 구현함.
3. 팩토리 패턴을 적용하여, Entity와 Dto에서 of() 메서드를 생성자로 활용함. (@Builder 활용X)

![1.png](https://github.com/Kernel360/blog-image/blob/main/2024/0219/1.png?raw=true)

## 기존 코드

위와 같은 배경에서,
기존의 테스트코드에서 Fixture 생성메서드는 다음과 같이 구현돼있습니다.

```java
  private UserDto createUserDto() {
      return UserDto.of(
              1L,
              "test@test.com",
              "test",
              "password",
              1,
              1,
              LocalDate.now().minusYears(20),
              "test.png",
              LocalDateTime.now(),
              LocalDateTime.now(),
              UserStatus.ENABLE
      );
  }

```

이렇게 작성하면, 테스트 메서드에서 아래와 같이 매번 생성자를 구현할 필요없이 간편하게 UserDto를 생성할 수 있다는 장점이 있습니다.

```java
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto();

      ...
  }

```

그리고 테스트 메서드에서 Entity나 Dto의 특정 필드에 대한 설정을 확인해야 한다면, 아래와 같이 구현하여 활용했습니다.

```java
  @Test
  void when_UserDto_then_**() {
      // given
      Long userId = 1L;
      UserDto userDto = createUserDto(userId);

      ...
  }

  private UserDto createUserDto(Long userId) {
      return UserDto.of(
              userId,
              "test@test.com",
              "test",
              "password",
              1,
              1,
              LocalDate.now().minusYears(20),
              "test.png",
              LocalDateTime.now(),
              LocalDateTime.now(),
              UserStatus.ENABLE
      );
  }
```

테스트 메서드에서 요구하는 인자가 달라지면, 다른 인자에 따른 생성 메서드를 따로 구현해줬습니다.

예를 들어, 테스트코드에서 유저의 id 말고, 이름과 생일에 대해서 설정이 필요하다면 아래와 같이 구현했어야 합니다.

```java
	@Test
  void when_UserDto_then_**() {
      // given
      String userName = "김민협";
      LocalDate birthday = LocalDate.of(1999, 3, 1);
      UserDto userDto = createUserDto(userName, birthday);

      ...
  }

  private UserDto createUserDto(String userName, LocalDate birthday) {
      return UserDto.of(
              1L,
              "test@test.com",
              userName,
              "password",
              1,
              1,
              birthday,
              "test.png",
              LocalDateTime.now(),
              LocalDateTime.now(),
              UserStatus.ENABLE
      );
  }
```

## 문제 상황

위의 기존 방식이 문제가 되는 상황은 생각보다 여러 군데에서 확인되었습니다.

![2.png](https://github.com/Kernel360/blog-image/blob/main/2024/0219/2.png?raw=true)

1. Entity나 Dto의 필드가 변경된다면, 많은 테스트코드들에서 private 메서드들을 수정해줘야 합니다.
  - 실례로, User의 한 필드가 boolean isDeleted로 설정돼 있었는데,
    탈퇴 말고도 제재된 경우도 고려하기 위해 커스텀Enum인 UserStatus status로 변경하게 된 경우가 있었습니다.
  - 이때, 거의 모든 도메인에서 UserDto를 활용하였기 때문에 해당 수정 작업이 모든 테스트코드에서 이뤄져야 했습니다.
2. 테스트코드 가독성을 위해, 설정이 필요한 필드들에 대해서만 인자로 받는 create*() 메서드를 구현하다보니, create*() 메서드들이 너무 많아지게 됐습니다.
  - 예를 들어 기존에 이름과 생일을 인자로 받는 createUserDto() 메서드가 있더라도,
    테스트 메서드에서 생일만 따로 설정이 필요하다면, 생일만을 인자로 받는 createUserDto()를 새로 만들어줘야 했습니다.
3. 같은 타입을 인자로 받는 create*() 메서드는 혼재가 불가능합니다.
  - String name만을 인자로 받는 createUserDto(String name)이 기존에 존재하는 상황에서
    String email만을 인자로 받는 createUserDto(String email)은 생성이 불가능합니다.
  - 이런 경우, createUserDtoWithEmail()과 같이 메서드명을 달리하여 임시방편으로 해결했습니다.

1번의 경우는 모든 테스트코드에 있는 createUserDto()와 같이 생성을 수행하는 private 메서드를 한 군데 모은, 별도의 클래스 분리해주는 Object Mother 패턴으로 해결되겠지만,

2번과 3번은 다른 해결책이 필요해 보입니다.

사실 이러한 문제점이 보일 때, 마땅한 해결책이 생각나지 않아 묻어두었던 것 같기도 합니다.

1번의 경우는 그냥 모든 메서드의 필드를 수정해줬고,
2번의 경우는 필요한 인자에 따른 create*()를 일일이 만들어줬고,
3번의 경우는 메서드명을 달리해서 충돌을 피해왔습니다.

그러나, 이번에 새로 추가되는 크루 기능을 구현하고 테스트코드를 작성하면서 더는 이러한 임시방편을 적용하지 말아야겠다고 결심했습니다. 왜냐하면 크루에는 “크루 성별기준, 크루 나이기준, 가입 시 신청 여부, 가입질문에 대한 답변 필요 유무”와 같이 테스트코드에서 각각 검증해줘야 할 필드들이 많았기 때문입니다. 필요한 필드에 대해서만 인자로 받는 메서드를 구현할 경우, 과도하게 createCrewDto() 메서드가 많아진다고 생각했기에, 해결책이 필요했습니다.

![3.png](https://github.com/Kernel360/blog-image/blob/main/2024/0219/3.png?raw=true)

## 해결책

가장 이상적인 해결책은 Entity나 Dto의 필드 값을 변경해줄 수 있게 만드는 것입니다.

```java
	@Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto();
      userDto.setName("김민협");
      userDto.setBirthday(LocalDate.of(1999, 3, 1));

      ...
  }
    
	@Test
  void when_User_then_**() {
      // given
      User user = createUser();
      user.setName("김민협");
      user.setBirthday(LocalDate.of(1999, 3, 1));

      ...
  }
```

위와 같이 구현할 수 있다면, 테스트코드의 가독성과 유지보수성 모두 향상시킬 수 있을 것으로 생각합니다.

그러나, ”Entity에서는 @Setter 활용을 하지 않기, Dto는 record(final class)로 구현”이라는 합당한 컨벤션을 지키기 위해서는 위 방법이 활용될 수 없습니다.

### 1. Reflection 활용

Java에서는 Reflection로 클래스의 정보를 검사하고 동적으로 조작할 수 있는 기능을 제공하고 있습니다. 해당 조작에서 필드값에 대해서도 변경이 가능하기에, User의 name을 변경한다면 아래와 같이 변경할 수 있습니다.

```java
	@Test
  void when_User_then_**() throws NoSuchFieldException, IllegalAccessException {
      // given
      User user = createUser();
      Field nameField = User.class.getDeclaredField("name");
      nameField.setAccessible(true);
      nameField.set(user, "김민협");

      ...
  }
```

코드를 보면, createUser()로 임의의 User를 생성한 다음,
Field nameField를 갖고 온뒤, 해당 필드의 setter를 가능하게 한 다음, 원하는 이름으로 필드를 변경합니다.

이 방법으로 Entity 코드에 setter 설정 없이, setter를 활용할 수 있습니다.

그러나 다음과 같은 문제점이 존재합니다.

1. 테스트 메서드명 옆에 Exception을 줄줄이 붙여 중요하지 않은 코드가 늘어나고,
2. Field를 선언하고, setAccessible을 true로 변경하는 코드가 가독성을 해칩니다.
   해당 코드를 메서드로 분리한다면, 기존의 문제상황과 차이가 없어집니다.
3. 가장 치명적으로, record로 구현된 Dto에 적용할 수 없습니다.
   IllegalAccessException (Can not set final * field) 에러가 반환되어, final로 선언된 필드들에 대해서는 이 방식으로도 변경이 불가능함을 알 수 있었습니다.

### 2. 설정하는 필드를 List로 받는, 유연한 Test Fixture 생성

이 방법을 검색해서 알아낸 게 아니라서, 이게 맞다라는 확신은 없지만 생각해낸 것 중에 가장 합리적이라고 생각하여 남겨봅니다.

우선, 시작은 테스트 메서드에서 create*() 시, 필요한 인자들을 유연하게 받을 수 있는 메서드가 있으면 좋을 것이라고 생각했습니다. 그래서 가상의 메서드를 상상하며 호출하는 방식 먼저 생각해봤습니다.

```java
	@Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
              new Object[]{NAME, "김민협"},
              new Object[]{EMAIL, "gbgreenbravo@gmail.com"}
      ));

      ...
  }
    
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
              new Object[]{BIRTHDAY, LocalDate.now().minusYears(25)}
      ));

      ...
  }
```

위와 같이, List<Object[]>로 받고 createUserDto() 메서드에서 해당 Object[]에 맞게 값을 생성해주면 되지 않을까? 하고 생각했습니다.

그래서 Enum과 삼항연산자를 활용하여 아래와 같이 인자에 유연한 createUserDto를 구현했습니다.

```java
  enum UserDtoField {
      ID, EMAIL, NAME, PASSWORD, SIGN_UP_TYPE, USER_GENDER, BIRTHDAY, PROFILE_IMAGE, USER_STATUS
  }

	createUserDto(List<Object[]> objects) {
      List<UserDtoField> fields = objects.stream().map(o -> o[0])
              .map(String::valueOf)
              .map(UserDtoField::valueOf)
              .toList();
      List<Object> values = objects.stream().map(o -> o[1])
              .toList();

      return UserDto.of(
              (fields.contains(ID)) ? (Long) values.get(fields.indexOf(ID)) : 1L,
              (fields.contains(EMAIL)) ? (String) values.get(fields.indexOf(EMAIL)) : "userEmail",
              (fields.contains(NAME)) ? (String) values.get(fields.indexOf(NAME)) : "userName",
              (fields.contains(PASSWORD)) ? (String) values.get(fields.indexOf(PASSWORD)) : "userPassword",
              (fields.contains(SIGN_UP_TYPE)) ? (Integer) values.get(fields.indexOf(SIGN_UP_TYPE)) : 1,
              (fields.contains(USER_GENDER)) ? (Integer) values.get(fields.indexOf(USER_GENDER)) : NumberConstants.MALE,
              (fields.contains(BIRTHDAY)) ? (LocalDate) values.get(fields.indexOf(BIRTHDAY)) : LocalDate.of(1999, 3, 1),
              (fields.contains(PROFILE_IMAGE)) ? (String) values.get(fields.indexOf(PROFILE_IMAGE)) : "userProfileImage",
              LocalDateTime.now(),
              LocalDateTime.now(),
              (fields.contains(USER_STATUS)) ? (UserStatus) values.get(fields.indexOf(USER_STATUS)) : UserStatus.ENABLE
      );
  }
```

설명을 드리자면, UserDtoField로 UserDto의 필드들을 구분해주고,

createUserDto()에서는 List<Object[]>를 인자로 받아,
Object[]의 0번째 인덱스 값은 field로 분류하고,
Object[]의 1번째 인덱스 값은 value로 분류했습니다.

그리고 return의 UserDto of()메서드 내에 각 인자를 지정해주는 곳에서 삼항연산자를 활용하여,
fields에 해당 필드가 있다면 values에서 해당 필드에 따른 값을 꺼내어 설정하는 것입니다.

위 방식으로도 충분하겠지만, 테스트코드의 가독성을 위해 create*()를 호출할 때 사용되는 불필요한 new Object[]를 지우고 싶었습니다. 아래와 같이 메서드를 호출하고 싶었습니다.

```java
	@Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
              NAME, "김민협",
              EMAIL, "gbgreenbravo@gmail.com"
      ));

      ...
  }
    
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
              BIRTHDAY, LocalDate.now().minusYears(25)
      ));

      ...
  }
```

따라서 메서드 인자를 List<Object[]>가 아닌 List<Object>로 받고,
짝수번째 index의 값은 필드명으로 인식하고,
홀수번째 index의 값은 필드의 값으로 인식하여
아래의 코드들로 변경시켰습니다.

#### 2-1. enum 활용

```java
  enum UserDtoField {
      ID, EMAIL, NAME, PASSWORD, SIGN_UP_TYPE, USER_GENDER, BIRTHDAY, PROFILE_IMAGE, USER_STATUS
  }

  UserDto createUserDto(List<Object> objects) {
      List<UserDtoField> fields = objects.stream().filter(o -> objects.indexOf(o) % 2 == 0)
              .map(String::valueOf)
              .map(UserDtoField::valueOf)
              .toList();
      List<Object> values = objects.stream().filter(o -> objects.indexOf(o) % 2 == 1)
              .toList();

      return UserDto.of(
              (fields.contains(ID)) ? (Long) values.get(fields.indexOf(ID)) : 1L,
              (fields.contains(EMAIL)) ? (String) values.get(fields.indexOf(EMAIL)) : "userEmail",
              (fields.contains(NAME)) ? (String) values.get(fields.indexOf(NAME)) : "userName",
              (fields.contains(PASSWORD)) ? (String) values.get(fields.indexOf(PASSWORD)) : "userPassword",
              (fields.contains(SIGN_UP_TYPE)) ? (Integer) values.get(fields.indexOf(SIGN_UP_TYPE)) : 1,
              (fields.contains(USER_GENDER)) ? (Integer) values.get(fields.indexOf(USER_GENDER)) : NumberConstants.MALE,
              (fields.contains(BIRTHDAY)) ? (LocalDate) values.get(fields.indexOf(BIRTHDAY)) : LocalDate.of(1999, 3, 1),
              (fields.contains(PROFILE_IMAGE)) ? (String) values.get(fields.indexOf(PROFILE_IMAGE)) : "userProfileImage",
              LocalDateTime.now(),
              LocalDateTime.now(),
              (fields.contains(USER_STATUS)) ? (UserStatus) values.get(fields.indexOf(USER_STATUS)) : UserStatus.ENABLE
      );
  }
```

우선 enum UserDtoField를 사용한다면, 기존 방식과 비슷하고, stream의 map()을 filter()로 바꿔주기만 하면 됩니다.

그러나 이 방식에는 아주 작은 문제점이 존재합니다.

enum의 특성상, 테스트케이스에서 UserDtoField.PassWord로 쓰기보다 import static으로 활용하여 Password 그대로 쓰는 것이 가독성이 좋습니다. 그러나, UserDtoField의 enum 필드들을 UserDto와 통일한다고 하면 ID와 같이 다른 Dto에서도 활용되는 필드명과 중복된다는 점입니다. 이러한 경우들 때문에 USER_ID나 CREW_ID와 같이 Dto 필드명과 상이한 값을 활용하기를 기피한 것도 있고,

enum을 사용하게 된다면, 나중에 혹시 모를 필드명의 변경에 대해 create*()메서드 뿐만 아니라, *Field enum의 값도 변경해줘야 하는 잠재적인 불편함도 있습니다.

#### 2-2. getDeclaredFields() 활용

따라서 getDeclaredFields()를 활용하여 enum말고 런타임 시 해당 Entity/Dto의 필드명을 받아오는 방법을 고안했습니다.

```java
  static final List<String> USER_DTO_FIELD_NAMES = Arrays.stream(UserDto.class.getDeclaredFields()).map(Field::getName).toList();

  UserDto createUserDto(List<Object> objects) {
      List<String> fields = objects.stream().filter(o -> objects.indexOf(o) % 2 == 0)
              .map(String::valueOf)
              .toList();
      List<Object> values = objects.stream().filter(o -> objects.indexOf(o) % 2 == 1)
              .toList();
      
      if (!USER_DTO_FIELD_NAMES.containsAll(fields))
          throw new IllegalArgumentException("Invalid field name in : " + fields);

      return UserDto.of(
              (fields.contains(USER_DTO_FIELD_NAMES.get(0))) 
                      ? (Long) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(0))) : 1L,
              (fields.contains(USER_DTO_FIELD_NAMES.get(1))) 
                      ? (String) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(1))) : "userEmail",
              (fields.contains(USER_DTO_FIELD_NAMES.get(2))) 
                      ? (String) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(2))) : "userNickname",
              (fields.contains(USER_DTO_FIELD_NAMES.get(3))) 
                      ? (String) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(3))) : "userPassword",
              (fields.contains(USER_DTO_FIELD_NAMES.get(4))) 
                      ? (Integer) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(4))) : 1,
              (fields.contains(USER_DTO_FIELD_NAMES.get(5))) 
                      ? (Integer) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(5))) : NumberConstants.MALE,
              (fields.contains(USER_DTO_FIELD_NAMES.get(6))) 
                      ? (LocalDate) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(6))) : LocalDate.of(1999, 3, 1),
              (fields.contains(USER_DTO_FIELD_NAMES.get(7))) 
                      ? (String) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(7))) : "userProfileImage",
              LocalDateTime.now(),
              LocalDateTime.now(),
              (fields.contains(USER_DTO_FIELD_NAMES.get(10))) 
                      ? (UserStatus) values.get(fields.indexOf(USER_DTO_FIELD_NAMES.get(10))) : UserStatus.ENABLE
      );
  }
```

우선, UserDto.class.getDeclaredFields()로 Dto의 필드를 가져오고 이를 List<String>으로 변환하여 상수로 지정합니다.

그런 다음, createUserDto()가 호출되면 List<Strings> fields에 필드명을 담고, values에는 필드값들을 담습니다.

삼항연산자의 경우는 코드는 길어졌으나, 기존과 같은 로직이 적용되어 있습니다.

한가지 추가된 부분은 !USER_DTO_FIELD_NAMES.containsAll(fields)으로, fields에 있는 값들을 검증해주는 부분입니다. 예를 들어 “nama”와 같이 유효하지 않은 필드명이 인자로 담기더라도, 해당 사항에 대해서 예외를 반환하는 곳이 없기 때문에 휴먼에러가 발생할 수 있는 부분입니다. 따라서 입력에 들어온 필드명이 유효한 것인지 검증하고, 유효하지 않다면 에러를 반환하도록 구현했습니다.

#### 리팩토링

위 코드를 가독성 좋게 리팩토링한다면 아래와 같은 코드로 정리될 수 있습니다.

아래 코드를 (Object Mother 패턴을 적용하여) 테스트용 FixtureFactory와 같은 클래스로 분리하여 사용한다면, 테스트코드 전반에서 활용할 수 있습니다.

```java
  static List<String> extractFields(List<Object> objects) {
      return objects.stream()
              .filter(o -> objects.indexOf(o) % 2 == 0)
              .map(String::valueOf)
              .toList();
  }

  static List<Object> extractValues(List<Object> objects) {
      return objects.stream()
              .filter(o -> objects.indexOf(o) % 2 == 1)
              .toList();
  }

  static void validateFields(List<String> fields, List<String> objectsFields) {
      if (!fields.containsAll(objectsFields))
          throw new IllegalArgumentException("Invalid field name in : " + objectsFields);
  }

  static <T> T extractFieldValue(List<String> fields, List<Object> values, String fieldName, T defaultValue) {
      int index = fields.indexOf(fieldName);
      return index != -1 ? (T) values.get(index) : defaultValue;
  }
    
  static final List<String> USER_DTO_FIELD_NAMES = Arrays.stream(UserDto.class.getDeclaredFields()).map(Field::getName).toList();

  static UserDto createUserDto(List<Object> objects) {
      List<String> fields = extractFields(objects);
      List<Object> values = extractValues(objects);

      validateFields(USER_DTO_FIELD_NAMES, fields);

      return UserDto.of(
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(0), 111L),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(1), "userEmail"),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(2), "userName"),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(3), "userPassword"),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(4), 1),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(5), 1),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(6), LocalDate.now().minusYears(25)),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(7), "userProfileImage"),
              LocalDateTime.now(),
              LocalDateTime.now(),
              extractFieldValue(fields, values, USER_DTO_FIELD_NAMES.get(10), UserStatus.ENABLE)
      );
  }
```

그렇다면 최종적으로, 기존에 enum의 값으로 호출되던 createUserDto()가

아래와 같이, 실제 Dto의 필드명으로만 인자에 담겨야 유효한 UserDto를 받을 수 있게 됩니다.

```java
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
              "name", "김민협",
              "email", "gbgreenbravo@gmail.com"
      ));

      ...
  }
    
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto(List.of(
		        "id", 31L
      ));

      ...
  }
```

이제까지의 2번의 방법은 제가 임의로 고안해낸 방법이다보니, 몇 가지 단점이 존재합니다.

- 암묵적인 컨벤션 : List에 짝수번째 인덱스는 필드명을 기입하고, 홀수번째 인덱스는 필드값을 넣어야 합니다.
- 테스트코드 작성 시 클래스의 필드명을 정확하게 모른다면, 계속 해당 클래스를 보면서 정확한 값을 넣어줘야 하는 불편함이 존재합니다.

### 3. 테스트용 @Builder static 클래스

이 방법은 이전의 2번의 방식대로 프로젝트에 반영하여 PR을 올렸었는데 그 [피드백](https://github.com/Kernel360/f1-Orury-Backend/pull/391#discussion_r1529875190)으로 나온 더 나은 해결책입니다.

가독성 측면에서도 2번과 비슷하고, 테스트코드 작성 시 불명확한 점이 없다는 점에서 이 방법을 최종적으로 채택했습니다. 그리고 Builder로 생성한 클래스를 record로 변환하는 것에는 제약이 없기 때문에, record에도 적용 가능한 방법입니다.

먼저 최종 코드를 보시겠습니다.

```java
  static final ObjectMapper mapper = new ObjectMapper().registerModule(new JavaTimeModule());

  @Getter
  @Builder
  static class TestUser {
      @Builder.Default Long id = 1L;
      @Builder.Default String email = "gbgreenbravo@gmail.com";
      @Builder.Default String name = "김민협";
      @Builder.Default String password = "비밀~!";
      @Builder.Default int signUpType = 1;
      @Builder.Default int gender = NumberConstants.MALE;
      @Builder.Default LocalDate birthday = LocalDate.now().minusYears(25);
      @Builder.Default String profileImage = "찰칵~!";
      @Builder.Default UserStatus status = UserStatus.ENABLE;
      @Builder.Default LocalDateTime createdAt = LocalDateTime.now();
      @Builder.Default LocalDateTime updatedAt = LocalDateTime.now();

      static TestUser.TestUserBuilder createUser() {
          return TestUser.builder();
      }

      User get() {
          return mapper.convertValue(this, User.class);
      }
  }
```

2번과 비교했을 때 가독성이나 명확성에서 큰 장점이 있습니다.

이제 코드를 읽어보면, 기존의 User라는 Entity 클래스의 필드값을 모두 가지는 TestUser를 테스트용 정적 클래스로 생성해줍니다. 그리고 ObjectMapper로 변환해주는 과정에서 Getter메서드가 있어야 하므로, @Getter을 설정해준 다음, @Builder을 통해 기본값과 설정값을 모두 할당하도록 하겠습니다.

각 필드들을 보면 타입과 필드명 말고도, @Builder.Default로 기본값이 지정돼있습니다. 이 덕분에 TestUser.builder().build();를 하더라도 각 필드에 null이 아닌 기본값들이 들어가게 되는 것입니다.

테스트코드에서는 createrUser()을 import static으로 가져와서 원하는 값을 설정하면,
TestUser.TestUserBuilder를 반환하게 됩니다. 이를 그대로 .build()로 닫아주면 TestUser이 반환되기에, 테스트코드에서 원하는 타입은 TestUser이 아닌 User이므로 get() 메서드를 통해 타입을 변환해줄 수 있습니다.

따라서, 위 코드가 테스트코드에서 쓰이면, 아래와 같이 작성될 수 있습니다.

```java
  @Test
  void when_User_then_**() {
      // given
      User user1 = createUser()
              .id(31L)
              .name("김민협").build().get();
                
      User user2 = createUser()
              .email("gbgreenbravo@gmail.com").build().get();

      ...
  }
    
  @Test
  void when_UserDto_then_**() {
      // given
      UserDto userDto = createUserDto().build().get();

      ...
  }
```

## 마무리 (w/ 멀티모듈)

3번의 방식이 가장 합리적이라 생각되어 이를 팀에서 적용하였고, 도메인별로 클래스를 나눠 각 도메인에 맞는 TestFixture 클래스를 구현했습니다!

추가로, 현재 프로젝트가 멀티모듈로 구성되어 있어, 각 모듈에서 중복적으로 사용되는 Fixture 코드 또한 존재했습니다. Domain 모듈의 테스트코드에서 UserDto를 생성하기도 하고, Client 모듈의 테스트코드에서도 UserDto를 생성하기에, Domain 모듈에서 생성하는 TestUserDto를 의존받을 필요가 있었습니다.

이에 대해 다른 모듈의 테스트 패키지 전체가 필요한 게 아니라, TestFixtures에 대해서만 의존이 필요했기에, java-test-fixtures 플러그인을 적용했습니다. (TestFixtures 의존성: 아래 도식의 빨간색 화살표)

![4.png](https://github.com/Kernel360/blog-image/blob/main/2024/0219/4.png?raw=true)

최종적으로 이 모든 적용을 끝내, 테스트코드 작성/유지보수의 편의성에 큰 향상을 가져올 수 있었습니다 :)
