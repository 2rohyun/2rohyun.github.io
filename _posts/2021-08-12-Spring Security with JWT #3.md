---
layout: post
title:  "Spring Security with JWT #3. Access token, Refresh token 구현"
summary: "Spring Security with JWT #3. Access token, Refresh token 구현"
author: 2rohyun
date: '2021-08-12 13:45:00 +0900'
category: spring
thumbnail: /assets/img/posts/jwt.png
keywords: spring security, jwt, access token, refresh token
permalink: /blog/spring-jwt-with-security-three/
usemathjax: false
---

## 0. 들어가기 앞서

본격적으로 Spring boot + Spring Security + JWT library 를 이용한 access token 과 refresh token 을 구현해보도록 한다.

## 1. 환경 세팅 및 유저 엔티티 생성

#### 프로젝트 구조

![projecttree](/assets/img/posts/projecttree.png){: width="40%" height="40%"}

#### build.gradle

```groovy
plugins {
    id 'org.springframework.boot' version '2.5.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'com.dohyun'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // https://mvnrepository.com/artifact/com.auth0/java-jwt
    implementation group: 'com.auth0', name: 'java-jwt', version: '3.18.1'
    // https://mvnrepository.com/artifact/org.json/json
    implementation group: 'org.json', name: 'json', version: '20210307'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}

test {
    useJUnitPlatform()
}
```

- 기본적으로 jpa, security, web, lombok 을 추가했고, mariadb 를 사용할 것이므로 mariadb-java-client 를 의존성으로 추가하였다. 또한 앞선 포스트에서 설명했듯, com.auth0 에서 제공해주는 jwt implementation 라이브러리를 추가하였고, HttpServletRequest 를 json 으로 받기 위해 json 라이브러리를 추가하였다.

#### AppUser.class

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class AppUser {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String username;

    private String password;

    @ManyToMany(fetch = FetchType.EAGER)
    private List<Role> roles = new ArrayList<>();

    public void changePassword(String password) {
        this.password = password;
    }

    public AppUser(Long id, String name, String username, String password, List<Role> roles) {
        this.id = id;
        this.name = name;
        this.username = username;
        this.password = password;
        this.roles = roles;
    }
}
```

#### Role.class

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    public Role(Long id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

- Spring security 에서 제공하는 UserDetails 타입의 User 클래스와 헷갈리게 하지 않기 위해 AppUser 클래스 명을 사용하였다. 
- 하나의 유저는 N개의 역할을 가질 수 있고, 하나의 권한은 N 개의 유저를 가질 수 있기 때문에 AppUser 와 Role 의 연관 관계는 @ManyToMany 로 하였다. @ManyToMany 로 연관관계를 매핑할 경우, JPA 에서는 내부적으로  app_user_roles 라는 조인 테이블을 생성해준다.
- 유저는 역할을 알 필요가 있고, 역할은 유저를 알 필요가 없기 때문에 단방향으로 연관관계를 매핑하였다.
- AppUser 객체를 불러올 때, roles 를 같이 불러오기 위해 FetchType.EAGER 를 사용하였다. 사실 예상치 못한 쿼리를 방지하기 위해 프로젝트 내 모든 FetchType 을 LAZY 로 하고 같이 불러와야 할 필요가 있는 객체만 fetch join, @EntityGraph 등 을 사용하여 불러오는 것이 더 나은 정답이지만, 간단한 예제이므로 EAGER 를 사용하였다.

#### AppUserRepository.class

```java
public interface AppUserRepository extends JpaRepository<AppUser, Long> {
    Optional<AppUser> findByUsername(String username);
}
```

#### RoleRepository.class

```java
public interface RoleRepository extends JpaRepository<Role, Long> {
    Optional<Role> findByName(String name);
}
```

## 2. 서비스 구현 및 샘플 데이터 삽입

#### AppUserService.class

```java
public interface AppUserService {

    AppUser saveAppUser(AppUser appUser);
    Role saveRole(Role role);
    void addRoleToAppUser(String username, String roleName);
    AppUser getUser(String username);
    List<AppUser> getUsers();
}
```

#### AppUserServiceImpl.class

```java
@Service
@RequiredArgsConstructor
@Slf4j
@Transactional
public class AppUserServiceImpl implements AppUserService {

