---
layout: post
title:  "spring boot 비밀번호 암호화하기"
summary: "비밀번호 암호화"
author: 2rohyun
date: '2021-04-15 12:42:23 +0900'
category: spring
thumbnail: /assets/img/posts/springboot.png
keywords: spring, spring boot, password, encrypt
permalink: /blog/spring-boot-비밀번호-암호화/
usemathjax: false
---
# spring boot 비밀번호 암호화하기

## 비밀번호를 암호화해야하는 이유

**단순 텍스트**로 DB에 사용자의 비밀번호를 저장할 경우, DB가 해킹당하는 순간 해커에게 사용자의 비밀번호를 그대로 넘겨주게 된다.

따라서 **단방향 해쉬 함수의 digest** 를 사용하여 비밀번호를 암호화한다.

**단방향 해시 함수**는 수학적인 연산을 통해 원본 메세지를 변환하여 암호화된 메세지인 다이제스트를 생성한다. 원본 메세지를 알면 암호화된 메세지를 구하기는 쉽지만 암호화된 원본 메세지를 구할 수 없어야 하며 이를 ‘**단방향성**’이라고 한다. **단방향 암호화**는 복호화할 수 없는 암호화 방법이다. 복호화란 문자열을 다시 원래 문자열로 돌려놓는 것을 의미한다.

#### 단방향 해쉬 함수의 digest 의 한계

하지만, 동일한 메시지가 언제나 동일한 다이제스트를 갖는다면, 공격자가 전처리(pre-computing)된 다이제스트를 가능한 한 많이 확보한 다음 이를 탈취한 다이제스트와 비교해 원본 메시지를 찾아내거나 동일한 효과의 메시지를 찾을 수 있다. 이와 같은 다이제스트 목록을 레인보우 테이블(rainbow table)이라 하고, 이와 같은 공격 방식을 **레인보우 공격(rainbow attack)**이라 한다. 게다가 다른 사용자의 패스워드가 같으면 다이제스트도 같으므로 한꺼번에 모두 정보가 탈취될 수 있다.

#### 단방향 해쉬 함수의 보완

**솔트(salt)**는 단방향 해시 함수에서 다이제스트를 생성할 때 추가되는 바이트 단위의 랜덤의 문자열이다. 그리고 이 원본 메시지에 문자열을 추가하여 다이제스트를 생성하는 것을 **솔팅(salting)**이라 한다. 이 방법을 사용하면, 공격자가 패스워드의 다이제스트를 알아내더라도 솔팅된 다이제스트를 대상으로 패스워드 일치 여부를 확인하기 어렵다. 또한 사용자 별로 다른 솔트를 사용한다면 동일한 패스워드를 사용하는 사용자의 다이제스트가 다르게 생성되어 인식 가능성 문제가 크게 개선된다.

## spring boot password encoding

#### build.gradle

```groovy
compile "org.springframework.boot:spring-boot-starter-security:2.3.9.RELEASE"
compile group: "org.springframework.security", name: "spring-security-core", version: '5.3.8.RELEASE'
compile group: "org.springframework.security", name: "spring-security-config", version: '5.3.8.RELEASE'
compile group: "org.springframework.security", name: "spring-security-web", version: '5.3.8.RELEASE'
compile group: "org.springframework.security", name: "spring-security-oauth2-jose", version: '5.3.8.RELEASE'
compile group: "org.springframework.security", name: "spring-security-oauth2-resource-server", version: '5.3.8.RELEASE'
```

#### password encoder

```java
@Configuration
@EnableWebSecurity
public class JavaConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .cors().disable()      // cors 비활성화
                .csrf().disable()      // csrf 비활성화
                .formLogin().disable() //기본 로그인 페이지 없애기
                .headers().frameOptions().disable();
    }

}
```

#### sign up

```java
// 1. request data 가져오기
// 2. user 생성
// 3. 생성된 데이터 -> UserApiResponse return
@Override
@Transactional
public Header<UserApiResponse> create(Header<UserApiRequest> request) {

    try {
        //1.
        UserApiRequest userApiRequest = request.getData();

        User checkAccount = userRepository.finByAccount(userApiRequest.getAccount());

        if (checkAccount != null) {
            return Header.ERROR("이미 존재하는 아이디 입니다.");
        }
        
        //비밀번호 암호화
        String encodePassword = passwordEncoder.encode(userApiRequest.getPassword());

        //2.
        User user = User.builder()
                .account(userApiRequest.getAccount())
                .password(encodePassword)
                .email(userApiRequest.getEmail())
                .nation(userApiRequest.getNation())
                .registeredAt(LocalDateTime.now())
                .status(UserStatus.REGISTERED)
                .authMethods(
                        userApiRequest.getAuthMethods()
                                .stream()
                                .map(String::valueOf)
                                .collect(Collectors.joining(","))
                )
                .authHistory(
                        userApiRequest.getAuthHistory()
                                .stream()
                                .map(String::valueOf)
                                .collect(Collectors.joining(","))
                )
                .phoneNumber(userApiRequest.getPhoneNumber())
                .university(userApiRequest.getUniversity())
                .lastLoginAt(userApiRequest.getLastLoginAt())
                .profileIcon(userApiRequest.getProfileIcon())
                .build();

        User newUser = userRepository.save(user);
        unactiveUserService.create();

        //3.
        return response(newUser);
    }catch (Exception e){
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        log.error(e.getMessage());
        return Header.ERROR("DB_ERROR");
    }
}
```

비밀번호 암호화 코드
`String encodePassword = passwordEncoder.encode(userApiRequest.getPassword());`

#### sign in

```java
public Header<UserApiResponse> signIn(Header<SignInApiRequest> request){
    try{
        SignInApiRequest userApiRequest = request.getData();

        User checkAccount = userRepository.finByAccount(userApiRequest.getAccount());

        //account check
        if (checkAccount == null) {
            return Header.ERROR("아이디가 틀렸습니다.");
        }

        //password check
        boolean check = passwordEncoder.matches(userApiRequest.getPassword(), checkAccount.getPassword());
        if(check){
            return Header.OK();
        }else{
            return Header.ERROR("비밀번호가 틀렸습니다.");
        }

    }catch (Exception e){
        log.error(e.getMessage());
        return Header.ERROR("DB_ERROR");
    }
}
```

비밀번호 매치 코드
`boolean check = passwordEncoder.matches(userApiRequest.getPassword(), checkAccount.getPassword());`

