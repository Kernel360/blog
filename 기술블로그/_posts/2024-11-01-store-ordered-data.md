---
layout: post  
title: 'RDB에 순서가 있는 데이터를 저장하는 방법'
author: '박수형'
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ['MySQL','RDB', 'JPA']
---

# RDB에 순서가 있는 데이터를 저장하는 방법

![스크린샷 2024-11-12 오전 10 15 13](https://github.com/user-attachments/assets/2fc91595-71b1-4cff-92da-4aa29d7be4fe)
TechPick 프로젝트를 진행하며 순서가 있는 데이터들을 RDB(MySQL)에 저장해야하는 상황을 자주 마주쳤습니다.
- 사용자가 등록한 태그의 순서를 저장
- 폴더안에 다른 폴더와 픽들의 순서를 각각 저장
- 픽에 속하는 태그들의 순서를 저장

RDB에 순서를 가지는 데이터들을 어떻게 저장할지 고민했던 방법을 소개하려고 합니다.

# 첫번째 시도 : Delete all & Insert all
![delete-all-save-all](https://github.com/user-attachments/assets/5071043d-4b81-4ec1-9ec4-9f4d0aac2b0a)

가장 단순한 방법입니다.<br>
테이블 안에서 각 아이템들이 순서를 가지게 저장하기때문에 별도의 순서 컬럼이 필요 없습니다.<br>
하지만, 순서 변경이 발생했을때, 관련된 아이템들을 모두 제거한후 변경된 순서로 전부 삽입합니다.<br>

### 장점
- 매우 단순한 로직​
- 높은 조회성능 (테이블에 정렬된 상태로 저장되어 추가적인 정렬이 필요 없음)​

### 단점
- 삭제후 삽입으로 인한 낮은 Update 성능​
- 영향을 받지 않아도 되는 아이템들까지 같이 영향을 받음​ <br>
  ex) 1, 2, 3, 4, 5 -> 1, 3, 2, 4, 5 로 변경하는 경우 2, 3번만 수정하면 되는데 전체 삭제 후 삽입 발생​

# 두번째 시도 : Convert to String​ 
![convert-to-string](https://github.com/user-attachments/assets/4b81b77c-9a66-4d84-a8e6-9285ab20ee59)

현재 프로젝트에서 사용하고 있는 방법입니다.<br>
아이템들의 순서를 idList형태로 저장하고 [AttributeConverter](https://www.baeldung.com/jpa-attribute-converters)를 사용해 db에 저장할때는 String으로 변환해 저장합니다.
조회시에는 db에 저장된 스트링을 파싱하여 List형태로 받습니다.

```
@Converter
public class OrderConverter implements AttributeConverter<List<Long>, String> {

	@Override
	public String convertToDatabaseColumn(List<Long> idList) {
		if (idList == null) {
			return "";
		}
		StringBuilder sb = new StringBuilder();
		for (Long id : idList) {
			sb.append(id).append(" ");
		}
		return sb.toString().trim();
	}

	@Override
	public List<Long> convertToEntityAttribute(String s) {
		List<Long> idList = new ArrayList<>();
		StringTokenizer st = new StringTokenizer(s);
		while (st.hasMoreTokens()) {
			idList.add(Long.parseLong(st.nextToken()));
		}
		return idList;
	}
}
```

### 장점
- List의 원소를 add/remove 하여 순서를 변경할 수 있기에 Update 성능이 높음
- 단순한 로직
### 단점
- List <-> String 컨버팅에 대한 오버헤드 존재​
- 숫자를 문자열로 저장하기때문에 저장 공간이 많이 필요​ <br> (int) 54187232 : 4byte / (String) 54187232 : 8byte​
- db는 순서와 관련된 컬럼을 가지고 있지 않기 때문에 정렬된 상태로 데이터를 조회할 수 없어 조회 성능이 낮음 <br> 애플리케이션에서 정렬 필요


# 세번째 시도 : Order with Interval​
![save-with-interval-1](https://github.com/user-attachments/assets/fab2e954-4dae-4502-9101-c7561765ea80)
이후 기능고도화때 고려하고 있는 방법 중 하나입니다.<br>
순서 컬럼을 가지는데, 최초 저장시 연속적으로 순서를 저장하는것이 아닌 **일정 간격을 두고 저장**합니다.<br>
이렇게 되면, 순서 변경이 일어나도 옮겨야 하는 위치에 끼워넣을 수 있어 실제 변경이 발생한 아이템만 업데이트 하면 됩니다.<br><br><br>

![save-with-interval-2](https://github.com/user-attachments/assets/025b9728-0f27-43c5-9970-4f2f8606f0d6)
하지만 순서변경이 계속 발생하다 보면, 위 그림처럼 순서가 붙어있는(299, 300) 위치에 이동이 발생할 수도 있습니다.<br>
이 경우에는 어쩔수 없이 해당 위치부터 마지막까지 아이템들의 순서를 업데이트하는 작업이 필요합니다.

### 장점
- 변경되는 row를 최소화시켜 나쁘지 않은 Update 성능을 가짐
- db에서 정렬(order by)해서 조회하기 때문에 조회 성능이 괜찮음​


# 성능 비교
![성능비교](https://github.com/user-attachments/assets/f2807c8d-7f2c-4d67-bb9c-fd3ab95b8cab)

마지막으로 성능비교입니다.<br>
