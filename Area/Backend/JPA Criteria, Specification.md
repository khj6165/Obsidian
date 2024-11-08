`JPA Criteria`는 동적 쿼리를 사용하기 위한 JPA 라이브러리이다.

JPQL과 같이 Entity 조회를 기본으로 하며, 컴파일 시점에 에러를 확인할 수 있다.
JPQL은 문자열을 사용하여 쿼리를 정의하는 반면, Criteria는 자바 객체 인스턴스로 정의한다.

Repository 예
```java
@Repository
public interface LectureRepository extends JpaRepository<Lecture, Long>, JpaSpecificationExecutor<Lecture> {
	...
}
```

JpaSpecificationExecutor 인터페이스
```java
/**
 * Interface to allow execution of {@link Specification}s based on the JPA criteria API.
 *
 * @author Oliver Gierke
 * @author Christoph Strobl
 */
public interface JpaSpecificationExecutor<T> {

    /**
     * Returns a single entity matching the given {@link Specification} or {@link Optional#empty()} if none found.
     *
     * @param spec can be {@literal null}.
     * @return never {@literal null}.
     * @throws org.springframework.dao.IncorrectResultSizeDataAccessException if more than one entity found.
     */
    Optional<T> findOne(@Nullable Specification<T> spec);

    /**
     * Returns all entities matching the given {@link Specification}.
     *
     * @param spec can be {@literal null}.
     * @return never {@literal null}.
     */
    List<T> findAll(@Nullable Specification<T> spec);

    /**
     * Returns a {@link Page} of entities matching the given {@link Specification}.
     *
     * @param spec can be {@literal null}.
     * @param pageable must not be {@literal null}.
     * @return never {@literal null}.
     */
    Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);

    /**
     * Returns all entities matching the given {@link Specification} and {@link Sort}.
     *
     * @param spec can be {@literal null}.
     * @param sort must not be {@literal null}.
     * @return never {@literal null}.
     */
    List<T> findAll(@Nullable Specification<T> spec, Sort sort);

    /**
     * Returns the number of instances that the given {@link Specification} will return.
     *
     * @param spec the {@link Specification} to count instances for. Can be {@literal null}.
     * @return the number of instances.
     */
    long count(@Nullable Specification<T> spec);
}
```

Specification 인터페이스
```java
public interface Specification<T> {
  Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb);
}
```
