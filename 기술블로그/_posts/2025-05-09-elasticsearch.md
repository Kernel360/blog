---
layout: post  
title: "엘라스틱서치"
author: "허성은"
categories: "기술블로그"
banner:
  image: 썸네일로 넣고 싶은 이미지 링크
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`elasticsearch`]
---

# 스프링 프로젝트에서 Elasticsearch로 태그 기반 매물 추천 시스템 구현하기

## 목차
1. [Elasticsearch 소개](#elasticsearch-소개)
2. [Elasticsearch의 장단점](#elasticsearch의-장단점)
3. [검색 동작 원리](#검색-동작-원리)
4. [스프링 부트에 Elasticsearch 연동하기](#스프링-부트에-elasticsearch-연동하기)
5. [태그 기반 매물 추천 시스템 구현](#태그-기반-매물-추천-시스템-구현)
6. [마무리](#마무리)

## Elasticsearch 소개

Elasticsearch는 Lucene 기반의 오픈소스 분산형 RESTful 검색 및 분석 엔진입니다. 대용량 데이터를 거의 실시간(Near Real-Time)으로 저장, 검색, 분석할 수 있는 기능을 제공합니다. JSON 기반의 문서를 저장하고 검색할 수 있으며, 복잡한 쿼리와 집계를 통해 데이터로부터 인사이트를 도출할 수 있습니다.

Elasticsearch는 ELK 스택(Elasticsearch, Logstash, Kibana)의 핵심 구성 요소로, 로그 분석, 전문 검색(Full-text search), 보안 인텔리전스, 비즈니스 분석, 모니터링 등 다양한 사용 사례에 적용됩니다.

## Elasticsearch의 장단점

### 장점

1. **강력한 전문 검색 기능**
   - 형태소 분석, 자동완성, 오타 수정, 동의어 처리 등 다양한 검색 기능 제공
   - 다국어 처리 및 자연어 처리 기능 지원

2. **확장성(Scalability)**
   - 수평적 확장이 용이하여 대용량 데이터 처리 가능
   - 샤드(Shard) 기반으로 데이터를 분산 저장하여 부하 분산 가능

3. **고가용성(High Availability)**
   - 레플리카(Replica) 기능을 통해 데이터 복제 및 장애 대응
   - 자동 복구 기능으로 안정적인 서비스 제공

4. **실시간 분석**
   - 거의 실시간(Near Real-Time) 검색 및 분석 가능
   - 복잡한 집계(Aggregation) 기능으로 데이터 분석 가능

5. **RESTful API**
   - HTTP 프로토콜 기반의 REST API 제공
   - 다양한 언어로 쉽게 연동 가능

### 단점

1. **리소스 사용량**
   - 메모리 사용량이 높아 충분한 리소스 확보 필요
   - 대규모 클러스터 운영 시 관리 비용 증가

2. **학습 곡선**
   - 초기 설정과 최적화에 전문 지식 필요
   - 복잡한 쿼리와 매핑 설정 이해 필요

3. **트랜잭션 지원 부족**
   - ACID 트랜잭션을 완전히 지원하지 않음
   - 데이터 일관성보다 가용성과 분산 처리에 최적화

4. **실시간 업데이트의 한계**
   - 인덱싱 후 검색 가능해지기까지 약간의 지연 발생
   - 완벽한 실시간 시스템이 아닌 '거의 실시간' 시스템

5. **비용**
   - 클라우드 환경에서 운영 시 상당한 비용 발생 가능
   - 최적화되지 않은 쿼리는 리소스 낭비 초래

## 검색 동작 원리

Elasticsearch에서 검색이 동작하는 원리를 간략히 설명하면 다음과 같습니다:

### 1. 인덱싱 과정

1. **문서 분석(Analysis)**
   - 텍스트를 토큰(Term)으로 분리
   - 불용어(stopwords) 제거, 소문자 변환, 어간 추출(stemming) 등 처리

2. **역색인(Inverted Index) 생성**
   - 각 토큰이 어떤 문서에 존재하는지 매핑
   - 빠른 검색을 위한 자료구조 구축

```
[일반 색인]
문서1: "Elasticsearch는 검색 엔진입니다"
문서2: "Elasticsearch로 빠른 검색을 구현합니다"

[역색인]
"Elasticsearch": 문서1, 문서2
"검색": 문서1, 문서2
"엔진": 문서1
"빠른": 문서2
"구현": 문서2
```

### 2. 검색 과정

1. **쿼리 분석**
   - 사용자 쿼리를 분석하여 검색 토큰 추출
   - 검색 조건에 맞는 쿼리 실행 계획 수립

2. **분산 검색**
   - 클러스터 내 여러 노드에 분산된 샤드에서 병렬 검색
   - 검색 결과 취합 및 정렬

3. **스코어링(Scoring)**
   - 관련성 점수(relevance score) 계산
   - TF-IDF, BM25 등의 알고리즘 사용

4. **결과 반환**
   - 관련성 높은 순으로 정렬된 결과 반환
   - 하이라이팅, 집계 등 후처리 적용

## 스프링 부트에 Elasticsearch 연동하기

### 1. 의존성 추가

`build.gradle` 파일에 다음 의존성을 추가합니다:

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
    // 기타 의존성...
}
```

### 2. 설정 파일 작성

`application.properties` 또는 `application.yml` 파일에 Elasticsearch 연결 정보를 설정합니다:

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic  # 필요한 경우
    password: password  # 필요한 경우
```

### 3. Elasticsearch 설정 클래스 생성

```java
@Configuration
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {
    
    @Value("${spring.elasticsearch.uris}")
    private String elasticsearchUrl;
    
    @Override
    public RestHighLevelClient elasticsearchClient() {
        ClientConfiguration clientConfiguration = ClientConfiguration.builder()
            .connectedTo(elasticsearchUrl.replace("http://", ""))
            .build();
        return RestClients.create(clientConfiguration).rest();
    }
}
```

### 4. 매물 문서 모델 정의

```java
@Document(indexName = "properties")
public class Property {
    
    @Id
    private String id;
    
    @Field(type = FieldType.Text, analyzer = "standard")
    private String title;
    
    @Field(type = FieldType.Text, analyzer = "standard")
    private String description;
    
    @Field(type = FieldType.Keyword)
    private String propertyType; // 아파트, 빌라, 단독주택 등
    
    @Field(type = FieldType.Double)
    private Double price;
    
    @Field(type = FieldType.Keyword)
    private List<String> tags; // 태그 목록: 역세권, 신축, 주차가능, 반려동물, 남향 등
    
    @Field(type = FieldType.GeoPoint)
    private GeoPoint location;
    
    // 생성자, getter, setter
}
```

### 5. Repository 인터페이스 생성

```java
public interface PropertyRepository extends ElasticsearchRepository<Property, String> {
    
    // 태그 기반 검색
    List<Property> findByTagsIn(List<String> tags);
    
    // 태그와 가격 범위로 검색
    List<Property> findByTagsInAndPriceBetween(List<String> tags, Double minPrice, Double maxPrice);
    
    // 특정 태그를 모두 포함하는 매물 검색 (커스텀 쿼리)
    @Query("{\"bool\": {\"must\": [{\"terms\": {\"tags\": ?0}}]}}")
    List<Property> findByTagsAll(List<String> tags);
}
```

## 태그 기반 매물 추천 시스템 구현

### 1. 서비스 레이어 구현

```java
@Service
public class PropertyService {
    
    private final PropertyRepository propertyRepository;
    private final ElasticsearchOperations elasticsearchOperations;
    
    public PropertyService(PropertyRepository propertyRepository, ElasticsearchOperations elasticsearchOperations) {
        this.propertyRepository = propertyRepository;
        this.elasticsearchOperations = elasticsearchOperations;
    }
    
    // 매물 저장/업데이트
    public Property save(Property property) {
        return propertyRepository.save(property);
    }
    
    // 태그 기반 기본 검색
    public List<Property> searchByTags(List<String> tags) {
        return propertyRepository.findByTagsIn(tags);
    }
    
    // 태그 기반 매물 추천
    public List<Property> recommendProperties(List<String> userPreferenceTags, Double budget) {
        // 검색 조건 생성
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        // 태그 필터 (가중치 부여)
        if (userPreferenceTags != null && !userPreferenceTags.isEmpty()) {
            // 태그 매칭에 가중치 부여 - 많은 태그가 일치할수록 점수 높음
            for (String tag : userPreferenceTags) {
                boolQuery.should(QueryBuilders.termQuery("tags", tag));
            }
            // 적어도 하나의 태그는 일치해야 함
            boolQuery.minimumShouldMatch(1);
        }
        
        // 예산 필터
        if (budget != null) {
            boolQuery.filter(QueryBuilders.rangeQuery("price").lte(budget));
        }
        
        // 검색 쿼리 생성 (최대 10개 추천)
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(boolQuery)
                .withSort(SortBuilders.scoreSort())
                .withPageable(PageRequest.of(0, 10))
                .build();
        
        // 검색 실행
        SearchHits<Property> searchHits = elasticsearchOperations.search(searchQuery, Property.class);
        
        // 결과 변환
        return searchHits.stream()
                .map(SearchHit::getContent)
                .collect(Collectors.toList());
    }
}
```

### 2. 컨트롤러 구현

```java
@RestController
@RequestMapping("/api/properties")
public class PropertyController {
    
    private final PropertyService propertyService;
    
    public PropertyController(PropertyService propertyService) {
        this.propertyService = propertyService;
    }
    
    @PostMapping
    public ResponseEntity<Property> createProperty(@RequestBody Property property) {
        return ResponseEntity.ok(propertyService.save(property));
    }
    
    @GetMapping("/recommend")
    public ResponseEntity<List<Property>> recommendProperties(
            @RequestParam List<String> tags,
            @RequestParam(required = false) Double budget) {
        
        return ResponseEntity.ok(propertyService.recommendProperties(tags, budget));
    }
}
```

### 3. 고객 프로필 및 태그 관리 구현

```java
@Document(indexName = "user_profiles")
public class UserProfile {
    
    @Id
    private String userId;
    
    @Field(type = FieldType.Keyword)
    private List<String> preferredTags;
    
    @Field(type = FieldType.Double)
    private Double budget;
    
    // 생성자, getter, setter
}

@Service
public class RecommendationService {
    
    private final PropertyService propertyService;
    private final UserProfileRepository userProfileRepository;
    
    public RecommendationService(PropertyService propertyService, UserProfileRepository userProfileRepository) {
        this.propertyService = propertyService;
        this.userProfileRepository = userProfileRepository;
    }
    
    // 사용자 맞춤 추천
    public List<Property> getPersonalizedRecommendations(String userId) {
        // 사용자 프로필 조회
        UserProfile userProfile = userProfileRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("사용자 프로필을 찾을 수 없습니다"));
        
        // 사용자 취향에 맞는 매물 추천
        return propertyService.recommendProperties(
                userProfile.getPreferredTags(), 
                userProfile.getBudget()
        );
    }
}
```

### 4. 태그 기반 매핑 및 인덱스 설계

```java
@Document(indexName = "properties")
@Setting(settingPath = "elasticsearch/settings.json")
@Mapping(mappingPath = "elasticsearch/mappings.json")
public class Property {
    // 필드 정의
}
```

`src/main/resources/elasticsearch/settings.json`:
```json
{
  "index": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "korean": {
          "type": "nori",
          "tokenizer": "nori_tokenizer"
        }
      }
    }
  }
}
```

`src/main/resources/elasticsearch/mappings.json`:
```json
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "korean"
    },
    "description": {
      "type": "text",
      "analyzer": "korean"
    },
    "propertyType": {
      "type": "keyword"
    },
    "tags": {
      "type": "keyword"
    },
    "price": {
      "type": "double"
    },
    "location": {
      "type": "geo_point"
    }
  }
}
```

## 마무리

Elasticsearch를 활용한 태그 기반 매물 추천 시스템은 고객의 선호도와 요구사항에 맞는 매물을 효과적으로 찾아주는 강력한 도구입니다. 특히 태그 기반 검색은 복잡한 조건을 간단하게 표현할 수 있어 사용자 경험을 크게 향상시킵니다.

이 시스템의 주요 장점은 다음과 같습니다:

1. **개인화된 추천**: 사용자의 선호 태그와 예산에 맞춘 맞춤형 매물 추천
2. **다양한 검색 조건**: 태그, 위치, 가격 등 다양한 조건을 조합한 검색
3. **빠른 검색 속도**: Elasticsearch의 분산 처리 기능으로 대량의 매물 데이터에서도 빠른 검색 가능

실제 부동산 플랫폼에 적용할 때는 태그 시스템의 표준화와 사용자 프로필 데이터 수집에 주의를 기울여야 합니다. 또한 정기적인 인덱스 최적화와 데이터 동기화를 통해 검색 품질을 유지하는 것이 중요합니다.

Elasticsearch와 스프링 부트를 결합하면 빠르고 유연한 태그 기반 매물 추천 시스템을 구축할 수 있으며, 이는 고객 만족도와 매물 매칭 정확도를 크게 향상시킬 수 있습니다.
