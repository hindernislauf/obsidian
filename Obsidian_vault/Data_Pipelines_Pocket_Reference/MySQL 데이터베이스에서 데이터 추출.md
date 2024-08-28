### MySQL 데이터베이스에서 데이터 추출 방법

- SQL을 사용한 전체 또는 증분 추출
    - 구현이 간단
    - 자주 변경되는 대규모 데이터세트에서는 확장성이 떨어짐
    - 전체 추출과 증분 추출 사이에도 트레이드오프가 있음(다음 파트)
- 이진 로그(binlog) 복제
    - 구현이 복잡
    - 스트리밍 데이터 수집을 수행하는 경로
    - 원본 테이블의 변경되는 데이터 볼륨이 크거나
    - MySQL 소스에서 데이터를 더 자주 수집해야 하는 경우 적합함

### 설치 방법

1. 로컬 시스템에 설치
2. AWS에서 Amazon RDS를 생성(MySQL 인스턴스 관리)
    1. 샘플을 학습, 작업 → 데이터베이스를 공개적으로 액세스할 수 있게 설정
    2. 프로덕션 설정 → 강력한 보안이 필요하기에 Amazon RDS 보안 모범 사례 따르기
    3. 단, 비용 발생

```sql
# 테이블 생성
CREATE TABLE Orders ( 
	OrderId int,
	OrderStatus varchar(30), 
	LastUpdated timestamp
);

# 행 삽입
INSERT INTO Orders
	VALUES (1, 'Backordered', '2020-06-01 12:00:00'); 
INSERT INTO Orders
	VALUES (1, 'Shipped', '2020-06-09 12:00:25');
INSERT INTO Orders
	VALUES (2, 'Shipped', '2020-07-11 3:05:00'); 
INSERT INTO Orders
	VALUES (1, 'Shipped', '2020-06-09 11:50:00'); 
INSERT INTO Orders
	VALUES (3, 'Shipped', '2020-07-12 12:00:00');
```

# 전체 또는 증분 MySQL 테이블 추출

전체 추출과 증분 추출은 데이터베이스에서 데이터를 처리하거나 분석할 때 사용하는 두 가지 주요 접근 방식입니다. 각각의 접근 방식은 장단점이 있으며, 선택할 때는 여러 가지 트레이드오프를 고려해야 합니다.

### 전체 추출 (Full Extraction)

```sql
SELECT *
FROM Orders;
```

**장점:**

1. **완전성**: `전체 데이터를 새로 추출`하므로 데이터의 일관성과 정확성을 보장할 수 있습니다. 이전 데이터 상태와 관계없이 항상 최신 상태를 제공합니다. 이전 데이터는 삭제합니다.
2. **단순성**: 데이터베이스 쿼리나 로직이 간단해질 수 있습니다. 모든 데이터를 한 번에 가져오면 별도의 추적이나 비교 작업이 필요 없습니다.

**단점:**

1. **성능**: 대량의 데이터를 매번 새로 추출해야 하므로 데이터베이스와 네트워크에 큰 부하를 줄 수 있습니다. 데이터가 많을수록 성능에 더 큰 영향을 미칩니다.
2. **시간**: 전체 데이터를 매번 추출하고 처리하는 데 시간이 많이 걸릴 수 있습니다.

### 증분 추출 (Incremental Extraction)

```sql
SELECT *
FROM Orders
WHERE LastUpdated > {{ last_extraction_run }};
```

- `LastUpdated`: 데이터베이스 테이블의 데이터가 마지막으로 업데이트된 시점을 나타내는 필드입니다. 이 필드는 일반적으로 `TIMESTAMP` 또는 `DATETIME` 데이터 타입을 가집니다.
    
- `{{ last_extraction_run }}`: 이 값은 템플릿 문법을 사용하여 외부에서 주입되는 변수입니다. 추출 작업의 최근 실행 시간을 나타내는 타임스탬프입니다. 이 값은 쿼리 실행 시점에 동적으로 설정됩니다.
    
    ```sql
    SELECT MAX(LastUpdated)
    FROM warehouse.Orders;
    ```
    
- 마지막 데이터 추출 이후에 업데이트된 레코드만을 가져오려면 이 조건을 사용합니다.
    
- 변경할 수 없는 데이터(레코드를 삽입 가능, 업데이트 불가)가 포함된 테이블의 경우, Last updated 열 대신 레코드가 생성된 시간에 대한 타임스탬프를 사용
    

**장점:**

1. **성능**: 데이터의 `변화만 추출`하므로, 전체 데이터셋을 새로 추출하는 것보다 효율적입니다. 특히 데이터가 자주 변경되지 않는 경우 유리합니다.
2. **시간**: 데이터 변경 사항만 처리하므로, 전체 추출보다 빠르게 완료될 수 있습니다.

**단점:**

