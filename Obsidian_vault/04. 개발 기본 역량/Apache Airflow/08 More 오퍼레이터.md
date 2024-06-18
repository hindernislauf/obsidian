- # Airflow 제공 기본 오퍼레이터
	- ## Airflow 설치시 기본 제공되는 오퍼레이터들이 존재함
		https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html
		![400](https://i.imgur.com/IEwOuCJ.png)
		![400](https://i.imgur.com/LFASTl0.png)
	- Airflow의 가장 큰 장점은 확장성으로 기본 오퍼레이터 외에도 수많은 오퍼레이터가 제공됨
		https://airflow.apache.org/docs/
	- 현재 설치되어 있는 Prodivder 목록은 Airflow Web - admin - Providers 에서 확인 가능
	- 그 외의 대상은 필요시 설치
- # Trigger Dag Run 오퍼레이터
	- ## DAG간 의존관계를 걸기 위해서
		- Airflow를 사용하다보면 DAG 간의 선후행 관계를 설정해야 하는 경우가 자주 생김
			- DAG1 수행 왼료되면 DAG2 수행
		- DAG 간 의존 관계 설정 방법은 크게 3가지 존재 그중 하나가 TriggerDagRun
			- (1. TriggerDagRun/ 2. External Sensor/ 3. Dataset)
		![500](https://i.imgur.com/7P52vYL.png)
	- ### 실습) Trigger Dag Run 오퍼레이터
		- docs: https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/trigger_dagrun/index.html
		- dag_trigger_dag_run_operator
			```jsx
			# Package Import
			from airflow import DAG
			from airflow.operators.bash import BashOperator
			from airflow.operators.trigger_dagrun import TriggerDagRunOperator
			import pendulum
			
			with DAG(
			    dag_id='dags_trigger_dag_run_operator',
			    start_date=pendulum.datetime(2024,6,17, tz='Asia/Seoul'),
			    schedule='30 9 * * *',
			    catchup=False
			) as dag:
			
			    start_task = BashOperator(
			        task_id='start_task',
			        bash_command='echo "start!"',
			    )
			
			    trigger_dag_task = TriggerDagRunOperator(
			        task_id='trigger_dag_task',
			        trigger_dag_id='dags_python_operator',
			        trigger_run_id=None,
			        logical_date='{{data_interval_start}}',
			        reset_dag_run=True,
			        wait_for_completion=False,
			        poke_interval=60,
			        allowed_states=['success'],
			        failed_states=None
			        )
			
			    start_task >> trigger_dag_task
			```
			- 오류....
- # SimpleHttp 오퍼레이터 사용하기
	- HTTP 요청을 하고 결과로 text를 리턴 받는 오퍼레이터(리턴 값은 Xcom에 저장)
	- HTTP를 이용하여 API를 처리하는 RestAPI 호출시 사용 가능
		(https://airflow.apache.org/docs/apache-airflow-providershttp/stable/_api/airflow/providers/http/operators/http/index.html)
	- 어떤 데이터를 가져올 것인가?
		- 서울시 공공자전거 실시간 대여정보 https://data.seoul.go.kr/dataList/OA-15493/A/1/datasetView.do
		- API로 가져오려면 http://openapi.seoul.go.kr:8088/(인증키)/json/bikeList/1/5 형태로 조회 필요
	- ## Connection 등록
		- SimpleHttp 오펄에터 사용을 위해 Connection 등록이 필수
		- Connection id: openapi.seoul.go.kr
		- Connection Type: HTTP
		- Host: openapi.seoul.go.kr
		- Port : 8088
			![300](https://i.imgur.com/bm243Gy.png)
	- ## Variable 이용
		- 개인 키를 variable 에 등록하고 꺼내 쓰기 (key: apikey_openapi_seoul_go_kr / value : 개인키)
		- endpoint는 아래와 같이 변경 가능
		- dags_simple_http_operator
			```jsx
			# Package Import
			from airflow import DAG
			from airflow.providers.http.operators.http import SimpleHttpOperator
			from airflow.decorators import task
			import pendulum
			
			with DAG(
			    dag_id='dags_simple_http_operator',
			    start_date=pendulum.datetime(2024, 6, 16, tz='Asia/Seoul'),
			    catchup=False,
			    schedule=None
			) as dag:
			
			    '''서울시 공공자전거 대여소 정보'''
			    tb_cycle_station_info = SimpleHttpOperator(
			        task_id='tb_cycle_station_info',
			        http_conn_id='openapi.seoul.go.kr',
			        endpoint='{{var.value.apikey_openapi_seoul_go_kr}}/json/bikeList/1/10/',
			        method='GET',
			        headers={'Content-Type': 'application/json',
			                        'charset': 'utf-8',
			                        'Accept': '*/*'
			                        }
			    )
			
			    @task(task_id='python_2')
			    def python_2(**kwargs):
			        ti = kwargs['ti']
			        rslt = ti.xcom_pull(task_ids='tb_cycle_station_info')
			        import json
			        from pprint import pprint
			        pprint(json.loads(rslt))
			        
			    tb_cycle_station_info >> python_2()
			```
	- ## Variable 활용의 장점
		1) 보안 강화
			- API 키를 githup에 노출하지 않을 뿐만 아니라 Web에서 조회한 Variable 또한 마스킹되어 보임.
			- Airflow는 기본적으로 키에 아래와 같은 이름들이 들어가면 값을 마스킹처리
			- _‘access_token’, ‘api_key’, ‘apikey’,’authorization’, ‘passphrase’, ‘passwd’, ‘password’, ‘private_key’, ‘secret’, ‘token’_
			- 실제 값은 메타 DB의 variable 테이블에서 볼 수 있음
		2) 일원화된 관리
			- 서울시 공공데이터에서 데이터를 추출하는 DAG이  여러개라 한다면 DAG마다 API를 명시해야함.
			- API 키가 바뀐다면 모든 DAG을 찾아 바꿔둬야함
			- Variable에서 한번만 변경해주면 모든 DAG에서 변경된 키를 바라볼 수 있음
	- ### SimpleHttp 오퍼레이터의 불편함
		- SimpleHttp는 기본적으로 1회 호출만 가능
		- 그러나 서울시 공공데이터 데디터셋은 몇 개의 row가 존재할 지 미리 알기 어려움
		- 결과 데이터를 전처리하고 싶은데 하나의 task로 모든 row 를 추출하면서 csv로 깔끔하게 저장할 수 있는 기능을 만들 수 없을까?
- # Custom 오퍼레이터
	- ## Custom 오퍼레이터란?
		- Airflow는 필요한 오퍼레이터를 직접 만들어 사용할 수 있도록 확장성을 지원
		- BaseOperator를 상속하면 원하는 기능은 파이썬으로 직접 구현 가능
			https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html
		- BaseOperator 상속시 두 가지 메서드를 재정의해야 함 (Overriding)
			(1) def __ init__
				-> 클래스에서 객체 생성시 객체에 대한 초기값 지정하는 함수
			(2) def execute(self, context)
				->