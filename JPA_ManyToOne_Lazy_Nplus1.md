# N+1

## N+1 쿼리란
- 하위 엔티티들을 첫 쿼리 실행시 한번에 가져오지 않고, Lazy Loading으로 필요한 곳에서 사용되어 쿼리가 실행될 때 발생하는 문제가 N+1 쿼리 문제입니다.

> 코드 예시
- 코드 
```java
academyRepository.findAll()
```
- 발생 쿼리 
```sql
    select
        academy0_.id as id1_0_,
        academy0_.name as name2_0_ 
    from
        academy academy0_
```

- 연관객체 호출 코드 
```java
academies.stream()
			.map(a -> a.getSubjects().get(0).getName())
			.collect(Collectors.toList());
```

- 발생 쿼리 : N+1 쿼리 발생
 - 아래 쿼리가 10번(academy 객체 갯수) 발생 
```sql
     select
        subjects0_.academy_id as academy_3_7_0_,
        subjects0_.id as id1_7_0_,
        subjects0_.id as id1_7_1_,
        subjects0_.academy_id as academy_3_7_1_,
        subjects0_.name as name2_7_1_,
        subjects0_.teacher_id as teacher_4_7_1_ 
    from
        subject subjects0_ 
    where
        subjects0_.academy_id=?
```           

## 연관관계가 맺어진 Entity를 한번에 가져오기 위한 방법

### Join Fetch 
- 조회 시 바로 가져오고 싶은 Entity 필드를 지정하는 것
```java
@Query("select a from Academy a join fetch a.subjects")
List<Academy> findAllJoinFetch();
```

- 발생 쿼리 : `바로 가져오고 싶은 Entity` 필드인 subjects를 `지정`해서 쿼리에서 바로 가져오게 되었다.
```sql
    select
        academy0_.id as id1_0_0_,
        subjects1_.id as id1_7_1_,
        academy0_.name as name2_0_0_,
        subjects1_.academy_id as academy_3_7_1_,
        subjects1_.name as name2_7_1_,
        subjects1_.teacher_id as teacher_4_7_1_,
        subjects1_.academy_id as academy_3_7_0__,
        subjects1_.id as id1_7_0__ 
    from
        academy academy0_ 
    inner join
        subject subjects1_ 
            on academy0_.id=subjects1_.academy_id
```

- 해당 방법으로 join fetch 쿼리를 통해 하위의 하위 Entity 까지 가져올 수 있다.
- 이 필드는 Eager 조회, 저 필드는 Lazy 조회를 해야한다까지 쿼리에서 표현해야 한다. 
  - 즉, 여기서는 Academy가 Subject를 Eager조회하고, Subject의 Teacher는 Lazy조회 해야 한다고 표현.

> 단점 
- Join Fetch, EntityGraph 둘 다  `카테시안 곱(Cartesian Product)이 발생`하여 Subject의 수만큼 Academy가 중복 발생하게 됩니다.

> 해결법
- 일대다 필드의 타입을 Set으로 선언하는 것입니다. : Set은 중복을 허용하지 않는 자료구조이기 때문에 중복등록이 되지 않습니다
- distinct를 사용하여 중복을 제거
```sql
@Query("select DISTINCT a from Academy a join fetch a.subjects s join fetch s.teacher")
List<Academy> findAllWithTeacher();
```

### @EntityGraph
- 하위 그래프 탐색
```java
@EntityGraph(attributePaths = "subjects")
@Query("select a from Academy a")
List<Academy> findAllEntityGraph();
```
- 하위의 하위 그래프 탐색
```java
@EntityGraph(attributePaths = {"subjects", "subjects.teacher"})
@Query("select a from Academy a")
List<Academy> findAllEntityGraphWithTeacher();
```
- Entity Graph는 Outer Join 으로 join된다.
- 원본 쿼리의 손상 없이 Eager/Lazy 필드를 정의하고 사용
- [ ] 공통적으로 `카테시안 곱(Cartesian Product)이 발생`하여 Subject의 수만큼 Academy가 중복 발생하게 됩니다.
  - 이 부분에 대해서 정확한 이해 못함
- 해결법
```java
@EntityGraph(attributePaths = {"subjects", "subjects.teacher"})
@Query("select DISTINCT a from Academy a")
List<Academy> findAllEntityGraphWithTeacher();
```


> 정리
- N+1을 해결하기 위해서 Eager조회를 엔티티 속성에 명시하지 않고,  repositort에 직접 쿼리로 명시함으로 해결하는 것 같습니다.
- 방법은 쿼리로 명시하느냐, 엔티티그래프로 명시하느냐이구요. 둘다 조인을 사용한다는 것입니다.





## @ManyToOne Lazy N+1

FetchType.LAZY로 변경했을 때 주의점은 트랜잭션에 내에서 toString()이나 property 복사 같은 작업을 하면서 Employee의 Company에 접근하게 되면 추가 쿼리가 실행되면서 역시 N+1이 발생하게 됩니다.

트랜잭션 밖에서 Employee의 Company에 접근하게 되면 LazyInitializationException 오류가 발생합니다.

> LazyInitializationException
LazyInitializationException은 영속성 컨텍스트가 종료 된 후
관계 설정된 엔티티 참조하려고 할 때 발생

- FetchType을 LAZY로 변경하게 되면 이런 현상은 발생하지 않습니다. Company 를 proxy 객체로 가지고 있고 `필요한 시점에 쿼리를 호출`하기 때문입니다.

- 해결법 : DAO 레어이 상위에서 @Transaction사용으로 Transaction 세션 사용

> fetch join
- DAO 레이어에서 관계된 엔티티를 join으로 함께 조회 후 상위 레어어에게 전달
- JPQL , QueryDSL을 사용해야 한다
- 쿼리로 함께 연관관계 조회 후  연관관계 매핑이 되므로, 연관관계 객체 조회시 다시 쿼리를 날리지 않습니다.
- @ManyToOne의 FetchType을 LAZY로 설정했지만 Company까지 필요한 경우에는 Fetch Join을 이용


> 참고 자료
- https://jojoldu.tistory.com/165
- https://web-km.tistory.com/44