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
* User 클래스를 사용하면 안되기에 SessionUser를 만들었다.   
* 왜 User 클래스를 사용하면 안되나요?**
만약 User 클래스를 그대로 사용했다면 다음과 같은 에러가 발생합니다.   

Failed to convert from type [java.lang.Object] 
to type [byte[]] for value 'com.jojoIdu.book.springboot.domain.user.User@4a43d6'    
이는 User 클래스에 직렬화를 구현하지 않았다는 의미의 에러입니다.     
그렇다면 User 클래스에 직렬화 코드를 넣으면 될까요? 그렇기에는 생각할 것이 많습니다.   

바로 User 클래스가  데이터베이스와 직접 연결되는 엔티티이기 때문입니다       
엔티티 클래스에는 언제 다른 엔티티와의 관계가 형성될지 모릅니다.       
   
예를 들면 @OneToMany , @ManyToMany등 자식 엔티티를 갖고 있다면      
직렬화 대상에 자식들까지 포함되니 성능 이슈, 부수 효과가 발생할 확률이 높습니다.      
      
그래서 직렬화 기능을 가진 DTO를 하나 추가로 만드는 것이 이후 운영 및 유지보수 때 많은 도움이 됩니다.  
```
구글 사용자 정보가 업데이트 되었을 때를 대비하여 update 기능도 같이 구현되었습니다.      
사용자의 이름이나, 프로필 사진이 변경되면 User 엔티티에도 반영이 됩니다.     
정확히 말하면 기존 Email 이 있다면 최신 정보로 받아오고         
Email 이 없다면 지금 정보로 Entity를 만들어라 하고 있습니다.    
   
CustomOAuth2UserService 클래스까지 생성되었다면 OAuthAttributes 클래스를 생성합니다.      
필자의 경우 OAuthAttributes는 DTO로 보기 때문에 config.auth.dto 패키지를 만들어 해당 패키지에 생성했습니다.   

**OAuthAttributes**     
```java   
```   
**소스코드 해석**
```java
public static OAuthAttributes of(){}   

* OAuth2User에서 반환하는 사용자 정보는 Map 자료구조 형태이기에 값 하나하나를 변환해야한다.   
__________________________________________________________________________________________
toEntity()    

