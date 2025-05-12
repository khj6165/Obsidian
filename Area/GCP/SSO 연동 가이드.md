### 1. Keycloak 설정

먼저 Keycloak과 Google Cloud 간의 SSO를 설정합니다. 이 과정은 앞서 설명한 대로 Keycloak에서 Google Identity Provider를 설정하고, Google Cloud에서 OAuth 클라이언트를 생성하는 과정입니다.
<keycloak sso 구글 공식 문서> 
https://cloud.google.com/architecture/identity/keycloak-single-sign-on?hl=ko#keycloak-17-or-later
#### 1.1. Keycloak 클라이언트 설정

1. Keycloak 관리 콘솔에 로그인합니다.
2. 해당 Realm을 선택합니다.
3. "Clients" 메뉴에서 새로운 클라이언트를 생성합니다.
4. 클라이언트 설정:
    - **Client ID**: `google`
    - **Client Protocol**: `openid-connect`
    - **Root URL**: `https://accounts.google.com`
    - **Redirect URIs**: `https://your-site.com/*`
    - **Web Origins**: `*`
5. "Save" 버튼을 클릭하여 클라이언트를 생성합니다.
#### 1.2. 클라이언트 설정 수정

1. 생성한 클라이언트를 선택합니다.
2. "Settings" 탭에서 다음 설정을 확인합니다:
    - **Access Type**: `confidential`
    - **Valid Redirect URIs**: `https://your-site.com/*`
    - **Base URL**: `https://your-site.com`
3. "Credentials" 탭에서 클라이언트 시크릿(Client Secret)을 복사합니다.
#### 1.3. Identity Provider 설정

1. "Identity Providers" 메뉴에서 "Add provider"를 클릭하고 "OpenID Connect v1.0"을 선택합니다.
2. Identity Provider 설정:
    - **Alias**: `google`
    - **Display Name**: `Google`
    - **Authorization URL**: `https://accounts.google.com/o/oauth2/auth`
    - **Token URL**: `https://oauth2.googleapis.com/token`
    - **Client ID**: Google Cloud에서 생성한 OAuth 클라이언트 ID
    - **Client Secret**: Google Cloud에서 생성한 OAuth 클라이언트 시크릿
    - **Default Scopes**: `openid email profile`
3. "Save" 버튼을 클릭하여 Identity Provider를 생성합니다.
### 2. Google Cloud 설정

Google Cloud에서 OAuth 클라이언트를 생성하고 필요한 권한을 설정합니다.

#### 2.1. OAuth 클라이언트 생성

1. Google Cloud Console에 로그인합니다.
2. "APIs & Services" > "Credentials"로 이동합니다.
3. "Create Credentials" > "OAuth client ID"를 선택합니다.
4. OAuth 클라이언트 ID 설정:
    - **Application type**: `Web application`
    - **Authorized redirect URIs**: `https://your-keycloak-server/auth/realms/YOUR_REALM/broker/google/endpoint`
5. "Create" 버튼을 클릭하여 OAuth 클라이언트를 생성합니다.
6. 생성된 클라이언트 ID와 클라이언트 시크릿을 복사합니다.

### 3. Spring Boot, Nuxt3 애플리케이션 설정
Spring Boot 애플리케이션에서 Keycloak을 사용하여 인증을 처리하고, Google Cloud API에 접근할 수 있도록 설정합니다.

#### 3.1. Spring Boot 의존성 추가

`pom.xml` 파일에 Keycloak과 Google Cloud 의존성을 추가합니다.
```xml
<dependencies>
    <!-- Spring Boot dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>
    <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>google-cloud-bigquery</artifactId>
        <version>2.1.0</version>
    </dependency>
</dependencies>
```
#### 3.2. Keycloak 설정

`application.yml` 파일에 Keycloak 설정을 추가합니다.
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: your-client-id
            client-secret: your-client-secret
            scope: openid, profile, email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
        provider:
          keycloak:
            issuer-uri: https://your-keycloak-server/auth/realms/YOUR_REALM
