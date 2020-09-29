08 Travis CI 배포 자동화
=======================
여러 개발자의 코드를 실시간으로 병합하고,     
테스트가 수행되는 환경, master 브랜치가 푸시되면 배포가 자동으로 이루어지는 환경을 구축해줍니다.      
   
# 1. CI & CD    
* CI : VCS(버전관리) 시스템 등에 PUSH가 되면 자동으로 테스트와 빌드가 수행되어 **안정적인 배포 파일을 만드는 과정**        
  * CI == 지속적인 통합        
* CD : 빌드 결과를 자동으로 운영 서버에 **무중단 배포까지 진행되는 과정**       
  * CD == 지속적인 배포      
   
일반적으로 CI만 구축되어 있지는 않고, CD도 함께 구축된 경우가 대부분입니다.     
   
**여기서 주의할 점은 단순히 CI 도구를 도입했다고 해서 CI를 하고 있는 것은 아닙니다.**     
마틴 파울러는 CI에 대해 다음과 같은 4가지 규칙을 이야기합니다. (http://bit.ly/2Yv0vFp)   
   
* 모든 소스 코드가 살아 있고(실행 되고) 누구든 현재의 소스에 접근할 수 있는 단일 지점을 유지할 것        
* 빌드 프로세스를 자동화해서 누구든 소스로부터 시스템을 빌드하는 단일 명령어를 사용할 수 있게 할 것         
* 테스팅을 자동화 해서 단일 명령어로 언제든지 시스템에 대한 건전한 테스트 수트를 실행할 수 있게 할 것     
* 누구나 현재 실행 파일을 얻으면 지금까지 가장 완전한 실행 파일을 얻었다는 확신을 하게 할 것     
     
여기서 특히나 중요한 것은 테스팅 자동화입니다.     
지속적으로 통합하기 위해서는 무엇보다 이 프로젝트가 완전한 **상태임을 보장**하기 위해 테스트 코드가 구현되어 있어야만 합니다.     
TDD 에 대한 전반적인 이해가 필요하신 분들은 ->      
https://www.youtube.com/watch?v=wmHV6L0e1sU&index=7&t=1538s&list=PLagTY0ogyVkIl2kTr08w-4MLGYWJz7lNK   
     
***
# 2. Travis CI 연동하기    
Travis CI는 깃허브에서 제공하는 무료 CI 서비스입니다.        
젠킨스와 같은 CI 도구도 있지만, 젠킨스는 설치형이기 때문에 이를 위한 EC2 인스턴스가 하나 더 필요합니다.     
이제 시작하는 서비스에서 배포를 위한 EC2 인스턴스는 부담스럽기 때문에 오픈소스 웹 서비스인 TravisCI를 사용하겠습니다.     

```     
AWS에서 Travis CI와 같은 CI 도구로 CodeBuild를 제공합니다.   
하지만 빌드 시간만큼 요금이 부과되는 구조라 초기에 사용하기에는 부담스럽습니다.   
실제 서비스되는 EC2, RDS, S3 외에는 비용 부분을 최소화 하는 것이 좋습니다.   
```
   
## 2.1. Travis CI 웹 서비스 설정   
1. https://travis-ci.org/ 에서 깃허브 계정으로 로그인합니다.  
2. 왼쪽 위 [계정명 => Settings]를 클릭합니다.   
3. 이동된 설정 페이지 아래쪽을 보면 깃허브 저장 검색창이 있습니다.   
여기서 저장소 이름을 입력해서 찾은 다음 오른쪽의 상태바를 활성화 시킵니다.  
4. 활성화한 저장소를 클릭하면 저장소 빌드 히스토리 페이지로 이동합니다.   

## 2.2. 프로젝트 설정   
Travis CI의 상세한 설정은 프로젝트에 존재하는 ```.travis.yml``` 파일로 할 수 있습니다.    
그럼 이제부터 ```.travis.yml``` 파일로 Travis CI 설정을 진행해보겠습니다.     

1. ```build.gradle```과 같은 위치에서 ```.travis.yml``` 을 생성합니다.   
2. 아래와 같은 코드를 입력합니다.  

**.travis.yml**
```yml
language: java
jdk:
  - openjdk8
branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - kwj1270@naver.com
```
   
___
   
```yml
language: java
jdk:
  - openjdk8
```
서버에서 실행되는 프로그래밍 언어와 해당 jdk 입력   
참고로 우리는 EC2에서 일반 jdk8이 아닌 openjdk8을 다운 받아 사용하므로 이렇게 해준다.   
   
___
   
```yml
branches:
  only:
    - master
```
* Travis CI 를 어느 브랜치가 푸시될 때 수행할지 지정합니다.   
* 현재 옵션은 오직 Master 브랜치에 push 될 때만 수행합니다.
     
___
   
```yml   
# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'
```
* gradle을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여,       
같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정   

___
   
```yml
script: "./gradlew clean build"
```
* master 브랜치에서 푸시되었을 때 수행하는 명령어입니다.   
* 여기서는 프로젝트 내부에 둔 gradlew 을 통해 clean & build를 수행합니다.   
* 이는 빌드 디렉토리를 삭제하고 다시 빌드한다는 뜻이다.   
    
___ 

```yml
# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - kwj1270@naver.com
```
* Travis CI 실행 완료시 자동으로 알람이 가도록 설정합니다.   
   
___  
  
위 과정까지 진행되었다면 master 브랜치에 커밋과 푸시를 하고,       
좀전의 travis CI 저장소 페이지를 확인합니다.    
빌드가 성공한 것이 확인되면, .travis.yml 에 등록한 이메일을 확인할 수 있습니다.      
빌드가 성공했다면 성공 이메일이 오고 실패했다면 실패 이메일이 도착합니다.     
   
## 2.3. Travis CI 와 AWS S3 연동하기   
**S3** : AWS에서 제공하는 일종의 **파일 서버**   
이미지 파일을 비롯한 정적 파일들을 관리하거나 지금 진행하는 것처럼 배포 파일들을 관리하는 등의 기능을 지원합니다.   
보통 이미지 업로드를 구현한다면 이 S3를 이용하여 구현하는 경우가 많습니다.     
    
S3를 비롯한 AWS 서비스와 TravisCI를 연동하게 되면 전체 구조는 다음과 같습니다.   
  
[사진]      
   
첫 번째 단계로 Travis CI와 S3를 연동하겠습니다.      
실제 배포는 AWS CodeDeploy라는 서비스를 이용합니다.        
하지만, S3 연동이 먼저 필요한 이유는 **Jar 파일을 전달하기 위해서입니다.**     
     
CodeDeploy는 저장 기능이 없습니다.     
그래서 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요합니다.     
그리고 이러한 공간으로 사용할 것이 바로 AWS S3 입니다.     

```
CodeDeploy가 빌드도 하고 배포도 할 수 있습니다.      
CodeDeploy 에서는 깃 허브 코드를 가져오는 기능을 지원하기 때문입니다.     
하지만 이렇게 할 때 빌드 없이 배포만 필요할 때 대응하기 어렵습니다.      
   
빌드와 배포가 분리되어 있으면 예전에 빌드되어 만들어진 Jar 파일을 재사용하면 되지만   
CodeDeploy가 모든 것을 하게 될 땐 항상 빌드를 하게 되니 확장성이 많이 떨어집니다.   
그래서 웬만하면 빌드와 배포는 분리하는 것을 추천합니다.   
``` 

여태까지의 과정을 요약하면 이렇습니다.     
**기존**  
1. 코드 깃허브에 PUSH
2. EC2에 접속 
3. 깃허브에 있는 코드를 클론, PULL 하여 merge
4. 빌드 진행
5. 빌드로 생긴 jar 파일을 실행
   
**쉘 스크립트**    
1. 코드 깃허브에 PUSH    
2. EC2에 접속   
3. 쉘 스크립트 실행으로 3가지 과정 생략    
    
**Travis**   
1. 코드를 push 하면서 테스트와 빌드 동시에 진행   
2. 테스트 진행시나 빌드 진행시 문제 있으면 이메일로 알람       
3. 코드 검증뿐만 아니라 테스트&빌드를 진행하면서 생긴 JAR 파일을 사용or전달 가능     
4. 다르게 보자면 3번 과정에서의 jar 파일은 테스트와 빌드가 성공한 안정적인 배포 파일이라는 의미를 준다.     
  
그리고 이제는 Travis CI 와 AWS S3 연동을 진행해보겠습니다.     
      
## 2.4. AWS Key 발급       
일반적으로 AWS 서비스에 외부 서비스가 접근 할 수 없습니다.             
그러므로 **접근 가능한 권한을 가진 Key 를 생성해서 사용**해야 합니다.            
즉, Travis CI 와 AWS 연동을 위한 Key를 발급하겠습니다.         
     
AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 **IAM**이 있습니다.           
**IAM은 AWS에서 제공하는 서비스의 접근 방식과 권한을 관리합니다.**          
이 IAM을 통해 Travis CI가 AWS의 S3와 CodeDeploy에 접근할 수 있도록 해보겠습니다.         
 
1. AWS 웹 콘솔에서 IAM을 검색하여 이동합니다.                 
2. IAM 페이지 왼쪽 사이드바에서 [사용자 -> 사용자 추가] 버튼을 차례대로 클릭합니다.      
3. 생성할 사용자의 이름을 입력하고 액세스 유형을 선택합니다. (엑세스 유형은 프로그래밍 방식 액세스입니다.)    
4. 권한 설정 방식은 [기존 정책 직접 연결]을 클릭해줍니다.   
5. 화면 아래 정책 검색 화면에서 ```s3full```로 검색하여 체크하고 다음 권한으로 CodeDeployFull을 검색하여 체크합니다.     
   * AmazoneS3FullAccess  
   * AWSCodeDeployFull    
   * 참고로 실서비스 제 회사에서는 권한도 S3와 CodeDeploy를 분리해서 관리하고 합니다. (여기서는 합쳐서 관리)        
6. 다음으로 넘어갑니다.     
7. 태그는 Name 값을 지정하는데, 본인이 인지 가능한 정도의 이름으로 만듭니다.   
8. 마지막으로 본인이 생성한 권한 설정 항목을 검토합니다.    
9. 최종 생성되었다면 **액세스 키** 와 **비밀 액세스 키**가 생성됩니다. (Travis CI에서 사용될 키)      
    
위 과정까지 거쳐서 생성한 **액세스 키** 와 **비밀 액세스 키**를 Travis CI에 등록하겠습니다.   

## 2.5. Travis CI에 IAM Key 등록   

1. Travis CI 페이지에 들어와 레포지토리 옆에 Setting 을 통해 상세 설정 화면으로 이동합니다.  
2. 화면 아래로 내려오면 Environment Variables 항목이 있습니다.   
3. 여기에 AWS_ACCESS_KEY, AWS_SECRET_KEY 를 변수로 해서 IAM 사용자에서 발급한 키를 등록합니다.   
   * AWS_ACCESS_KEY: 액세스 키 ID
   * AWS_SECRET_KEY: 비밀 액세스 키
4. 여기에 등록된 값들은 이제 ```.travis.yml``` 에서    
```&AWS_ACCESS_KEY```,```&AWS_SECRET_KEY```라는 이름으로 사용할 수 있습니다.      
   
이제 이 키를 사용해서 jar를 관리할 S3 버킷을 생성하겠습니다.   

## 2.6. S3 버킷 생성       
다음으로 S3에 관해 설정을 진행하겠습니다.        
AWS의 S3 서비스는 일종의 **파일 서버**입니다.    
순수하게 파일들을 저장하고 접근 권한을 관리, 검색 등을 지원하는 파일 서버의 역할을 합니다.     
       
S3는 게시글을 쓸 때 나오는 첨부파일 등록을 구현할 때 많이 이용합니다.(첨부파일 저장소)          
파일 서버의 역할을 하기 때문인데, Travis CI에서 생성된 **Build 파일을 저장하도록 구성하겠습니다.**       
그리고 S3에서 저장된 Build 파일은 이후 AWS의 CodeDeploy 에서 배포할 파일로 가져가도록 구성할 예정입니다.     
 
1. AWS 서비스에서 S3를 검색하여 이동하고 [버킷 만들기]를 클릭하여 버킷을 생성합니다.     
2. 원하는 버킷명을 작성합니다. (**배포할 Zip 파일이 모여있는 장소라는 의미를 가진 이름 추천**)
   * 예제에서는 ```freelec-springboot-build```라는 이름을 사용했습니다.   
3. 다음으로 넘어갑니다.   
4. 버전관리를 관리를 설정해주어야 하지만 별다른 설정이 없어도 되므로 넘어갑니다.  
5. 버킷의 보안과 권한 설정 부분인데 **모든 차단**으로 설정해주어야 합니다.   
   * 실제 서비스에서는 Jar 파일이 퍼블릭일 경우 누구나 내려받을 수 있으므로   
   코드나 설정값, 주요 키값들이 다 탈취될 수 있습니다.      
6. 그리고 퍼블릭이 아니더라도 우리는 IAM 사용자로 발급받은 키를 사용하니 접근 가능합니다.     
그러므로 **모든 액세스를 차단하는 설정에 체크합니다.**   
7. 버킷이 생성되면 버킷 목록에서 확인할 수 있습니다. 이를 확인해줍시다.   
      
S3가 생성되었으니 이제 이 S3로 배포 파일을 전달해보겠습니다.   
   
## 2.7. travis.yml 코드내용 추가     
Travis CI에서 빌드하여 만든 Jar 파일을 S3에 올릴 수 있도록 ```.tavis.yml```에 다음 코드를 추가해줍니다.   
   
```yml
before_deploy:
  - zip -r freelec-springboot2-webservice *
  - mkdir -p deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true
```
      
**.travis.yml 전체 코드**
```yml
language: java
jdk:
  - openjdk8
branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

before_deploy:
  - zip -r freelec-springboot2-webservice * # 해당 디렉토리에 있는 모든 요소들을 zip으로 묶음 
  - mkdir -p deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true
    
# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - kwj1270@naver.com
```
   
___
   
```yml
before_deploy:
  - zip -r freelec-springboot2-webservice *
  - mkdir -p deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip
```
```yml
before_deploy:
```
* deploy 명령어가 실행되기 전에 수행합니다.
  
```yml
  - zip -r freelec-springboot2-webservice *
```
* CodeDeploy는 Jar 파일은 인식하지 못하므로 ```Jar + 기타 설정 파일```들을 모아 **압축**합니다.  
* 그래야 CodeDeploy가 zip 파일을 인식하고 거기서 Jar 파일을 꺼내 사용가능합니다.   
* 현재 위치의 모든 파일을 ```freelec-springboot2-webservice``` 이름으로 압축합니다.   
* 명령어의 마지막 위치는 본인의 프로젝트 이름이어야 합니다.  
   
```yml
  - mkdir -p deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
```
* deploy라는 디렉토리를 Travis CI가 실행 중인 위치에서 생성합니다.   

```yml
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip
```
* ```freelec-springboot2-webservice.zip``` 파일을 
```deploy/freelec-springboot2-webservice.zip``` 로 이동시킵니다.   

___   
   
```yml
deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true
```
```yml
deploy:
```
* S3로 파일 업로드 혹은 CodeDeploy로 배포 등 **외부 서비스와 연동될 행위들을 선언**합니다.    

```yml
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
```
* Travis CI 페이지에서 지정한 변수와 값이 실행시 들어가진다.

```yml
    bucket: kwj1270-freelec-springboot-build # s3
```
* 앞서 생성한 S3의 버킷이름을 넣어준다.

```yml
    region: ap-northeast-2
```
* 지역을 입력 EC2 리전에 맞춰서 입력해주면 되는데 ```ap-northeast-2``` 는 아시아 태평양(서울)을 의미한다.   
* 자세히 알고 싶다면 https://docs.aws.amazon.com/ko_kr/general/latest/gr/rande.html 

```yml
    skip_cleanup: true
```
* Travis CI가 빌드중에 디렉토리를 재설정하고 빌드중에 수행 된 모든 변경 사항을 삭제하지 않도록해준다.

```yml
    acl: private # zip 파일 접근을 private로
```
* zip 파일 접근을 private로 한다.   

```yml
    local_dir: deploy # before_deploy에서 생성한 디렉토리
```
* 해당 위치의 파일들만 S3로 전송합니다.     
* 해당 코드는 before_deploy에서 생성한 deploy 디렉토리를 지정했습니다.     
 
```yml
    wait-until-deployed: true
```
* deploy 가 진행되는 동안 기다린다.   
   
___
   
설정이 다 완료 되었다면 **깃허브로 푸시**합니다.       
Travis CI에서 자동으로 빌드가 진행되는 것을 확인하고, 모든 빌드가 성공하는지 확인합니다.     
   
```
Done. Your build exited with 0 
```
위와 같은 로그가 나온다면 Travis CI의 빌드가 성공한 것입니다.     
그리고 S3 버킷을 가보면 업로드가 성공한 것을 확인할 수 있습니다.      
   
## 2.8. Travis CI 와 AWS S3, CodeDeploy 연동하기     
AWS의 배포 시스템인 CodeDeploy를 이용하기 전에       
배포 대상인 EC2 가 CodeDeploye를 연동받을 수 있게 IAM 역할을 생성해주어야 합니다.     

### IAM 역할 추가하기 
1. 서비스 찾기(검색)에서 `IAM` 입력후 클릭
2. 왼쪽 대시보드의 `액세스 관리` 카테고리의 `역할` 클릭 
   * 기존에는 IAM의 사용자를 만들었는데 역할과의 차이점은?   
      * 사용자 : **AWS 서비스 외에 사용할 수 있는 권한**, 로컬 PC, IDC 서버(데이터센터) 등
      * 역할 : **AWS 서비스에만 할당할 수 있는 권한** , EC2, CodeDeploy, SQS 등
3. 역할 만들기 클릭
4. `AWS 서비스` 블록을 누른 후 EC2 찾아서 클릭 후 `[다음 권한]` 클릭 
5. '권한정책연결' 에서 **AmazonEC2RoleforAWSCodeDeploy** 검색후 체크박스 체크 후 `[다음]` 클릭   
해당 권한은 이름 그대로 EC2가 CodeDeploy를 사용할 수 있도록 하는 것이다.     
6. '태그 추가' 에서 원하는 key 와 value 를 입력한 후 `[다음]` 클릭   
7. 알맞은 역할 이름을 입력해주고 최종적으로 확인 후 `[역할만들기]` 클릭하여 역할 만들기   

### EC2 서비스에 IAM 역할 등록하기 
1. EC2 서비스로 이동후 대시보드에서 `인스턴스`를 클릭하여 EC2 인스턴스로 이동   
2. 본인의 인스턴스에 마우스 오른쪽 클릭   
3. 클릭하면 나오는 목록에서 **`인스턴스 설정`** -> **`IAM 역할 연결/바꾸기('수정'으로 바뀜)`** 클릭     
4. 리스트 박스를 클릭하여 방금 생성한 역할 선택후 `[저장]` 버튼 클릭   
5. 다시 인스턴스에 돌아와서 본인의 인스턴스 재부팅(마우스오른쪽 -> 인스턴스 상태 -> 재부팅)   

### CodeDeploy 에이전트 설치   
AWS CodeDeploy 사용 설명서에 보면    
EC2 각 인스턴스에 'CodeDeploy 에이전트'가 설치되어 실행중이어야 한다고 명시되어 있다.        

'CodeDeploy 에이전트'는 CodeDeploy 를 이용한 배포를 진행할시 필히 설치해야하며        
EC2 서버가 CodeDeploy 이벤트를 수신할 수 있도록 해주는 것이다.       
   
1. cmd 나 terminal 을 이용해서 EC2 서버에 접속   
2. `aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2` 입력    
3. install 이란 파일에 CodeDeploy 에이전트가 설치되었다.   
4. `chmod +x ./install`를 이용하여 현재 디렉토리에 있는 `install` 에 실행권한을 추가한다.
5. `sudo ./install auto`를 이용하여 install 파일로 설치를 진행합니다.   
6. `sudo service codedeploy-agent statis`를 이용하여 Agent가 정상적으로 실행되고 있는지 상태검사를 합니다.
7. `The AWS CodeDeploy agent is running as PID xxx` 메시지가 출력되면 정상입니다.

* 만약 설치 중에 다음과 같은 에러가 발생한다면 루비라는 언어가 설치 안된 상태라서 그렇습니다.   
   * `/usr/bin/env:ruby: No such file or directory
   * 이럴 경우 `yum install`로 루비를 설치하면 됩니다.  
      * `sudo yum install ruby`
      
### CodeDeploy를 위한 권한 생성  
기존 우리는 EC2에 대한 역할을 생성했다.    
하지만 반대로 CodeDeploy 에 대한 권한도 생성해주어야 한다.   


1. 서비스 찾기(검색)에서 `IAM` 입력후 클릭
2. 왼쪽 대시보드의 `액세스 관리` 카테고리의 `역할` 클릭 
3. 역할 만들기 클릭
4. `AWS 서비스` 블록을 누른 후 CodeDeploy 찾아서 클릭 후 `[다음 권한]` 클릭 
5. '권한정책연결' 에서 **AWSCodeDeployRole** 검색 후 체크박스 체크 후 `[다음]` 클릭   
6. '태그 추가' 에서 원하는 key 와 value 를 입력한 후 `[다음]` 클릭   
7. 알맞은 역할 이름을 입력해주고 최종적으로 확인 후 `[역할만들기]` 클릭하여 역할 만들기  

### CodeDeploy 생성   

CodeDeploy 는 AWS의 배포 삼형제 중 하나이다.      

* **Code Commit**
      * 깃허브와 같은 **코드 저장소 역할**
      * private 지원이라는 강점이 있었짐나 깃허브도 private 지원으로 잘 사용 안함
* **Code Build**
      * Tracis CI와 마찬가지로 **빌드용 서비스**      
      * 멀티모듈 사용시 사용해볼만 하지만, 규모가 있을시 **젠킨스/팀시티 등을 더 이용해서** 잘 사용 안함     
* **CodeDeploy**
      * AWS의 **배포 서비스**
      * 대체재 자체가 없음
      * 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포등 많은 기능을 지원       
         
필자의 의문점        
`Code Build` 를 이용해서 빌드를 할 수 있었는데 왜 `Travis CI`를 이용한 것일까?        
일단 기존에 사용한 이유와 같이 곁들어서 정리해보겠다.   
   
* 젠킨스 사용시 서버 1대 더 필요 -> 프리티어 불가   
* Code Build 사용시 IAM 역할을 생성, 단 후에 젠킨스로 변경시 IAM 사용자 생성해야 하므로 경험차 미리 사용 
* Travis CI 는 쉬운 설정을 제공해줌 
* 무료   
 
이제 어느정도 개념은 복습했으니 CodeDeploy를 설치해보겠다.      
   
1. CodeDeploy 서비스로 이동 
2. `[애플리케이션 생성]` 클릭   
3. 생성할 CodeDeploy 애플리케이션 이름 기술 및 컴퓨팅 플랫폼에 `[EC2/온프레미스]` 선택    
4. 생성이 완료되면 `[배포 그룹 생성]` 을 클릭    
5. 배포 그룹 이름과 기술 및 Code Deploy 서비스 역할을 선택하여 등록해준다.(EC2가 아니다)       
6. 배포 유형에는 `현재 위치`를 선택, 만일 서비스가 2대 이상이라면 `블루/그린` 을 선택하면된다.   
7. 화면 구성에서는 `Amazone EC2 인스턴스` 체크박스를 클릭, Key 와 Value에 적절한 값을 넣어준다.   
8. 마지막으로 배포 구성은 `CodeDeployDefault.AllAtOnce`로 선택하고 `로드밸런싱 활성화` 체크는 해제해준다.   
   * 배포 구성이란 한번 배포할 대 몇 대의 서버에 배포할지를 결정합니다.   
   1대씩 배포할지, 30%, 50%로 나눠서 배포할지 등등이 있지만    
   우리는 1대밖에 사용하지 않으므로 전체 배포 옵션으로 선택했습니다.    
   
### Travis CI, S3, CodeDeploy 연동 
먼저 S3 에서 넘겨줄 zip 파일을 저장할 디렉토리를 하나 생성하겠습니다. (빌드된 jar가 있는 zip파일)     

1. EC2 서버에 접속 후 `mkdir ~/app/step2 && mkdir ~/app/step2/zip` 입력      
2. 프로젝트에서 CodeDeploy의 설정을 위한 `appspec.yml` 파일 생성      
   
**`appspec.yml`**
```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwriter: yes
```    
  
```yml
version: 0.0
```
* CodeDeploy 버전을 이야기합니다.   
* 프로젝트 버전이 아니므로 0.0 외에 다른 버전을 사용하면 오류가 발생합니다.

```yml
os: linux
```
* CodeDeploy 가 실행될 os 환경을 기술해줍니다.   
     
```yml
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwriter: yes
```
* `source:` CodeDeploy에서 전달해준 파일 중 destination으로 이동 시킬 대상을 지정합니다.   
루트 경로 `/`를 지정하면 전체 파일을 이야기합니다.     
      
* `destination:` source에서 지정된 파일들을 받을 위치입니다.  
이후 jar 를 실행하는 등은 detination에서 옮긴 파일들로 진행됩니다.   
   
* `overwrite:` 기존에 파일들이 있으면 덮어쓸지를 결정합니다.   
현재 yes라고 했으니 파일들을 덮어쓰게 됩니다.     
    
3. `.travis.yml` 에도 CodeDeploy 내용을 추가합니다.   
    
**`.travis.yml`**   
```yml
deploy:
... S3 설정문...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    key: freelec-springboot2-webservice.zip # 빌드 파일을 압축해서 전달 
    bundle_type: zip # 압축 확장자
    application: freelec-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: freelec-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹 
    region: ap-northeast-2
    wait-until-deployed: true
```
S3 옵션과 유사하지만 CodeDeploy의 애플리케이션 이름과 배포  그룹명을 지정합니다.     

**.travis.yml*
```yml
language: java
jdk:
  - openjdk8
branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

before_deploy:
  - zip -r freelec-springboot2-webservice * # 해당 디렉토리에 있는 모든 요소들을 zip으로 묶음 
  - mkdir -p deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip # deploy로 zip파일 이동

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: kwj1270-freelec-springboot-build # s3
    key: freelec-springboot2-webservice.zip # 빌드 파일을 압축해서 전달 
    bundle_type: zip # 압축 확장자
    application: freelec-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: freelec-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - kwj1270@naver.com
```

CodeDeploy 설정이 완료되었다면 한번 테스트를 위해 프로젝트에서 Commit 하고 Push를 진행해봅니다.     
푸시가 되면 Travis CI가 동작될 것이고      
Travis CI가 끝나면 CodeDeploy 서비스에서 배포까지 수행된것을 알 수 있을 것입니다.      
  
배포가 끝났다면 다음 명령어로 파일들이 잘 도착했는지 확인해보겠습니다.   
```
cd /home/ec2-user/app/step2/zip
```
    
파일 목록들을 확인해봅시다.   
```
ll
```
파일 목록이 정상적으로 있다면 Travis CI, S3, CodeDeploy가 연동되어 배포까지 이루어진것을 알 수 있습니다.     
```
README.md
appspec.yml
build
build.gradle
gradle
gradlew
gradlew.bat
schema.sql
settings.gradle
src
```

# 3. 배포 자동화 구성  
현재 Travis CI, S3, CodeDeploy 연동까지 구현하여        
```Jar파일 + 기타 설정 파일```들을 모아 압축한 zip 폴더까지 EC2로 가져왔습니다. (프로젝트 내용이긴하다.)           
(S3에서 CodeDeploy로 넘어올떄 Jar 파일을 인식하지 못하므로 JAR + 기타 설정파일을 zip하여 가져온 것입니다.)       
(Jar는 build 파일 안에 있습니다.)       
   
이제 이것을 기반으로 시레졸 Jar를 배포하여 실행까지 해보겠습니다.  

## deploy.sh 파일 추가   
1. scripts 디렉토리를 생성합니다. (src,build와 같은 프로젝트 루트 디렉토리 안에서 만듭니다.)    
2. scripts 디렉토리에서 deploy.sh 파일을 생성합니다.   
3. 아래 코드를 입력합니다.  

**deploy.sh**
```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl freelec-springboot2-webservice | grep jar | awk '{print $1}')

echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
```sh

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/
```
* zip 폴더내에 있는 즉, 프로젝트내에 있는 jar 파일을 `step2`로 복사합니다.    

```sh
echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl freelec-springboot2-webservice | grep jar | awk '{print $1}')

echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi
```
* 현재 실행중인 jar 가 있으면 종료시키는 역할을 합니다.   

```sh
echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

```
* jar의 이름을 얻어온 뒤 해당 jar에 실행권한을 부여합니다.   

```sh
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
```
* nohup 방식으로 jar를 실행하고 jar 내의 properties, EC2에 있는 properties, 그리고 profile을 적용합니다.   
    
```sh
$JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
* **nohup 실행시 CodeDeploy 는 무한 대기상태가 됩니다.**   
* 이러한 이슈를 해결하기 위해 nohup.out 파일을 표준 입출력용으로 별도로 사용합니다.   
* 이렇게 하지 않을 경우 `nohup.out`파일이 생기지 않고, CodeDeploy 로그에 표준 입출력이 출력됩니다.   
* nohup이 끝나기 전까지 CodeDeploy도 끝나지 않는다는 의미로 꼭 이렇게 해주도록 합시다.   
  
step1과의 차이점은 `git pull` 을 통해 직접 빌드했던 부분을 제거해준 것입니다.    

## .travis.yml 수정    
현재 프로젝트의 모든 파일을 zip 파일로 만드는데,         
jar 파일은 build 에 있고, appspec.yml, 배포를 위한 스크립트들을 제외하면 불필요한 파일들입니다.        

그렇기에 이 외 나머지는 배포에 필요하지 않으므로 `.travis.yml`을 수정하겠습니다.   

```yml
# 참고로 .travis.yml 에서 작업은 travis 서버에서 동작하는 것이다.   
before_deploy:
  - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip # deploy로 zip파일 이동
