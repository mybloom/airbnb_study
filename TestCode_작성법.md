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

## hasSize()로 비교하기


- 만약 hasSize로 비교하시기 위해서는 테스트 전후로 테스트에 의한 DB 변경사항을 초기화해주시는 것이 좋겠습니다. 이를 위해서는 beforeEach, afterEach와 같은 JUnit 5의 API를 학습해보시구요.


## Test 애노테이션

> 학습사항: @SpringBootTest 외에도 테스트 환경을 제공하는 다른 어노테이션이 있습니다. 
### 현재 Repository를 테스트하는 환경에서는 어떤 어노테이션을 사용하면 좋은지, 그리고 본인이 생각해보신 @SpringBootTest 와 비교하여 어떤 이점이 있는지 자신만의 언어로 설명해주세요.

> **1** @SpringBootTest
- 통합테스트 : 실제 운영 환경처럼 전체 플로우가 제대로 동작하는지 보기 위한 테스트 
  - [ ] 그렇다면 `인수테스트`를 위해서는 SpringBootTest를 사용해야 할 것 같다. 확인해보자. 
- @SpringBootApplication을 찾아가서 모든 빈 스캔

> **2** MockMvc : @AutoConfigureMockMvc
- [ ] ServletContainer를 테스트용으로 띄우지않고 서블릿을 mocking 한 것이 동작한다. 즉 내장 톰캣이 구동되지 않는다.
  - 무슨 말인지 모르겠다. 느낌은 파악
- 여튼 POSTMAN에서 요청 날리는 것을 테스트코드로 하는 것이다. 
  - 예를 들어 `GET요청을 요청 파라미터와 함께 날리고 응답을 확인하는 테스트를 할 수 있다.`

- 셋팅 방법
```java
@SpringBootTest(webEnvironment = WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class Test클래스명
```

- 전체코드
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class SampleControllerTest {
  
    @Autowired
    MockMvc mockMvc;
  
    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string("hello henry"))
            .andDo(print());
    }
}
```

> **3** TestRestTemplate
- 내장 톰캣을 사용하는 실제 서블릿이 동작한다.
- RANDOM_PORT로 실제 사용할 포트 설정 
- [ ] API 요청을 테스트할 수 있는 것으로 보이며 MockMvc와 같은 기능인 것 같은데, 코드상으로 바로 이해되지는 않는다.
  - 전체코드를 확인해보아라.
- Controller를 테스트하는 것이고, Service는 함께 테스트시 무거워지니 @MockBean을 이용해서 해결할 수 있다. 
- 설정 방법
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class SampleControllerTest
```
- 전체 코드
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class SampleControllerTest {
  
    @Autowired
    TestRestTemplate testRestTemplate;
  
    @MockBean
    SampleService mockSampleService;
  
    @Test
    public void hello() throws Exception {
        // 테스트 환경에서 mockSampleService.getName()이 호출되면 "wooody92"로 응답한다.
        when(mockSampleService.getName()).thenReturn("wooody92");
        String result = testRestTemplate.getForObject("/hello", String.class);
        assertEquals("hello wooody92", result);
    }
}
```

> **4** WebTestClient
- WebClient와 동일한 api를 사용 가능
- WebClient는 비동기 방식으로 요청을 보내고 기다리지 않고 요청에 대한 응답이 오면 콜백(이벤트)이 오면 콜백을 실행하는 방식으로 동작
- webFlux 의존성을 추가





> 참고 자료
- https://wooody92.github.io/spring%20boot/Spring-Boot-%ED%86%B5%ED%95%A9%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%99%80-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8/




## 테스트 코드 생성자 주입
- 테스트 코드에서도 생성자 주입이 가능한가요? 만약 가능하다면, 왜 그런가요? 가능하지 않다면, 또 왜 그렇닫고 생각하시나요?

