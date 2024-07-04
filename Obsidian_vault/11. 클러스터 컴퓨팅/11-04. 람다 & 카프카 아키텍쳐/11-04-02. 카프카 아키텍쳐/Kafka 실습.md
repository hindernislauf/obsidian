https://github.com/freepsw/kafka_basic
- ## Java 설치 및 JAVA_HOME 설정
	```
	$ sudo dnf install java-17-openjdk java-17-openjdk-devel
	
	$ java --version
	openjdk version "17.0.6" 2023-01-17 LTS
	OpenJDK Runtime Environment (Red_Hat-17.0.6.0.10-3.el9) (build 17.0.6+10-LTS)
	OpenJDK 64-Bit Server VM (Red_Hat-17.0.6.0.10-3.el9) (build 17.0.6+10-LTS, mixed mode, sharing)
	
	# java home 경로 확인
	$ ls /usr/lib/jvm/jre-17-openjdk
	bin  conf  include  legal  lib  release  tapset
	
	# JAVA_HOME 설정
	$ echo 'export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))' | sudo tee -a /etc/profile
	
	$ source /etc/profile
	
	$ echo $JAVA_HOME
	/usr/lib/jvm/java-17-openjdk-17.0.6.0.10-3.el9.x86_64
	
	$ ls $JAVA_HOME
	bin  conf  include  legal  lib  release  tapset
	```
- ## 1. Install & start kafka [link](http://kafka.apache.org/documentation.html#quickstart)
	- ### Step 1: Download the apache kafka binary files
		```
		$ sudo dnf install -y  wget
		$ mkdir ~/apps
		$ cd ~/apps/
		$ wget https://downloads.apache.org/kafka/3.6.2/kafka_2.12-3.6.2.tgz
		$ tar -xzvf kafka_2.12-3.6.2.tgz
		```
	- ### Step 2: Start Zookeeper server
		```
		$ cd ~/apps/kafka_2.12-3.6.2
		
		# 1) Foreground 실행 (테스트 용으로 zookeeper 로그를 직접 확인)
		$ bin/zookeeper-server-start.sh config/zookeeper.properties
		
		# 2) Background 실행
		$ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
		$ ps -ef | grep zookeeper
		```
	- ### Step 3: Start Kafka server
		```
		$ cd ~/apps/kafka_2.12-3.6.2
		
		# 1) Foregroud 
		$ bin/kafka-server-start.sh config/server.properties
		
		# 2) background 실행
		$ bin/kafka-server-start.sh -daemon config/server.properties
		```
		- #### Kafka Broker 설치후 생성되는 로그파일
			- config/server.properties의 log.dir에 정의된 파일
		    - https://github.com/apache/kafka/blob/3.0/config/server.properties
			```
			# A comma separated list of directories under which to store log files
			log.dirs=/tmp/kafka-logs
			```
		- 아직 topic이 생성되지 않아서, 메타정보 관련된 파일만 생성됨.
			```
			$ ls /tmp/kafka-logs/
			cleaner-offset-checkpoint    meta.properties                   replication-offset-checkpoint
			log-start-offset-checkpoint  recovery-point-offset-checkpoint
			```
		- 메타 정보 출력
			```
			## Broker 관련 메타 정보를 출력 
			$ cat /tmp/kafka-logs/meta.properties
			#
			#Sun Mar 27 00:10:12 UTC 2022
			cluster.id=OX9OhkJmRZWJ9xFCE8CIgw
			version=0
			broker.id=0
			```
	- ### Step 4: Create a topic
		- #### topic 생성 후 조회
		```
		$ cd ~/apps/kafka_2.12-3.6.2
		$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
		
		$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
		```
			- test-0 : 0은 파티션
		- #### Topic 생성 후 log 디렉토리에 추가되는 파일들
		- __consumer_offsets 파티션정보 (첫번째 topic 생성시 자동으로 생성됨)__
		```
		$ ls /tmp/kafka-logs/
		cleaner-offset-checkpoint    meta.properties                   replication-offset-checkpoint
		log-start-offset-checkpoint  recovery-point-offset-checkpoint  test-0
		.....
		```
		- .log 파일에 데이터가 없음. (전송된 메지가 없음)
		```
		$ ls -al /tmp/kafka-logs/test-0
		total 16
		drwxr-xr-x.  2 freepsw18 freepsw18      167 Jul  1 16:59 .
		drwxr-xr-x. 53 freepsw18 freepsw18     4096 Jul  1 17:01 ..
		-rw-r--r--.  1 freepsw18 freepsw18 10485760 Jul  1 16:59 00000000000000000000.index
		-rw-r--r--.  1 freepsw18 freepsw18      160 Jul  1 17:00 00000000000000000000.log
		-rw-r--r--.  1 freepsw18 freepsw18 10485756 Jul  1 16:59 00000000000000000000.timeindex
		-rw-r--r--.  1 freepsw18 freepsw18        8 Jul  1 16:59 leader-epoch-checkpoint
		-rw-r--r--.  1 freepsw18 freepsw18       43 Jul  1 16:59 partition.metadata
		
		```
	- ### Step 5: Send some messages
	```
	$ cd ~/apps/kafka_2.12-3.6.2
	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
	my message
	second message
	```
	- ### Step 6: Start a consumer
	```
	$ cd ~/apps/kafka_2.12-3.6.2
	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
	my message
	second message
	```
- #### 메시지 전송 후 log 디렉토리에 추가되는 파일들
	- __consumer_offsets 파티션 생성 (첫번째 topic 생성시 자동으로 생성됨)
		```
		$ ls /tmp/kafka-logs/
		__consumer_offsets-0   __consumer_offsets-22  __consumer_offsets-36  __consumer_offsets-5
		.....
		```
	- .log 파일에 실제 데이터가 저장됨 (기본 7일간 데이터 보관, 이후에 자동 삭제)
		```
		$ ls -al /tmp/kafka-logs/test-0
		drwxr-xr-x.  2 freepsw18 freepsw18      167 Jul  1 16:59 .
		drwxr-xr-x. 53 freepsw18 freepsw18     4096 Jul  1 17:02 ..
		-rw-r--r--.  1 freepsw18 freepsw18 10485760 Jul  1 16:59 00000000000000000000.index
		-rw-r--r--.  1 freepsw18 freepsw18      160 Jul  1 17:00 00000000000000000000.log
		-rw-r--r--.  1 freepsw18 freepsw18 10485756 Jul  1 16:59 00000000000000000000.timeindex
		-rw-r--r--.  1 freepsw18 freepsw18        8 Jul  1 16:59 leader-epoch-checkpoint
		-rw-r--r--.  1 freepsw18 freepsw18       43 Jul  1 16:59 partition.metadata
		```
	- log 파일의 내용 확인
		```
		$ cd ~/apps/kafka_2.12-3.6.2
		
		## log 파일 확인 
		$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /tmp/kafka-logs/test-0/00000000000000000000.log
		
		Dumping /tmp/kafka-logs/test-0/00000000000000000000.log
		Log starting offset: 0
		baseOffset: 0 lastOffset: 0 count: 1 baseSequence: 0 lastSequence: 0 producerId: 0 producerEpoch: 0 partitionLeaderEpoch: 0 isTransactional: false isControl: false deleteHorizonMs: OptionalLong.empty position: 0 CreateTime: 1719853215613 size: 78 magic: 2 compresscodec: none crc: 3210426395 isvalid: true
		| offset: 0 CreateTime: 1719853215613 keySize: -1 valueSize: 10 sequence: 0 headerKeys: [] payload: my message
		baseOffset: 1 lastOffset: 1 count: 1 baseSequence: 1 lastSequence: 1 producerId: 0 producerEpoch: 0 partitionLeaderEpoch: 0 isTransactional: false isControl: false deleteHorizonMs: OptionalLong.empty position: 78 CreateTime: 1719853219329 size: 82 magic: 2 compresscodec: none crc: 2827246318 isvalid: true
		| offset: 1 CreateTime: 1719853219329 keySize: -1 valueSize: 14 sequence: 1 headerKeys: [] payload: second message
		
		
		## index 파일 확인
		$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /tmp/kafka-logs/test-0/00000000000000000000.index
		
		Dumping /tmp/kafka-logs/test-0/00000000000000000000.index
		offset: 0 position: 0
		
		## timestamp index 파일 확인 
		$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /tmp/kafka-logs/test-0/00000000000000000000.timeindex
		
		Dumping /tmp/kafka-logs/test-0/00000000000000000000.timeindex
		timestamp: 1719853219329 offset: 1
		
		
		```
	- Topic의 partition 별로 마지막 commit 된 위치 확인
	    - 40번 partition에 commit 2로 저장됨
		```
		$ cat /tmp/kafka-logs/replication-offset-checkpoint
		0
		51
		test 0 2
		__consumer_offsets 29 0
		__consumer_offsets 43 0
		__consumer_offsets 0 0
		__consumer_offsets 40 2
		
		```
	- ### Step 7: Describe Topic
		```
		$ cd ~/apps/kafka_2.12-3.6.2
		$ bin/kafka-topics.sh --describe  --bootstrap-server localhost:9092 --topic test
		Topic: test	TopicId: w1XGVh69Q32nB6-TK4yV9A	PartitionCount: 1	ReplicationFactor: 1	Configs:
			Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
		```
	- ### Steo 8: Offset monitoring
		- https://kafka.apache.org/documentation/#basic_ops_consumer_lag
		```
		# find consumer group list
		$ bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
		console-consumer-29099
		
		# view offset of group
		$ bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group console-consumer-{29099}
		
		TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
		test            0          2               2               0               consumer-1-ff412a90-fec0-45e4-89d9-06179c7bd8e3 /10.146.0.6     consumer-1
		```
	- ### Delete Topic ( **관리자 아닌 이상 사용 안함?** )
		- 실습에서는 삭제하지 말고, 명령어 참고만 할 것
		```
		$ bin/kafka-topics.sh --delete --bootstrap-server localhost:9092  --topic test
		
		# 삭제 후 topic이 삭제되었는지 확인
		$ bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
		
		```
