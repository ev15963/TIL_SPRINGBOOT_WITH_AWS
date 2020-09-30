# NGINX 무중단 배포      
배포를 진행하면 **새로운 Jar가 실행되기전까지 기존 Jar를 종료시켜 놓기 때문에**서비스가 중단됩니다.            
          
## 무중단 배포 소개        
배포가 서비스를 정지해야만 가능할 때는 롤백이 어렵고 서비스가 정지되니 곤혹스럽습니다.     
그래서 서비스를 중단시키지 않고 배포를 진행하는 **무중단 배포**를 해야합니다.    
  
* AWS에서 블루 그린 무중단 배포       
* 도커를 이용한 웹 서비스 무중단 배포     
  
이외에도 L4 스위치를 이용한 무중단 배포 방법도 있지만,   
L4가 워낙 고가의 장비이다 보니 대형 인터넷 기업외에는 쓸일이 거의 없습니다.    

**NGINX**는 웹 서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스 소트프웨어입니다.   
**리버스 프록시**란 NGINX가 **외부의 요청을 받아 백앤드 서버로 요청을 전달하는 행위**를 합니다.        
리버스 프록시 서버(NGINX)는 요청을 전달하고, 실제 요청에 대한 처리는 뒷단의 웹 애플리케이션 서버들이 처리합니다.    
즉 NGINX 자체로도 서버로 이용이 가능하고 서버에 외부 요청을 전달하는 역할도 할 수 있습니다.          
또한 NGINX 는 AWS 서비스가 아니고 오픈소스 소프트웨어이기 때문에     
    
우리는 이 `리버스 프록시`를 사용하여 무중단 배포를 이용할 것입니다.       
**하나의 서버에 NGINX 1대와 스프링부트 Jar를 2대 사용하면 됩니다.**         

* NGINX 는 80번 포트, 443번 포트를 할당합니다.    
* 스프링부트1 은 8081 포트로 실행합니다.   
* 스프링부트2 는 8082 포트로 실행합니다.   
    
**앤진엑스 무중단 배포는 다음과 같은 구조가 됩니다.**
   
[사진]   
   
## 운영과정 

[사진]   
   
**최초 배포**
1. 사용자는 서비스 주소로 접속합니다. (**80 혹은 443 포트**)    
2. NGINX는 사용자의 요청을 받아 현재 연결된 스프링 부트1(8081)로 요청을 전달합니다.   
3. 현재 스프링 부트2(8081)는 NGINX와 연결된 상태가 아니니 요청받지 못합니다.   
     
[사진]        
        
**2번재 배포**    
1. 연결되지 않은 서버로 새로운 배포를 진행합니다.   
2. 배포하는 동안에도 기존 서비스는 중단되지 않습니다.(NGINX는 기존 최초 서버를 바라바보기 때문입니다.)      
3. 배포가 끝나면 스프링 부트2가 구동하기 시작합니다.   
4. NGINX 는 스프링 부트2가 구동 중인지 확인하고 구동 중이면 nginx reload 명령어를 통해 스프링부트2를 바라보도록합니다.    
5. nginx reload는 0.1초 이내로 완료되기에 딜레이가 없이 서버가 바뀌게 됩니다.   
   
[사진]    
   
**3번재 배포**    
1. 스프링 부트1로 배포를 진행합니다. 
2. 배포하는 동안에도 기존 서비스는 중단되지 않습니다.(NGINX는 스프링 부트2를 바라바보기 때문입니다.)      
3. 배포가 끝나면 스프링 부트1가 재구동하기 시작합니다.   
4. NGINX 는 스프링 부트1이 구동 중인지 확인하고 구동 중이면 nginx reload 명령어를 통해 스프링 부트1를 바라보도록합니다.    
5. nginx reload는 0.1초 이내로 완료되기에 딜레이가 없이 서버가 바뀌게 됩니다. 

[전체 구조]   
    
## 엔진엑스 설치와 스프링 부트 연동하기  
### 엔진엑스 설치 
1. EC2에 접속한 후 `sudo yum install nginx` 입력
2. 설치가 완료되었다면 `sudo service nginx start`로 nginx 실행  
3. NGINX가 잘 실행되었다면 `starting nginx: [ ok ]` 출력이 되므로 이를 확인   

### 보안 그룹 추가 
먼저 NGINX의 포트번호를 보안그룹에 추가해야합니다.    
NGINX의 기본적인 포트번호는 80이므로 EC2 보안그룹에 80번 트를 할당해줍시다.    