* User 엔티티를 생성합니다.   
* OAuthAttributes 에서 엔티티를 생성하는 시점은처음 가입할 때입니다.   
* 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용합니다.   
```   
OAuthAttributes 클래스 생성이 끝났으면 같은 패키지에 SessionUser 클래스를 생성합니다.     
   
**SessionUser**
```java
```
SessionUser에는 인증된 사용자 정보만 필요합니다.         
그 외에 필요한 정보들은 없으니 name, email, picture 만 필드로 선언합니다.          

## 2.3. 로그인 테스트   
스프링 시큐리티가 잘 적용되었는지 확인하기 위해 화면에 로그인 버튼을 추가하자   
```index.mustache```에 로그인 버튼과 로그인 성공 시 사용자 이름을 보여주도록 하자   

**index.mustache**    
```mustache  
```   
**소스코드 해석**
```mustache
{{#userName}}   

* 머스테치는 다른 언어와 같은 if문을 제공하지 않습니다.     
* true/false 여부만 판단할 뿐입니다.      
* 그래서 머스터치에서는 항상 최종값을 넘겨줘야 합니다.      
* 여기서도 역시 userName 이 있다면 userName을 노출시키도록 구성했습니다.    
______________________________________________________________________________
a href="/logout"    

* 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL 입니다.   
* 즉, 개발자가 별도로 저 URL에 해당하는 컨트롤러를 만들 필요가 없습니다.   
* SecurityCofing 클래스에서 URL을 변경할 순 있지만 기본 URL을 사용해도 충분하니 여기서는 그대로 사용합니다.   
______________________________________________________________________________
{{^userName}}

* 머스테치 에서 해당 값이 존재하지 않는 경우에는 ^ 를 사용합니다.   
* 여기서는 userName이 없다면 로그인 버튼을 노출시키도록 구성했습니다.  
______________________________________________________________________________
a href = "/oauth2/authorization/google"     

* 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL 입니다.   
* 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요가 없습니다.  
* 후에 네이버 로그인은 따로 설정을 해주어야 할 것입니다.  
```
```index.mustache```에서 userName을 사용할 수 있게 IndexController 에서 UserName을 model에 저장하자  

**IndexController**
```java
```
**소스코드 해석**
```java
(SessionUser) httpSession.getAttribute("user");
    
* 앞서 작성된 CustomOAuth2UserService에서 로그인 성공 시 세션에 SessionUser를 지정하도록 구성했습니다.          
* 즉, 로그인 성공시 httpSession.getAttribute("user")에서 값을 가져올 수 있습니다.     
______________________________________________________________________________
if(user != null)   

* 세션에 저장된 값이 있을 때만 model에 userName 으로 등록합니다.      
* 세션에 저장된 값이 없으면 model엔 아무런 값이 없는 상태이니 로그인 버튼이 보이게 된다.   
```

***
# 3. 어노테이션 기반으로 개선하기     
일반적인 프로그래밍에서 개선이 필요한 나쁜 코드의 대표적인 예로 **같은 코드가 반복되는 부분**이 있습니다.   
같은 코드를 계속해서 복사 & 붙여넣기로 만든다면 이후에 수정이 필요할 때 모든 부분을 일일이 수정해줘야 합니다.  
이렇게 될 경우 유지보수성이 떨어질 수 밖에 없으며, 혹시나 수정이 반영되지 않은 부분이 생길 수 있습니다.   
   
```java
Session user = (SessionUser) httpSession.getAttribute("user");
```
위 코드는 그 대표적인 예로 값이 바뀔 경우 모든 소스를 바꿔줘야합니다.      
그렇기에 위 코드를 **메소드의 인자로 세션값을 바로 받을 수 있도록 변경해보겠습니다.**   
   
우선 ```config.auth``` 패키지에 ```@LoginUser``` 어노테이션을 생성합니다.    

**@LoginUser**
```java

```

**소스코드 해석**
```java
@Target(ElementType.PARAMETER)    
  
* 이 어노테이션이 생성될 수 있는 위치를 지정합니다.         
* PARAMETER 로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있습니다.    
* 이 외에도 클래스 선언문에 쓸 수 있는 TYPE등이 있습니다.      
______________________________________________________________________________
@Retention(RetentionPolicy.RUNTIME)     
* 어노테이션의 범위(?)라고 할 수 있겠습니다.     
* 어떤 시점까지 어노테이션이 영향을 미치는지 결정합니다.    
______________________________________________________________________________
@interface 
* 이 파일을 어노테이션 클래스로 지정합니다.       
* LoginUser 라는 이름을 가진 어노테이션이 생성되었다고 보면 됩니다.     
```
   
그리고 같은 위치에 ```LoginUserArgumentResolver``` 를 생성합니다.   
```LoginUserArgumentResolver``` 는 ```HandlerMethodArgumentResolver``` 인터페이스를 구현한 클래스입니다.   
   
```HandlerMethodArgumentResolver```는 한 가지 기능을 제공합니다.    
바로 조건에 맞는 경우의 메소드가 있다면     
```HandlerMethodArgumentResolver```의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있습니다.        
   
**LoginUserArgumentResolver**   
```java
```

**소스코드 해석**
```java
@Override
public boolean supportsParameter(MethodParameter parameter){ 사용자 정의}

* 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단합니다.  
* 여기서는 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환합니다.  
* 즉, @LoginUser SessionUser 이면 합격  
______________________________________________________________________________
@Override
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
   NativeWebRequest webRequest, WebDataBinderFactory binderFactroy) throws Exception {사용자 정의}

* 파라미터에 전달할 객체를 생성합니다.   
* 여기서는 세션에서 객체를 가져와서 넣습니다.  
* 즉 소셜로그인 하면서 httpSession에 user로 값이 저장되었는데 그것을 꺼내서 
@LoginUser SessionUser user 에 넣어주겠다는 의미입니다.   
```
이제 이렇게 생성된 ```LoginUserArgumentResolver```를 스프링에서 인식될 수 있도록      
```WebMvcConfiguration```에 추가하도록 하자.         
config 패키지에 ```WebConfig``` 클래스를 생성           

**WebConfig**
```java
``` 
```HandlerMethodArgumentResovler```는 항상              
```WebMvcConfigure``` 의 ```addArgumentResolvers()```를 통해 추가해야 합니다.            
다른 ```Handler-MethodArgumentResovler```가 필요한다면 같은 방식으로 추가해주면 됩니다.         
   
모든 설정이 끝났으니 처음 언급한 대로     
```IndexController``` 의 코드에서 반복되는 부분들을 모두 ```@LoginUser```로 개선하겠습니다.        
     
**IndexController**    
```java
```

**소스코드 해석**
```java
@LoginUser SessionUser user   
  
* 기존에(User) httpSession.getAttribute("user")로 가져오던 세션 정보 값이 개선되었습니다.     
* 이제는 어느 컨트롤러든지 @LoginUser만 사용함녀 세션 정보를 가져올 수 있게 되었습니다.   
```   


***
# 4. 세션 저장소로 데이터베이스 사용하기   
지금 우리가 만든 서비스는 애플리케이션을 재실행하면 로그인이 풀립니다.        
이는 **세션이 내장 톰캣의 메모리에 저장되기 때문입니다.**    
   
기본적으로 세션은 실행되는 **WAS의 메모리에서 저장되고 호출**됩니다.      
메모리에 저장되다 보니 **내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에선 항상 초기화 되는 것**입니다.      
즉, **배포할 때마다 톰캣이 재시작**되는 것입니다.     
   
이외에도 한 가지 문제가 더 있습니다.   
2대 이상의 서버에서 서비스하고 있다면 **톰캣마다 세션 동기화** 설정을 해야만 합니다.    
그래서 실제 현업에서는 세션 저장소에 대해 다음의 3가지중 한 가지를 선택합니다.   

1. 톰캣 세션을 사용한다.  
   * 일반적으로 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식입니다.       
   * 이렇게 될 경우 톰캣 (WAS)에 세션이 저장되기 때문에     
   2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요합니다.     
   
2. MySQL과 같은 데이터베이스를 세션 저장소로 사용합니다.
   * 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법입니다.      
   * 많은 설정이 필요 없지만, 결국 로그인 요청마다 DB IO가 발생하여 성능상 이슈가 발생할 수 있습니다.   
   * 보통 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도에서 사용합니다.  
     
3. Redis, Memcached 와 같은 메모리 DB를 세션 저장소로 사용한다.     
   * B2C 서비스에서 가장 많이 사용하는 방식입니다.    
   * 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요합니다.  
   
여기서 2번째 방식인 **데이터베이스를 세션 저장소로 사용하는 방식**을 선택하여 진행하겠습니다.  
선택한 이유는 **설정이 간단**하고 사용자가 많은 서비스가 아니며 비용 절감을 위해서입니다.   
   
이후 AWS에서 이 서비스를 배포하고 운영할 때를 생각하면 레디스와 같은 메모리 DB를 사용하기는 부담스럽습니다.   
왜냐하면, 레디스와 같은 서비스(엘라스틱 캐시)에 별도로 사용료를 지불해야 하기 때문입니다.   
    
사용자가 없는 현재 단계에서는 데이터베이스로 모든 기능을 처리하는게 부담이 적습니다.   
만약 본인이 운영 중인 서비스가 커진다면 한번 고려해 보고, 이 과정에서는 데이터베이스를 사용하겠습니다.  

## 4.1. spring-session-jdbc 등록  
먼저 ```Build.gradle``` 에 다음과 같이 의존성을 등록합니다.  




