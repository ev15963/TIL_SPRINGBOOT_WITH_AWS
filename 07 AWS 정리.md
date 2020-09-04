# 목차
**[1. EC2](#ec2)**     
**[2. RDS](#rds)**     
**[3. 배포](#배포)**    
**[4. AWS 와 OAuth2](#AWS-와-OAuth2)**    

# EC2
EC2 는 AWS에서 제공하는 성능, 용량등을 유동적으로 사용할 수 있는 서버입니다.   

0. AWS 서비스에서 EC2 검색 
1. EC2 인스턴스 시작 
2. 알맞은 OS 선정 -> 아마존 리눅스1 AMI 선정 (2는 유료)
3. 인스턴스 유형에는 프리티어로 표기된 것 사용 (무료로 사용가능하다는 말)   
4. 세부정보는 설정하지 않고 넘어가기
5. 스토리지는 30GB 까지 프리티어로 사용가능하므로 업시킨다.     
6. 여러 인스턴스가 있을 경우 태그별로 구분하면 검색이나 그룹짓기 편하므로 태그를 추가해준다.  
7. 보안그룹은 특정 포트만 허용하는 것이므로 22(ssh),8080(http),443(https)을 설정해준다.     
8. 검토하고 문제없으면 다음으로 
9. pem 키를 생성한다. -> EC2 접근을 하기 위한 키 -> 이름을 입력하고 키 페어 다운로드를 클릭  
  * 인스턴스는 지정된 pem키와 매칭되는 공개키를 가지고 있어 해당 pem 키외에는 허용하지 않는다.   
  * 또한 일종의 마스터키이기 때문에 유출하면 안된다.  
  * 잘 관리할 수 있는 디렉토리로 저장하자  
10. ec2 목록으로 이동
11. 생성된 인스턴스를 클릭해보면 IP와 도메인을 확인할 수 있다.    
12. 인스턴스도 하나의 서버이기에 IP가 존재하는데 아마존에서는 인스턴스를 중지하고 다시 켤때 새 IP를 할당한다.   
13. 매번 확인하고 고쳐주는 귀찮은 작업을 해결하기 위해 탄력적 IP를 도입해준다.  
  * EC2 인스턴스 페이즈의 왼쪽 카테고리에서 탄력적IP를 눌러 선택하고 주소가 없으므로 [새 주소 할당] 클릭  
  * 아마존 풀 클릭 하고 할당하면 생성됨
14. 기존 EC2와 탄력적 IP연결 
  * 방금 생성한 탄력적 IP를 확인하고 페이지 위에 있는 [작업]->[주소 연결] 클릭
  * 연결을 위해 EC2 이름과 IP를 선택하고 연결 클릭
  * 연결이 완료되면 왼쪽 카테고리의 인스턴스 목록의 인스턴스 클릭
  * 해당 인스턴스의 퍼블릭, 탄력적 IP가 모두 잘 연결되었는지 확인
  * 탄력적 IP는 주소연결을 안하면 비용발생하므로 꼭 연결하기  
15. EC2서버에 접속하기
  * ```ssh -i pem '키 위치' 'EC2의 탄력적 IP주소' ``` 입력 
  * 하지만 매번 긴 명령어를 입력하는 것은 귀찮으므로 쉽게 접속해보자
  * pem 파일을 ```./ssh```로 복사한다.
  * ```./ssh``` 디렉터로리 pem 파일을 옮겨 놓으면 ssh 실행기 pem키 파일을 자동으로 읽어온다.  
  * 이후 별도로 pem 키 위치를 명령어로 지정할 필요가 없게 됩니다.  
  * 복사하기 ```cp 'pem 키를 내려받은 위치(.pem파일)' ~./ssh```
  * 이동후 확인하기 ```cd ~./ssh`` -> ```ll``` 있는지 확인
  * 복사가 되었다면 ```chmood 600 ~/.ssh/pem키 이름``` 으로 권한 변경 -> 읽고 쓰기  
  * 권한을 변경하였다면 pem 키가 있는 ```./ssh``` 디렉토리에 config 파일 생성
  * ```vim ~/.ssh/config```
  ```
  # 주석
  HostName ec2의 탄력적 IP주소 
  User ec2-user
  IdentityFile ~/.ssh/pem키 이름
  ```
  * :wq로 저장
  * ```chmod 700 ~/.ssh/config```로 권한 변경
  * ssh config에 등록한 서비스명 
  * yes 입력 
  * 접속 성공 
  
## EC2 서버에서 꼭 설정해야할 요소들 
   
* 자바8 설정 : 
* 타임존 변경 : 기본 서버의 시간은 미국 시간대이므로 한국 시간대가 되어야 우리가 사용하는 시간이 모두 한국 시간으로 등록되고 사용  
* 호스트 네임 변경 : 햔제 접속한 서버의 별명을 등록,  
실무에서는 한 대의 서버가 아닌 수십 대의 서버가 작동되는데,   
IP만으로 어떤 서버가 어떤 역할을 하는지 알 수 없습니다.   
이를 구분하기 위해 보통 호스트 네임을 필수로 등록합니다.  

### Java 8 설치   
```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```
자바8 설치 
```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```
EC2 인스턴스의 자바 버전을 8로 설정
```
sudo /usr/sbin/alternatives --config java
```
```
1.   자바7
2.   자바8
```
이런식으로 뜰 텐데 자바8이 있는 번호 선택 -> 2번 선택   
```
sudo yum remove java-1.7.0-openjdk
```
자바7 삭제   
```
java -version
```
자바 버전 확인 -> 8이면 완벽    
    
### 타임존 변경   
EC2 서버의 기본 타임존은 UTC입니다.   
이는 세계 표준 시간으로 한국의 시간대가 아닙니다. -> 9 시간 차이   
이렇게 되면 서버에서 수행되는 Java 애플리케이션에서 생성되는 시간도 모두 9시간씩 차이가 나기 때문에 수정해야한다.   
서버의 타임존을 한국 시간으로 변경하겠습니다.   
    
```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```
정상적으로 수행되었다면 date 명령어로 타임존이 KST로 변경된 것을 확인할 수 있습니다.   
    
### Hostname 변경    
여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어렵습니다.     
그래서 각 서버가 어느 서비스인지 표현하기 위해 HOSTNAME을 변경하겠습니다.       
다음 명령어로 편집 파일을 열어봅니다.   
   
```
sudo vim /etc/sysconfig/network    
```
화면에서 노출되는 항목 중 HOSTNAME으로 되어 있는 부분을 **본인이 원하는 서비스명으로 변경합니다.**    
```
NETWORK=yes
HOSTNAME=localhost.localdomain <- 여기
NOZEROCONF=yes
```
```
NETWORK=yes
HOSTNAME=freelec-springboot2-webservice <- 여기  
NOZEROCONF=yes
```
변경한 후 다음 명령어로 서버를 재부팅 합니다.   
```
sudo reboot
```
재부팅이 끝나고 나서 다시 접속해 보면 HOSTNAME이 잘 변경된 것을 확인할 수 있습니다.    

___  

Hostname이 등록되었다면 한 가지 작업을 더 해야 합니다.      
호스트 주소를 찾을 때 가장 먼저 검색해보는 ```/etc/hosts```에 변경한 hostname을 등록합니다.     
만약 하지 않았을 경우 https://woowabros.github.io/experience/2017/01/20/billing-event.html 들어가보기   
    
내용요약 : 
과거 버전에서 hostname이 local 호스트이면 바로 반환하게끔 했지만 
localhost가 아닐 경우 lookupAllHostAddr() 메서드가 사용된다.   
이름 그대로 모든 host 주소를 찾는 것이기에 시간이 오래걸리고 (6초)     
시간이 오래걸리다 보니 빌링 데이터베이스의 connect_timeout 설정값이 5초였기 때문에 당연히 에러가 발생   
mariadb-connector-j 에서는 JNA를통하여 현재 pid만 반환하도록 하는 내용이 commit되었다.   

우리는 호스트 네임을 localhost 가 아닌 판별을 위한 이름을 지어주었으므로   
/etc/hosts에 IP주소와 hostname을 등록하여 최우선적으로 IP 주소를 반환하도록 합니다.    

```
sudo vim /etc/hosts
```
해당 폴더에 들어가서 
```
127.0.0.1 등록한 호스트네임
```
위와 같이 바꿔준다.  
```:wq```로 저장하고  
정상적으로 등록이 되었는지 확인하기 위해 ```curl 등록한 호스트네임```을 입력해본다.   
만약 잘못 등록되었다면 찾을 수 없는 주소라 뜨고      
잘 등록되었다면 80번 포트로 접근이 안된다는 에러가 발생합니다.     
  
이는 아직 80번 포트로 실행된 서비스가 없음을 의미합니다.(우리는 8080 사용함)   
즉, curl 호스트 이름으로 실행은 잘 되었음을 의미합니다.   
   
# RDS 
AWS에서는 모니터링, 알람, 백업, HA 구성등을 모두 지원하는 관리형 서비스인 RDS를 제공합니다.      
RDS는 AWS에서 지원하는 클라우드 기반 관계형 데이터베이스입니다.  
하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화하여    
개발자가 개발에 집중할 수 있게 지원하는 서비스입니다.      
추가로 조정 가능한 용량을 지원하여    
예상치 못한 양의 데이터가 쌓여도 비용만 추가로 내면 정상적으로 서비스가 가능한 기능도 있습니다.    

1. 서비스에 RDS라 검색
2. [데이터베이스 생성] 클릭 
3. 아마존 자체 데이터베이스인 Amazone Aurora 로 교체하기 쉽고 무료인 MariaDB 선택  
4. 무료버전인 프리티어 클릭 
5. 스토리지에서 범용(SSD), **할당된 스토리지는 20으로 바꿔줍니다**  
6. 화면을 내리면 DB 인슽턴스와 마스터 정보를 등록할 수 있다.  
 * 본인만의 DB 인스턴스 이름과 사용자 정보를 등록합니다.  
 * 여기서 입력된 정보로 실제 데이터베이스에 접근하게 되니 어딘가 내용을 메모합시다.  
7. 네트워크 퍼블릭 에세스를 [예] 로 변경합니다. -> 이후 보안 그룹에서 저장된 IP로만 접근하도록 막을 것이므로 상관x   
8. 데이터베이스 이름을 정하고 포트는 3306, db 파라미터 그룹은 옵션 그룹이랑 동일하게 맞춘다.  
9. [완료] 버튼 클릭 
10. [DB 인스턴스 세부 정보 보기]를 클릭하여 상태를 확인하고 다 생성되었다면 본격적으로 설정 시작   

## RDS 운영환경에 맞는 파라미터 설정 
1. 타임존 
2. Character Set 
3. Max Connection   
   
1. 왼쪽 카테고리에서 [파라미터 그룹] 탭을 클릭해서 이동합니다.     
2. 화면 오른쪽 위의 [파라미터 그룹 생성] 버튼을 클릭하빈다.    
3. 세부 정보 위쪽에 DB 엔진을 선택하는 항목이 있는데 **방금 생성한 MariaDB와 같은 버전으로 맞춰줘야합니다.**    
4. 생성이 완료되면 파라미터 그룹 목록창에 새로 생성된 그룹을 볼 수 있고 해당 파라미터 그룹을 클릭합니다.     
5. 이동한 페이지의 오른쪽 위쪽을 보면 [파라미터 편집] 버튼이 있으며 해당 버튼을 클릭해줍니다.     
6. time_zone 을 검색하여 [Asia/Seoul]을 선택합니다.      
7. 다음으로 Character Set을 변경합니다.           
**character 항목들은 utf8mb4 로**      
**collation 항목들은 utf8mb4_general_ci 로 변경합니다**         
utf8과 utf8mb4의 차이는 **이모지 저장 가능 여부입니다**     
utf8mb4는 이모지를 저장할 수 있으므로 해당 인코딩을 사용합니다.         
  
 * character_set_client  
 * character_set_connection 
 * character_set_database    
 * character_set_filesystem  
 * character_set_results    
 * collation_connection   
 * collation_server     
    
8. 마지막으로 Max Connection을 수정합니다.   
RDS의 MaxConnection은 인스턴스 사양에 따라 자동으로 정해집니다.    
현재 프리티어 사양으로는 약 60개의 커넥션만 가능해서 좀 더 넉넉한 값으로 지정합니다. (150)     
이후에 RDS 사양을 높이게 된다면 기본값으로 다시 돌려 놓으면 됩니다.     
설정이 다 되었다면 오른쪽 위의 [변경사항 저장]또는[수정] 버튼을 클릭해 최종 저장합니다.      
9. 우리가 4번 처럼 파라미터 그룹을 만들어주어도 db파라미터 그룹의 기본은 default 입니다.      
그렇기에 왼쪽 카테고리의 데이터베이스에서 옵션항목에서 방금 생성한 파라미터 그룹으로 변경해줍니다.     
10. 저장을 하면 수정 사항이 요약된 것을 볼 수 있으며 여기서 반영시점을 [즉시 적용]으로 합니다. (예약-새벽시간)   
11. 그럼 데이터베이스가 변경중이라는 상태가 됩니다.  
12. 간혹 변경이 안될 때도 있으므로 정상적용을 위해 한번더 재부팅을 해줍니다.  

## EC2 RDS 연결하기    
1. RDS의 세부정보 페이지에서 [보안 그룹] 항목내에 있는 하이퍼링크를 클릭합니다.   
2. RDS의 보안 그룹에 있는 하이퍼링크를 클릭하여 세부 정보로 들어간다. 
3. RDS는 잠시 멈추고 새 브라우저를 띄운뒤 EC2로 가서 왼쪽 카테고리 보안그룹으로 들어간다.
4. 보안 그룹 목록 중 EC2에 사용된 보안 그룹의 그룹 ID를 복사합니다.  
5. 복사된 보안 그룹 ID와 본인의 IP를 가지고 다시 RDS 보안 그룹의 세부 정보로 돌아온다
6. 밑에 인바운드 규칙이 있는데 [인바운드 규칙 편집]을 클릭한다.   
7. 규칙 추가 버튼을 누른후 유형은 [MYSQL/AURORA], 소스는 내 IP
8. 하나더 만듭니다. 규칙 추가 버튼을 누른후 유형은 [MYSQL/AURORA], 소스는 [사용자 정의] 후 복사한 IP 입력   
9. 이렇게 하면 EC2와 RDS간에 접근이 가능해집니다.  
10. EC2 같은 경우 이후에 2대 3대가 될 수 있는데 매번 IP를 등록할 수는 없으니 보편적으로 이렇게 보안 그룹간에 연동을 진행합니다.   

## IntelliJ 와 RDS 연결하기    
1. 이제 IntelliJ 에서 플러그인마켓에 들어간 후 DatabaseNavigator를 검색하고 다운해줍니다.    
2. command shift a 를 눌러서 Action 검색창을 띄운뒤 DatabaseBrowser를 검색후 실행합니다.   
3. 그럼 프로젝트 사이드바 왼쪽에 DB Browser가 노출됩니다.   
4. 다시 자바 프로젝트를 보고 싶다면 하단 탭을 클릭하면 됩니다.   
5. 플러스 버튼을 누른 후 MySQL을 클릭합니다.  
6. 본인이 생성한 RDS 정보를 차례대로 등록합니다.  
7. HOST 부분에는 RDS의 엔드포인트가 필요합니다.
 * RDS 정보 상세 페이지에서 보인&그룹에 있는 엔드포인트를 확인하고 이를 복사하여 붙여넣어줍시다.  
8. 이후 db 정보 입력 화면 아래에 있는 [Test Connection]을 이용하여 연결 테스트를 해봅니다.   
9. Connection Successful 메시지를 보았다면 [Apply->ok] 버튼을 차례로 눌러 최종 확인을 합니다.   
10. 그럼 이제 인텔리제이에 RDS의 스키마가 노출됩니다.  
11. 위쪽에 있는 [Open SQL console]을 클릭하고 [New SQL Console]을 클릭합니다.   
12. 이후 뜨는 창에는 새로 생성될 콘솔창의 이름을 정해줍니다.   
13. 생성된 콘솔창에서 SQL을 실행해보겠습니다. ```user 'AWS RDS 웹콘솔에서 지정한 DB명';``` <- DB 생성시 이름  
14. 만약 본인의 db명을 잊었다면    
인텔리제이의 schema 항목을 보면 MYSQL에서 기본으로 생성하는 스키마 이외에 다른 스키마가 있는데 이 이름을 참고하면된다.   
15. 필자 같은 경우는 ```use freelec_springboot2_webservice;```   
16. 콘솔에 입력을 한후 화살표 아이콘으로 된 [Execute Statement]를 클릭하면 됩니다.   
17. 이후 Execute Console에서 SQL statement executed successfully 메시지가 떴다면 쿼리가 정상적으로 수행된것입니다.   
18. 데이터베이스가 선택된 상태에서 현재의 character_set, collation 설정을 확인합니다.   
 * 콘솔창에 ```show variables like 'c%';``` 입력 
 * 나머지 항목은 괜찮은데 ```character_set_database```,```collation_connection``` 2가지는 latin1 로 되어있습니다.   
 * 이 2개 항목은 MariaDB 에서만 RDS 파라미터 그룹으로는 변경이 안되기에 직접 수정해 줍시다.      
 ```sql
 ALTER DATABASE 데이터베이스명
 CHARACTER SET = 'utf8mb4'
 COLLATE = 'utf8mb4_general_ci';
 ```
 * 쿼리를 수행후 다시한번 character set을 확인해줍니다.   
 * 성공적으로 모든 항목이 변경되었습니다!
19. 이제 타임존도 정확한지 확인합시다.   
 * ```select @@time_zone, now();```를 콘솔에 입력하여 실행합니다.  
 * [Asia/Seoul]로 되어있으면 잘 된 것이고 RDS 파라미터 그룹이 잘 적용된 것을 알 수 있습니다.   
20. 마지막으로 한글명이 잘 들어가는지 확인해주는 insert 쿼리문을 작성해봅시다.      
참고로 테이블 생성은 인코딩이 정확한지 확인후 해야합니다.    
왜냐하면 이후에 바꾸더라도 테이블 생성 당시 설정값을 그대로 유지하기 때문입니다.   
 * 다음 쿼리를 차례대로 실행해봅니다.   
 ```sql
 CREATE TABLE test(
  id bigint(20) NOT NULL AUTO_INCREMENT,
  content varchar(255) DEFAULT NULL,
  PRIMARY KEY (id)
 ) ENGIN=InnoDB;
 
 insert into test(content) values('테스트');
 
 select * from test;
 ```
 * 한글데이터가 잘 등록되었는지 확인합니다.   
    
## EC2에서 RDS에서 접근 확인 
1. EC2에 접근을 합니다. (필자는 맥이므로 터미널에서 ssh)   
2. 접속이 되었다면 MySQL 접근 테스트를 위해 MySQL CLI를 설치하겠습니다.   
 * ```sudo yum install mysql```
3. 설치가 다 되었으면 로컬에서 접근하듯이 계정, 비밀번호, 호스트 주소를 사용해 RDS에 접속합니다.   
 * ```mysql -u 아이디 -p -h HOST주소(엔드포인트)```
 * 이후 콘솔에 비밀번호도 입력하라 나오면 비밀번호도 입력 ```비밀번호```
 * 접속이 완료되면 EC2에서 RDS로 접속이 된다는 것을 알 수 있습니다.   
4. RDS에 접속되었으면 실제로 생성한 RDS가 맞는지 간단한 쿼리르 한번 실행해보겠습니다.   
 * ```show database;```  
5. 우리가 생성한 데이터베이스가 있으면 완벽히 검증한 것입니다.   

# 배포 
## EC2에 프로젝트 Clone 받기   
1. ec2에 접속
2. ```sudo yum install git```으로 깃 다운   
3. ```git --version``` 으로 버전 확인이지만 이로인해 설치가 완료되었는지 확인 
4. 이제 ```git clone```명령어로 프로젝트를 저장할 공간 생성
 * ```mkdir ~/app && mkdir ~/app/step1```   
5. ```cd ~/app/step1``` 명령어를 통해 생성된 디렉토리로 이동   
6. 본인의 깃허브에 들어가서 해당 레포지토리의 http 주소를 복사    
7. 복사한 주소를 가지고 ec2에서 이동한 디렉토리에서 ```git clone 복사한 주소``` 명령어 수행하여 복사  
8. git clone 이 끝났다면 생성된 프로젝트 디렉토리로 이동해서 잘 복사되었는지 확인
 * ```cd 프로젝트명```
 * ```ll```
 * 프로젝트 코드들이 모두 있으면 성공임
9. 코드들이 잘 수행되는지 테스트 검증 
 * ```./gradlew test``` 맥기준으로 테스트 실행
 * BUILD SUCCESSFUL 이 뜨면 성공
 * 만약 실패했을 경우에는 코드를 수정하고 깃허브에 푸시한뒤 ```git pull``` 로 바뀐 코드랑 현재 코드랑 merge 해준다.   
 * 만약 ```-bash: ./gradlew: Permission denied```와 같이 그래들 gradlew 실행 권한이 없다는 메시지가 뜬다면      
 ```chmod +x ./gradlew```로 실행 권한을 얻어 다시 수행하면 ㄷ뵈니다.   
10. 참고로 그래들을 다운 받지 않았는데 사용 가능한 이유는 Wrapper 파일 덕분이다. -> TIL_FIRST_SPRINGBOOT 참고바람     

## 배포 스크립트 만들기   
배포 : 작성한 코드를 실제 서버에 반영하는 것       
   
여러 배포의 의미들      
* git clone 혹은 git pull을 통해서 새버전의 프로젝트를 받음   
* Gradle이나 Maven을 통해 프로젝트 테스트와 빌드   
* EC2 서버에서 해당 프로젝트 실행 및 재실행    
      
앞선 과정에서는 배포할때마다 git clone 부터 다시 반복해주어야 한다는 문제점이 있다.      
즉, **배포할 때마다 개발자가 하나하나 명령어를 실행**하는 것이고 불편함이 많아집니다.      
그래서 이를 **쉡 스크립트로 작성해 스크립트만 실행하면 앞의 과정이 진행되도록 하겠습니다.** 
  
쉘 스크립트 파일의 확장자는 ```.sh```

1. ```vim ~/app/step1/deploy.sh``` 명령어 입력
2. 아래와 같은 스크립트를 넣어준다.   

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=freelec-springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
	echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다"
else
	echo "> kill -15 $CURRENT_PID"
	kill -15 $CURRENT_PID
	sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

nohup java -jar \
	-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
```
    
___
    
```sh
REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=freelec-springboot2-webservice
```
* REPOSITORY 라는 변수를 만들고 프로젝트를 배포할 디렉토리 선정 -> ```/home/ec2-user/app/step1```
* PROJECT_NAME 배포할 프로젝트의 이름을 저장 -> 후에 해당 프로젝트로 이동하기 위해서 
   
___

```sh
echo "> Git Pull"

git pull
```
로그를 남기고 최신버전으로 머지한다.   
   
___   
     
```sh

echo "> 프로젝트 Build 시작"

./gradlew build
```
프로젝트를 빌드하여 jar 파일을 만든다.   
   
___
   
```sh
echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/
```
jar 파일을 넣어둘 위치로 이동하고   
프로젝트 디렉토리에 생긴 jar 파일을 앞서 이동한 위치에 복사한다.   
즉, ```프로젝트디렉토리/프로젝트/jar 파일```을 ```프로젝트디렉토리/```로 뺀다. (프로젝트 밖으로 뺀것이다 && 프로젝트랑 동일 위치)   
      
___
   
```sh
echo "현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
	echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다"
else
	echo "> kill -15 $CURRENT_PID"
	kill -15 $CURRENT_PID
	sleep 5
fi
```
이미 구동중이라면 해당 PID를 얻어서 kill 하고 아니면 놔둔다.   
kill -15 는 kill -9 에 비해 권장되는 방법으로 kil -9는 강제종료이며
kill -15 는 응답 요청 종료라고 보면된다.    
   
___
   
```sh
echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)
```
* 새로 실행할 jar 파일명을 찾습니다.   
* 여러개가 있을 수 있으므로 ```tail -n 1```가장 나중에 생긴 jar 파일명을 가져옵니다.   

___

```sh
echo "> JAR NAME: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
```
실행할 jar 명을 출력하고   
* 찾은 jar 파일명으로 해당 jar 파일을 nohup으로 실행합니다.   
* 내장 톰캣만으로 서버 실행이됩니다.  
* 일반적으로 터미널 종료시 같이 종료되는데 nohup java -jar 이런식으로 실행을 하면 터미널이 끊겨도 종료되지 않습니다.   
* Dspring 옵션으로 active는 real, 그리고 나머지 프로퍼티도 실행되게끔 클래스 패스에 넣어줍니다.      
* 마지막은 ```2>&1 &``` 잘 모르겠습니다.       
   
___   
   
3. 이렇게 생성한 스크립트에 실행 권한을 추가합니다. ```chmod +x ./deploy.sh```   
4. ```ll```로 확인해보면 실행 권한이 추가되었음을 알 수 있습니다.   
5. ```./deploy.sh``` 명령어로 실행시킵니다.   
6. 로그가 출력되면 애플리케이션이 실행됩니다.   
7. ```vim nohup.out``` 명령어로 nohup 파일을 열어 로그를 확인해봅니다.   
8. nohup 파일 맨 마지막에 보면 ```APPLICATION FAILED TO START``` 가 뜨면서 실패했음을 알 수 있다.  
9. 또한 내용을 보면 ClientRegistrationRepository를 찾을 수 없다는 에러가 발생했다.   
10. 필자 기준 여러 액션에 관련된 프로퍼티를 만들고 oauth를 사용하다 보니 외부 시큐리티를 등록해주어야 했다.   

## 외부 Security 파일 등록하기   
ClientRegistrationRepository를 생성하려면 ClientId 와 ClientSecret이 필수입니다.   
로컬PC에서 실행할때는 application-oauth-properties가 있어서 괜찮았지만      
해당 파일을 우리는 **```.gitignore```로 지정해서 깃허브에 올라왔을 때는 제외되어 올라왔고 우리는 없는 상태로 배포했습니다.**        
   
**그래서 우리는 서버에 디렉토리를 만들고 해당 디렉토리에 따로 ```application-oauth-properties``` 를 만들어주겠습니다.**   
1. 기존 저희 위치인 ```/home/ec-user/app/step1```이 아닌 ```/home/ec2-user/app``` 으로 이동하겠습니다. 
2. ```cd /home/ec2-user/app```   
3. 필요한 프로피터를 만들어줍니다 ```vim /home/ec2-user/app/application-oauth-properties```  
4. 기존에 저장된 내용들을 그대로 복사 붙여넣기 해주신후 ```:wq```로 저장합니다.  
5. 그리고 방금 저장한 프로퍼티를 사용하도록 ```deploy.sh``` 파일을 수정하겠습니다.   
6. ```vim ~/app/step1/deploy.sh``` 입력   
7. 아래와 같이 수정
```sh
nohup java -jar \
	-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
```
* 스프링 설정 파일 위치를 지정하여 ```classpath:/application.properties``` 와   
OAuth 설정을 담은 ```application-oauth.properties``` 위치를 지정합니다.   
* ```classpath:/```는 jar 안에 있는 resource 디렉토리를 기준으로 경로가 생성됩니다.   
* ```application-oauth.properties```는 외부에 있기 때문에 ```/home/ec2-user/app/```같은 절대경로를 사용합니다.     
   
**전체 코드**
```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=freelec-springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
	echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다"
else
	echo "> kill -15 $CURRENT_PID"
	kill -15 $CURRENT_PID
	sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

nohup java -jar \
	-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
```
8. 수정이 완료되었다면 ```./deploy.sh``` 또는 ```~/app/step1/deploy.sh```입력하여 쉡 스크립트를 실행합니다.   
9. 그럼 오타가 있지 않은 이상 정상적으로 실행될 것입니다.     

# 스프링 부트 프로젝트로 RDS 접근하기    
학습기준으로 RDS MariaDB를 사용중입니다.        
MariaDB에서 스프링부트 프로젝트를 실행하기 위해서는 몇가지 작업이 더 필요합니다.        
       
* 테이블 생성 : H2에서 자동 생성해주던 테이블들을 MariaDB에선 직접 쿼리를 이용해 생성합니다.       
* 프로젝트 설정 : 자바 프로젝트가 MariaDB에 접근하려면 데이터베이스 드라이버가 필요합니다.         
MariaDB에서 사용 가능한 드라이버를 프로젝트에 추가합니다.        
* EC2-리눅스 설정 : 데이터베이스의 접속 정보는 중요하게 보호해야 할 정보입니다.          
공개되면 외부에서 데이터를 모두 가져갈 수 있기 때문입니다.           
프로젝트 안에 접속 정보를 갖거 있다면 깃허브와 같이 오픈된 공간에서 누구나 해킹할 위험이 있습니다.         
EC2 서버 내부에서 접속 정보를 관리하도록 설정합니다.     

## RDS 테이블 생성      
1. JPA가 사용될 엔티티 테이블   
2. 스프링 세션이 사용될 테이블    
    
위 2가지 테이블을 생성할 것입니다.        
**JPA가 사용할 테이블은 테스트 코드 수행 시 로그로 생성되는 쿼리를 사용하면 됩니다.**       
기존 절차대로 진행했다면 테스트 코드 수행시 발생하는 로그에서 SQL쿼리문을 가져오면 됩니다.     

1. 데이터베이스를 사용해야하므로 콘솔창에 ```use 데이터베이스```를 사용해서 데이터베이스 사용을 선언합니다.       
	* ```use freelec_springboot2_webservice;```    
2. 아래 쿼리문 2개를 콘솔창에 넣어줍니다.  
```sql
create table posts (id bigint not null auto_increment, created_date datetime, modified_date datetime, author varchar(255), content TEXT not null, title varchar(500) not null, primary key (id)) engine=InnoDB
```
```sql
create table user (id bigint not null auto_increment, created_date datetime, modified_date datetime, email varchar(255) not null, name varchar(255) not null, picture varchar(255), role varchar(255) not null, primary key (id)) engine=InnoDB;
```
     
___      
   
스프링 세션 테이블은 schema-mysql.sql 파일에서 확인할 수 있습니다.   
1. 파일 검색 (Mac 기준 - Command Shift O)를 눌러줍니다.   
2. schema-mysql.sql 검색합니다.  
3. 아래와 같은 sql문이 있고 이를 기존 콘솔창에 추가합니다.   
```sql
CREATE TABLE SPRING_SESSION (
	PRIMARY_ID CHAR(36) NOT NULL,
	SESSION_ID CHAR(36) NOT NULL,
	CREATION_TIME BIGINT NOT NULL,
	LAST_ACCESS_TIME BIGINT NOT NULL,
	MAX_INACTIVE_INTERVAL INT NOT NULL,
	EXPIRY_TIME BIGINT NOT NULL,
	PRINCIPAL_NAME VARCHAR(100),
	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
	ATTRIBUTE_BYTES BLOB NOT NULL,
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
RDS에 필요한 테이블은 모두 생성했으니 프로젝트로 넘어갑니다.    

## 프로젝트 설정   
```build.gradle```에 MariaDB 관려 드라이버를 추가해줍니다.  
```gradle
    compile('org.mariadb.jdbc:mariadb-java-client')
```

그리고 서버에서 구동될 환경을 하나 구성합니다.(properties)     
1. ```src/main/resources/```에 ```application-real.properties``` (profile 이 real)
2. 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하면 RDS 환경 profile 설정이 추가됩니다.   
    
**application-real.properties**   
```properties
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.session.store-type=jdbc
```   
모든 설정이 완료되었다면 깃허브로 푸시합니다.   

## EC2 설정   
OAuth 와 마찬가지로 RDS 접속 정보도 보호해야할 정보이니    
EC2 서버에 ```/home/ec2-user/app/application-oauth.properties```를 만들었듯이    
```/home/ec2-user/app/application-real-db.properties```를 만들어주도록 한다.     

1. ec2 서버에 접속 
2. ```vim ~/app/application-real-db.properties``` 명령어 입력    
3. 아래와 같은 코드 입력     
   
**application-real-db.properties**   
```properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://{RDS주소}:{포트명-기본 3306}/{db 이름}
spring.datasource.username={db 계정}
spring.datasource.password={db 비밀번호}
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```
```properties
spring.jpa.hibernate.ddl-auto=none
```
우리가 실제 운영으로 사용될 테이블은 JPA의 자동생성을 해주면 안되므로 이러한 처리를 해줍니다.      
만약 자동 생성을 했다면 기존 데이터가 날라가는 현상이 발생하므로 각별히 신경써야 합니다.     
   
___
   
마지막으로 사용할 프로퍼티를 만들었으므로 기존 ```deploy.sh```의 내용도 조금 수정해줍니다.   

```sh
nohup java -jar \
	-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
```
* profile을 real로 주어 ```application-real-properties``` 을 활성화 시킵니다.       
* ```application-real-properties``` 내용을 보면       
```spring.profiles.include=oauth,real-db```를 가지고 있어 다른 프로필도 활성화 시켜 다른 프로퍼티도 사용가능해집니다.      
* ```/home/ec2-user/app/application-real-db.properties``` 는 외부에 있는것이므로 설정파일에 추가시켜줍니다.     
   
이후 deploy.sh를 실행하고 nohup.out 으로 로그를 확인해봅니다. (성공 여부 판별)   
```
curl localhost:8080
```
또한 curl 명령어로 html 코드가 정상적으로 보인다면 성공입니다.    

# AWS 와 OAuth2   
## AWS 보안 그룹 변경   
1. ec2에 스프링 부트 프로젝트가 8080포트로 배포되어 있으므로, 8080포트가 보안 그룹에 열려 있는지 확인합니다.     
2. 보안 그룹에 8080 포트가 열려있지 않다면 [인바운드 규칙 편집]을 통해 추가해줍니다.     
3. 2가지 인바운드를 추가해줄 것입니다.     
	* [사용자 지정 TCP]-[TCP]-[8080]-[0.0.0.0/0]-[공백]     
	* [사용자 지정 TCP]-[TCP]-[8080]-[::/0]-[공백]     

## AWS EC2 도메인으로 접속     
1. AWS 브라우저에서 왼쪽 사이드바의 [인스턴스 메뉴]를 클릭합니다.        
2. 본인이 생성한 EC2 인스턴스를 생성하면 상세 정보에서 **퍼블릭 DNS**를 확인할 수 있습니다.       
3. 이 주소가 EC2에 자동으로 할당된 도메인입니다. (이 주소를 통해 EC2 서버 접속가능 -> 웹 주소)         
4. 기존 우리는 SNS OAuth 등록을 localhost만 해주었기에 위 주소를 통해 추가로 설정해주어야 합니다.  
5. 
   