1. EC2 서비스로 이동     
2. 왼쪽 대시보드의 `[네트워크 보안 및 관리 카테고리]`에서 `[보안 그룹]`으로 이동    
3. EC2 관련 보안그룹을 클릭         
4. 인바운드 규칙에서 NGINX 관련 보안그룹 추가     

|유형|프로토콜|포트 범위|소스|설명-선택사항|
|---|---|---|---|---|
|HTTP|TCP|80|사용자 지정-0.0.0.0/0|NGINX|
|HTTP|TCP|80|사용자 지정-::/0|NGINX|

### 리다이렉션 주소 추가   
8080 포트가 아닌 80 포트로 주소가 변경되니 구글과 네이버 로그인에도 변경된 주소를 등록해야만 합니다.            
기존에 등록된 리디렉션 주소에서 8080부분을 제거하여 추가 등록합니다.            
               
추가한 후에는 EC2의 도메인으로 접속을 해봅시다.         
단, 기존의 8080포트가 아니라 80포트를 사용해서 접속을 해봅시다.(즉, 포트번호 없이 입력)           
    
### NGINX와 스프링 부트 연동   
NGINX가 현재 실행중인 스프링 부트 프로젝트를 바라볼 수 있도록 NGINX 설정 파일에서 프록시 설정을 하겠습니다.     
   
```
sudo vim /etc/nginx/nginx.conf
```
이후 `server{}`안에 있는 `location / ` 부분을 찾아서 다음과 같이 추가합니다.   

```
        location / {
                proxy_set_header X-Real_IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
```
```
                proxy_set_header X-Real_IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
```
* 실제 요청 데이털르 header의 각 항목에 할당합니다.      
* 예) `proxy_set_header X-Real_IP $remote_addr` : Request Header의 X-Real-IP에 요청자 IP를 저장합니다.       
  
수정이 끝났으면 `:wq` 명령어로 저장하고 종료합니다.   
설정사항이 바뀌었으니 `sudo service nginx restart`를 통해 재시작을 해주어야 합니다.   

## 무중단 배포 스트립트 만들기    
무중단 배포 스크립트를 만들기 전에 API를 하나 추가시켜줄 것입니다.   
해당 API는 배포시에 8081을 사용할지, 8082를 사용할지 판단을 하는 기준이 됩니다.   

### profile API 추가    
profileController를 만들어 아래와 같은 코드를 추가합니다.     
   
1. `com.jojoldu.book.springboot.web`에 `profileController` 클래스를 생성합니다.     
2. 아래와 같은 코드를 넣어줍니다.   

**ProfileController**   
```java
import lombok.RequiredArgsConstructor;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile(){
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        return profiles.stream().filter(realProfiles::contains).findAny().orElse(defaultProfile);
    }

}
```
```java
    private final Environment env;
```
Enviroment 클래스는 외부 설정파일을 가져와서 프로퍼티를 추가하거나 추출하는 역할을 합니다.
외부 설정파일은 config 파일을 의미하고 현재 실행중인 상태에 대해서도 저장을 하고 있습니다.
그리고 Enviroment 클래스는 외부 설정값을 자바에서 변경할 수도 있게끔 해줍니다.   

```java
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
```
* 외부 설정파일에서 현재 실행중인 ActiveProfile을 모두 가져옵니다. (실행중인 서버 profile 상태)
* 즉, real, oauth, real-db 등이 활성화되어 있다면 3개가 모두 담겨있습니다.   
* 참고로 `asList()`는 가변 매개변수로 개수가 지정되어 있지 않습니다.   
    
```java
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
```
* 여기서 `real`, `real1`, `real2`는 모두 배포에 사용될 profile이라 이들을 담은 List를 하나 만듭니다.   
* 실제로 우리는 `real1`, `real2`만 사용할 것이지만 기존 상태로 다시 사용할 수도 있으니 real도 남겨줍니다.   
* 밑에서 배포에 사용되는 profile이 있는지 없는지 검사할 때 기준이되므로 만들어줍니다.   

```java
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);
```
* `defaultProfile` 변수는 활성화 되어 있는 Profile이 없으면 default를 넣고 있으면 가장 처음 profile을 가져옵니다.   

