---
layout: post
title: "회원 - 회원관리/회원가입"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day10 (회원 관리/가입)
---
## 회원

로그인 서비스를 위하여 `User`를 만들어 주었다.

```java
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue
    private Long id;

    @NotEmpty
    @Setter
    private String email;

    @NotEmpty
    @Setter
    private String name;

    @NotNull
    @Setter
    private Long level;

    private String password;

    public boolean isAdmin() {
        return level >= 100;
    }

    public boolean isActive() {
        return level > 0;
    }
    public void deactive(){
        level = 0L;
    }
}
```

현재는 최소한의 데이터만 갖도록 하였다.  
level이 0:비활성화 1:customer 2:restaurant owner 3:admin
 

admin-api에서는 user에 대한 CRUD를 만들었다.
customer-api에서는 user에 대한 create를 만들었다.

spring security사용을 사용하기 위해 `build.gradle`에 security를 추가해 주었다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-security'
```

추가해주고 실행하면 spring security에서 제공하는 로그인폼이 뜬다.
![security_login](../../images/20200317/security_login.PNG){:width="300"}

해당프로젝트에서는 REST api를 제공하는 것 이므로 폼을 disable 해주겠다.

그리고 h2-console에 접근하면 아이프레임이 보이지 않는다.
![iframe1](../../images/20200317/iframe.PNG){:width="500"}

헤더의 프레임옵션을 disable해준다. 그럼 정상적으로 h2 console이 보인다.
![iframe1](../../images/20200317/frame_disable.PNG){:width="500"}

```java
@Configuration
@EnableWebSecurity
public class SecurityJavaConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .cors().disable()
                .csrf().disable()
                .formLogin().disable()
                .headers().frameOptions().disable();
    }

}
```

회원가입 시 이메일 중복체크와 password에 대한 hashing을 해주었다.
```java
    public User registerUser(String email, String name, String password){
        Optional<User> existed = userRepository.findByEmail(email);
        if(existed.isPresent()){
            throw new EmailExistedException(email);
        }

        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encodedPassword = passwordEncoder.encode(password);

        User user = User.builder()
                .email(email)
                .name(name)
                .password(encodedPassword)
                .level(1L)
                .build();

        return userRepository.save(user);
    }
```

이제 회원을 추가해 주었고, 인가와 권한을 추가할 것이다.