    private final AppUserRepository appUserRepository;
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Override
    public AppUser saveAppUser(AppUser appUser) {
        appUser.changePassword(passwordEncoder.encode(appUser.getPassword()));
        return appUserRepository.save(appUser);
    }

    @Override
    public Role saveRole(Role role) { return roleRepository.save(role); }

    @Override
    public void addRoleToAppUser(String username, String roleName) {
        AppUser appUser = appUserRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("No such username in DB"));
        Role role = roleRepository.findByName(roleName)
                .orElseThrow(() -> new NoSuchElementException("No such role name in DB"));
        appUser.getRoles().add(role);
    }

    @Override
    public AppUser getUser(String username) {
        return appUserRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("No such username in DB"));
    }

    @Override
    public List<AppUser> getUsers() { return appUserRepository.findAll(); }
}
```

- 기본적인 저장, 조회 기능을 추가하였다.

#### AmigoscodeJwtApplication.class

```java
@SpringBootApplication
public class AmigoscodeJwtApplication {

    public static void main(String[] args) {
        SpringApplication.run(AmigoscodeJwtApplication.class, args);
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    CommandLineRunner run(AppUserService appUserService) {
        return args -> {
            appUserService.saveRole(new Role(null,"ROLE_USER"));
            appUserService.saveRole(new Role(null,"ROLE_MANAGER"));
            appUserService.saveRole(new Role(null,"ROLE_ADMIN"));
            appUserService.saveRole(new Role(null,"ROLE_SUPER_ADMIN"));

            appUserService.saveAppUser(new AppUser(null, "이도현", "도팔","1234", new ArrayList<>()));
            appUserService.saveAppUser(new AppUser(null, "김철수", "김마스","1234", new ArrayList<>()));
            appUserService.saveAppUser(new AppUser(null, "이영희", "이마스","1234", new ArrayList<>()));
            appUserService.saveAppUser(new AppUser(null, "홍길동", "홍마스","1234", new ArrayList<>()));

            appUserService.addRoleToAppUser("도팔", "ROLE_USER");
            appUserService.addRoleToAppUser("도팔", "ROLE_ADMIN");
            appUserService.addRoleToAppUser("도팔", "ROLE_SUPER_ADMIN");
            appUserService.addRoleToAppUser("김마스", "ROLE_ADMIN");
            appUserService.addRoleToAppUser("이마스", "ROLE_MANAGER");
            appUserService.addRoleToAppUser("이마스", "ROLE_USER");
            appUserService.addRoleToAppUser("홍마스", "ROLE_USER");

        };
    }
}
```

- PasswordEncoder 로 BCryptPasswordEncoder 를 사용하기 위해 빈으로 등록하였다.
- 애플리케이션이 실행되고 나서 DB 에 AppUser 와 Role 에 대한 샘플 데이터를 삽입하기 위해 CommandLineRunner 와 익명 클래스를 이용하여 초기화 코드를 넣어주었다.

## 3. Spring Security 환경 설정 및 Access, Refresh token 생성

#### SecurityConfig.class

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CustomAuthenticationFilter customAuthenticationFilter = new CustomAuthenticationFilter(authenticationManagerBean());
        customAuthenticationFilter.setFilterProcessesUrl("/api/login");

        http
                .csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                .antMatchers("/api/login/**", "/api/token/refresh/**").permitAll()
                .antMatchers(HttpMethod.GET, "/api/user/**").hasAnyAuthority("ROLE_USER")
                .antMatchers(HttpMethod.POST, "/api/user/save/**").hasAnyAuthority("ROLE_ADMIN")
                .anyRequest().authenticated()
                .and()
                .addFilter(customAuthenticationFilter)
                .addFilterBefore(new CustomAuthorizationFilter(), UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

- 유저 정보를 UserDetails 타입으로 가져오기 위해 AppUserService 에 UserDetailsService 를 상속 받아 loadUserByUsername() 메소드를 구현한다.

```java
public class AppUserServiceImpl implements AppUserService, UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        AppUser appUser = appUserRepository.findByUsername(username).orElseThrow(() -> new UsernameNotFoundException("No such username in DB"));
        Collection<SimpleGrantedAuthority> authorities = new ArrayList<>();
        appUser.getRoles().forEach(role -> {
            authorities.add(new SimpleGrantedAuthority(role.getName()));
        });
        return new User(appUser.getUsername(),appUser.getPassword(), authorities);
    }
