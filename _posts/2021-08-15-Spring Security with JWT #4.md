---
layout: post
title:  "Spring Security with JWT #4. Refresh token 관리 방안과 로그아웃"
summary: "Spring Security with JWT #4. Refresh token 관리 방안과 로그아웃"
author: 2rohyun
date: '2021-08-15 18:45:00 +0900'
category: spring-security
thumbnail: /assets/img/posts/jwt.png
keywords: spring security, jwt, refresh token, logout
permalink: /blog/spring-jwt-with-security-four/
usemathjax: false
---

## 1. 의사결정

저번 포스트에서까지 Access token 과 Refresh token 의 구현에 대해서 알아 보았다. 이번 포스트에서는 Refresh token 에 대한 보안성 강화 전략과 로그아웃 시 서버에서 처리해주어야 할 것들에 대해 알아보려고 한다.

#### Refresh token 관리 방안 및 보안성 강화 전략

Refresh token 은 Access token 의 만료 기간을 짧게 두고, Access token 이 만료될 때 마다 초기 로그인 시 발급한 Refresh token 으로 Access token 을 재발급 받아, Access token 이 탈취당했을 때를 보완하며, 또한 로그인 유지의 목적을 가진다. 하지만 Refresh token 이 탈취된다면? 탈취한 사용자는 Refresh token 을 이용해 Access token 을 계속해서 발급 받아 악의적 접근을 하게된다. 따라서 Refresh token 에 대한 관리 방안이 중요하다고 여겨지는데, 찾아보니 "Refresh token 을 발급 받은 클라이언트 사이드에서 안전하게 보관해야 한다." 라는 글이 대부분이였다. 나에게 충분한 해답이 되지 못했고, 서버를 공부하고 있으므로 서버에서 Refresh token 에 대한 보안성을 높일 수 있는 전략이 무엇이 있을까? 라고 고민하던 중, auth0.com 의 Refresh token 관리 전략을 찾게 되었다.

```
Refresh Token Automatic Reuse Detection

Refresh tokens are bearer tokens. It's impossible for the authorization server to know who is legitimate or malicious when receiving a new access token request. We could then treat all users as potentially malicious.

How could we handle a situation where there is a race condition between a legitimate user and a malicious one? For example:

🐱 Legitimate User has 🔄 Refresh Token 1 and 🔑 Access Token 1.

😈 Malicious User manages to steal 🔄 Refresh Token 1 from 🐱 Legitimate User.

🐱 Legitimate User uses 🔄 Refresh Token 1 to get a new refresh-access token pair.

The 🚓 Auth0 Authorization Server returns 🔄 Refresh Token 2 and 🔑 Access Token 2 to 🐱 Legitimate User.

😈 Malicious User then attempts to use 🔄 Refresh Token 1 to get a new access token. Pure evil!

What do you think should happen next? Would 😈 Malicious User manage to get a new access token?

This is what happens when your identity platform has 🤖 Automatic Reuse Detection:

The 🚓 Auth0 Authorization Server has been keeping track of all the refresh tokens descending from the original refresh token. That is, it has created a "token family".

The 🚓 Auth0 Authorization Server recognizes that someone is reusing 🔄 Refresh Token 1 and immediately invalidates the refresh token family, including 🔄 Refresh Token 2.

The 🚓 Auth0 Authorization Server returns an Access Denied response to 😈 Malicious User.

🔑 Access Token 2 expires, and 🐱 Legitimate User attempts to use 🔄 Refresh Token 2 to request a new refresh-access token pair.

The 🚓 Auth0 Authorization Server returns an Access Denied response to 🐱 Legitimate User.

The 🚓 Auth0 Authorization Server requires re-authentication to get new access and refresh tokens.

It's critical for the most recently-issued refresh token to get immediately invalidated when a previously-used refresh token is sent to the authorization server. This prevents any refresh tokens in the same token family from being used to get new access tokens.

This protection mechanism works regardless of whether the legitimate or malicious user is able to exchange 🔄 Refresh Token 1 for a new refresh-access token pair before the other. Without enforcing sender-constraint, the authorization server can't know which actor is legitimate or malicious in the event of a replay attack.

Automatic reuse detection is a key component of a refresh token rotation strategy. The server has already invalidated the refresh token that has already been used. However, since the authorization server has no way of knowing if the legitimate user is holding the most current refresh token, it invalidates the whole token family just to be safe.
```