```java
        return profiles.stream().filter(realProfiles::contains).findAny().orElse(defaultProfile);
```
* 현재 실행중인 profile 등 중에서 배포에 사용하는 프로파일이 있는 경우 순서에 상관없이 하나를 리턴하고 없으면 defaultProfile 리턴합니다. 
* `realProfiles::contains`는 `contains(profiles 인스턴스중 하나)`로 동작하여 있는지 없는지 검사합니다.   
    * `public boolean contains(Object o)`
   
이제 해당 코드가 잘 동작하는지 테스트 코드를 작성해보겠습니다.        
해당 컨트롤러는 특별히 **스프링 환경이 필요하지는 않습니다.**          
Spring 관련 어노테이션과 클래스를 사용하지만 실제적으로는 Spring config 파일을 조회하기만 하는 것이기 때문입니다.        
그래서 `@SpringBootTest`없이 테스트 코드를 작성합니다.
     
패키지는 test 폴더의 `package com.jojoldu.book.springboot.web`입니다.    
   
**ProfileControllerUnitTest**
```java
package com.jojoldu.book.springboot.web;

import org.junit.Test;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;

public class ProfileControllerUnitTest {

    @Test
    public void real_profile이_조회된다() {
        //given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        //given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void active_profile이_없으면_default가_조회된다() {
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
}
```
ProfileController나 Environment 모두 자바 클래스(인터페이스)이기 때문에 쉽게 테스트할 수 있습니다.              
Enviroment는 인터페이스라 **가짜 구현체인 `MockEnvironment`를 사용해서 테스트하면됩니다.**       
   
또한 생성자 주입을 통해 쉽게 값을 넘겨줄 수 있는 장점도 발견할 수 있습니다.      
만약 `@Autowired`를 사용하여 DI를 했다면 스프링 테스트를 사용했어야 했을겁니다.      

```java
    @Test
    public void real_profile이_조회된다() {
        //given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
```
* 가짜 Environment 구현체인 MockEnvironment 객체를 만듭니다.   
* MockEnvironment 객체에 여러 profile을 넣어줍니다.   
* ProfileController 에 해당 MockEnvironment 객체를 넣어줍니다.
* 실제 배포용 profile이 존재하기에 Controller는 실제 배포용 profile을 리턴할 것입니다.   
* 실제 배포용 profile이 기존에 준비한 profile과 일치하는지 검증합니다.    

```java
    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        //given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
```
* 가짜 Environment 구현체인 MockEnvironment 객체를 만듭니다.     
* MockEnvironment 객체에 여러 profile을 넣어주는데 실제 배포 profile 들을 제외하고 넣어줍니다.    
* ProfileController 에 해당 MockEnvironment 객체를 넣어줍니다.
* 실제 배포용 profile이 존재하지는 않지만 다른 profile 들이 있으니 가장 먼저 들어간 profile을 리턴할 것입니다.     
* 가장 먼저 들어간 profile이 리턴되었는지 검증합니다.      

```java
    @Test
    public void active_profile이_없으면_default가_조회된다() {
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
```

* 가짜 Environment 구현체인 MockEnvironment 객체를 만듭니다.     
* MockEnvironment 생성자에 객체에 넣어주지 않고 객체를 생성합니다. 
* 어떤 profile이 존재하지 않으므로 `default` 를 리터할 것입니다.    
* 리턴된 값이 `default`와 일치하는지 검증해줍니다.   
    
그리고 앞선 securityConfig에서도 `/profile`에 접근할 수 있도록 http 설정을 수정해줍시다.    

```java

package com.jojoldu.book.springboot.config.auth;

import com.jojoldu.book.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```  
그리고 securityConfig 설정이 잘 되었는지도 검증도 해주도록 합니다.         
아 검증은 스프링 시큐리티 설정을 불러와야 하니 `@SpringBootTest`를 사용하는 테스트 클래스로 작성해줍니다.         
    
**securityConfig** 자체에 대한 테스트를 진행하는 것이 아닌      
`ProfileController`가 실제 스프링 시큐리티 환경에서 동작하는지에 대한 테스트를 진행하는 것입니다.      
     
**ProfileControllerTest**
```java
package com.jojoldu.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ProfileControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void profile은_인증없이_호출된다() throws Exception {
        String expected = "default";

        ResponseEntity<String> response = restTemplate.getForEntity("/profile", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isEqualTo(expected);
    }
}
```
코드를 간단히 해석하자면 단순히 `/profile`로 이동했을 때 얻는 값이 `default` 인지 확인합니다.        
서버를 실제 실행한것도, 그리고 profile을 준것도 아니기 때문에 `default`가 나올수밖에 없습니다.         
    
