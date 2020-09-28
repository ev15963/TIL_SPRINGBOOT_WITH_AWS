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
	      proxy_pass $service_url;
                proxy_set_header X-Real_IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
```
```
	      proxy_pass $service_url;
```
* NGINX 로 요청이 오면 `http://localhost:8080`으로 전달합니다.  

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

조금있다가 추가 