요약하자면,

1. 합법적인 사용자는 인증 시, Access Token 1 과 Refresh Token 1 을 발급 받는다.
2. 악의적인 사용자는 합법적인 사용자로 부터 Refresh Token 1 을 탈취한다.
3. 합법적인 사용자는 Refresh Token 1 을 사용하여 새로운 Access Token 2 와 Refresh Token 2 를 발급 받는다.
4. 인증 서버에서는 발급한 모든 Refresh Token 을 Token Family 로 저장해둔다.
5. 악의적인 사용자는 인증 서버에 탈취한 Refresh Token 1 로 접근을 요청한다.
6. 인증 서버에서는 Refresh Token 재사용을 탐지하여 Refresh Token 2 를 포함한 Token Family 의 모든 Refesh Token 을 Invalidating 한다.

위의 시나리오 대로 동작하게 되는데, 서버 측에서 관리할 수 있는 좋은 전략이라고 생각되어 위와 같이 구현하려고 하였다.

#### 로그아웃 전략

JWT 로 인증과 권한을 관리하는 방식의 특성 상 STATELESS 하기 때문에 서버에서는 요청 받은 token 이 로그아웃한 사용자의 token 인지 구분해낼 수 있는 방법이 없다고 생각했다. 따라서 JWT 의 로그아웃 전략에 대해 찾아보니 로그아웃 즉시 클라이언트 사이드에서 Access Token 과 Refresh Token 을 지운다고 한다. 하지만 실제 서비스에서는 로그아웃 요청이 들어왔을 때, 해당 Access Token 에 대한 blacklist 를 만들어 요청 받은 token 이 로그아웃한 사용자의 것인지 체크하는 전략을 사용해야 한다고 한다. 따라서 나는 로그아웃 요청을 받았을 때, 해당 Access Token 을 blacklist 에 만료 기간을 주어 올려두고, 해당 사용자의 Refresh Token Family 를 지우는 방식으로 구현하려고 하였다.

#### 그렇다면 어디에 저장할 것인가?

결론부터 말하자면 Redis 를 채택하기로 하였다. 이유는,

1. 단순한 구조의 데이터 모델인 Key-Value 방식을 통한 빠른 속도와 데이터 저장소로 입력/출력이 가장 빠른 메모리를 채택하여 속도가 매우 빠르다.
2. Inmemory Cache Solution 이지만, 디스크에 데이터를 기록하고 있기 때문에 영속성을 가져 메모리가 날아가도 데이터를 복구할 수 있다.
3. List, Set, SortedSet 과 같은 Collection 을 지원한다.
4. 만료 시간 ( Expired time ) 을 줄 수 있어 데이터를 휘발성으로 사용할 수 있다.

위와 같은 특징으로 보아, Refresh Token 및 Access Token 을 저장하는 데에 최적의 특징을 지녔다고 생각하였다.

## 2. Redis 를 이용한 Refresh Token 관리 전략 및 로그아웃 구현