RestTemplate 클래스에 대해서 알고싶다면 https://advenoh.tistory.com/46 블로거님이 잘 정리했으니 보도록하자.      
참고로 EndPoint의 의미를 몰라서 검색했는데      
`ENDPOINT란 API가 서버에서 리소스에 접근할 수 있도록 가능하게 하는 URL이다`          
라고 하지만 내 개인적인 생각으로는 중간에 에러가 발생하지 않고 마지막 로직까지 처리되는 것을 의미하는 것 같다.        
     
여기까지 모든 테스트가 성공했다면 깃허브로 푸시하여 배포합니다.      
배포가 끝나면 브러우저에서 `/profile`로 접속해서 profile이 잘 나오는지 확인합니다.       
     
### real, real2 profile 생성       
현재 EC2 환경에서 실행되는 profile은 `real`로 **Travis CI 배포 자동화를 위한 profile**이니         
무중단 배포를 위한 profile 2개(real1, real2)를 추가합니다.       

**application-real1.properties**
```java
server.port=8081
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```
**application-real2.properties**
```java

server.port=8082
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```
2개의 profile은 대부분의 내용은 비슷하지만 **포트 번호만 다릅니다.**   

### NGINX 설정 수정   
무중단 배포의 핵심은 **NGINX 설정입니다.**    
배포때마다 NGINX의 프록시 설정이 순식간에 교체됩니다.(스프링 부트로 요청을 흘려보내는)     
여기서 프록시 설정이 교체될 수 있도록 설정을 추가하겠습니다.    

NGINX 설정이 모여있는 `/etc/nginx/conf.d/`에 `service-url.inc`아는 파일을 하나 생성합니다.    
   
```   
sudo vim /etc/nginx/conf.d/service-url.inc`    
``` 
그리고 다음 코드를 입력합니다.      
      
```
set $service_url http://127.0.0.1:8080;
```
`:wq`로 저장하고 종료한 뒤 해당 파일은 엔직엔스가 사용할 수 있게 설정하겠습니다.        
   
```
sudo vim /etc/nginx/nginx.conf
```
nginx 파일을 열어줍니다.   

```
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        include /etc/nginx/conf.d/service-url.inc;

        location / {
                proxy_pass $service_url;
                proxy_set_header X-Real_IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }		
```
`location /` 부분을 찾아서 위와 같이 수정해줍니다.      

```
	      proxy_pass $service_url;
```
* NGINX 로 요청이 오면 `http://localhost:8080`으로 전달한다는 뜻입니다.      
  
`:wq`로 저장하고 종료한 뒤 설정이 변경되었으니 nginx를 재시작해주어야 합니다.   

```
sudo service nginx restart
```
다시 프로젝트에서 서버를 킨 후 브라우저에서 정상 동작을 한다면 설정이 잘 완료된 것입니다.    
   
### 배포 스크립트들 작성 
먼저 기존 step2와 중복되지 않기 위헤 EC2에 step3 디렉토리를 생성합니다.    

```
mkdir ~/app/step3 && mkdir ~/app/step3/zip
```
무중단 배포는 앞으로 step3를 사용하겠습니다.   
그래서 `appspec.yml` 역시 step2로 되어있는 부분을 step3로 배포되도록 수정합니다.           
또한 무중단 배포를 위한 스크립트가 총 5개여서 이에 관한 설정을 해주겠습니다.        

* stop.sh : 기존 NGINX에 연결되어 있지는 않지만, 실행중이던 스프링부트 종료   
* start.sh : 배포할 신규 버전 스프링 부트 프로젝트를 stop.sh로 종료한 `profile`로 실행     
* health.sh : `start.sh`로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크     
* switch.sh : NGINX가 바라보는 스프링 부트를 최신 버전으로 변경      
* profile.sh : 앞선 4개 스크립트 파일에서 공용으로 사용할 `profile`과 포트 체크 로직    
    
**appspec.yml**
```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step3/zip/
    overwriter: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료한다.
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작한다.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 스프링 부트가 정상적으로 실행됐는지 확인한다.
      timeout: 60
      runas: ec2-user
```

스크립트 부분은 JAR 파일이 복사된 이후부터 차례로 앞선 스크립트들이 실행된다고 보면 됩니다.       
   
___ 
이제 각 스크립트들에 대해서 코드를 작성해보겠습니다.   
전체적으로 scripts 디렉토리 안에서 생성하겠습니다.    

