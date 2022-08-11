<br>

# 카프카 브로커 설치 및 튜토리얼
> 이 문서는 **서버 1대에 카프카 브로커 3대 설치**  예제입니다. 
<br>

### 목차
- Kafka 패키지 다운로드
- Zookeeper 설정
- Zookeeper 실행
- Kafka 설정
- Kafka 실행
<br>

## Kafka 패키지 다운로드
Kafka 설치시, Zookeeper 내장되어 있음

```
wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz

tar -xvf kafka_2.12-3.2.0.tgz
```

## Zookeeper 설정
Zookeeper 3개의 노드를 하나의 클러스터로 구성<br>

- Apache Kafka 폴더 config 내에 zookeper.properties 파일을 3개 복사한다.
  ```
  cp zookeeper.properties zookeeper_1.properties
  cp zookeeper.properties zookeeper_2.properties
  cp zookeeper.properties zookeeper_3.properties
  ```

- Zookeeper Log 저장하는 폴더를 노드별로 생성한다.<br>
(zookeeper_1,2,3 각각 저장 폴더 별도 구성)
  ```
  cd /kafka_2.12-3.2.0
  mkdir data
  cd data
  mkdir zookeeper_1

  ./kafka_2.12-3.2.0/data/zookeeper_1
  ./kafka_2.12-3.2.0/data/zookeeper_2
  ./kafka_2.12-3.2.0/data/zookeeper_3
  ```

- zookeeper_1.properties 설정
zookeeper_2, zookeeper_3도 마찬가지로 설정한다.<br>
(*) 표시의 경우 노드별로 다르게 설정한다.

  ```
  cd config/zookeeper.properties // 이동
  vi config/zookeeper.properties
  
  dataDir=/kafka_2.12-3.2.0/data/zookeeper_1 // (*) 각각 폴더 경로 설정 (zookeeper_1,zookeeper_2,zookeeper_3)

  clientPort=2181 // (*) 각각 다른 포트 설정 (2181, 2182, 2183)

  maxClientCnxns=0 

  admin.enableServer=false

  tickTime=2000
  initLimit=30
  syncLimit=2

  server.1=<IP>:28881:38881 // 서버 정보, 서버 1대이므로 IP는 통일
  server.2=<IP>:28882:38882 // (*) 각각 다른 포트 설정
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

  // 실행 확인
  ps -ef | grep zookeeper
  bin/zookeeper-shell.sh <IP>:2181
  ```

## Kafka 설정
- Apache Kafka 폴더 config 내에 zookeper.properties 파일을 3개 복사한다.
  ```
  cp server.properties server_1.properties
  cp server.properties server_2.properties
  cp server.properties server_3.properties
  ```
- Kafka Log 저장하는 폴더를 노드별로 생성한다.<br>
  ```
  cd /tmp/kafka-logs
  mkdir kafka_1

  /tmp/kafka-logs/kafka_1
  /tmp/kafka-logs/kafka_2
  /tmp/kafka-logs/kafka_3
  ```
- server_1.properties 설정
  server_2, server_3도 마찬가지로 설정한다.<br>
  (*) 표시의 경우 노드별로 다르게 설정한다.
  ```
  broker.id=0 (*) 브로커 카프카를 구분하기 위한 ID, 카프카마다 다르게 설정해야함 (0, 1, 2)
  listeners=PLAINTEXT://:9092 (*) 각각 다른 포트 설정 (9092, 9093, 9094)
  advertised.listeners=PLAINTEXT://<IP>:9092 
  log.dirs=/tmp/kafka-logs/kafka_1 (*) 각각 로그 경로 설정 (kafka_1, kafka_2, kafka_3)
  zookeeper.connect=<IP>:2181, <IP>:2182, <IP>:2183 // 각 서버별 zookeeper 설정 포트
  ```

## Kafka 실행
  ```
  bin/kafka-server-start.sh -daemon ./config/server_1.properties
  bin/kafka-server-start.sh -daemon ./config/server_2.properties
  bin/kafka-server-start.sh -daemon ./config/server_3.properties
  
  jps // kafka 3개가 떠있는 것을 확인할 수 있다.
  ```

## 로컬컴퓨터에서 카프카 통신 확인

1. 로컬 컴퓨터에 서버와 동일한 버전의 카프카 바이너리 패키지 다운
    
    ```
    java
    wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz
    
    tar xvf kafka_2.12-3.2.0.tgz
    ```
    
2. 카프카 브로커 정보 가져오기
    
    ```
    java
    bin/kafka-broker-api-versions.sh --bootstrap-server (퍼블릭IP):9092
    ```

