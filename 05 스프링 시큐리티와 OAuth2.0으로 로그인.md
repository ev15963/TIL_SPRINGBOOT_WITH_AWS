스프링 시큐리티와 OAuth2.0으로 로그인
=======================
# 1. 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트
왜 많은 서비스에서 소셜 로그인을 사용할까? 이동욱 저자님의 생각으로는 **배보다 배꼽이 커지는 경우**라 하셨다.      
   
직접 구현한다면 다음을 전부 구현해야한다.        
* 로그인 시 보안 
* 비밀번호 찾기
* 비밀번호 변경
* 회원정보 변경
* 회원가입시 이메일 혹은 전화번호 인증
    
그렇기에 OAuth2 로그인 구현을하면 앞선 목록을 소셜 기업에 맡기고 서비스에만 집중할 수 있다.       
  
## 1.1. 스프링부트 1.5 vs 스프링 부트 2.0  
스프링 부트 1.5에서의 OAuth2 연동 방법이 2.0에서는 크게 변경되었다.  
하지만 ```spring-security-oauth2-autoconfigure``` 라이브러리 덕분에 **설정 방법의 큰 차이를 없게 할 수 있다**   
```
spring-security-oauth2-autoconfigure
```
해당 라이브러리를 사용할 경우 스프링 부트2에서도 기존 설정을 그대로 사용할 수 있다.   
아무래도 새로운 방식보다는 기존에 안전하게 작동하는 방법이 확실하므로 많은 개발자가 해당 방식을 이용했다.  
    
**하지만**    
우리는 스프링 부트2 방식인 ```Spring Security Oauth2 Client``` 라이브러리를 사용할 것이고    
이유는 아래와 같다.  

* 스프링 팀에서 기존 1.5에서 사용되던 spring-security-oauth 프로젝트는 유지 상태로 경정했으며   
더는 신규 기능은 추가하지 않고 버그 수정 정도의 기능만 추가될 예정이다.   
즉, 신규 기능은 oauth2 라이브러리에서만 지원하겠다고 선언한 것이다.  
* 스프링 부트용 라이브러리(starter)가 출시 되었다.  
* 기존에 사용되던 방식은 확장 포인트가 적절하게 오픈되어 있지 않아 직접 상속하거나 오버라이딩 해야 하고 
신규 라이브러리의 경우 확장 포인트를 고려해서 설계된 상태이다.    
   
그렇기에 이제 새롭게 배우는 학생 입장에서는 스프링 부트2 방식으로 배우는 것이 좀 더 나을 것이다.   
      
**추가 이야깃거리**   
스프링 부트2 방식의 자료를 찾고 싶은 경우 인터넷 자료들 사이에서 다음 2가지만 확인하면 된다.  
1. spring-security-oauth2-autoconfigure 라이브러리를 사용했는지  
2. application.properties 혹은 application.yml 정보가 다음 사진과 같이 차이가 있는지  
   
[사진]   
   
스프링 부트 1.5 방식에서는 url 주소를 모두 명시해야 하지만,     
스프링 부트 2.0 방식에서는 client 인증 정보만 입력하면 된다.      
     
1.5 버전에서 직접 입력했던 값들은 2.0 버전으로 오면서 모두 **enum**으로 대체되었다.     
**CommonOAuth2Provider**라는 enum이 새롭게 추가되어 구글, 깃허브, 페이스북, 옥타의 기본 설정값은 모두 여기서 제공한다.    
이외에 다른 소셜 로그인을 추가한다면 직접 다 추가해 주어야 한다.  

## 1.2. 구글 서비스 등록 

## 1.3. application-oauth 등록  
```application.properties```가 있는 src/main/resources/디렉토리에       
```application-oauth.properties``` 파일을 생성한다.           
그리고 해당 파일에 클라이언트ID와 클라이언트 보안 비밀 코드를 다음과 같이 등록한다.       
    
**application-oauth.properties**    
```
맥북에서 가져오자  
```
```
scope=profile,email
```
* 많은 예제에서는 이 scope를 별도로 등록하지 않고 있다.  
* 기본값이 openid, profile, email이기 때문이다.
* 강제로 profile, email을 등록한 이유는 openid가 scope에 있으면 Open Id Provider로 인식하기 때문이다.  
* 이렇게 되면 Open Id Provider 인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 Oauth2Service를 만들어야 한다.
* 그렇기에 하나의 OauthService로 사용하기 위해 일부러 scope 에서 openid를 빼고 등록해주자.  
  
스프링 부트에서는 properties의 이름을 application-xxx.properties로 생성하면   
xxx라는 이름의 profile이 생성되어 이를 통해 관리를 할 수 있다.  
즉, profile=xxx 라는 식으로 호출하면 해당 properties의 설정들을 가져올 수 있다.   
호출하는 방식은 여러 방식이 있지만 스프링 부트의 기본 설정 파일인 application.properties에서 
appication-oauth.properties를 포함하도록 구성해보자  

**appication.properties**
```
spring.profiles.include=oauth
```

## 1.4. .gitignore 등록  
우리의 프로젝트는 깃허브와 연동하여 사용하다 보니 application-oauth.properties 파일이 깃허브에 올라갈 수 있다.   
보안을 위해 해당 파일이 올라가는 것을 금지해주자    
```
application-oauth.properties
```
추가한 뒤 커밋했을 때 커밋 파일 목록에 **application-oauth.properties**가 나오지 않으면 성공이다.   
만약 .gitignore에 추가했음에도 여전히 커밋 목록에 노출된다면 이는 Git의 캐시문제이다.  