**profile.sh**
```sh

#!/usr/bin/env bash

# bash는 return value가 안되니 *제일 마지막줄에 echo로 해서 결과 출력*후, 클라이언트에서 값을 사용한다

# 쉬고 있는 profile 찾기: real1이 사용중이면 real2가 쉬고 있고, 반대면 real1이 쉬고 있음
function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}"
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}
```
```sh
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)
```
해당 코드를 해석하기 전에 curl에 대해서 설명하자면 **http 메시지를 쉘 상에서 요청하여 결과를 확인하는 명령어이다.**   

* `-s` : `-silent` 라는 의미로 부가정보 없이 조회하도록 합니다.   
* `-o` : `-option` 이라는 뜻으로     
remote 에서 받아온 데이타를 기본적으로는 콘솔에 출력한다. -o 옵션 뒤에 FILE 을 적어주면 해당 FILE 로 저장한다.     
* `-w` : http 응답 코드를 조회할 수 있도록 해주는 옵션이다.
* `"%{http_code}"` : 응답 코드 출력시 포멧을 지정한 것으로 200, 403 같은 응답코드형식으로 나타내준다.   
* `http://localhost/profile` : 우리가 정상적으로 수행하고 있는지 검사하는 대상 url

자세한 내용은 https://qastack.kr/superuser/272265/getting-curl-to-output-http-status-code
쉡게 말해서 `curl -s -o /dev/null -I -w "%{http_code}" http://www.example.org/`를 이용하면    
구문 분석이 필요하지 않아 스크립트에서 사용하기 편한 '응답 상태 확인'을 한다는 것이다.           
     
* 현재 NGINX 가 바라보고 있는 스프링 부트가 정상적으로 수행중인지 확인합니다.   
* 응답값을 httpStatus로 받습니다.   

```sh
    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi
```
* 정상이면 200, 오류가 발생한다면 400-503 사이로 발생하니 400 이상은 모두 예외로 보고 real2를 현재 profile로 합니다.    

```sh
    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi
```
* NGINX와 연결되지 않은 profile을 조회하는 코드입니다.   
* IDLE_PROFILE는 이렇듯 연결되지 않은 profile을 담는 변수로 real1 이면 real2, real2 이면 real1로 세팅될 것입니다.   

```sh
    echo "${IDLE_PROFILE}"
```
* **bash라는 스크립트는 값을 반환하는 기능이 없습니다.**   
* 그래서 **제일 마지막 줄에 echo로 결과를 출력한 후** 클라이언트에서 그 값을 잡아서 `($(find_idle_profile))`사용합니다.   
* 위 코드를 보면 메서드 마지막에 echo를 사용했둣이 중간에 echo를 사용해서는 안 됩니다.    

```sh
# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}
```
* 기존 위에 정의한 메서드를 사용해서 현재 사용중이지 않는 profile을 구합니다.       
* real1 일 경우 저희는 8081로 하자 했으니 8081을 출력합니다.         
* real1 이 아닐 경우 (real2) 저희는 8082로 하자 했으니 8082를 출력합니다.   
    
**stop.sh**
```sh

#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```   
```sh
ABSPATH=$(readlink -f $0)
```
* `readlink` : 심볼릭 링크의 원본 파일 확인  
* `readlink -f` : 심볼릭 링크를 따라 최종의 파일을 절대경로로 반환 -> 가장 원본 파일       
* `심볼릭 링크` : 링크를 연결하여 원본 파일을 직접 사용하는 것과 같은 효과를 내는 링크이다    
* `$0` : 스크립트에서 실행 시 실행된 쉘 스크립트 경로를 포함한 파일명을 의미   
* 즉, `readlink -f $0`는 현재 쉘 스크립트를 실행하는 원본 파일의 경로를 의미한다.    
* 이를 더 간단히 말하면 **현재 스크립트가 있는 경로를 구한것입니다.**  

```sh
ABSDIR=$(dirname $ABSPATH)
```
* 앞서 구한 쉘 스크립트를 실행하는 원본 파일이 속한 디렉토리를 구합니다.
* 이를 더 간단히 말하면 **현재 스크립트가 있는 디렉토리를 구한것입니다.**