```

- 요청이 dispatcherServlet 에 도달하기 전에 SecuriyFilterChain 의 UsernamePasswordAuthenticationFilter 를 거치며 로그인 성공 여부를 확인하고, access, refresh token 을 발급하기 위해 UsernamePasswordAuthenticationFilter 를 상속받은 CustomAuthenticationFilter 를 생성하고, attemptAuthentication() 메소드와 successfulAuthentication() 메소드를 Override 한다.

#### CustomAuthenticationFilter.class

```java
@Slf4j
public class CustomAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    
    public CustomAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }
}
```

- 인증을 수행하기 위해 AuthenticationManager 를 의존관계로 설정한다.

```java
    @SneakyThrows
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        JSONObject body = RequestToJsonUtil.readJsonFromRequestBody(request);
        String username = body.getString("username");
        String password = body.getString("password");

        log.info("[attemptAuthentication] username is : {}, password is : {}", username, password);

        try {
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username,password);
            return authenticationManager.authenticate(authenticationToken);
        } catch (Exception e) {
            log.error("[attemptAuthentication] Login failed : {}", e.getMessage());
            response.setStatus(HttpStatus.FORBIDDEN.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);

            Map<String, String> error = new HashMap<>();
            error.put("error_message",e.getMessage());
            new ObjectMapper().writeValue(response.getOutputStream(), error);

            return null;
        }
    }
```

#### RequestToJsonUtil.class

```java
public class RequestToJsonUtil {
    public static JSONObject readJsonFromRequestBody(HttpServletRequest request){
        StringBuilder json = new StringBuilder();
        String line = null;
        try {
            BufferedReader reader = request.getReader();
            while((line = reader.readLine()) != null) {
                json.append(line);
            }

        }catch(Exception e) {
            System.out.println("Error reading JSON string: " + e.toString());
        }
        return new JSONObject(json.toString());
    }
}
```

- 요청받은 body 안의 username, password 를 JSONObject 로 변환하여 username 과 password 를 얻고, 얻은 username 과 password로 UsernamePasswordAuthenticationToken 을 생성하여 AuthenticationManager 에게 전달하여 authentication 을 수행한다.
- 인증에 실패한 경우, HTTP status code 403 과 에러 메세지를 리턴한다.

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

        log.info("[successAuthentication] login successfully done");

        Map<String, String> tokens = new HashMap<>();
        tokens.put("access_token",accessToken);
        tokens.put("refresh_token",refreshToken);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        new ObjectMapper().writeValue(response.getOutputStream(), tokens);
    }
```

- 인증에 성공한 경우, successfulAuthentication() 을 수행한다.

1. Authentication 객체의 principal 을 가져와 User 타입으로 캐스팅한다.
2. Signature 를 위한 Secret key 로 "secret" 의 바이트를 HMAC 256 방식으로 encrypt 한다. ( 실제로 사용할 경우, Secret key 를 안전하게 보관해야 한다. )
3. Access Token 의 만료 시간을 10분으로 설정하여 생성한다.
4. Refresh Token 의 만료 시간을 1주일로 설정하여 생성한다. ( Refresh Token 의 경우 Access Token 을 재발급하기 위한 토큰이므로 claim 이 필요하지 않다. )
5. 생성한 Access Token 과 Refresh Token 을 JSON 으로 리턴해준다.

## 4. 요청 받은 토큰 검증

- 요청 받은 request 의 token 을 검증하기 위해 OncePerRequestFilter 를 상속 받은 CustomAuthorizationFilter 를 생성하고, UsernamePasswordAuthenticationFilter 전에 추가한다. 

> OncePerRequestFilter 를 사용하는 이유? Spring Security에서 인증과 접근 제어 기능이 Filter로 구현되어진다. 이러한 인증과 접근 제어는 RequestDispatcher 클래스에 의해 다른 서블릿으로 dispatch되게 되는데, 이 때 이동할 서블릿에 도착하기 전에 다시 한번 filter chain을 거치게 된다. 바로 이 때 또 다른 서블릿이 우리가 정의해둔 필터가 Filter나 GenericFilterBean로 구현된 filter를 또 타면서 필터가 두 번 실행되는 현상이 발생할 수 있다. 이런 문제를 해결하기 위해 OncePerRequestFilter 를 사용하여 한 번의 요청 당 딱 한번만 실행되는 필터를 만들 수 있다.

