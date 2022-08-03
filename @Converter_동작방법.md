# @Converter 동작 방법

```java
@Enumerated(value = EnumType.STRING)
@Convert(converter = AccommodationTypeConverter.class)
private AccommodationType type;
```

## @Converter란
- DB 저장 값 또는 타입이 애플리케이션에서 사용하는 것과 다를 때 변환해주는 것
- Id,version,연관관계, Enumerated or Temporal가 표시된 속성에는 사용할 수 없다


## 동작방법

> 동작방법을 확인한 방법
- 인텔리제이에서 `AttributeConverter` 인터페이스의 메서드 구현체를 따라가보았다.
- `JpaAttributeConverterImpl`이 나왔고, `convertToDatabaseColumn` 메서드에 포인트 되었다.
```java
@Override
	public R toRelationalValue(O domainForm) {
		return attributeConverterBean.getBeanInstance().convertToDatabaseColumn( domainForm );
	}
```
- 구현체 메서드에도 별 내용이 없어 해당 메서드를 구글에 검색해봤는데, 누군가의 글이 나왔다.
  - 참고 자료 : https://namocom.tistory.com/892

- 동작방법을 확인하는 방법중에 로그스택을 따라가면서 구현체들을 하나씩 보는 방법이 있다는 것을 알았다.
- 블로그의 글을 보면서 중간에 나온 `ValueExtractor` 클래스는 갑자기 어떻게 나왔는지는 모르겠지만 ResultSet에서 추출하는 역할을 하는 클래스로 보인다. 
- 그렇게 따라가다가 `JdbcTypeJavaClassMappings` 클래스를 보았고, 해당 클래스가 JDBC type과 Java Class를 매핑해주는 거 같았다. `buildJavaClassToJdbcTypeCodeMappings()` 메서드를 확인해보니 아래처럼 어떤 jdbc를 어떤 자바 타입으로 바꿔주는 것인지 맵이 나왔다. 

```java
// these mappings are the ones outlined specifically in the spec
		workMap.put( String.class, Types.VARCHAR );
		workMap.put( BigDecimal.class, Types.NUMERIC );
		workMap.put( BigInteger.class, Types.NUMERIC );
		workMap.put( Boolean.class, Types.BIT );
		workMap.put( Byte.class, Types.TINYINT );
```

> 블로그 글의 결론
- DB -> Object 로 가는 경우 :  ValueExtractor가 사용됨
- Object -> DB 로 가는 경우 : ValueBinder 에 의하여 converter.toRelationalValue 가 수행됨



## 사용법
- 변환하려고 하는 타입이 기본 타입이아닐 경우 `a`ttributeName` 요소를 지정해야 한다.


## 예제

- AttributeConverter<A, B>
  - A 타입을 B 타입으로 바꾸는 Converter
  
> 예시1 : 기본 속성 변경
- Boolean 을 Integer로 변환
```java
    @Converter
       public class BooleanToIntegerConverter 
          implements AttributeConverter<Boolean, Integer> {  ... }
  
       @Entity
       public class Employee {
           @Id long id;
  
           @Convert(converter=BooleanToIntegerConverter.class)
            boolean fullTime;
            ...
       }
```

> 예시2 : 기본 속성의 자동 변경
- EmployeeDate 를 Date 로 변환
```java
 @Converter(autoApply=true)
       public class EmployeeDateConverter 
          implements AttributeConverter<com.acme.EmployeeDate, java.sql.Date> {  ... }
  
       @Entity
       public class Employee {
           @Id long id;
           ...
           // EmployeeDateConverter is applied automatically
           EmployeeDate startDate;
       }
```

> 예시3 : 자동 변경 converter Disable
- EmployeeDate 를 Data로 변환하는 것을 자동으로 설정한 converter를 해지한다.
```java
 @Convert(disableConversion=true)
 EmployeeDate lastReview;
```
     
> 예시4 : 기본타입의 컬렉션을 변환
- List<String> 을 특정 타입으로 변환
- 유의점 : @ElementCollection 를 붙여줘야 한다.
```java
@ElementCollection
// applies to each element in the collection
@Convert(converter=NameConverter.class) 
List<String> names;
```

> 예시5 : 기본 타입의 Map을 변환
```java
@ElementCollection
@Convert(converter=EmployeeNameConverter.class)
Map<String, String> responsibilities;
```

> 예시6 : maq의 기본 타입의 키를 변환
```java
@OneToMany
@Convert(converter=ResponsibilityCodeConverter.class, attributeName="key")
Map<String, Employee> responsibilities;
```

> 예시7 : @Embedded 된 embeddable 속성의 변환
```java
@Embedded
@Convert(converter=CountryConverter.class, attributeName="country")
Address address;
```