***
# 2. 구글 로그인 연동하기 
## 2.1. User Entity 설정하기

구글의 로그인 인증정보를 발급 받았으니 프로젝트 구현을 진행해보자    
우선 사용자 정보를 담당할 도메인인 ```User```클래스를 생성해주자.    

**User**
```java
```

**소스코드 해석**
```java
@Enumerated(EnumType.STRING)

* JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정합니다.    
* 기본적으로 int로 된 숫자가 저장됩니다.   
* 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수가 없습니다.   
* 그래서 문자열 (EnumType.STRING)로 저장될 수 있도록 선언합니다.  
```


**Role**
```java

```
**스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야 합니다.**      
그래서 코드별 키 값을 ```ROLE_GEUST```, ```ROLE_USER```등으로 지정합니다.     
   

**UserRepository**
```java

```
**소스코드 해석**
```java
findByEmail(String email);   

* 소셜 로그인으로 반환되는 값중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드
* PK 를 사용한 것이 아니라 Unique 를 사용한 것을 알 수 있다.   
```

## 2.2. 스프링 시큐리티 설정   
먼저 ```build.gradle```에 스프링 시큐리티 관련 의존성 하나를 추가해주자      
    
**build.gradle**    
```gradle  
complie('org.springframework.boot::spring-boot-starter-oauth2-client')
```    

**소스코드 해석**
```gradle
spring-boot-starter-oauth2-client

* 소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현시 필요한 의존성입니다.  
* spring-boot-starter-oauth2-client 와 spring-boot-starter-oauth2-jose를 기본으로 관리해줍니다.
```

``` build.gradle```설정이 끝났으면 OAuth 라이브러리를 이용한 소셜 로그인 코드를 작성해보자      
```config.auth``` 패키지를 생성합니다.         
앞으로 **시큐리티 관련 클래스는 모두 이곳에 담는다**고 보면 될 것 같습니다.     
    
그리고 SpringConfig 클래스를 생성해줍시다.   

**SpringConfig**
```java
```
**소스코드 해석**
```java
@EnableWebSecurity   

* Spring Security 설정들을 활성화시켜줍니다.     
_____________________________________________________________
http.csrf().disable().headers().frameOptions().disable()    

* h2-console 화면을 사용하기 위해 해당 옵션들을 disable합니다.   
_____________________________________________________________
.authorizeRequests()  

* URL 별 권한 관리를 설정하는 옵션의 시작점입니다.   
* authorizeRequests() 가 선언되어야만 andMatchers() 옵션을 사용할 수 있습니다.   
_____________________________________________________________
.andMatchers("url", "url2")    

* 권한 관리 대상을 지정하는 옵션입니다.   
* URL, HTTP 메소드별로 관리가 가능합니다.   
* "/" 등 지정된 URL들은 permitAll() 옵션을 통해 전체 열람 권한을 주었습니다.   
* "api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 했습니다.  
_____________________________________________________________
.anyRequest()

* 설정된 값 이외 나머지 URL 들을 나타냅니다.  
* 여기서는 .authenticated()을 추가하여 나머지 URL 들은 모두 인증된 사용자들에게만 허용합니다.   
* 인증된 사용자 즉, 로그인 한 사용자들을 이야기합니다.   
_____________________________________________________________
.logout().logoutSuccessUrl("/")     
* 로그아웃 기능에 대한 여러 설정의 진입점입니다.   
* 로그아웃 성공시 / 주소로 이동합니다.  
_____________________________________________________________
.oauth2Login()   

* OAuth2 로그인 기능에 대한 여러 설정의 진입점입니다.   
_____________________________________________________________
.userInfoEndpoint()   

* OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당합니다.   
_____________________________________________________________
.userService(customOAuth2UserService)    

* 소셜 로그인 성공시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록합니다.       
* 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있습니다.  
```
설정 코드 작성이 끝났다면 ```CustomOAuth2UserService```클래스를 생성하자   
이 클래스는 구글 로그인 이후 가져온 사용자 정보들을 기반으로    
가입 및 정보수정, 세션 저장등의 기능을 지원해준다.       
      
**CustomOAuth2UserService**    
```java
```
**소스코드 해석**
```java
String registrationId = userRequest.getClientRegistration().getRegistrationId();

* 현재 로그인 진행중인 서비스를 구분하는 코드   
* 구글로 로그인, 네이버로 로그인하는지 구분하기 위해 사용되는 코드이다.   
_________________________________________________________________________________________
String userNameAttributeName =   
userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

* OAuth2 로그인 진행시 키가 되는 필드값을 이야기합니다. PK 같은 역할   
* 구글의 경우 기본적으로 코드를 지원하지만 ("sub") , 네이버 카카오등은 지원하지 않습니다.   
* 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용할 것입니다.  
_________________________________________________________________________________________
OAuthAttributes attributes = 
OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

* oAuth2User 는 OAuth2UserService 로 만들어진 OAuth2User 객체를 참조하는 변수이다.  
* OAuthAttributes attributes는 OAuth2UserService 를 통해 가져온 OAuth2User 클래스의 attribute를 담을 클래스입니다. 
* 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용합니다.   
* 이것은 우리가 직접 정의해주는 클래스로 밑에서 클래스를 작성할 것입니다.   
________________________________________________________________________________________  
httpSession.setAttribute("user", new SessionUser(user));

* 세션에 자용자 정보를 저장하기 위한 DTO 클래스    
* 왜 User 클래스를 사용하지 않고 새로 만들어서 쓰냐면  
*
*
```








> 인용
## 2.1. 소 주제
### 2.1.1. 내용1
```
내용1
```   

***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
