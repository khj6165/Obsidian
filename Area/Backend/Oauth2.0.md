## Oauth 2.0 Token Types
1.
Access Token
•
클라이언트에서 사용자의 보호된 리소스에 접근하기 위해 사용하는 일종의 자격 증명 으로서 역할을 하며 리소스 소유자가 클라이언트에게 부여한
권한 부여의 표현이다
•
일반적으로 JWT(JSON Web Tokens) 형식을 취하지만 사양에 따라 그럴 필요는 없다
•
토큰에는 해당 액세스 기간 , 범위 및 서버에 필요한 기타 정보가 있다
•
타입에는 식별자 타입 (Identifier Type) 과 자체 포함타입 (Self contained Type) 이 있다
2.
Refresh Token
•
액세스 토큰이 만료된 후 새 액세스 토큰을 얻기 위해 클라이언트 응용 프로그램에서 사용하는 자격 증명
•
액세스 토큰이 만료되는 경우 클라이언트는 권한 부여 서버로 인증하고 Refresh Token 을 전달한다 .
•
인증 서버는 Refresh Token 의 유효성을 검사하고 새 액세스 토큰을 발급한다
•
Refresh Token 은 액세스 토큰과 달리 권한 서버 토큰 엔드포인트에만 보내지고 리소스 서버에는 보내지 않는다
3.
ID Token
•
OpenID Connect 챕터에서 학습함
4.
Authorization Code
•
권한 부여 코드 흐름에서 사용 되며 이 코드는 클라이언트가 액세스 토큰과 교환할 임시 코드 임
•
사용자가 클라이언트가 요청하는 정보를 확인하고 인가 서버로부터 리다이렉트 되어 받아온다



OAuth2User

OidcUser
기본 구현체 : DefaultOidcUser
id토큰 내의 claim정보를 통해 인증처리를 한다.
![[Pasted image 20250212104655.png]]

## OAuth2 Resource Server
클라이언트의 접근을 제한하는 인가 정책을 설정한다.
인가서버에서 발급한 Access Token의 유효성을 검증하고 접근 범위에 따라 적절한 자원을 전달하도록 설정한다.

### JWT
JWT로 전달되는 토큰을 검증하기 위한 JwtDecoder, BearerTokenAuthenticationFilter, JwtAuthenticationProvider등의 클래스 모델들을 제공한다.


application.yml파일에 keycloak설정을 하는 순간 로그인기본창은 안뜨고 401error만뜸.
토큰 검증을 거쳐야만 