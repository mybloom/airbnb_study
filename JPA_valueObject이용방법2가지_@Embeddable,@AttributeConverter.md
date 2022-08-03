# @Embeddable , @AttributeConverter 차이

- @Embeddable 와 @AttributeConverter 의 장단점


> @Embeddable
- I want to draw the attention to the email fields in the User entity code:
- This should use regex or a library to validate that the passed in String is a valid email address
- 
- 같은 엔티티에 다른 필드 이름을 지정하기 위해 AttributeOverrides를 추가해야 한다!없으면 다음과 같은 예외가 발생한다.
- 왜냐면  same database column value.에 매핑되기 때문이다. 그래서 `@AttributeOverrides`를 추가해줘야 한다.
```java
 @Embedded
    @AttributeOverrides(@AttributeOverride(name = "value", column = @Column(name = "personal_email")))
    private Email personalEmail;

    @Embedded
    @AttributeOverrides(@AttributeOverride(name = "value", column = @Column(name = "work_email")))
    private Email workEmail;
```

## AttributeConverter
- AttributeConverter는 value object 와 db의 single column 과 매핑을 정의할 수 있다.
- 값 개체를 여러 열에 매핑하는 데 사용할 수 있는 @Embeddable 보다 더 제한적
> 다른점 
- There is no need to have a protected constructor
- The value field can be final.
- DB 컬럼과 매핑하기 위해서 Converter생성 
-  AttributeConverter<Email, String> : 명시된 타입은 value object , database field type 를 나타낸다.
```java
import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter(autoApply = true) 
public class EmailConverter implements AttributeConverter<Email, String> { 
    @Override
    public String convertToDatabaseColumn(Email attribute) { 
        return attribute.getValue();
    }

    @Override
    public Email convertToEntityAttribute(String dbData) { 
        return new Email(dbData);
    }
}
```

>  @Embeddable
- 장점 
  - DB에 값 객체의 필드가 각각 저장되어야 할 때 사용하기 좋다.
- 단점
  - 같은 타입의 값 객체가 엔티티 내에 여러 개 있을 때 override 해줘야 한다. 
> AttributeConverter
- 장점
- 한 개의 속성을 가지는 값 객체에는 AttributeConverter로 코드 작성이 심플해지는 장점이 있다. `autoApply = true`의 옵션으로 해당 값 객체를 사용하는 엔티티에서는 자동으로 맵핑해줄 수 있다.
- 그리고 한 개의 속성을 가지면서 해당 속성의 값 검증등의 속성을 위한 로직이 필요하면서, 여러 엔티티에서 사용할 수 있는 속성이 있다면 값 객체로 wrapping하여 AttributeConverter를 사용하면 편리할 것이다. 
- 엔티티의 속성 이름이 자동으로 db 컬럼과 매핑되어 같은 타입의 값 객체 사용시 편리하다.
- 값 객체가 @Entity일 필요가 없으므로, JPA가 엔티티로 인식하기 위한 protected 기본생성자를 명시하지 않아도 된다.
- 단점 
  - 값 객체다 converter를 생성해줘야 하는 단점이 있지만 `상속`을 이용해 반복적인 코드를 줄일 수 있습니다.


> 참고자료
- https://www.wimdeblauwe.com/blog/2021/03/01/attributeconverter-vs-embeddable-in-jpa/
- JPA Entity Mapping(Enum Converter): https://techblog.woowahan.com/2600/