1. **복잡성**: 증분 추출 로직이 더 복잡할 수 있습니다. 변경된 데이터만 추적하고 비교하는 메커니즘이 필요하며, 이를 위한 추가적인 관리가 필요합니다.
2. **일관성 문제**: 변경 사항을 정확하게 추적하지 못하거나 처리 과정에서 데이터 일관성 문제가 발생할 수 있습니다. 예를 들어, 데이터 누락이나 중복이 발생할 수 있습니다.
    1. 삭제된 행은 캡처되지 않는다.
    2. 원본 테이블에는 마지막으로 업데이트된 시간에 대한 신뢰할 수 있는 타임스탬프가 있어야 한다.

### 트레이드오프 요약

- **성능과 시간**: 증분 추출은 성능과 시간 면에서 유리하지만, 전체 추출은 데이터의 일관성과 정확성을 보장합니다.
- **복잡성과 관리**: 증분 추출은 더 복잡한 로직과 관리가 필요하며, 전체 추출은 상대적으로 단순하지만 성능적인 제약이 클 수 있습니다.
- **데이터의 일관성**: 전체 추출은 데이터의 일관성을 유지하는 데 유리하지만, 증분 추출은 일관성을 유지하는 데 더 많은 주의가 필요합니다.

결론적으로, 전체 추출과 증분 추출 중 어떤 방법을 선택할지는 데이터의 크기, 업데이트 빈도, 성능 요구사항, 시스템의 복잡도 등에 따라 달라질 수 있습니다.

### 파이썬 스크립트에 의해 트리거되는 SQL 쿼리를 사용하여 구현

1. 라이브러리 설치

```sql
(env)  $ pip install pymysql
```

1. 연결 정보를 저장하기 위해 pipeline.conf 파일 업데이트

```sql
[mysal_config] 
hostname = my_host.com 
port = 3306
username = my_user_name
password = my_password 
database = db_name
```

1. `extract_mysql full.py`라는 새 파이썬 스크립트를 생성
    1. 수집 로드 단계에서 데이터 웨어하우스로 추출된 데이터를 쉽게 정형화하고 사용 가능

```sql
import pymysql 
import csv
import boto3          # S3 버킷에 업로드 위해
import configparser
```

1. MySQL 데이터베이스에 대한 연결을 초기화

```sql
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
hostname = parser.get("mysal_config", "hostname")
port = parser.get("mysal_config", "port")
username = parser.get("mysal_config", "username")
dbname = parser.get("mysal_config", "database")
password = parser.get("mysal_config", "password")

conn = pymysql.connect(host=hostname,
	user=username, 
	password=password, 
	db=dbname, 
	port=int(port))
	
if conn is None:
	print("Error connecting to the MySQL database") 
else:
	print("MySQL connection established!")
```

1. Orders 테이블의 전체 추출

```sql
m_query = "SELECT * FROM Orders;" 
local_filename = "order_extract.csv"

m_cursor = conn.cursor() 
m_cursor.execute(m_query)
results = m_cursor.fetchall()

with open(local_filename, 'w') as fp: 
	csv_w = csv.writer(fp, delimiter='|') 
	csv_w.writerows(results)
fp.close() 
m_cursor.close() 
conn.close()
```

1. CSV 파일을 S3 버킷에 업로드

```sql
# aws_boto_credentials 값을 로드
parser =configparser.ConfigParser() 
parser.read("pipeline.conf")
access_key = parser.get("aws_boto_credentials", "access_key") 
secret_key = parser.get("aws_boto_credentials", "secret_key") 
bucket_name = parser.get("aws_boto_credentials", "bucket_name")

S3 = boto3.client('53', 
aws_access_key_id=access_key, 
aws_secret_access_key=secret_key)

S3_file = local_filename

S3.upload_file(local_filename, bucket_name, S3_file)
```

1. 스크립트 실행

```sql
(env) $ python extract_mysql_full.py
```

### 데이터 증분 추출을 위한 스크립트 변경사항

1. 스크립트 시작점으로 `extract_mysql_full.py` 파일을 복사하여 `extract_mysql_incremental.py` 사본에서 시작
    
    ⇒ 소스 데이터베이스에 부담을 주며 프로덕션 쿼리 실행도 차단할 수 있기 때문에 복제본 설정을 고려
    