```sh
source ${ABSDIR}/profile.sh
```
* 쉘 스크립트를 실행하는 원본 파일이 속한 디렉토리에서 `profiles.sh` 스크립트 코드를 가져옵니다.        
* 자바로 보면 일종의 import 같은 구문 
* 이를 더 간단히 말하면 **현재 스크립트에 같은 디렉토리에 있는 profiles.sh 스크립트 코드를 가져와서 사용하는것입니다.**              
	 
```sh
IDLE_PORT=$(find_idle_port)
```
* 앞서 `source ${ABSDIR}/profile.sh`로 인하여 `profile.sh` 스크립트를 사용할 수 있으므로   
메서드인 `find_idle_port`를 통해서 사용하지 않는 포트값을 가져옵니다.   

```sh
echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```
* 해당 포트번호가 사용중인지 아닌지 알기 위해 `lsof`를 이용해서 탐색을 진행합니다.   
	* `lsof` : list open files 이라는 뜻으로 열려있는 파일이나 **실행중인 프로세스** 목록을 출력합니다.   
	* `-t` : 자세한 정보를 출력하지 않고 pid 정보만 출력한다.
	* `-i` : 모든 네트워크 포트를 표시하고 i 뒤에 프로토콜을 명시하면 해당 프로토콜 관련 포트만 표시한다.   
	* `tcp:${IDLE_PORT}` : 사용하지 않는 포트로 조회된 포트가 tcp 프로토콜로 사용하는지 검사    
* `-z` : 문자열의 길이가 0이면 true, 즉 기존에 사용안했다면 문자열이 없고 사용했다면 문자열이 있다.    
* `  kill -15 ${IDLE_PID}` : 사용중이면 처리되는 로직에 있던 코드, 해당 포트를 죽인다
	* `-9`도 있던데 차이는? `-9`는 강제종료 , `-15`는 종료 요청후 안전하게 종료    	
* `sleep 5` : 5초간 대기 
    
**start.sh**
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
```sh
ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
```
* 앞선 내용과 똑같이 `profile.sh`를 사용하기 위한 코드입니다.   

```sh
REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=freelec-springboot2-webservice
```
* 프로젝트를 실행할 레포지토리랑 프로젝트 이름을 기술합니다.   

```sh
echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/
```
* 기존 Travis CI 와 S3를 이용하면서 zip 폴더안에는 Jar와 스크립트들이 들어있습니다.      
* 해당 jar파일을 zip보다 한단계 위 디렉토리로 옮깁니다.(step3로)         

```sh

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"
```
* 애플리케이션 배포를 위해 jar 파일의 이름을 얻습니다.   
* 또한 해당 jar 파일이 실행될 수 있도록 실행권한을 줍니다.   
* `-t` : 파일을 시간순으로 정렬합니다. 최신순으로 내림차순합니다.(최신이 맨위)
* `-r` : 반대로 출력합니다.   
* 즉 최신이 가장 밑에 있도록 하는 것입니다.   
* `tail` : 파일의 내용을 뒤에서부터 출력하는 명령어입니다. 여기서는 목록의 맨뒤를 의미합니다.   
* `-n 숫자` : K개의 줄을 출력합니다. 즉 맨뒤에서 1개만 출력하는 것입니다.   
* 종합하자면 최신 jar 파일을 1개 가져온다 생각하면 될 것입니다.   

```sh
IDLE_PROFILE=$(find_idle_profile)
```
* 사용하지 않는 profile 을 받아옵니다. (real1 or real2)

```sh
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
* nohup 으로 jar파일을 실행하는데 `application-$IDLE_PROFILE.properties` 으로 실행합니다.   
* 즉, 사용하지 않는 서버로 jar를 실행합니다.   
* 또한 `profile.active=$IDLE_PROFILE` 을 이용하여 profile을 바꿔서 실행합니다.   
* CodeDeploy에서 무한 대기상태에 빠지는 것을 막기위해 `$JAR_NAME > $REPOSITORY/nohup.out 2>&1 &`를 기술해줍니다.    

**switch.sh**
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> 엔진엑스 Reload"
    sudo service nginx reload
}
```
우선 해당 sh는 `health.sh`에서 사용할 것입니다.   

```sh
ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
```
* 앞선 내용과 똑같이 `profile.sh`를 사용하기 위한 코드입니다.   

```sh
    IDLE_PORT=$(find_idle_port)
```
* 현재 사용하지 않는 포트를 가져옵니다. 