- ## 3. Check Zookeeper data
	- 접속할 zookeeper 정보를 입력하여, zookeeper 내부로 접속
	```
	$ cd ~/apps/kafka_2.12-3.6.2
	
	$ bin/zookeeper-shell.sh localhost:2181
	Connecting to localhost:2181
	Connecting to localhost:2181
	Welcome to ZooKeeper!
	JLine support is disabled
	
	WATCHER::
	
	WatchedEvent state:SyncConnected type:None path:null
	
	## Zookeeper에서 제공하는 정보 조회
	ls /
	[admin, brokers, cluster, config, consumers, controller, controller_epoch, feature, isr_change_notification, latest_producer_id_block, log_dir_event_notification, zookeeper]
	
	
	## Broker 관련 정보 조회 
	ls /brokers
	[ids, seqid, topics]
	
	
	## 현재 broker 목록을 조회 (broker 0번만 존재)
	ls /brokers/ids
	[0]
	
	## 0번 broker에 대한 정보 확인 
	get /brokers/ids/0
	{"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT"},"endpoints":["PLAINTEXT://instance-20240701-162846.asia-northeast3-b.c.beaming-grid-428115-d6.internal:9092"],"jmx_port":-1,"features":{},"host":"instance-20240701-162846.asia-northeast3-b.c.beaming-grid-428115-d6.internal","timestamp":"1719917952131","port":9092,"version":5}
	
	
	## topic 정보 조회 
	ls /brokers/topics
	[__consumer_offsets]
	
	
	## topic 상세 정보 조회
	get /brokers/topics/test
	Node does not exist: /brokers/topics/test
	
	# test topic을 삭제하지 않은 경우에는 아래와 같은 topic 정보가 출력됨
	{"removing_replicas":{},"partitions":{"0":[0]},"topic_id":"ZOLr9CLNQjSW70hyIHx6BQ","adding_replicas":{},"version":3}
	
	
	## controller broker 조회
	get /controller
	{"version":2,"brokerid":0,"timestamp":"1719917952299","kraftControllerEpoch":-1}
	```