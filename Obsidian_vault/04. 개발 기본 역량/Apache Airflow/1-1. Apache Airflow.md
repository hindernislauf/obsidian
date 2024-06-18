- # 01. Apache Airflow 개요
	- 파썬을 이용해 워크플로우를 만들고 관리할수 있는 오픈소스 기반 워크플로우 관리 도구
	- ## 특징
		- 파이썬으로 제작된도구이며 이용자가 워크플로우 생성시에도 파이썬으로 구현해야함
		- 하나의 워크플로우는 DAG(Directed Acyclic Graph) 이라 부르며 DAG 안에는 1개 이상의 Task가 존재
		- Task간 선후행 연결이 가능하되 순환되지 않고 방향성을 가짐(=DAG)
		- Cron 기반의 스케줄링
		- 모니터링 및 실패 작업에 대한 재실행 기능이 간편
			![](https://i.imgur.com/hyi5C6a.png)
	- ## 왜 Airflow 인가?
		- 워크플로우를 관리하는 여러 오픈소스 도구중 가장 인기가 많은 도구
		- UI로 워크플로우를 만들지는 못해 처음에 쓰기가 어려우나 익숙해지면 
		- 파이썬 언어가 하락하거나 한 거의 모든 유형의 파이프라인을 만들 수 있음
	- ## 장점
		- 파이썬에 익숙하다면 러닝 커브 빠르게 극복 가능
		- 대규모 워크플로우 환경에서 부하 증가시 수평적 확장 가능한 Kubernetes 등 아키텍처 지원
		- 파이썬에서 지원되는 라이브러리 활용하여 다양한 도구 컨트롤 가능
			(GCP, AWS등 대다수 클라우드에서 제공하는 서비스)
		- Airflow에서 제공하는 파이썬 소스 기반으로 원하는 작업을 위한 커스터마이징이 가능
			(오퍼레이터, Hook, 센서 등)
	- ## 단점
		- 실시간 워크플로우 관리에 적합하지 않음(최소 분 단위 실행)
		- 워크플로우(DAG) 개수가 많아질 경우 모니터링이 쉽지 않음
		- 워크플로우를 GUI환경에서 만들지 않기에 파이썬에 익숙하지 않다면 다루기 쉽지않음
			(협업 환경에서 개발 표준이 없으면 유지관리가 쉽지 않음)
- # 02. Apache Airflow 설치
	- ### AWS 인스턴스 생성
		- 보안그룹  8080포트 생성
	- ### EC2 접속
		- `ssh -i ~/Downloads/mungio.pem ubuntu@퍼블릭IP`
	- ### 도커 엔진 설치
		```
		https://docs.docker.com/engine/install/ubuntu/
		
		# Add Docker's official GPG key:
		sudo apt-get update
		sudo apt-get install ca-certificates curl
		sudo install -m 0755 -d /etc/apt/keyrings
		sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
		sudo chmod a+r /etc/apt/keyrings/docker.asc
		
		# Add the repository to Apt sources:
		echo \
		  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
		  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
		  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
		sudo apt-get update
		
		
		
		sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
		
		
		
		sudo docker run hello-world
		```
	- ### Airflow 설치
		```
		https://airflow.apache.org/docs/apache-airflow/2.9.1/howto/docker-compose/index.html
		
		curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.1/docker-compose.yaml'`
		
		mkdir -p ./dags ./logs ./plugins ./config
		echo -e "AIRFLOW_UID=$(id -u)" > .env
		
		sudo docker compose up airflow-init
		```
	- ### Airflow 시작
		```
		# 시작
		sudo docker compose up
		# 접속
		http://43.203.113.15:8080
		
			# 종료
			sudo docker compose down
		```
	- ## DAG 개발
		- DAG은 1개 이상의 오퍼레이터로 정의되며 오퍼레이터는 하고자 하는 기능에 대한 설계도라 할 수 있음
		- Task는 오퍼레이터(클래스)를 통해 생성된 객체로 실제 jop을 수행하는 대상이 됨
			![300](https://i.imgur.com/xA5F6cL.png)
		- DAG 개발을 위해 EC2 서버의 /home/ubuntu/dags 디렉토리에 DAG 파일 정의
			![400](https://i.imgur.com/N5ajSCQ.png)
		- Bash Operator 작성
			```
			cd dags
			vi dags_bash_operator.py
			
			# Airflow DAG 및 필요한 모듈을 가져옵니다.
			from airflow import DAG
			import datetime
			import pendulum
			from airflow.operators.bash import BashOperator
			
			# DAG 정의를 시작합니다.
			# 여기서는 DAG ID, 스케줄, 시작 날짜, 그리고 catchup 설정을 정의합니다.
			with DAG(
				# DAG의 고유 식별자입니다.
			    dag_id="dags_bash_operator",  
			    # 매일 0시에 실행되는 크론 표현식입니다.
			    schedule="0 0 * * *",  
			    # DAG 실행의 시작 날짜 및 시간대입니다.
			    start_date=pendulum.datetime(2023, 3, 1, tz="Asia/Seoul"),  
			    # 시작 날짜 이후의 기간 동안의 미실행 DAG 실행을 방지합니다.
			    catchup=False  
			) as dag:
			    # 첫 번째 태스크 정의
			    bash_t1 = BashOperator(
				    # 태스크의 고유 식별자입니다.
			        task_id="bash_t1",  
			        # 실행될 bash 명령어입니다. 여기서는 현재 사용자를 출력합니다.
			        bash_command="echo whoami",  
			    )
			
			    # 두 번째 태스크 정의
			    bash_t2 = BashOperator(
				    # 두 번째 태스크의 고유 식별자입니다.
			        task_id="bash_t2",
			        # 실행될 bash 명령어입니다. 여기서는 호스트 이름을 출력합니다.
			        bash_command="echo $HOSTNAME",  
			    )
			
			    # 태스크 종속성을 정의합니다.
			    # bash_t1 태스크가 성공적으로 완료된 후에 bash_t2 태스크가 실행됩니다.
			    bash_t1 >> bash_t2

			```
		- ### Bash Operator 수행
			![](https://i.imgur.com/YjBV2zO.png)
			- 초록색 네모 버튼 클릭해서 실행
			![400](https://i.imgur.com/drgLeUN.png)
			- Logs 확인
			![400](https://i.imgur.com/LFrdeQO.png)
		`sudo docker ps`
		![](https://i.imgur.com/HrcnGp4.png)
		```
		sudo docker exec -it a50c801c8cbe bash
		echo $HOSTNAME
		```
		![](https://i.imgur.com/B50t9RX.png)
	- ## task는 누가 실행하는가
		![400](https://i.imgur.com/DUS2Jvv.png)
- # 03. 개발 환경 구성
	- ## 개발환경 Flow
		- dags 개발시 서버에서 직접 개발하지 않으면 일반적으로 git을 활용한 CI/CD 환경을 주로 이용함
		- Airflow 서버가 별도로 존재한다고 가정할 때 코드 개발은 로컬컴퓨터에서 개발 후 완성된 코드를 서버로 배포하는 식으로 진행
			![400](https://i.imgur.com/5QXkh8y.png)
		- 파이썬 인터프리터 버전 확인 & 설치
			![400](https://i.imgur.com/xfCr9NJ.png)
		- python 다운로드
			https://www.python.org/downloads/release/python-3123/
			- 설치 확인
				```
				cd /user/local/bin
				python3 -V
				
				ls python*
				```
		- 가상환경 생성(Mac OS)
			```
			# 로컬 환경
			cd
			/usr/local/bin/python3.12 -m venv airflow-venv
			cd airflow_venv
			sourch bin/activate
			(airflow_venv)$ python -V
			```
		- 파이썬 가상환경 사용 이유
			- Airflow 라이브러리 버전 충돌 방지
			- 프로젝트에 따라 필요한 파이썬 인터프리터 버전이 다른경우
			- 파이썬은 의존 관계에 있는 특정 라이브러리에 대한 요구하는 버전이 상이한 경우
			- 설치 시첨에 따라서도 설치되는 버전이 상이한 경우가 많아 가상환경 생성은 필수
		- ### Airflow 라이브러리 설치
			```
			pip install "apache-airflow[celery]==2.9.1" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.9.1/constraints-3.12.txt"
			```
		- 로컬 환경 git 디렉토리 구성
			- /Users/{user명}/airflow 디렉토리에서 소스를 작성, git push 하면 서버의 /home/ubuntu 디렉토리에 git pull 받도록 구성
			![400](https://i.imgur.com/4nU1Xem.png)
		- github Repository 만들기
			- 레파지토리명: yeardream_airflow
		- github 레파지토리 권한 부여
			- 로컬 -> 원격 레파지토리에 코드를 올리려면 권한 필요
			- Settings -> Devleoper settings -> Personal Access Tokens -> Tokens(classic)
			- 필요 권한: repo
	- 로컬 -> github 연동
		```
		git init
		ls -al
		mkdir dags
		```
	- VScode
		- docker-compose.yaml 생성
			```
			# 로컬 터미널
			curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.1/docker-compose.yaml'
			```
		- volumes 수정
			```
			- /home/ubuntu/yeardream_airflow/dags:/opt/airflow/dags
			- /home/ubuntu/logs:/opt/airflow/logs
			- /home/ubuntu/config:/opt/airflow/config
			- /home/ubuntu/plugins:/opt/airflow/plugins
			```
		- github 올리기
			```
			# 로컬
			git branch -M master
			git status
			git add .
			git status
			git commit -m "first commit"
			git status
			git remote add origin https://github.com/mun-gio/yeardream_airflow.git
			git push -u origin master
			```
		- EC2로 github 내려받기
			```
			# ubuntu
			cd /home/ubuntu
			git clone https://github.com/mun-gio/yeardream_airflow.git
			ls -al
			cd yeardream_airflow
			ls -al
			```
		- 기존 올렸던 터미널 나오기 Ctrl + C
		- git 으로 동기화된 docker-compose.yaml 파일을 이용해 도커 컨테이너 재가동
			```
			sudo docker compose up
			```
			- 오류 나면 해보기
				```
				# 모든 컨테이너 종료, 삭제
				sudo docker ps
				sudo docker container stop {컨테이너ID} 
				sudo docker container stop {컨테이너ID} 
				
				# 8080포트 종료
				sudo lsof -i :8080
				sudo kill -9 <PID>
				```
	- ## 오퍼레이터 기본
		![](https://i.imgur.com/x1wtYV5.png)
		- Task 연결 방법
			- VScode 파일 작성
				```
				dags_conn_test1.py
				
				from airflow import DAG
				
				import pendulum
				
				from airflow.operators.empty import EmptyOperator
				
				  
				
				with DAG(
				
				dag_id="dags_conn_test1",
				
				schedule=None,
				
				start_date=pendulum.datetime(2024, 6, 14, tz="Asia/Seoul"),
				
				catchup=False
				
				) as dag:
				
				t1 = EmptyOperator(
				
				task_id="t1"
				
				)
				
				  
				
				t2 = EmptyOperator(
				
				task_id="t2"
				
				)
				
				  
				
				t3 = EmptyOperator(
				
				task_id="t3"
				
				)
				
				  
				
				t4 = EmptyOperator(
				
				task_id="t4"
				
				)
				
				  
				
				t5 = EmptyOperator(
				
				task_id="t5"
				
				)
				
				  
				
				t6 = EmptyOperator(
				
				task_id="t6"
				
				)
				
				  
				
				t7 = EmptyOperator(
				
				task_id="t7"
				
				)
				
				  
				
				t8 = EmptyOperator(
				
				task_id="t8"
				
				)
				
				  
				
				t1 >> t2
				
				t1 >> t3
				
				t3 >> t4
				
				t5 >> t4
				
				t4 >> t6
				
				t7 >> t6
				
				t6 >> t8
				```
				
				```
				dags_conn_test2.py
				
				from airflow import DAG
				
				import pendulum
				
				from airflow.operators.empty import EmptyOperator
				
				  
				
				with DAG(
				
				dag_id="dags_conn_test2",
				
				schedule=None,
				
				start_date=pendulum.datetime(2024, 6, 14, tz="Asia/Seoul"),
				
				catchup=False
				
				) as dag:
				
				t1 = EmptyOperator(
				
				task_id="t1"
				
				)
				
				  
				
				t2 = EmptyOperator(
				
				task_id="t2"
				
				)
				
				  
				
				t3 = EmptyOperator(
				
				task_id="t3"
				
				)
				
				  
				
				t4 = EmptyOperator(
				
				task_id="t4"
				
				)
				
				  
				
				t5 = EmptyOperator(
				
				task_id="t5"
				
				)
				
				  
				
				t6 = EmptyOperator(
				
				task_id="t6"
				
				)
				
				  
				
				t7 = EmptyOperator(
				
				task_id="t7"
				
				)
				
				  
				
				t8 = EmptyOperator(
				
				task_id="t8"
				
				)
				
				  
				
				t1 >> [t2, t3]
				
				[t3, t5] >> t4
				
				[t4, t7] >> t6 >> t8
				```
			- 로컬 터미널
				```
				git add dags
				git commit -m "dags_conn_test2"
				git push
				```
			- EC2 터미널
				```
				git pull
				```
	- ## Bash 오퍼레이터를 이용한 Shell 파일 수행
		- Bash 오퍼레이터는 Bash 명령을 수행하기 위한 오퍼레이터
		- bash 명령이 길고 복잡할 경우 서버에 Shell 파일을 만들어두고 Airflow를 이용하여 쉘 파일을 수행하도록 하는 것은 매우 일반적인 방법
		- ex) sftp를 통해 파일을 받은 후 DB에 Insert & tar.gz으로 압축 -> Shell 파일로 작성후 Airflow 로 실행
		- ### 고려할 사항
			- Airflow에서 Task를 처리하는 것은 worker 이며 현재 실습 환경에서 서버의 worker컨테이너가 실제 task를 처리하게 됨
			- 따라서 worker 컨테이너가 Shell 파일을 처리하려면 Shell 파일을 컨테이너에 넣어주면 됨
		- 
		  ![400](https://i.imgur.com/3BIJlj1.png)
			- $1은 입력받는 첫번째 를 말한다
		- docker-compose.yaml 수정
			![400](https://i.imgur.com/sXpClxP.png)
			- `yeardream_airflow/`
		- dags디렉토리에 dags_bash_select_fruit.py 만들기
			```
			from airflow import DAG
			
			import pendulum
			
			from airflow.operators.bash import BashOperator
			
			  
			
			with DAG(
			
			dag_id="dags_bash_select_fruit",
			
			schedule="10 0 * * 6#1",
			
			start_date=pendulum.datetime(2024, 6, 14, tz="Asia/Seoul"),
			
			catchup=False
			
			) as dag:
			
			t1_orange = BashOperator(
			
			task_id="t1_orange",
			
			# bash_command="/opt/airflow/plugins/shell/select_fruit.sh ORANGE",
			# sh 추가 권한 부여
			bash_command="sh /opt/airflow/plugins/shell/select_fruit.sh ORANGE",
			)
			
			  
			
			t2_avocado = BashOperator(
			
			task_id="t2_avocado",
			
			# bash_command="/opt/airflow/plugins/shell/select_fruit.sh AVOCADO",
			# sh 추가 권한 부여
			bash_command="sh /opt/airflow/plugins/shell/select_fruit.sh AVOCADO",
			)
			
			  
			
			t1_orange >> t2_avocado
			```
			(쉘 실행 안됨)
- # 과제
	![400](https://i.imgur.com/mWTF1HK.png)
	- ## dags_bash_operator_standard.py
	```
	from airflow import DAG
	import datetime
	import pendulum
	from airflow.operators.bash import BashOperator
	
	my_dag = DAG(
		dag_id="dags_bash_operator_standard",
		schedule="0 9 * * 1,5",
		start_date=pendulum.datetime(2024, 6, 1, tz="Asia/Seoul"),
		catchup=True,
		tags=["homework"],
	)
	
	bash_t1 = BashOperator(
		task_id="bash_t1",
		bash_command="echo whoami",
		dag = my_dag,
	)
	
	bash_t2 = BashOperator(
		task_id="bash_t2",
		bash_command="echo $HOSTNAME",
		dag = my_dag,
	)
	
	bash_t1 >> bash_t2
	```
	- ## dags_bash_operator_decorator.py
	```
	from airflow import DAG
	import datetime
	import pendulum
	from airflow.decorators import dag
	from airflow.operators.bash import BashOperator
	
	@dag(
		dag_id="dags_bash_operator_decorator",
		schedule="0 13 * * 5#2",
		start_date=pendulum.datetime(2024, 5, 1, tz="Asia/Seoul"),
		catchup=True,
		tags=["homework"],
	)
		
		def my_dag():
			bash_t1 = BashOperator(
				task_id="bash_t1",
				bash_command="echo whoami",
		)
		
			bash_t2 = BashOperator(
				task_id="bash_t2",
				bash_command="echo $HOSTNAME",
			)
	
		bash_t1 >> bash_t2
	
	dag_instance = my_dag()
	```
