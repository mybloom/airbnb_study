# 테스트코드 작성법

## assertAll 사용하기
- assertAll 사용하는 가장 좋은 방법은 AssertJ의 assertThat과 함께 사용하는 것이다. JUnit의 순수 기능을 이용함과 동시에 AssertJ를 쓰면 강력한 테스트를 만들 수 있다. 이 방식이 테스트코드 작성 시 가장 많이 이용되는 방법이다.

```java
Assertions.assertAll(
			() -> assertThat(savedWish.getAccommodationId()).isEqualTo(wish.getAccommodationId()),
			() -> assertThat(savedWish.getCustomer()).isEqualTo(wish.getCustomer())
		);
```

> 참고자료
- https://escapefromcoding.tistory.com/358

## Nested 
- DisplayName을 그룹핑해서 보여주는 애노테이션 

> 참고자료
- @Nested : https://bcp0109.tistory.com/297
- @Nested 관련 참고 예제(공식문서) : https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested
