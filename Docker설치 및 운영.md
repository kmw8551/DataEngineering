
# Docker 설치 및 운영


![image](uploads/08111e74523204fcfd85bf025029b8fa/image.png)

## 목차
[[_TOC_]]



-------------------------------------------------------------------------------------------------------

##  내용   
### **1. 패키지 정보 업데이트와 CA certificates 설치**  


#### **1-1. 패키지 정보 업데이트 목록**       
   
`$ sudo apt-get update`
   
&nbsp;&nbsp;sudo 명령어가 안될 경우, 다음을 체크하면 된다.         


&nbsp;&nbsp;1. sudo 설치가 안된 리눅스(Docker 컨테이너에 설치된 최초의 리눅스 OS의 경우 해당)<br>
&nbsp;&nbsp;2. /usr/bin/sudo  가 권한을 잘못 갖고 있는 경우: ```bash $ pkexec chmod u+s,g-x /usr/bin/sudo``` 로 해결<br>


#### **1-2. CA certificates 설치**            
    
&nbsp;&nbsp;`$ sudo apt install apt-transport-https ca-certificates curl software-properties-common`<br>   

&nbsp;&nbsp;*※ ca 설명*<br>
&nbsp;&nbsp;apt-transport-https는 APT가 HTTPS 기반 repository 사용하도록 설정을 도와주는 것<br> 
&nbsp;&nbsp;ca-certificate는 certificate authority에서 발행되는 디지털 서명<br>
&nbsp;&nbsp;SSL 인증서의 PEM 파일이 포함되어 있어 SSL 기반 앱이 연결이 되어있는지 확인할 수 있다.<br>
 
#### **1-3. 패키지 저장소 추가(GPG키 등록)**       
    
&nbsp;&nbsp;`$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - `        
     
    command line용  데이터 전송 툴을 이용(download/upload 모두 가능)하여  해당 url에 있는 gpg 파일을 다운로드한다.
    이후 apt-key add -를 통해 패키지 저장소해 다운로드한 gpg 키들 등록한다. curl은 linux, Mac OS에 자동 설치되어 있다.

`$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`    

&nbsp;&nbsp;위의 실행을 통해 add-apt-repository에 등록 한다. add-apt-repository는 apt로 설치할 수 없다.<br>
&nbsp;&nbsp;Ubuntu 공식 저장소에서 항상 최신의 패키지를 다루는 것이 아니기 때문에, 때로는 개발자나 제공사에서 올려놓은<br>
&nbsp;&nbsp;패키지 사용이 필요한 상황이 있다.<br>
&nbsp;&nbsp;Ubuntu의 경우, Launchpad가 제공하는 Launchpad PPA(Personal Package Archive) 저장소를 사용할 수 있으며<br> 
&nbsp;&nbsp;add-apt-repository에서 해당 PPA를 지원한다.


&nbsp;&nbsp;**add-apt-repository not found** 문제 발생 시, `$ sudo apt-get install software-properties-common`을 통해 해결할 수 있다.<br>

---------------------------------------------------------------------------------------------------------------
### **2. Docker 엔진 (Community Version)**      

#### **2-1. Docker 설치**    

&nbsp;&nbsp;`$ sudo apt-get update && sudo apt-cache search docker-ce`<br>

&nbsp;&nbsp;설치 이전, Docker community version이 검색되는지 확인을 한다.<br> 