```
#### 3.3. Security 설정

Spring Security 설정을 추가하여 Keycloak을 통해 인증을 처리합니다.
```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.oauth2.client.oidc.web.logout.OidcClientInitiatedLogoutSuccessHandler;
import org.springframework.security.oauth2.client.registration.ClientRegistrationRepository;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(authorizeRequests ->
                authorizeRequests
                    .antMatchers("/").permitAll()
                    .anyRequest().authenticated()
            )
            .oauth2Login(oauth2Login ->
                oauth2Login
                    .loginPage("/oauth2/authorization/keycloak")
            )
            .logout(logout ->
                logout
                    .logoutSuccessHandler(oidcLogoutSuccessHandler())
            );
        return http.build();
    }

    @Bean
    public OidcClientInitiatedLogoutSuccessHandler oidcLogoutSuccessHandler(ClientRegistrationRepository clientRegistrationRepository) {
        OidcClientInitiatedLogoutSuccessHandler successHandler = new OidcClientInitiatedLogoutSuccessHandler(clientRegistrationRepository);
        successHandler.setPostLogoutRedirectUri("{baseUrl}");
        return successHandler;
    }
}
```

#### 3.4. Nuxt 3 설정

`nuxt.config.ts` 파일에 Keycloak 설정을 추가합니다.
```typescript
import { defineNuxtConfig } from 'nuxt3'

export default defineNuxtConfig({
  modules: [
    '@nuxtjs/auth-next',
    '@nuxtjs/axios'
  ],
  axios: {
    baseURL: 'https://your-keycloak-server/auth'
  },
  auth: {
    strategies: {
      keycloak: {
        scheme: 'oauth2',
        endpoints: {
          authorization: 'https://your-keycloak-server/auth/realms/YOUR_REALM/protocol/openid-connect/auth',
          token: 'https://your-keycloak-server/auth/realms/YOUR_REALM/protocol/openid-connect/token',
          userInfo: 'https://your-keycloak-server/auth/realms/YOUR_REALM/protocol/openid-connect/userinfo',
          logout: 'https://your-keycloak-server/auth/realms/YOUR_REALM/protocol/openid-connect/logout'
        },
        clientId: 'your-client-id',
        clientSecret: 'your-client-secret',
        scope: ['openid', 'profile', 'email'],
        codeChallengeMethod: '',
        responseType: 'code',
        grantType: 'authorization_code'
      }
    }
  }
})
```
#### 3.5. Keycloak 초기화

Keycloak을 초기화하고, Nuxt 3 애플리케이션에서 사용할 수 있도록 설정합니다.
```typescript
// plugins/keycloak.ts
import Keycloak from 'keycloak-js'

export default defineNuxtPlugin((nuxtApp) =&gt; {
  const keycloak = new Keycloak({
    url: 'https://your-keycloak-server/auth',
    realm: 'YOUR_REALM',
    clientId: 'your-client-id'
  })

  keycloak.init({ onLoad: 'login-required' }).then((authenticated) =&gt; {
    if (!authenticated) {
      keycloak.login()
    } else {
      nuxtApp.provide('keycloak', keycloak)
    }
  })

  return {
    provide: {
      keycloak
    }
  }
})
```
### 4. 클라이언트와 서버 간의 통신

클라이언트(예: Nuxt 3 애플리케이션)와 서버(Spring Boot 애플리케이션) 간의 통신에서 JWT를 사용하여 사용자의 권한을 관리합니다.

#### 4.1 Keycloak에서 사용자 인증 및 토큰 발급

사용자가 Keycloak을 통해 로그인하면, Keycloak은 사용자에게 액세스 토큰과 리프레시 토큰을 발급합니다. 이 토큰은 사용자의 인증 상태와 권한 정보를 포함합니다.

#### 4.2. 클라이언트에서 로그인

클라이언트 애플리케이션에서 사용자가 Keycloak을 통해 로그인합니다. 로그인 후, 클라이언트는 Keycloak으로부터 액세스 토큰을 받습니다.

#### 4.3. 액세스 토큰을 서버로 전달

클라이언트는 액세스 토큰을 HTTP 헤더에 포함하여 서버로 요청을 보냅니다. 예를 들어, 사용자가 특정 데이터셋에 접근하려고 할 때, 클라이언트는 다음과 같은 요청을 보냅니다:
```
GET /api/dataset
Authorization: Bearer <access_token>
```

### 5. 서버에서 토큰 검증 및 권한 확인

서버(Spring Boot 애플리케이션)는 클라이언트로부터 받은 액세스 토큰을 검증하고, 사용자의 권한을 확인합니다.

#### 5.1. 토큰 검증

서버는 Keycloak의 공개 키를 사용하여 액세스 토큰의 서명을 검증합니다. 이를 통해 토큰이 변조되지 않았음을 확인합니다.

#### 5.2. 권한 확인

서버는 액세스 토큰의 페이로드를 확인하여 사용자의 권한 정보를 추출합니다. 예를 들어, 토큰의 페이로드에 포함된 역할(role) 정보를 확인하여 사용자가 데이터셋에 접근할 수 있는지 확인합니다.

### 6. Google BigQuery API 호출

서버는 사용자의 권한을 확인한 후, Google BigQuery API를 호출하여 데이터셋에 접근합니다. 이 과정에서 서버는 Google Cloud와의 SSO를 통해 얻은 액세스 토큰을 사용합니다.

#### 6.1. Google Cloud 액세스 토큰 요청

서버는 Keycloak을 통해 Google Cloud 액세스 토큰을 요청합니다. 이를 위해 Keycloak의 토큰 엔드포인트에 요청을 보내어 Google Cloud 액세스 토큰을 받습니다.

```java
import org.springframework.web.client.RestTemplate;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.stereotype.Service;

