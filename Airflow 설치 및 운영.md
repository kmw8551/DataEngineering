## 목차



--------------------------------------------------------------------------------------------------------------------

## Ubuntu에 Airflow 설치
<br>

#### apt-get 업데이트 및 pip 설치
Airflow 설치는 python3 기반으로 설치되어야 한다. Linux OS마다 Python의 버전이 다르다는 점을 우선적으로 참고하여 진행하는 것을 권장한다. <br>Linux Debian buster와 Ubuntu 16.04 LTS에서는 Python2.7 버전이 Default이므로 미리 Python 버전을 변경해야 한다.<br><br>
Linux Ubuntu 18.04에서는 Python3가 기본이며, 3 버전을 기반으로 돌아가는 프로그램들이 존재한다. 따라서 Default Python을 지우는 것은<br> 지양해야 하며, 새로운 Python 버전을 설치하고 path를 새로 설치한 Python으로 맞춰주는 것이 보다 안전하다. <br> [Python 버전 변경하기 참고](#case-1-python-버전-문제)

<br>

예제는 Python 3.7을 대상으로 하며, Python3.6.2 이상의 버전이면 별다른 문제 사항 없이 수월하게 설치가 가능하다.<br>
Python3.7 설치와 더불어 Apache-airflow 설치 전에 추가적인 의존성 패키지를 설치한다.<br>

`$ sudo apt-get update`<br>
`$ sudo apt install software-properties-common`<br>
`$ sudo add-apt-repository ppa:deadsnakes/ppa`<br>
`$ sudo apt-get install python3.7`<br>
`$ sudo apt-get install python3.7-dev -y`<br>
`$ sudo apt-get install libssl-dev`<br>
`$ sudo apt-get install libmysqlclient-dev -y`<br>
`$ sudo apt-get install libkrb5-dev -y`<br>
`$ sudo apt-get install python3-setuptools -y`<br>

<br>

Airflow의 병렬 처리를 위한 PostgreSQL/MySQL 설치(여기에서는 PostgreSQL 설치<br>
`$ sudo apt-get install postgresql postgresql-contrib -y`



### PostgreSQL 접속`$ sudo -u postgres psql`

PostgreSQL 설치 시, airflow와 PostgresSQL이 같은 VM, Container안에 있을 경우, LocalExecutor 사용이 가능하다.<br>
하지만 서로 다른 Container 혹은 VM 에 존재할 경우, localExecutor 사용이 힘들 수 있다.

**PostgreSQL에 접속하여 유저(analyticsplatform)와 DB(airflow)를 생성한다.**

`postgres=# create role analyticsplatform;`<br>
CREATE ROLE<br>
`postgres=# create database airflow`<br>
CREATE DATABASE<br>
`postgres=# grant all privileges on database airflow to analyticsplatform;`<br>
GRANT<br>
`postgres=# alter role analyticsplatform superuser;`<br>
ALTER ROLE<br>
`postgres=# alter role analyticsplatform createdb;`<br>
ALTER ROLE<br>
`postgres=# alter role analyticsplatform with login;`<br>
ALTER ROLE<br>
`postgres=# grant all privileges on all tables in schema public to analyticsplatform;`<br>
GRANT<br>
`postgres=# \c airflow`<br>
You are now connected to database "airflow" as user "postgres".<br>
`airflow=# \conninfo`<br>
You are connected to database "airflow" as user "postgres" via socket in "/var/run/postgresql" at port "5432”.<br>
`airflow=# \q`

여기까지 설치하였으면 홈디렉토리에 airflow가 생성되었을 것이다. 생성되어 있지 않으면 airflow 폴더를 만든다.<br>
\
`$ mkdir ~/airflow`
<br>
airflow 디렉토리가 생성되었으면 .bash_profile에 등록한다.<br>
`$ echo 'export AIRFLOW_HOME=~/airflow' > ~/.bash_profile`
<br>
 

Python3.7 기반 airflow 실행을 위해서는 다음과 같은 사항을 설치하여야 하며, 주로 패키지 의존성 관리와 연관된 것들이다.<br>

`$ python3.7 -m pip install pip`<br>
`$ python3.7 -m pip install --upgrade pip`<br>
`$ python3.7 -m pip install --upgrade wheel`<br>
`$ python3.7 -m pip install --upgrade psutil`<br>
<br>

Apache-airflow를 설치한다.<br>
`$ python3.7 -m pip install apache-airflow`<br>
<br>

설치시 오류가 날 수도 있다. 그렇다면 아래와 같이 UNIDECODE를 yes로 변경하여 설치한다. <br>
`$ sudo SLUGIFY_USES_TEXT_UNIDECODE=yes python3.7 -m pip install apache-airflow`
<br>
<br>

병렬처리를 위한 CeleryWorker를 사용하기 위해 밑에 패키지를 또 설치한다.<br>

`$ python3.7 -m pip install celery` <br>
`$ python3.7 -m pip install psycopg2` <br>
`$ sudo apt-get install libpq-dev python-dev` (여기는 psycopg2 설치 에러 날 경우, 사용) <br>
`$ python3.7 -m pip install mysqlclient` <br>
`$ python3.7 -m pip install psycopg2-binary` <br>
`$ python3.7 -m pip install apache-airflow['kubernetes']` <br>
`$ python3.7 -m pip install apache-airflow['rabbitmq']` <br>
`$ python3.7 -m pip install apache-airflow[celery]` <br>
`$ python3.7 -m pip install apache-airflow[postgres]` <br>
`$ python3.7 -m pip install apache-airflow[mysql]` <br>

Celery Worker의 메시지 브로커로 RabbitMQ를 설치한다.<br>
`$ sudo apt install rabbitmq-server -y`<br>

RabbitMQ의 설정파일에 NODE_IP_ADDRESS를 바꿔준다.<br>
`$ sudo vi /etc/rabbitmq/rabbitmq-env.conf`

RabbitMQ를 기동한다.<br>
`$ sudo service rabbitmq-server start`

airflow의 데이터베이스를 초기화하고 폴더 내 airflow Config 파일을 열어 설정을 변경한다.<br>
`$ airflow initdb`<br>
`$ sudo vi ~/airflow/airflow.cfg`<br>

`~/airflow/airflow.cfg`의 아래 내용을 변경한다.
>executor = CeleryExecutor <br>
sql_alchemy_conn = postgresql+psycopg2:///airflow <br>
broker_url = amqp://guest:guest@localhost:5672// <br>
result_backend = amqp://guest:guest@localhost:5672// <br>

<br>
설정이 끝났으면 DB를 다시 초기화 한다.<br>
`$ airflow initdb`

<br>
airflow 디렉토리 내 dag(directed acyclic graph)파일을 두기 위한 디렉토리를 만든다.<br>
`$ mkdir ./airflow/dags`
<br>
template dag를 불러온다. <br>
`$ airflow list_dags`

airflow를 사용하기 위한 scheduler, worker, webserver 기동 Script를 작성한다. <br>
`$vi ./airflow_start.sh`<br>

<b>VI 편집기 접속 후 아래 내용을 입력한다.</b>
> #!/bin/bash <br>
airflow scheduler -D <br>
airflow worker -D <br>
airflow webserver -p 8070 -D <br>

Shell Script를 실행시킨다.<br>
`$ ./airflow_start.sh`

로컬에서 airflow가 잘 실행되었는지 확인한다.<br>
> http://[airflow-server-ip]:8080

## Trouble Shooting
#### Case 1. Python 버전 문제
&nbsp;&nbsp;Azure에서 Linux로 Airflow 서버를 구축할 때 기본적으로 Python2.7버전이 기본으로 잡혀있다. 이 경우 Python3.x 버전을 쓰고 Airflow를 설치해야 한다.<br>

##### Python 버전 확인
`$ python -V[or --version]`<br>
> python2.7
<br>

&nbsp;&nbsp;Python 대체 버전이 Linux에 등록되어 있는지 확인<br>
`$ sudo update-alternatives --config python` <br>
> update-alternatives: error: no alternatives for python<br>

&nbsp;&nbsp;Python 대체 버전을 등록한다.<br>
`$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1`<br>
`$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.6 2`<br>
<br>
&nbsp;&nbsp;Python 대체 버전을 등록하고 구성이 잘되었는지 확인한다.<br>
`$ sudo update-alternatives --config python`<br>
> There are 2 choices for the alternative python (providing /usr/bin/python). <br>

| Selection| Path| Priority | Status |
| ------ | ------ | ----- | ----- |
| * 0| /usr/bin/python3.6 | 2 | auto mode |
|   1| /usr/bin/python2.7 | 1 | manual mode |
|   1| /usr/bin/python3.6 | 2| manual mode |
> Press <enter> to keep the current choice[*], or type selection number: 2 <br>

&nbsp;&nbsp;다시 파이썬 버전을 확인하면 python 3.x대 버전으로 기본 경로가 설정되어 있다.<br>
`$ python --version`<br>
> Python 3.6.3

#### Case 2. setproctitle
setporctitle관련 문제가 발생할 경우 아래와 같은 에러가 발생한다.
> utilsmodule.c:1:20: fatal error: Python.h: No such file or directory <br>
compilation terminated.<br>
이 경우 setproctitle은 apt로 python-dev 설치 시 같이 설치 되는데 python3의 경우 제대로 설치 안되는 경우가 더러 있다. 이 경우 python3-dev로 재설치해준다.<br>

`$ sudo apt-get install python3-dev -y`

## Airflow 실행하기(feat. PostgreSQL)

#### Airflow 실행 방법

airflow 를 실행하기 위해서는 웹 서버를 띄워야 한다. 웹 서버를 실행하기 이전, airflow worker와 scheduler를 실행한다.<br>

`$ airflow worker`<br>
`$ airflow scheduler`<br>
`$ airflow webserver -p 8080`<br>

<br>
웹브라우저로 접속을 한다.
<br>

`http://<ip>:<port>/admin/`
<br>

![image](uploads/82be3ad329dedd9eddc762e1ceb553dd/image.png)

<br>
만약 DAG를 만들었다면, 해당 페이지에서 DAG를 off에서 on으로 바꿔준다.
<br>

![image](uploads/be6968c7bb2af23a5916ef2b2ce16aa6/image.png)
<br>
graph View를 통해 현재 Operator들의 목록을 볼 수 있다.


#### Airflow 설명

<작성 필요>


--------------------------------------------------------------------------------------------------------------------------
## Docker Airflow to PostgreSQL 작업

####  Airflow 컨테이너에서 Postgres 컨테이너 작업하기

Airflow와 PostgreSQL이 서로 다른 컨테이너에 존재할 경우, 몇 가지 설정에 유의하여야 한다.<br>
<br>
예시) Airflow 컨테이너의 `/airflow/dags/src` 경로에서 Postgresql 테이블을 생성
<br>
`/airflow/dags/src` 경로에 example.py를 만든다.

`$ vi example.py`
<br>
에디터에는 다음과 같은 내용을 적는다.<br>
![image](uploads/7aba96ef7b202e24e9d49aabffa20415/image.png)

```python 

from sqlalchemy import create_engine
from sqlalchemy_utils import database_exists, create_database
import psycopg2

def make_database():
    """
    Make the Postgres database and create the table.
    """

    dbname    = 'weatherdb'
    username  = 'postgres'
    tablename = 'weather_table'

    engine  = create_engine('postgresql+psycopg2://%s:analyticsplatform@20.39.168.172:5432/%s'%(username,dbname))


    if not database_exists(engine.url):
        create_database(engine.url)

    conn = psycopg2.connect(database = dbname,
                            user = username,
                            host ="172.17.0.3",
                            password="analyticsplatform")

    curr = conn.cursor()

    create_table = """CREATE TABLE IF NOT EXISTS %s
                (
                    city         TEXT,
                    country      TEXT,
                    latitude     REAL,
                    longitude    REAL,
                    todays_date  DATE,
                    humidity     REAL,
                    pressure     REAL,
                    min_temp     REAL,
                    max_temp     REAL,
                    temp         REAL,
                    weather      TEXT
                )
                """ % tablename

    curr.execute(create_table)
    conn.commit()
    conn.close()

if __name__ == "__main__":
    make_database()

```

위의 Python 코드는 "기상청 데이터 수집 및 저장"에서 DB 테이블을 생성하는 코드이다.<br>
SQLAlchemy [^1] 를 통해 테이블을 만들고, psycopg2를 이용하여 DB Connection을 수행한다. <br>
[^1]: 오픈 소스 SQL 툴킷으로, python 언어로 작성된 ORM(Object-relational mapper, 객체 관계형 매퍼)이다. MIT에서 작성하였으며 Django Framework의 ORM 또한 이와 매우 유사하다. 

```python
dbname    = 'weatherdb'
username  = 'postgres'
tablename = 'weather_table'

engine  = create_engine('postgresql+psycopg2://%s:analyticsplatform@20.39.168.172:5432/%s'%(username,dbname))

#중략

conn = psycopg2.connect(database = dbname,
                            user = username,
                            host ="172.17.0.3",
                            password="analyticsplatform" 
```
여기서 중요한 것은, engine 생성 시 주소 설정과 connect 부분에서 host 설정이다.<br>
engine 부분의 코드는 공식 문서를 참조하였으며, `postgresql+psycopg2:[DB User]:[Password]@[localhost or IP]/[DB Name]`이다.<br>
connection 부분에서는 host 설정이 반드시 필요하다. 기본적으로 docker가 설치된 VM 접속의 편의를 위해 고정 IP 설정을 해놓는 경우가 많다.<br>그렇기 때문에 고정 IP 접근만 가지고 원하는 컨테이너에 대한 세부 접속은 불가능하며, 따라서 해당 컨테이너의 IP를 이용하여 컨테이너에 접근 가능하게 해야 한다.
`conn = psycopg2.connect(database = ... host = "DB 컨테이너 IP")` 와 같이 host에 postgresql이 설치된 컨테이너의 IP를 적어준다.<br>

 




------------------------------------------------------------------------------------------------------------------------------
