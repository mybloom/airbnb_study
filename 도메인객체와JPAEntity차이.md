# DDD Entity 와 JPA Entity 구분하기

## AccommodationImage 가 Entity로 구분된 이유가 무엇일까요? 
Accommodation테이블 AccommodationImage 테이블을 분리하고 싶어서 사용하게 되었습니다.

## 본인이 생각하는 도메인 객체와 JPA Entity의 차이점이 있다면 무엇이라고 생각하시나요?
도메인 객체와 JPA Entity의 차이점을 보면서
`도메인에 대해 생각할 때 Repository에 종속되어 생각하지 말자` 고 느꼈습니다.

해당 내용에 대해 조사하다보니 `패키지 구분`을 다시 생각해보게 되었습니다.
해당 프로젝트를 예로 들면, 위시리스트 조회시 Customer에서 하게 되는데요 Wish와 Cusotomer 엔티티를 같은 패키지에 두어 같은 도메인으로 인식하고 사용해도 되지 않을까란 생각을 하게 되었습니다.

--- 
## 참고자료에서 의미 있게 본 것 

- 도메인 주도 설계(DDD)라는 세계관에서 비롯된 명명법으로, 도메인 주도 설계에서는 '내부'에 존재하는 모든 것의 이름은 반드시 '유비쿼터스 도메인 언어(ubiquitous domain language)'관점에서 기술하라고 조언한다. 바꿔 말하면, 도메인에 대해 논의할 때 우리는 '주문'에 대해 말하는 것이지, '주문 리포지터리'에 대해 말하는 것이 아니다.
- 책 클린코드 내용

- 실전에서는?
ID라는 비슷한 개념을 가지고 있기에 JPA의 엔티티가 비즈니스 로직에 침범하는 것을 자주 봤다.
심지어 Repository를 넘어, Service 심지어 Controller까지 JPA의 엔티티가 와서 사용되는 경우를 보았다.
나는 그 JPA 엔티티를 Repository와 맞닿아 있는 Service까지로 하기로 타협을 하고 경계(boundary)를 그었다.
그 경계를 넘어서는 JPA의 엔티티가 아닌 일반 POJO 객체가 움직였다. POJO를 세부적으로 나누어보면 구분할 수 있는 ID가 있었으므로 (DB의 PK랑은 같은 경우도 다른 경우도 있었다) VO보다는 DDD의 Entity였다.


> 참고자료
- https://namocom.tistory.com/980