@Service
public class GoogleTokenService {

    private final RestTemplate restTemplate = new RestTemplate();

    public String getGoogleAccessToken(String keycloakAccessToken) {
        String url = "https://your-keycloak-server/auth/realms/YOUR_REALM/protocol/openid-connect/token";
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + keycloakAccessToken);
        HttpEntity<String> entity = new HttpEntity<>(headers);

        ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.POST, entity, String.class);
        // Parse the response to extract the Google access token
        // ...
        return googleAccessToken;
    }
}
```
#### 6.2. Google BigQuery API 호출

서버는 Google Cloud 액세스 토큰을 사용하여 Google BigQuery API를 호출합니다.
```java
import com.google.cloud.bigquery.BigQuery;
import com.google.cloud.bigquery.BigQueryOptions;
import com.google.cloud.bigquery.Dataset;
import com.google.cloud.bigquery.DatasetId;
import org.springframework.stereotype.Service;

@Service
public class BigQueryService {

    private final BigQuery bigQuery;

    public BigQueryService(String googleAccessToken) {
        this.bigQuery = BigQueryOptions.newBuilder()
                .setCredentials(GoogleCredentials.create(new AccessToken(googleAccessToken, null)))
                .build()
                .getService();
    }

    public Dataset getDataset(String projectId, String datasetId) {
        DatasetId dataset = DatasetId.of(projectId, datasetId);
        return bigQuery.getDataset(dataset);
    }
}
```

#### 6.3. 클라이언트에 데이터셋 정보 전달

서버는 Google BigQuery API를 통해 얻은 데이터셋 정보를 클라이언트에 전달합니다.
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BigQueryController {

    @Autowired
    private GoogleTokenService googleTokenService;

    @GetMapping("/api/dataset")
    public Dataset getDataset(@RequestParam String projectId, @RequestParam String datasetId, @RequestHeader("Authorization") String authorizationHeader) {
        String keycloakAccessToken = authorizationHeader.replace("Bearer ", "");
        String googleAccessToken = googleTokenService.getGoogleAccessToken(keycloakAccessToken);

        BigQueryService bigQueryService = new BigQueryService(googleAccessToken);
        return bigQueryService.getDataset(projectId, datasetId);
    }
}
```

### 7. Nuxt3에서 데이터셋 접근
Nuxt 3 애플리케이션에서 Spring Boot 애플리케이션의 엔드포인트를 호출하여 BigQuery 데이터셋에 접근합니다.
``` vue
<template>
  <div>
    <h1>BigQuery Dataset</h1>
    <div v-if="dataset">
      <p>Dataset ID: {{ dataset.id }}</p>
      <p>Dataset Name: {{ dataset.friendlyName }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useNuxtApp } from '#app'

const dataset = ref(null)
const { $axios } = useNuxtApp()

onMounted(async () => {
  const response = await $axios.get('/api/dataset', {
    params: {
      projectId: 'your-project-id',
      datasetId: 'your-dataset-id'
    }
  })
  dataset.value = response.data
})
</script>
```