```sh
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc
```
* `"set \$service_url http://127.0.0.1:${IDLE_PORT};"`을 출력합니다. 
* 쌍따옴표 `"` 를 넣어주지 않으면 해당 문자열을 파이프라인으로 넘겨지지 않게 됩니다.    
* 앞에서 출력한 값을 `service-url.inc`에 덮어쓰기를 진행합니다 (참고로 내용 전체를 덮어씁니다.)     
* `tee` 명령어는 입력과 출력을 동시에 하는 명령어로 덮어쓴후 해당 내용을 출력할 것입니다.   

```sh
    echo "> 엔진엑스 Reload"
    sudo service nginx reload
```
* `service-url.inc`의 내용이 변경되었으나 설정파일만 다시 불러오면 되므로 nginx 를 `reload` 해줍니다.      
   
**health.sh**
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```
```sh
ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh
```
* 앞선 내용과 똑같이 `profile.sh`를 사용하기 위한 코드입니다.   
* 하지만 기존과 다른점은 `switch.sh`도 사용하도록 기술해줍니다.   

```sh
IDLE_PORT=$(find_idle_port)
```
* 현재 사용하지 않는 포트를 가져옵니다.     

```sh
for RETRY_COUNT in {1..10}
```
* 반복문을 진행합니다.   
* 리눅스 for는 닫는 괄호가 아니여서 1~9까지가 아닌 1~10까지 반복합니다.      

```sh
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
```
* `-s` : `-silent` 정숙모드를 이용해서 진행 내역, 메시지등을 제외하여 출력할때 주로 사용합니다. (주로 응답코드때 사용)     
* 여기서는 응답코드 대신에 real1, real2 와 같이 현재 사용하지 않는 포트에 대한 profile을 리턴합니다.      
    
```sh
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)
```
* 현재 사용하지 않는 포트의 profile을 출력하고 
* 해당 profile에 real 이란 이름이 있는지 검색합니다.   
* 그리고 있으면 라인 수가 1이 되고, 없으면 라인 수가 0이 되는데 이를 `UP_COUNT` 변수에 할당합니다.
* `grep` : 입력되는 파일에서 주어진 패턴 목록과 매칭되는 라인을 검색한 다음 표준 출력으로 검색된 라인을 복사해서 출력합니다. 
* 즉 real이란 단어를 검색한 다음 출력합니다.   
* `wc` : 파일내의 문자/라인/단어 의 수를 출력하도록 하는 명령어입니다.   
* `-l` : wc의 출력 대상을 라인으로 지정합니다.   
* 즉, 현재 사용하지 않는 profile이 real을 가리키면 1을 넣고 없으면 0을 넣습니다.     

```sh
  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi
```
* UP_COUNT 변수값이 1이 이상인지 체크합니다.  
* `-ge` : 는 `A >= B`와 같은 의미로 왼쪽 피연산자가 오른쪽 피연산자에 대해 **이상**인지 검증합니다.   
* 1 이상이면 `real` 관련 profile 즉, 배포 profile 이므로 `switch_proxy` 메서드를 실행합니다.  
	* switch_proxy 는 switch.sh 에 정의된 메서드로 `service-url.inc`의 내용을 사용하지 않는 port로 바꿉니다.  
	* break 가 있으므로 해당 구문 이후에는 아래 스크립트들이 실행되지 않도록 해줍니다.  
* `real` 관련 profile이 아닐경우 아니라는 출력을 해줍니다.   

```sh
  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi
```
* 만약 10회 이상동안 `real` 관련 `profile`이 나오지 않아 `break` 되지 않았다면 
* NGINX 를 연결하지 않고 종료시킵니다.   

```sh
  echo "> Health check 연결 실패. 재시도..."
  sleep 10
