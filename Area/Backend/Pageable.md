```java
@GetMapping("/users") public Page<User> getAllUsers(Pageable pageable) { 
	return userRepository.findAll(pageable);
}
```

Springboot 내부에서 url 파라미터가 컨트롤러에 바인딩이 될 때, Pageable이 존재하면 PageRequest 객체를 생성한다.

해당 객체에서 역시 정렬도 제공하는데, url을 다음과 같이 치면 정렬과 페이징이 동시에 수행되게 할 수 있다.

- `http://localhost:8080/members?page=0`
    - 0번 페이지 부터 20개 조회한다.
        - default 가 20개로 default를 수정하는 방법도 존재한다.
- `http://localhost:8080/members?page=0&size=5`
    - 0번 페이지부터 5개 조회한다.
- `http://localhost:8080/members?page=0&size=5&sort=id.desc`
    - 0번 페이지부터 5개 조회 하는데, id의 역순으로 조회한다.