#### CustomAuthorizationFilter.class

```java
@Slf4j
public class CustomAuthorizationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (request.getServletPath().equals("/api/login") || request.getServletPath().equals( "/api/token/refresh")) {
            filterChain.doFilter(request,response);
        } else {
            String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);
            if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
                try {
                    String token = authorizationHeader.substring("Bearer ".length());

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
}
```

1. 요청이 로그인이나 토큰 재발급을 위한 것이라면 filterChain 의 다음 filter 를 태운다.
2. 로그인이나 토큰 재발급 요청 외에 다른 요청이라면 HTTP message Header, Authorization 필드의 Bearer token 을 가져온다.
3. 토큰을 JWTVerifier 와 Secret key 를 이용해 decode 하고, username 과 roles 를 얻는다.
4. 얻은 username 과 roles 를 이용하여 UsernamePasswordAuthenticationToken 을 생성하여 SecurityContext 에 넣는다.
5. 토큰을 검증하는 데에 실패하면 Http Status code 403 과 에러 메세지를 리턴한다.

## 5. Refresh Token 을 이용한 Access Token 재발급

- 만료된 Access Token 으로 요청한 경우, 토큰이 만료됨을 응답해주고, 프런트에서는 Refresh Token 으로 Access Token 재발급을 요청한다. 요청에 따라 서버에서는 Access Token 을 재발급해준다.

#### AppUserController.class

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

                String accessToken = JWT.create()
                        .withSubject(user.getUsername())
                        .withExpiresAt(new Date(System.currentTimeMillis() + 10 * 60 * 1000))
                        .withIssuer(request.getRequestURL().toString())
                        .withClaim("roles",user.getRoles().stream().map(Role::getName).collect(Collectors.toList()))
                        .sign(algorithm);

                Map<String, String> tokens = new HashMap<>();
                tokens.put("access_token",accessToken);
                tokens.put("refresh_token",refreshToken);
                response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                new ObjectMapper().writeValue(response.getOutputStream(), tokens);
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

- CustomAuthorizationFilter 와 비슷한 구조를 지닌다. "/token/refresh" 로 요청이 들어올 경우, Refresh Token 을 검증한 뒤, username 과 roles 를 이용하여 새로운 Access Token 을 생성한 뒤, 발급해준다.

## 6. 정리

- 사용자의 로그인 요청에 따른 로그인 성공 여부를 확인하였고, 로그인에 성공한 경우 Access Token 과 Refresh Token 을 발급해주었다. 
- 로그인 요청 및 Access Token 재발급 요청을 제외한 모든 요청은 HTTP header 의 토큰을 검증하고, username 과 roles 을 이용하여 UsernamePasswordAuthenticationToken 을 생성하고 SecurityContext 에 넣어두었다.
- "/token/refresh" 로 요청이 들어올 경우, Refresh Token 을 검증한 뒤, username 과 roles 를 이용하여 새로운 Access Token 을 생성한 뒤, 발급해주었다.

- 사용자가 로그아웃 요청을 한 경우, 사용자의 Access Token 과 Refresh Token 을 invalidating 해야한다. 따라서 다음 포스트에서는 Access Token 과 Refresh Token 을 invalidating 하는 방법, Refresh Token 관리 방안과 Redis 에 대해서 다루도록 한다.

### REFERENCE

https://velog.io/@ehdrms2034/Spring-Security-JWT-Redis%EB%A5%BC-%ED%86%B5%ED%95%9C-%ED%9A%8C%EC%9B%90%EC%9D%B8%EC%A6%9D%ED%97%88%EA%B0%80-%EA%B5%AC%ED%98%84
https://www.youtube.com/watch?v=VVn9OG9nfH0&t=3s
https://velog.io/@tlatldms/%EC%84%9C%EB%B2%84%EA%B0%9C%EB%B0%9C%EC%BA%A0%ED%94%84-Refresh-JWT-%EA%B5%AC%ED%98%84
https://minkukjo.github.io/framework/2020/12/18/Spring-142/

  