- Redis 의 Java Client 에는 Jedis, Lettuce 2가지가 존재한다. [jojoldu 님 블로그](https://jojoldu.tistory.com/418) 에서 Jedis 와 Lettuce 에 대해 성능적 관점으로 잘 비교를 해놓으신 포스트를 참고하여 비동기로 요청을 처리하여 고성능을 자랑하는 Lettuce 를 사용하기로 하였다.

#### build.gradle

```groovy
    // https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis
    implementation group: 'org.springframework.data', name: 'spring-data-redis', version: '2.5.2'
    // https://mvnrepository.com/artifact/io.lettuce/lettuce-core
    implementation group: 'io.lettuce', name: 'lettuce-core', version: '6.1.4.RELEASE'
```

- spring data 에서 제공해주는 spring-data-redis 와 lettuce 를 dependency 로 추가하였다.

#### RedisConfig.class

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<byte[], byte[]> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

```yml
spring:
  cache:
    type: redis
  redis:
    host: localhost
    port: 6379
```

- localhost 의 Redis 와 연결하는 환경을 설정하기 위해 RedisConfig 클래스를 만들었고, LettuceConnectionFactory 를 이용하여 연결하였다.

#### RedisUtils.class

```java
@Component
public class RedisUtils {

    private final RedisTemplate<String, String> redisTemplate;

    public RedisUtils(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public String getData(String key){
        ValueOperations<String,String> valueOperations = redisTemplate.opsForValue();
        return valueOperations.get(key);
    }

    public void setData(String key, String value){
        ValueOperations<String,String> valueOperations = redisTemplate.opsForValue();
        valueOperations.set(key,value);
    }

    public void setDataExpire(String key,String value,long duration){
        ValueOperations<String,String> valueOperations = redisTemplate.opsForValue();
        Duration expireDuration = Duration.ofSeconds(duration);
        valueOperations.set(key,value,expireDuration);
    }

    public void addDataFromList(String key, String value) {
        ListOperations<String, String> listOperations = redisTemplate.opsForList();
        listOperations.leftPush(key,value);
    }

    public String getRecentDataFromList(String key) {
        ListOperations<String, String> listOperations = redisTemplate.opsForList();
        return listOperations.index(key,0);
    }

    public Long getIndexOfList(String key, String value) {
        ListOperations<String, String> listOperations = redisTemplate.opsForList();
        return listOperations.indexOf(key, value);
    }

    public void deleteData(String key){
        redisTemplate.delete(key);
    }
}
```

- Redis 에 String 타입으로 만료 기간 없이 key, value 를 저장하고 가져오는 메소드와, 만료 기간을 설정하여 데이터를 저장하는 메소드, 그리고 하나의 key 에 여러 개의 value 를 리스트로 저장하는 메소드와 데이터를 삭제하는 메소드를 편하게 사용하기 위해 RedisUtils 클래스를 구현하였다.

### Refresh Token 관리 구현

#### CustomAuthenticationFilter 의 SuccessfulAuthentication() 메소드

```java
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) throws IOException, ServletException {
        User user = (User) authentication.getPrincipal();
        Algorithm algorithm = Algorithm.HMAC256("secret".getBytes());

        String accessToken = JWT.create()
                .withSubject(user.getUsername())
                .withExpiresAt(new Date(System.currentTimeMillis() + 10 * 60 * 1000))
                .withIssuer(request.getRequestURL().toString())
                .withClaim("roles",user.getAuthorities().stream().map(GrantedAuthority::getAuthority).collect(Collectors.toList()))
                .sign(algorithm);

        String refreshToken = JWT.create()
                .withSubject(user.getUsername())
                .withExpiresAt(new Date(System.currentTimeMillis() + 10080 * 60 * 1000))
                .withIssuer(request.getRequestURL().toString())
                .sign(algorithm);

        // Refresh token 발급 시 레디스에 { username : token } 으로 저장
        redisUtils.addDataFromList(user.getUsername(), refreshToken);

        log.info("[successAuthentication] login successfully done");

        Map<String, String> tokens = new HashMap<>();
        tokens.put("access_token",accessToken);
        tokens.put("refresh_token",refreshToken);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        new ObjectMapper().writeValue(response.getOutputStream(), tokens);
    }
```

- 기존의 코드에 Refresh token 생성 시 Redis 에 { key:username, value:[refresh token] } 형식으로 저장하였다. 

#### AppUserController 의 refreshToken() 메소드

```java
    @GetMapping("/token/refresh")
    public void refreshToken(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            try {
                String refreshToken = authorizationHeader.substring("Bearer ".length());
                Algorithm algorithm = Algorithm.HMAC256("secret".getBytes());
                JWTVerifier verifier = JWT.require(algorithm).build();
                DecodedJWT decodedJWT = verifier.verify(refreshToken);
                String username = decodedJWT.getSubject();
                AppUser user = appUserService.getUser(username);

                // 해당 유저의 예전 refresh token 으로 접근한 경우 모든 refresh token 을 지운다.
                if (redisUtils.getIndexOfList(user.getUsername(), refreshToken) != null && !redisUtils.getRecentDataFromList(user.getUsername()).equals(refreshToken)) {
                    redisUtils.deleteData(user.getUsername());
                    throw new Exception("Invalid Refresh token access, Please Login again!");
                }
                // 레디스에 refresh token 이 없는 경우, refresh token 이 아니거나 이미 로그아웃한 유저이다.
                else if (redisUtils.getIndexOfList(user.getUsername(), refreshToken) == null) {
                    throw new Exception("This is not refresh token or refresh token doesn't exist!");
                }
                else {
                    String accessToken = JWT.create()
                            .withSubject(user.getUsername())
                            .withExpiresAt(new Date(System.currentTimeMillis() + 10 * 60 * 1000))
                            .withIssuer(request.getRequestURL().toString())
                            .withClaim("roles",user.getRoles().stream().map(Role::getName).collect(Collectors.toList()))
                            .sign(algorithm);

                    String newRefreshToken = JWT.create()
                            .withSubject(user.getUsername())
                            .withExpiresAt(new Date(System.currentTimeMillis() + 10080 * 60 * 1000))
                            .withIssuer(request.getRequestURL().toString())
                            .withClaim("roles", user.getRoles().stream().map(Role::getName).collect(Collectors.toList()))
                            .sign(algorithm);

                    redisUtils.addDataFromList(user.getUsername(),newRefreshToken);

                    Map<String, String> tokens = new HashMap<>();
                    tokens.put("access_token",accessToken);
                    tokens.put("refresh_token",newRefreshToken);
                    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                    new ObjectMapper().writeValue(response.getOutputStream(), tokens);
                }
            } catch (Exception e) {
                Map<String, String> error = new HashMap<>();
                error.put("error_message",e.getMessage());

                response.setStatus(HttpStatus.FORBIDDEN.value());
                response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                new ObjectMapper().writeValue(response.getOutputStream(), error);
            }
        } else {
            throw new RuntimeException("Refresh token is missing");
        }
    }
```

- 사용자가 Refresh Token 을 이용해 Access Token 재발급을 요청했을 떄, 사용자의 예전 Refresh Token 으로 접근한 경우 해당 사용자의 모든 Refresh Token 을 삭제하여 재인증을 유도한다.
- Refresh Token 이 Redis 에 존재하지 않으면, Refresh Token 이 아니거나 이미 로그아웃하여 Refresh Token 을 지운 상태이다.

### 로그아웃 구현

#### AppUserController 의 logout() 메소드

```java
    @GetMapping("/logout")
    public void logout(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            try {
                String accessToken = authorizationHeader.substring("Bearer ".length());
                Algorithm algorithm = Algorithm.HMAC256("secret".getBytes());
                JWTVerifier verifier = JWT.require(algorithm).build();
                DecodedJWT decodedJWT = verifier.verify(accessToken);
                String username = decodedJWT.getSubject();

                // redis blacklist 에 해당 유저의 access 토큰 추가
                redisUtils.setDataExpire(accessToken, "blacklist", 10 * 60);

                // refresh token 삭제
                if (redisUtils.getRecentDataFromList(username)!= null) {
                    redisUtils.deleteData(username);
                }
            } catch (Exception e) {
                Map<String, String> error = new HashMap<>();
                error.put("error_message", e.getMessage());

                response.setStatus(HttpStatus.FORBIDDEN.value());
                response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                new ObjectMapper().writeValue(response.getOutputStream(), error);
            }
        } else {
                throw new RuntimeException("Access token is missing");
            }
    }
```

- 사용자의 로그아웃 요청 시, Access Token 을 {key:access token, value:blacklist} 로 저장하여 두고, 해당 username 을 key 로 가지는 Refresh Token Family 를 Redis 에서 삭제한다.

#### CustomAuthorizationFilter 의 doFilterInternal() 메소드

```java
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (request.getServletPath().equals("/api/login") || request.getServletPath().equals( "/api/token/refresh")) {
            filterChain.doFilter(request,response);
        } else {
            String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);
            if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
                try {
                    String token = authorizationHeader.substring("Bearer ".length());

                    // 로그아웃한 유저인지 check
                    if (redisUtils.getData(token) != null){
                        throw new Exception("User already logged out!");
                    }
                    else {
                        Algorithm algorithm = Algorithm.HMAC256("secret".getBytes());
                        JWTVerifier verifier = JWT.require(algorithm).build();
                        DecodedJWT decodedJWT = verifier.verify(token);
                        String username = decodedJWT.getSubject();
                        String[] roles = decodedJWT.getClaim("roles").asArray(String.class);
                        Collection<SimpleGrantedAuthority> authorities = new ArrayList<>();
                        Arrays.stream(roles).forEach(role -> {
                            authorities.add(new SimpleGrantedAuthority(role));
                        });
                        UsernamePasswordAuthenticationToken authenticationToken
                                = new UsernamePasswordAuthenticationToken(username,null,authorities);
                        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                        filterChain.doFilter(request,response);
                    }
                } catch (Exception e) {
                    log.error("[doFilterInternal] Error authorizing token : {}", e.getMessage());
                    Map<String, String> error = new HashMap<>();
                    error.put("error_message",e.getMessage());

                    response.setStatus(HttpStatus.FORBIDDEN.value());
                    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                    new ObjectMapper().writeValue(response.getOutputStream(), error);
                }
            } else {
                filterChain.doFilter(request,response);
            }
        }
    }
```

- 로그인 요청과 Refresh Token 을 이용한 Access Token 재발급 요청을 제외한 모든 요청에 담겨져 오는 Access Token 을 검사하여 Redis 에 존재한다면 로그아웃한 사용자의 Access Token 이므로 403 에러와 에러 메세지를 리턴한다.

## 3. 마치며

이번 포스트를 작성하기 전에는 단순하게 Access Token, Refresh Token 을 발급해주고 각자의 역할에 맞게 사용하면 그만인 단순하고 쉬운 구현이라고 생각했다. 하지만 구현을 실제로 진행하며 보안을 높이기 위해 다양한 관리 전략 및 시나리오가 존재하고 정석적으로 존재하는 방법이 없다는 것을 알았다. 또한 구현을 하며 생각보다 많은 조건을 생각하며 모두 분기처리 해주어야 하고, 성능적인 관점에서도 바라보아야 했다. 역시 이번에도 "직접 해보기 전에는 쉬워 보인다, 깊게 알면 알수록 디테일이 중요하다." 라는 진리를 체감하게 되었다. 구현을 해보며 아쉬웠던 점은, 첫 번째로, Refresh Token 으로 Token Family List 를 저장하는 것이다. 사용자의 최신 Refresh Token 이 아닌 예전의 Refresh Token 으로 접근한 경우, 해당 사용자의 Token Family 를 모두 삭제하고 에러 메세지를 날리고, Refresh Token 이 없는 경우 에러 메세지만 날린다. 결국 두 가지 경우 모두 재인증을 하여 Access Token 과 Refresh Token 을 다시 발급 받아야 하는데 과연 유의미한 분기 처리일까? 에 대한 문제이다. 결국 모두 재인증을 해야한다면, 새로운 Access Token 및 Refresh Token 을 발급해줄 때 기존의 Refresh Token 을 Redis 에서 삭제하고 새로 발급해준 Refresh Token 을 저장한다면, Token Family 를 List 로 가지고 있지 않아도 되므로 메모리 및 성능적으로 더 좋은 퍼포먼스를 발휘할 수 있지 않을까? 라는 생각이다. 그리고 두 번째로, 로그인 요청 및 Refresh Token 을 이용한 Access Token 재발급 요청을 제외한 모든 요청에 대해 토큰의 blacklist 여부를 검사하는 로직이 마음에 걸린다. 물론 Redis 는 매우 빠르고, 가벼운 데다 {key:value} 쌍으로 데이터가 저장되기 때문에 O(1) 의 시간 복잡도를 가져 트랜잭션을 길게 물고 있을 경우가 최소 blacklist 검사에서는 없다는 것을 알지만 그래도! 더 좋은 방법이 존재하지 않을까? 에 대한 아쉬움이 남는다. 이번 포스트에서 그치지 않고 더욱 깊게 찾아보고 알아볼 생각이다.

 ### REFERENCE

https://jojoldu.tistory.com/418

https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/#Refresh-Token-Rotation

https://sabarada.tistory.com/104

https://goodgid.github.io/Redis/

https://stackoverflow.com/questions/21978658/invalidating-json-web-tokens
  