&nbsp;&nbsp;`docker-ce - Docker: the open-source application container engine` 이라는 문구가 검색되면 설치 준비가 된 것이다.<br>    
&nbsp;&nbsp;Docker CE 버전은 아래 버전 중 하나의 64비트 버전이 필요하다(Debian 계열 기준, 타 OS는 링크 참조) 
&nbsp;&nbsp;[Docker OS Check](https://docs.docker.com/engine/install/) <br>

&nbsp;&nbsp;`$ apt-cache madison docker-ce docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages<br>
$ sudo apt-get install docker-ce=<VERSION>`<br> 

&nbsp;&nbsp;Docker CE 특정 버전 설치의 코드는 다음과 같다. 만약 Docker 최신버전을 설치하고자 하면<br>
&nbsp;&nbsp;아래와 같은 명령어를 작성하면 된다.<br>
&nbsp;&nbsp;`sudo apt-get install docker-ce`  
      
#### **2-2. 실행 확인**<br>

    일반 사용자 계정으로 Docker 명령어를 사용하기 위해서는 그룹을 추가해야 하며, 그룹을 추가하지 않고 바로 docker run 명령어를 
    실행할 시, permission error가 발생한다.    

&nbsp;&nbsp;`$ docker run hello-world
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.38/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'`     

    이를 해결하기 위해서는 사용자를 docker 그룹에 추가해야 한다. Docker는 기본적으로 root 권한이 필요하다.
    사용자 그룹에 대한 개념은 다음과 같다.     

  + USER GROUP은 공통된 목적으로 사용자를 묶는다
  + 그룹 내 사용자는 해당 그룹이 소유한 권한대로 파일을 read/write/execute 할 수 있다.
  + 각 사용자 및 그룹에는 User ID(UID)와 Group ID(GID)라는 고유한 식별번호가 있다.

&nbsp;&nbsp;`$ sudo usermod -aG docker $USER`  
   
&nbsp;&nbsp;위의 명령어를 통해 일반 사용자에서 docker 명령어 실행 시, permission denied 오류가 발생하지 않도록 사용자 그룹을    
&nbsp;&nbsp;추가해 준다.  
  
--------------------------------------------------------------------------------------------------------------------    

### **3. Docker -개요** <br>

#### **3-1. 버전 확인**<br>

![image](uploads/cdb58ca68708ed8818bbb2ae87efbaab/image.png)<br>

&nbsp;&nbsp;```$ docker version``` 명령어를 수행하면 설치된 Docker의 Client와 Server 버전을 알 수 있다.<br>

&nbsp;&nbsp;현재 "ETL" VM 인스턴스에 설치되어 있는 Docker의 버전이며 Client와 Server 정보가 잘 보이는지 확인한다.<br> 
&nbsp;&nbsp;만약 ```$  Error response from daemon: Bad response from Docker engine``` 이라는 메세지가 보인다면<br>
&nbsp;&nbsp;권한의 문제이거나 docker daemon의 문제(정상적으로 실행되지 않는 경우)를 위주로 확인하면 된다.<br>


#### **3-2. Docker 아키텍처**<br>

&nbsp;&nbsp;Docker는 컨테이너라는 독립된 환경에서 프로그램을 실행할 수 있도록 지원하며 Hypervisor의 추가적인 로드 없이 바로 host
시스템의 커널에서<br>&nbsp;&nbsp;실행하기 때문에 빠르며 가볍다.<br> 
&nbsp;&nbsp;컨테이너의 강점은 소위 말하는 "Snowflake Servers"의 문제로부터 자유롭다는 것이다. 하기의 그림은 기존 VM과 컨테이너 구조의 차이를 보여준다.<br>

![image](uploads/b99b6d07ace4998610014496c23b984d/image.png)

[Docker 도입 이전 상황-가상화 간략 설명]

&nbsp;&nbsp;이전의 VM은 Hypervisor 위에 올라가며, Guest OS 설치가 필요하다. Hypervisor는 각각의 Guest OS에서 오는 명령어를 번역하여 하드웨어에 넘기는 &nbsp;&nbsp;일과 가상화 된 Guest OS에게 자원을 할당하는 역할을 한다. 이를 전가상화라고 한다.<br>&nbsp;&nbsp;전가상화는 하드웨어를 완전히 가상화하는 방식으로,DOM0라고 하는 관리용 가상 머신이 실행된다.<br>
&nbsp;&nbsp;그에 반해, 반가상화는 전가상화와 달리 하드웨어를 전부 가상화 하지 않으며, 모든 명령을 &nbsp;&nbsp;DOM0를 통해 하이퍼바이저에 요청하지 않는다.<br>
&nbsp;&nbsp;반가상화는 Hyper Call이라는 인터페이스를 통해 하이퍼바이저에게 직접 요청을 날려, 각각의 가상화 된 Guest OS가 하드웨어 CPU<br>
&nbsp;&nbsp;또는 메모리와 같은 자원에 대한 직접적인 제어권을 가질 수 있다.<br><br>
&nbsp;&nbsp;이러한 설명은 Host OS 가상화 (시스템에 설치된 OS에 가상화 소프트웨어를 설치하고 가상 머신(VM)을 생성하여 각각의 VM에 OS를 설치하는 방법)<br>
&nbsp;&nbsp;하이퍼바이저 가상화(Host OS 없이 하드웨어에 하이퍼바이저를 설치하여 사용하는 방식)에 해당하는 내용이며, Docker는 이와는 조금 더 다르다.<br>
&nbsp;&nbsp;이전의 설명은 하드웨어 가상화 레벨(하이퍼바이저)이었다면, Docker는 컨테이너 기반 가상화로 OS 레벨의 가상화에 속하며 LXC(리눅스 컨테이너)와 비슷하다. 컨테이너의 장점은 "빠른 시작 및 종료 속도와 높은 독립성 보장"이라는 문장으로 요약할 수 있으며, <br>&nbsp;&nbsp;세부 장점 사항은 다음과 같다.
<br>

  * 빠른 시작/ 종료<br>
  * 낮은 오버헤드: OS 전체 부팅보다 메모리를 훨씬 적게 차지함<br>
  * 에플리케이션 컨테이너 지원 및 샌드박스화 [^1] 
  * HW 자원의 대부분을 활용<br>

[^1]: 외부로부터 받은 파일을 바로 실행하지 않고 보호된 영역에서 실행하여 내부 시스템에 악영향을 주는 것을 사전에 방지하는 기술.
<br>


&nbsp;&nbsp;단점 사항은 다음과 같다.

  * 결국 Host OS에 종속적(Host OS의 커널을 사용)<br>
  * 컨테이너별 커널 구성이 사실상 불가능<br>
   

[Docker Architecture]<br>
[Client-Server 구조]<br>
![image](uploads/d0afa07105f6c76990b76726e2f9bdbc/image.png)

![image](uploads/2ffc4892c49ba7d0b0eddb63cc3bbe13/image.png)

<br>
&nbsp;&nbsp;실행 및 운영에서 꼭 알아야 할 것은 위의 Docker Client-Server Architecture이다. Client와 Server는 REST API로 소통을 하며,    
&nbsp;&nbsp;실제의 작업은 데몬(Server)에서 수행한다. Client는 요청을 하기만 한다.<br>
<br>   

&nbsp;&nbsp;**Docker Daemon**
<br>
&nbsp;&nbsp; Docker 데몬(dockerd)은 Docker API 요청에 대한 접속 대기(Listening)와 Docker 객체(image, container, network, volume)등을 관리한다.<br>
<br>
&nbsp;&nbsp;**Docker Client**
<br>
&nbsp;&nbsp;Docker 클라이언트(docker)는 사용자와 Docker간 상호작용을 할 수 있는 방법이며 명령어를 받는 역할을 한다.<br>
&nbsp;&nbsp;사용자가 Docker 명령어를 통해 명령을 내리면 클라이언트는 1개 이상의 데몬과 communicate한다.<br>

&nbsp;&nbsp;**Docker Registry**
<br>
&nbsp;&nbsp;Docker registry는 Docker의 image들을 관리한다. Docker Hub는 공식 레지스트리로 모두가 사용가능하며, docker pull 또는 docker push<br>
&nbsp;&nbsp;명령어로 설정해두었던 레지스트리로 이미지를 받거나 올릴 수 있다.<br>
<br>
&nbsp;&nbsp;**Docker Volume**
<br>
&nbsp;&nbsp;Volume은 Docker에 의해 관리되는 저장 공간이며, Host File System의 특정 경로를 컨테이너로 마운트 하는 방법인 Bind Mount와는 다르다.<br>
<br>

-------------------------------------------------------------------------------------------------------------------------
### **4. Docker 실행** <br>       
      
#### **4-1. 컨테이너 생성**<br>

&nbsp;&nbsp;이미지로 컨테이너 생성하는 방법은 다음과 같다<br>
&nbsp;&nbsp; `$ docker create [옵션] [이미지 이름]:[태그]`
<br>
#### **4-2. 컨테이너 실행**<br>

##### **4-2-1. Run 명령어**<br>

&nbsp;&nbsp;이미지로 컨테이너를 실행하는 기본적인 방법은 다음과 같다<br>
&nbsp;&nbsp;`docker run <옵션> <이미지 이름, ID> <명령> <매개변수>`
<br>

&nbsp;&nbsp;옵션 값을 설정할 때 "=" 는 생략 가능하며[ex) docker run --name anaconda], run 명령어의 변수 인자는 다음과 같다.<br>
<br>

  * `--attach` | `-a` :컨테이너에 표준 입력, 표준 출력을 연결<br>
  * `--cpu-shares` | `-c` : CPU 자원 분배 설정, 기본 값은 1024이며 상대적으로 적용된다.<br>
  * `--detach` | `-d` : Detached모드이며 데몬 모드라고도 한다. 컨테이너가 백그라운드로 실행된다.<br>
  * `--env` | `-e` : 컨테이너의 환경 변수를 설정한다. 주로 비밀번호 또는 설정 값 전달에 쓰인다.<br>
  * `--expose` : 컨테이너의 포트를 host와 연결만 하고 외부로 노출하지 않는다.<br>
  * `--interactive` | `-i` : 표준 입력을 활성화하며 컨테이너와 연결되어 있지 않더라도 표준 입력이 유지된다.<br>
  * `--name` : 컨테이너의 이름을 설정하는데 쓰인다.<br>
  * `-p` : host와 컨테이너 간의 포트 배포 및 바인드를 위해 사용.<br>
  * `--restart` : 컨테이너 내 프로세스 종료 시, 재시작하는데 쓰인다.<br>
  * `--rm` : 컨테이너가 종료될 때, 컨테이너와 관련된 리소스 전부를 깨끗이 제거하는데 쓰이며, 주로 Test용 컨테이너를 일회성 실행할 때 쓰인다.<br>
  * `-v` : host와 컨테이너 간 볼륨 설정을 위해 사용, host 파일 시스템의 특정 경로를 컨테이너 안의 파일 시스템의 특정경로로 mount.<br>


##### **4-2-2. cp 명령어**<br>

##### **4-2-3. exec 명령어**<br>

##### **4-2-4. images 명령어**<br>

##### **4-2-5. ps 명령어**<br>

##### **4-2-6. rm, rmi 명령어**<br>

##### **4-2-7. start, stop 명령어**<br>

##### **4-2-8. 기타 명령어**<br>

#### **4-3. Docker 관련 발생 이슈 사례 및 해결 방안**<br>

##### **4-3-1. Docker airflow 웹서버가 뜨지 않는 이유**<br>

1. 웹 브라우저에서 err_connection_refused가 뜨는 경우<br>
![image](uploads/87c9405ade4e88ed66d306aad3d7da6d/image.png)

&nbsp;&nbsp;실제 Docker 컨테이너가 해당 포트(ex: 8080)과 외부로 뚫려있지 않은 경우가 대다수 이다.
이는 Docker run 당시 -p 인자로 외부 포트와 내부 포트 간 &nbsp;&nbsp;포워딩이 안되어 있어서 그렇다.
따라서 docker pull <container 이름>으로 이미지를 구축하고 docker run 실행에서 -p로 포트를 추가한다.
기존의<br>&nbsp;&nbsp;컨테이너는 지워진 상태인 것이 좋으며 새로운 이름을 사용한 컨테이너를 실행한다면 딱히 문제되지 않는다.
<br><br>
&nbsp;&nbsp;코드 예시(**포트는 여러 개 지정해도 무방, 대신 -p 인자를 그에 맞춰서 쓸 것!**)<br>
`docker run -it  -p (외부포트):(내부포트)  -p (외부포트2):(내부포트2) --name (컨테이너 이름) (이미지 명)`
<br> 
<br><br>
2. 설정이 필요한 SW들의 설정 문제<br>
&nbsp;&nbsp;jupyter notebook의 경우 jupyter notebook --generate-config 을 통해 설정 파일 생성과 설정 이후 다시 jupyter notebook --config <config파일명>을 통해 적용해야 한다. 컨테이너는 기본 루트 권한이라서 --allow-root 명령어 또한 실행할 시 적어주어야 한다. 또한 최초 실행시 jupyter notebook 다음 --ip 인자를 붙여서 --ip 0.0.0.0를 적어주는 것이 좋다. 이는 0.0.0.0이 '로컬 시스템의 모든 IPv4 주소'를 의미하는 것으로 외부와의 연결 시 모든 네트워크 인터페이스에서 수신 대기 중임을 의미한다. 물론 이 또한 config 파일에서 allow_ip 부분이 '*' 또는 '0.0.0.0'으로 되어 있어야 한다. 해당 내용의 자세한 부분은 Docker Container Anaconda 설치 및 JupyterLab 설치 부분을 참조할 것을 권장한다.<br>
<br>
&nbsp;&nbsp;코드 예시<br>
`jupyter notebook --generate-cofing` <br>
`jupyter notebook --config jupyter_notebook_config.py` <br>
`jupyter notebook --ip 0.0.0.0 -p 8080 --allow-root` <br>

