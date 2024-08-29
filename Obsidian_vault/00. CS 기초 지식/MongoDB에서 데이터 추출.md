# MongoDB에서 데이터 추출

- MongoDB Atlas - https://www.mongodb.com/ko-kr/atlas
1. MongoDB Atlas Cluster 생성
![](https://i.imgur.com/gqP7RiL.png)

2. Cluster 연결 주소 확인
![](https://i.imgur.com/sV6AAsk.png)

3. 생성된 Cluster 확인
![](https://i.imgur.com/UnO3KBn.png)

4. Cluster 내 Collection 생성
![](https://i.imgur.com/2yImW04.png)

- pipeline.conf - MongoDB 및 MinIO 설정 파일
```
[MONGO_ATLAS_CONFIG]
HOSTNAME=wp-cluster.01fs2.mongodb.net
USERNAME= {MongoDB Atlas ID}
PASSWORD= {MongoDB Atlas Password}
DATABASE=wp-db
COLLECTION=wp-collection
```

- sample_mingodb.py - MongoDB Atlas DB에 데이터 저장
```
from pymongo import MongoClient
import datetime
import configparser


# load the mongo_config values
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
hostname = parser.get("MONGO_ATLAS_CONFIG", "HOSTNAME")
username = parser.get("MONGO_ATLAS_CONFIG", "USERNAME")
password = parser.get("MONGO_ATLAS_CONFIG", "PASSWORD")
database_name = parser.get("MONGO_ATLAS_CONFIG", "DATABASE")
collection_name = parser.get("MONGO_ATLAS_CONFIG", "COLLECTION")

mongo_client = MongoClient(

"mongodb+srv://" + username
				 + ":" + password
				 + "@" + hostname
				 + "/" + database_name
				 + "?retryWrites=true&"
				 + "w=majority&ssl=true")

# connect to the db where the collection resides
mongo_db = mongo_client[database_name]

# choose the collection to query documents from
mongo_collection = mongo_db[collection_name]

event_1 = {
	"event_id": 1,
	"event_timestamp": datetime.datetime.today(),
	"event_name": "signup"
}

event_2 = {
	"event_id": 2,
	"event_timestamp": datetime.datetime.today(),
	"event_name": "pageview"
}

event_3 = {
	"event_id": 3,
	"event_timestamp": datetime.datetime.today(),
	"event_name": "login"
}

# insert the 3 documents
mongo_collection.insert_one(event_1)
mongo_collection.insert_one(event_2)
mongo_collection.insert_one(event_3)
```

- pipeline.conf - MongoDB 및 MinIO 설정 파일
```
[MINIO_CREDENTIALS]
ACCESS_KEY={MinIO Access key}
SECRET_KEY={MinIO Secret key}
BUCKET=minio-storage
ACOOUNT_ID=twinis
```

- extract_mongodb.py - MongoDB Atlas DB에 저장된 데이터 추출 후 MinIO Bucket에 저장
```
from pymongo import MongoClient
import csv
from minio import Minio
import datetime
from datetime import timedelta
import configparser

parser = configparser.ConfigParser()

parser.read("pipeline.conf")

hostname = parser.get("MONGO_ATLAS_CONFIG", "HOSTNAME")
username = parser.get("MONGO_ATLAS_CONFIG", "USERNAME")
password = parser.get("MONGO_ATLAS_CONFIG", "PASSWORD")
database_name = parser.get("MONGO_ATLAS_CONFIG", "DATABASE")
collection_name = parser.get("MONGO_ATLAS_CONFIG", "COLLECTION")

mongo_client = MongoClient(
"mongodb+srv://" + username
				 + ":" + password
				 + "@" + hostname
			     + "/" + database_name
				 + "?retryWrites=true&"
				 + "w=majority&ssl=true")

# connect to the db where the collection resides
mongo_db = mongo_client[database_name]

# choose the collection to query documents from
mongo_collection = mongo_db[collection_name]

start_date = datetime.datetime.today() + timedelta(days = -1)
end_date = start_date + timedelta(days = 1 )

mongo_query = { "$and":[{"event_timestamp" : { "$gte": start_date }}, {"event_timestamp" : { "$lt": end_date }}] }
## MongoDB 쿼리 옵션: https://fors.tistory.com/403

event_docs = mongo_collection.find(mongo_query, batch_size=3000)
## batch_size 값: 1번 연결에 가지고 오는 결과의 갯수
## event_docs -> {'event_id': value, 'event_timestamp': value, 'event_name': value} 형식의 list

# create a blank list to store the results
all_events = []

# iterate through the cursor
for doc in event_docs:
	# Include default values
	event_id = str(doc.get("event_id", -1))
	event_timestamp = doc.get(
							"event_timestamp", None)
	event_name = doc.get("event_name", None)
	
# add all the event properties into a list
current_event = []
current_event.append(event_id)
current_event.append(event_timestamp)
current_event.append(event_name)

# add the event to the final list of events
all_events.append(current_event)

export_file = "export_file.csv"

with open(export_file, 'w') as fp:
	csvw = csv.writer(fp, delimiter='|')
	csvw.writerows(all_events)

fp.close()

# load the MINIO_CREDENTIALS values
parser = configparser.ConfigParser()
parser.read("pipeline.conf")

access_key = parser.get("MINIO_CREDENTIALS", "ACCESS_KEY")
secret_key = parser.get("MINIO_CREDENTIALS", "SECRET_KEY")
bucket_name = parser.get("MINIO_CREDENTIALS", "BUCKET")

minio = Minio('localhost:9000', 
			access_key=access_key,
			secret_key=secret_key,
			secure=False)

minio_object_name = export_file

minio.fput_object(bucket_name, minio_object_name, minio_file)
```

- Min IO Bucket에 적재된 모습
![](https://i.imgur.com/SydmqIH.png)
