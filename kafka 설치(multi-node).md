<br>

# 카프카 브로커 설치 및 튜토리얼
> 이 문서는 **서버 1대에 카프카 브로커 3대 설치**  예제입니다. 
<br>

### 목차
- 
<br>
<br>
## Kafka 설치
Kafka 설치시, Zookeeper 내장되어 있음

```
wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz

tar -xvf kafka_2.12-3.2.0.tgz
```

## Zookeeper 설정
Zookeeper 3개를 하나의 클러스터로 구성<br>

- Apache Kafka 폴더 config 내에 zookeper.properties 파일을 3개 복사한다.
```
cp zookeeper.properties zookeeper_1.properties
cp zookeeper.properties zookeeper_2.properties
cp zookeeper.properties zookeeper_3.properties
```

- Zookeeper Log 저장 공간 폴더 생성<br>
Apache Kafka 폴더 내에 data 폴더 생성<br>
(zookeeper_1,2,3 각각 저장 폴더 별도 구성)
```
./kafka_2.12-3.2.0/data

./kafka_2.12-3.2.0/data/zookeeper_1
./kafka_2.12-3.2.0/data/zookeeper_2
./kafka_2.12-3.2.0/data/zookeeper_3
```

- zookeeper_1.properties 설정

```
dataDir=/kafka_2.12-3.2.0/data/zookeeper_1 // 각각 폴더 경로 입력 (zookeeper_1,zookeeper_2,zookeeper_3)

clientPort=2181 // 각각 입력 (2181, 2182, 2183)

maxClientCnxns=0 

admin.enableServer=false

tickTime=2000
initLimit=30
syncLimit=2

server.1=<IP>:28881:38881 // 서버 정보, 서버 1대이므로 IP는 통일
server.2=<IP>:28882:38882
server.3=<IP>:28883:38883
```

- dataDir
  로그가 쌓이는 경로<br>
  (1번 - /kafka_2.12-3.2.0/data/zookeeper_1)<br>
  (2번 - /kafka_2.12-3.2.0/data/zookeeper_2)<br>
  (3번 - /kafka_2.12-3.2.0/data/zookeeper_3)<br>
  
- clientPort
  서버를 3개 별도로 두면 2181로 통일한다.<br>
  하지만, 하나의 서버에서 같은 포트를 동시에 사용하는 것이 불가능하므로 각각 다른 번호를 설정한다.<br>
  (1번 - 2181)<br>
  (2번 - 2181)<br>
  (3번 - 2181)<br>
  
- Server 정보
  3개의 zookeeper를 하나의 클러스터로 묶기 위해서 서버 정보를 설정해야한다.<br>
  server.(myid) 형식으로 앙상블을 구성한다. 임의로 1,2,3으로 설정<br>
  <br>
  1,2,3번 모두 3줄을 추가한다.<br>
  server.1=<IP>:28881:38881 // 서버 정보, 서버 1대이므로 IP는 통일<br>
  server.2=<IP>:28882:38882<br>
  server.3=<IP>:28883:38883<br>
 
<br>
  
- server myid 설정
```
./kafka_2.12-3.2.0/data

cd ./kafka_2.12-3.2.0/data/zookeeper_1
touch myid

// 1,2,3번 모두 적용
./kafka_2.12-3.2.0/data/zookeeper_1/myid
./kafka_2.12-3.2.0/data/zookeeper_2/myid
./kafka_2.12-3.2.0/data/zookeeper_3/myid

// myid 내에 zookeeper.properties에 설정한 (myid) 입력
```

## Zookeeper 실행  
```
bin/zookeeper-server-start.sh -daemon ./config/zookeeper_1.properties
bin/zookeeper-server-start.sh -daemon ./config/zookeeper_2.properties
bin/zookeeper-server-start.sh -daemon ./config/zookeeper_3.properties
  
ps -ef | grep zookeeper
```
