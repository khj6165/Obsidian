#### DTO : Entity 노출하지 않기
api 스펙을 위한 별도의 DTO로 파라미터를 받아야한다. 
엔티티를 외부에오는 걸로 바인딩하면 장애발생
api만들때는 엔티티를 파라미터를 받아서도, 노출해서도 안된다.
```java
/**
* 등록 V2: 요청 값으로 Member 엔티티 대신에 별도의 DTO를 받는다.
*/
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid
	CreateMemberRequest request) {
	Member member = new Member();
	member.setName(request.getName());
	Long id = memberService.join(member);
	return new CreateMemberResponse(id);
}
@Data
static class CreateMemberRequest {
	private String name;
}
@Data
static class CreateMemberResponse {
	private Long id;
	public CreateMemberResponse(Long id) {
		this.id = id;
	}
}
```
validation도 api스펙에다가 @NotEmpty추가. 엔티티에 추가하지 않기
DTO를 안에넣을지 바깥에 파일로 넣을지는 판단하기.

서비스에서 Member를 직접 반환하지 않는다. 필요한 경우에 id정도만 반환. 엔티티를 반환하면 쿼리가 추가되므로. 커멘드와 쿼리 분리.
```java
/**
* 수정 API
*/
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id,
	@RequestBody @Valid UpdateMemberRequest request) {
	memberService.update(id, request.getName());
	Member findMember = memberService.findOne(id);
	return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}
```

응답또한 한 엔티티에 각각api 응답을 위한 프레젠테이션 계층이 담기는걸 지양해야함

```java
/**
* 조회 V2: 응답 값으로 엔티티가 아닌 별도의 DTO를 반환한다.
*/
@GetMapping("/api/v2/members")
public Result membersV2() {
	List<Member> findMembers = memberService.findMembers();
	//엔티티 -> DTO 변환
	List<MemberDto> collect = findMembers.stream()
	.map(m -> new MemberDto(m.getName()))
	.collect(Collectors.toList());
	return new Result(collect);
}
@Data
@AllArgsConstructor
static class Result<T> {
	private T data;
}
@Data
@AllArgsConstructor
static class MemberDto {
	private String name;
}
```

#### 지연 로딩과 조회 성능 최적화 - ManyToOne, OneToOne
**1. 양방향 연관관계 무한루프 문제 : 한곳을 @JsonIgnore로 양방향을 끊음**
**2. @ManyToOne(fetch = LAZY) ByteBuddyInterceptor 프록시**
**3. Hibernate5JakartaModule : 프록시인 애는 데이터를 안뿌리는 설정**
```gradle
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5-jakarta'
```

```java
@Bean
Hibernate5Module hibernate5Module() {
return new Hibernate5Module();
}
```

**4. 지연로딩 : 데이터가 있는애들은 강제초기화**
<span style="background:#fff88f">order.getMember()까지는 프록시 객체</span>
<span style="background:#fff88f">order.getMember().getName() : Lazy 강제 초기화 > 영속성 컨택스트확인 DB쿼리 날림.</span>

- 주의: 지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다! 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용해라!

order 건수 2개조회하기 위해 order 한번 실행 
order1 + 회원지연로딩2 + 배송지연로딩2 총 5개의 쿼리 수행
-> 모든 연관관계를 LAZY로 바꿈. 지연로딩은 영속성 컨텍스트에서 조회해서 이미 있는경우 조회쿼리 생략.

**5. 패치조인 - OrderRepository**
LAZY로딩을 고려하지 않아도 된다.
```java
public List<Order> findAllWithMemberDelivery() {
return em.createQuery(
	"select o from Order o" +
	" join fetch o.member m" +
	" join fetch o.delivery d", Order.class)
	.getResultList();
}
```
쿼리가 1번 나간다. -> 대신 엔티티로 조회해서 모두 select해옴
많은 곳에서 사용할 수 있어서 재활용 가능하다는 장점이 있다.

**6. 개선본 : api스펙에 의존한다는 단점. 사실 5번과 성능차이가 많이는 안남.**
- Controller
```java
private final OrderSimpleQueryRepository orderSimpleQueryRepository; //의존관계 주입
/**
* V4. JPA에서 DTO로 바로 조회
* - 쿼리 1번 호출
* - select 절에서 원하는 데이터만 선택해서 조회
*/
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
	return orderSimpleQueryRepository.findOrderDtos();
}
```

- SimpleQueryRepository로 따로 빼기 : 정말 복잡한 쿼리가 필요할 때 유지보수에 좋음. 조회전용으로 화면에 맞춰서 쓰기.
```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
private final EntityManager em;
public List<OrderSimpleQueryDto> findOrderDtos() {
	return em.createQuery(
		"select new
		jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name,
		o.orderDate, o.status, d.address)" +
		" from Order o" +
		" join o.member m" +
		" join o.delivery d", OrderSimpleQueryDto.class)
		.getResultList();
	}
}
```

**쿼리 방식 선택 권장 순서**
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. 대부분의 성능 이슈가 해결된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.

#### 컬렉션 조회 최적화 - OneToMany
order 안에 orderItems가 있을때, order만 dto로 변환하면 해결되지 않음. orderItems도 엔티티로 보내면 안된다. 엔티티 의존을 완전히 끊고 dto로 변환해야함. > OrderDto, OrderItemDto 필요.

order, orderitem을  sql입장에서 조인하면 결과데이터에서 orderid가 orderitem만큼 n배가 되어버림. 
orderid는 중복되지 않기를 원하는 상황에서는 distinct키워드를 넣는다.
JPA에서의 distinct = db에서의 distinct + jpa에서 자체적으로 order가 같은id값이면 한번더 중복제거 반환. (db에서는 데이터가 완전히 같아야 중복제거 가능)