```
Travis CI는 특정 파일만 S3로 보내는 것이 불가능하고 디렉토리 단위만 업로드 할 수 있습니다.       
그렇기에 ** Travis CI 서버에 특정 로직을 추가하여 원하는 파일만 zip으로 묶어서 올리겠습니다.**     

* `before-deploy` 디렉토리를 만들어 프로젝트 내에 있는 script, appspec.yml, build를 복사합니다.   
* `before-deploy` 디렉토리를 zip 파일 형태로 만듭니다.   
* `deploy` 디렉토리를 만든 뒤 `deploy/freelec-springboot2-webservice.zip`에 `before-deploy` 파일들을 복사합니다.    
* 후에 `deploy :` S3로 인하여 `deploy/freelec-springboot2-webservice.zip` 파일만 가져갑니다.     

## appspec.yml 수정  
`appspec.yml` 파일에 다음 코드를 추가합니다. 'location', 'timeout', 'runas'   

```yml
permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: deploy.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료한다.
      timeout: 60
      runas: ec2-user
```

**전체 코드**
```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwriter: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
```

```yml
permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user
```
* CodeDeploy 에서 EC2 서버로 넘겨준 파일들을 모두 ec2-user 권한을 갖도록 합니다.   

```yml
hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
```
* CodeDeploy 배포 단계에서 실행할 명령어를 지정합니다.   
* ApplicationStart라는 단계에서 deploy.sh를 ec2-user 권한으로 실행하게 합니다.   
* timeout:60 으로 스크립트 실행 60초 이상 수행되면 실패가 됩니다. (무한정 기다릴 수 없어야하므로)    
   
모든 설정이 완료되었으면 커밋과 푸시를 진행해봅시다.    
Travis CI에서의 로그가 정상적인지   
CodeDeploy 서비스에서 상태가 성공인지 확인하고   
EC2 도메인을 입력해서 확인해봅시다.    
     
## 기타      
실제 배포과정에서는       
`build.gradle` 에서 `version '1.0.1-SNAPSHOT'` 과 같이 스냅샵 번호를 갱신시키면서 배포해줍니다.     
    
AWS가 지원하는 서비스에서는 오류가 발생했을 때 로그를 찾기 힘듭니다.       
CodeDeploy에 관한 로그를 찾고자 한다면 `/opt/codedeploy-agent/deployment-root` 를 찾아보면 됩니다.       
`/opt/codedeploy-agent/deployment-root` 디렉토리로 이동한뒤 목록을 확인해보면 다음과 같은 내용을 볼 수 있습니다.      

**/opt/codedeploy-agent/deployment-root**
```
05as6gfd4ewq5tr6q7-08d94b6c54z131r674q68t4q6 // 임의의 값을 넣었습니다
deployment-instructions
deployment-logs
ongoing-deployment
```
* 맨위에 있는 디렉토리명은 **CodeDeploy ID** 입니다.   
   * 사용자마다 고유한 ID이므로 각기 다른 ID값으로 기술되어 있습니다.  
   * 해당 디렉토리에는 **배포한 단위별로 배포 파일들이 있습니다.**   
   * 본인의 배포 파일이 정상적으로 왔는지 확인해볼 수 있습니다.   
   
* deployment-logs
   * CodeDeploy 로그 파일입니다.   
   * CodeDeploy 로 이루어지는 배포 내용중 표준 입/출력 내용은 모두 여기에 담겨있습니다.  
   * 작성한 echo 내용도 모두 표기됩니다.   
   
## 마치며 
이제 우리는 Master 브랜치에 푸시만 하면 자동으로 EC2에 배포가 됩니다.     
하지만, 배포하는 동안 스프링 부트 프로젝트는 종료 상태가 되어 서비스를 이용할 수 없게 됩니다.       
이를 해결한 과정을 다음 챕터에서 보여드리겠습니다.     
    