<br>

##### **4-3-2. Docker 컨테이너 내/외부 연결**<br>
1. Docker 컨테이너 간 연결은 어떻게 하나?<br>
Docker에 컨테이너가 배포되면, 컨테이너는 격리된 네트워크 환경을 갖게 된다. 이 네트워크 환경(namespace)는 오로지 해당 컨테이너를 위한 것이며, 따라서 mac주소와 IP도 부여 받는다.<br>여기서 Container 포트의 외부 노출(expose)를 알면 외부 또는 컨테이너 간 서비스 연결 시, 도움이 될 것이다.<br>
컨테이너 간 연결 또한 ssh 를 이용하는데, ssh는 안타깝게도 Docker 컨테이너에 기본적으로 설치가 되어있지 않다.<br>
따라서 다음과 같이 설치가 필요하다.<br>

`$ sudo apt-get install ssh`<br>
<br>
![image](uploads/91302f00a012ac48239de15ac2e1db1b/image.png)
<br>
![image](uploads/db545d51f01323a635e9917a3c45d874/image.png)
<br>
<br>
&nbsp;&nbsp;*빨간 글씨가 떠도 겁낼 것이 없다. 물론 너무 대담하게 무시하면 안된다.*
<br><br>
&nbsp;&nbsp;기본적으로 Docker 컨테이너 포트는, 처음 외부 통신을 위해 노출된 포트 또는 DB 포트를 제외하면, 닫혀있다. 따라서 네트워크 관리 및 &nbsp;&nbsp; 포트 할당을 알아야 한다.<br>
&nbsp;&nbsp;Docker로 설치한 기본 Ubuntu는 없는 것이 많다. 그 중 두 개는, netstat과 ufw(우분투 방화벽) 이다. 따라서 두 패키지를 
설치하도록 한다.<br>
&nbsp;&nbsp;`$ sudo apt-get install ufw`<br>
&nbsp;&nbsp;`$ sudo apt-get install net-tools`<br>
 
도커 실행시 ufw 명령어 사용에서 주의해야할점 : https://seulcode.tistory.com/396
&nbsp;&nbsp;ssh [사용자 계정]@[서버 주소] 또는 ssh -l [사용자 계정] [서버 주소] 형식으로 사용한다. 
 












---------------------------------------------------------------------------------------------------------------------