2. Orders 테이블에서 추출한 마지막 레코드의 타임스탬프를 찾기
    
    1. Redshift 클러스터와 상호 작용하려면, psycopg2 라이브러리를 설치
    2. Redshift 클러스터에 연결하고 쿼리
    
    ```sql
    import psycopg2
    
    # Redshift db connection 정보를 가져옴 
    parser = configparser.ConfigParser() 
    parser.read("pipeline.conf")
    dbname = parser.get("aws_creds", "database")
    user = parser.get("aws_creds", "username") 
    password = parser.get ("aws_creds", "password") 
    host = parser.get("aws_creds", "host")
    port = parser.get ("aws_creds", "port")
    
    # Redshi ft 클러스터에 연결 
    rs_conn = psycopg2. connect(
    	"dbname=" + dbname
    	+ "user=" + user
    	+ "password=" + password 
    	+ "host=" + host
    	+ "port=" + port)
    
    rs_sql = "'SELECT COALESCE(MAXLastUpdated),
    	"1900-01-01')
    	FROM Orders;""" 
    rs_cursor = rs_conn.cursor()
    rs_cursor.execute(rs_sql) 
    result = rs_cursor.fetchone()
    
    # 오직 하나의 레코드만 반환됨 
    last_updated_warehouse = result[0]
    
    rs_cursor.close() 
    rs_conn.commit()
    ```
    
    - `last_updated_warehouse` 에 저장된 값을 사용하여 MySQL 데이터베이스에서 실행되는 추출 쿼리를 수정
    - 이전 추출 작업 실행 이후 Orders 테이블에서 업데이트된 레코드만 가져올 수 있음
    - (42p) ~44
    - 새 쿼리에는 last_update_warehouse 값에 대해 %s로 표시되는 자리 표시자가 포함되어 있다. 그러면 값이 튜플(데이터 집합을 저장하는데 사용되는 데이터 유형)로 cursor의 .execute() 함수에 전달된다 . 이는 SQL 주입(injection)을 방지하기 위해 SQL 쿼리에 매개변수를 추가해 주는 적절하고 안전한 방법이다. 다음은 MySQL 데이터베이스에서 SQL 쿼리를 실행해주는 업데이트된 코드다.
    
    ```python
    m_query = """SELECT * 
    	FROM Orders
    	WHERE LastUpdated > %s;""" 
    local_filename = "order_extract.csv"
    
    m_cursor = conn.cursor()
    m_cursor.execute(m_query, (last_updated_warehouse,))
    ```
    

- `extract_mysql_full.py`

```python
import pymysql
import csv
import boto3
import configparser

parser = configparser.ConfigParser()
parser.read("pipeline.conf")
hostname = parser.get("mysql_config", "hostname")
port = parser.get("mysql_config", "port")
username = parser.get("mysql_config", "username")
dbname = parser.get("mysql_config", "database")
password = parser.get("mysql_config", "password")

conn = pymysql.connect(host=hostname,
        user=username,
        password=password,
        db=dbname,
        port=int(port))

if conn is None:
  print("Error connecting to the MySQL database")
else:
  print("MySQL connection established!")

  m_query = "SELECT * FROM Orders;"
local_filename = "order_extract.csv"

m_cursor = conn.cursor()
m_cursor.execute(m_query)
results = m_cursor.fetchall()

with open(local_filename, 'w') as fp:
  csv_w = csv.writer(fp, delimiter='|')
  csv_w.writerows(results)

fp.close()
m_cursor.close()
conn.close()

# load the aws_boto_credentials values
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
access_key = parser.get("aws_boto_credentials", "access_key")
secret_key = parser.get("aws_boto_credentials", "secret_key")
bucket_name = parser.get("aws_boto_credentials", "bucket_name")

s3 = boto3.client('s3', aws_access_key_id=access_key, aws_secret_access_key=secret_key)

s3_file = local_filename

s3.upload_file(local_filename, bucket_name, s3_file)
```

- `extract_mysql_incremental.py`

```python
import pymysql
import csv
import boto3
import configparser
import psycopg2

# get db Redshift connection info
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
dbname = parser.get("aws_creds", "database")
user = parser.get("aws_creds", "username")
password = parser.get("aws_creds", "password")
host = parser.get("aws_creds", "host")
port = parser.get("aws_creds", "port")

# connect to the redshift cluster
rs_conn = psycopg2.connect(
    "dbname=" + dbname
    + " user=" + user
    + " password=" + password
    + " host=" + host
    + " port=" + port)

rs_sql = """SELECT COALESCE(MAX(LastUpdated), '1900-01-01')
    FROM Orders;"""
rs_cursor = rs_conn.cursor()
rs_cursor.execute(rs_sql)
result = rs_cursor.fetchone()

# there's only one row and column returned
last_updated_warehouse = result[0]

rs_cursor.close()
rs_conn.commit()

# get the MySQL connection info and connect
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
hostname = parser.get("mysql_config", "hostname")
port = parser.get("mysql_config", "port")
username = parser.get("mysql_config", "username")
dbname = parser.get("mysql_config", "database")
password = parser.get("mysql_config", "password")

conn = pymysql.connect(host=hostname,
        user=username,
        password=password,
        db=dbname,
        port=int(port))

if conn is None:
  print("Error connecting to the MySQL database")
else:
  print("MySQL connection established!")

m_query = """SELECT *
    FROM Orders
    WHERE LastUpdated > %s;"""
local_filename = "order_extract.csv"

m_cursor = conn.cursor()
m_cursor.execute(m_query, (last_updated_warehouse,))
results = m_cursor.fetchall()

with open(local_filename, 'w') as fp:
  csv_w = csv.writer(fp, delimiter='|')
  csv_w.writerows(results)

fp.close()
m_cursor.close()
conn.close()

# load the aws_boto_credentials values
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
access_key = parser.get(
    "aws_boto_credentials",
    "access_key")
secret_key = parser.get(
    "aws_boto_credentials",
    "secret_key")
bucket_name = parser.get(
    "aws_boto_credentials",
    "bucket_name")

s3 = boto3.client(
    's3',
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key)

s3_file = local_filename

s3.upload_file(
    local_filename,
    bucket_name,
    s3_file)
```