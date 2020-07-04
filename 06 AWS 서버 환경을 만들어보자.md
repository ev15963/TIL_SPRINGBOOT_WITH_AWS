06 AWS 서버 환경을 만들어보자
=======================
외부에서 본인이 만든 서비스에 접근하려면 24시간 작동하는 서버가 필수입니다.      
       
* 집에 PC를 24시간 구동시킨다.         
* 호스팅 서비스(Cafe24, 코리아호스팅등)을 이용한다.       
* 클라우드 서비스(AWS, AZURE,GCP 등)을 이용한다.          
          
일반적으로 비용은 호스팅 서비스나 집 PC를 이용하는 것이 저렴합니다.         
만약 특정시간에만 트래픽이 몰린다면 **유동적으로 사양을 늘릴 수 있는 클라우드가 유리합니다.**       
   
### 클라우드   
인터넷(클라우드)을 통해    
**서버, 스토리지(파일 저장소), 데이터베이스, 네트워크, 소프트웨어, 모니터링 등의 컴퓨팅 서비스를 제공**하는 것입니다.     
단순히 물리 장비를 대여하는 것으로 생각하는데 그렇지 않습니다.  
AWS의 EC2는 서버 장비를 대여하는 것이지만,    
실제로는 그 안의 **로그 관리, 모니터링, 하드웨어 교체, 네트워크 관리 등을 기본으로 지원**하고 있습니다.   
개발자가 직접 해야 할 일을 AWS가 전부 지원을 하는 것입니다.   
        
**IaaS(Infrastructure as a Service)**            
* 기존 물리 장비를 미들웨어와 함께 묶어둔 추상화 서비스입니다.        
* 가상머신, 스토리지, 네트워크, 운영체제 등의 IT 인프라를 대여해 주는 서비스라고 보면 됩니다.      
* AWS 의 EC2, S3 등       
        
**PaaS(Platform as a Service)**           
* IaaS 에서 한번 더 추상화한 서비스입니다.         
* 한번 더 추상화했기 때문에 많은 기능이 자동화되어 있습니다.       
* AWS 의 Beanstalk(빈스토), Heroku(헤로쿠) 등       
           
**SaaS(Software as a Service)**           
* 소프트웨어 서비스를 이야기합니다.        
* 구글 드라이브, 드랍박스, 와탭 등      

### AWS
많은 클라우드 서비스중에 AWS를 사용하려 하고 이유는 아래와 같습니다.    
         
* 첫 가입시 1년간 대부분 서비스가 무료입니다.         
단 서비스 제한이 있는데 이는 각 서비스를 설정할 때 해보겠습니다.      
* 클라우드에서는 기본적으로 지원하는 기능(모니터링, 로그관리, 백업, 복구, 클러스터링 등등)이 많아        
개인이나 소규모일 때 개발에 좀 더 집중할 수 있습니다.    
* 많은 기업이 AWS로 이전 중이기 때문에 이직할 때 AWS 사용 경험은 도움이 됩니다.      
국내에서는 AWS 점유율이 압도적입니다.        
쿠팡, 우아한형제들, 리멤버 등 클라우드를 사용할 수 있는 회사에서는 대부분 AWS를 사용합니다.      
* 사용자가 많아 국내 자료와 커뮤니티가 활성화되었습니다.    
         
참고로 Beanstalk 을 사용하면 대부분의 작업이 간소화되지만, **프리티어로 무중단 배포가 불가능합니다.**     
배포할 때마다 서버가 다운되면 제대로 된 서비스를 만들 수 없으니 무중단 배포는 필수이고 빈스톡은 사용할 수 없습니다.     

# 1. AWS 회원가입  
**AWS 가입을 위해서는 Master 혹은 Visa 카드가 필요합니다.**      
준비가 되었다면 AWS 공식 사이트로 이동하겠습니다.      

https://aws.amazon.com/ko/

이후 무료 계정 만들기를 선택하여 계정을 만들어줍시다.      

![AWS 가입](https://user-images.githubusercontent.com/50267433/86503414-f3e2a280-bde8-11ea-9b4b-5005d38d55e4.PNG)   
![AWS 가입2](https://user-images.githubusercontent.com/50267433/86503423-01982800-bde9-11ea-9a97-6b8e1145a245.PNG)    
![AWS 가입3](https://user-images.githubusercontent.com/50267433/86503424-06f57280-bde9-11ea-98ab-5b7a8bcb5aff.PNG)    
![AWS 가입4](https://user-images.githubusercontent.com/50267433/86503427-0c52bd00-bde9-11ea-95bd-bc7edfe8a81f.PNG)   
![AWS 가입5](https://user-images.githubusercontent.com/50267433/86503434-12e13480-bde9-11ea-9528-da053250957f.PNG)    
![AWS 가입6](https://user-images.githubusercontent.com/50267433/86503439-17a5e880-bde9-11ea-8024-65baea0254b6.jpg)    
![AWS 가입7](https://user-images.githubusercontent.com/50267433/86503442-1bd20600-bde9-11ea-80ed-4463348b6431.jpg)    
![AWS 가입8](https://user-images.githubusercontent.com/50267433/86503445-22607d80-bde9-11ea-9acd-50715ee9e907.jpg)    
![AWS 가입9](https://user-images.githubusercontent.com/50267433/86503448-27bdc800-bde9-11ea-9d3a-8ddf1a46bdd1.PNG)    
![AWS 가입10](https://user-images.githubusercontent.com/50267433/86503450-2be9e580-bde9-11ea-8a6b-247bfa1e8d3c.PNG)      
![AWS 가입11](https://user-images.githubusercontent.com/50267433/86503477-74a19e80-bde9-11ea-92c2-c4c7e59bb93b.PNG)     
     
***    
# 2. EC2 인스턴스 생성하기 