```
* 10회 이전에는 해당 스크립트가 실행되는데 10초 후에 다시 반복하도록 합니다.   
    
정리 하자면     
1. NGINX 와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크합니다.   
2. 잘 떴는지 확인되어야 NGINX 프록시 설정을 변경합니다.   
3. NGINX 프록시 설정 변경은 switch.sh 의 switch_proxy 메서드로 수행을합니다.    
4. 만약 프록시 설정 변경이 되지 않는다면 NGINX 연결을 시키지 않고 종료시킵니다.   
   
그리고 모든 과정을 총정리하자면   
* `stop.sh` : 현재 바라보고 있지 않는 즉, 이제 사용할 포트가 현재 사용중이라면 해당 포트를 죽입니다.  
* `start.sh` : 현재 바라보고 있지 않는 즉, 이제 사용할 profile을 이용하여 jar를 실행합니다.   
* `health.sh` : 현재 바라보고 있지 않는 즉, 이제 사용할 port 값을 통해 service_url.inc 를 갱신합니다.   
	* 또한 현재 바라보고 있지 않는 즉, 이제 사용할 profile 이 real 관련 profile인지 검증합니다.   
	* 아닐 경우 NGINX 연결을 하지 않습니다.  
* `profile.sh` : 현재 바라보고 있지 않는 즉, 이제 사용할 profile 과 port 값을 구해줍니다.   
* `switch.sh` : service_url.inc 값을 바꿔줍니다. 

## 무중단 배포 태스트 
배포 테스트를 진행하기 전에 한가지 해야할 일이 있습니다.   
잦은 배포로 Jar 파일명이 겹칠 수있습니다.   
매번 버전을 수동으로 올리는 것은 매우 귀찮은 일이므로 자동으로 버전값을 올릴 수 있도록 조치하겠습니다.  

```gradle
version '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
```

**build.gradle**
```gradle
buildscript {
    ext{
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'com.jojoldu.book'
version '1.0.3-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
sourceCompatibility = 1.8


repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
    compile('org.springframework.boot:spring-boot-starter-mustache')
    compile('org.springframework.boot:spring-boot-starter-oauth2-client')
    compile('org.springframework.session:spring-session-jdbc')
    compile('org.mariadb.jdbc:mariadb-java-client')
    testCompile('org.springframework.boot:spring-boot-starter-test')
    testCompile('org.springframework.security:spring-security-test')
}
```
* build.gradle 은 **Groovy 기반**의 '빌드 툴'입니다.   
* 그렇기에 Groovy 언어의 여러 문법을 사용할 수 있는데,   
여기서는 `newDate()`로 빌드할 때마다 그 시간이 버전에 추가되도록 구성했습니다.      
   
여기까지 구성한 뒤 최종 코드를 깃허브로 푸시합니다.      
배포가 자동으로 진행되면 CodeDeploy 로그로 잘 진행되는지 확인해봅니다.         

```
tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
```
그럼 아래와 같은 로그들이 출력될 것이고 이를 확인해주면 됩니다.   
```
[2020-06-28 15:19:25.563] [d-C6B8LCX54][stdout]> curl -s http://localhost:8081/profile
[2020-06-28 15:19:35.613] [d-C6B8LCX54][stdout]> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다.
[2020-06-28 15:19:35.613] [d-C6B8LCX54][stdout]> Health check:
[2020-06-28 15:19:35.613] [d-C6B8LCX54][stdout]> Health check 연결 실패. 재시도...
[2020-06-28 15:19:45.951] [d-C6B8LCX54][stdout]> Health check 성공
[2020-06-28 15:19:45.964] [d-C6B8LCX54][stdout]> 전환할 Port: 8081
[2020-06-28 15:19:45.964] [d-C6B8LCX54][stdout]> Port 전환
[2020-06-28 15:19:45.978] [d-C6B8LCX54][stdout]set $service_url http://127.0.0.1:8081;
[2020-06-28 15:19:45.979] [d-C6B8LCX54][stdout]> 엔진엑스 Reload
[2020-06-28 15:19:46.011] [d-C6B8LCX54][stdout]Reloading nginx: [  OK  ]
```
   
스프링 부트 로그도 보고 싶다면 다음 명령어로 확인할 수 있습니다.   

```
vim ~/app/step3/nohup.out
```
그럼 스프링 부트 실행 로그를 직접 볼 수 있습니다.    
한 번 더 배포하면 그때는 `real2`로 배포될 것입니다.    
이 과정에서 브라우저 새로고침을 해보면 전혀 중단 없는 것을 확인할 수 있습니다.   
2번 배포를 진행한 뒤에 다음과 같이 자바 애플리케이션 실행 여부를 확인합니다.   

```
ps -ef | grep java
```
  
다음과 같이 2개의 애플리케이션이 실행되고 있음을 알 수 있습니다.   

```
java -jar -Dspring.config.location=...-Dspring.profiles.active=real1 /home/ec2-user/app/step3/~~/jar
java -jar -Dspring.config.location=...-Dspring.profiles.active=real2 /home/ec2-user/app/step3/~~/jar
```
이제 이 시스템은 마스터 브랜치에 푸시가 발생하면 자동으로 서버 배포가 진행되고,      
서버 중단 역시 전혀 없는 시스템이 되었습니다.         



