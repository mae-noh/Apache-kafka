<br>

# 카프카 브로커 설치 및 튜토리얼
- [BOOK] 아파치 카프카 애플리케이션 프로그래밍 with 자바 - 최원영 지음. <br>
- 아파치 카프카를 EC2에 설치 후, 로컬 커맨드 툴을 이용하여 다양한 명령어 실습을 진행해본다.

<br>

### 목차
- AWS EC2 인스턴스 발급 및 보안설정
- 인스턴스 접속
- 인스턴스에 자바 설치
- 주키퍼, 카프카 브로커 실행 전 환경 설정
- 주키퍼, 카프카 실행
- 로컬 컴퓨터에서 카프카와 통신 확인
- 카프카 커맨드 라인 툴
- 카프카 명령어 모음

<br>
<br>
<br>

## AWS EC2 인스턴스 발급 및 보안설정

1. key pair 생성, 자동으로 pem 파일 다운로드.
2. 보안 그룹 Inbound 설정에서 포트 오픈.
`카프카 브로커 9092` `주키퍼 2181`

<br>

## 인스턴스 접속

1. ssh 명령어로 접속하기
    
    EC2 인스턴스에 접속하기 위해서는 발급받은 key파일(.pem)을 프라이빗 키로 추가
    
    ```java
    // pem 파일 권한 변경(read)
    chmod 400 test-kafka-server-key.pem
    ```
    
    ```java
    // -i 옵션을 이용하여 pem파일을 프라이빗 키로 추가
    // 퍼블릭IP는 EC2 인스턴스 정보에서 확인 가능
    ssh -i test-kafka-server-key.pem ec2-user@(퍼블릭IP)
    ssh -i (keypair파일명) (ec2유저이름)@(퍼블릭IP)
    ```
    
<br>

## 인스턴스에 자바 설치

1. JDK 설치
카프카 브로커는 스칼라와 자바로 작성되어 JVM환경 위에서 실행됨.
    
    ```java
    // java 1.8 설치
    sudo yum install -y java-1.8.0-openjdk-devel.x86_64
    
    // java 버전 확인
    java -version 
    ```
    
<br>

## 주키퍼, 카프카 브로커 실행 전 환경 설정
**EC2 서버 1대에 주키퍼, 카프카 1개 설치**
<br>
<br>
- 카프카 바이너리 패키지 다운로드

    [Apache Kafka](https://kafka.apache.org/downloads)

    ```java
    // kafka_2.12 압축파일 다운로드
    wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz

    // tar 압축해제 xvf 현재 디렉토리에 풀기
    // x 파일 추출
    // v 처리 과정을 자세하게 나열
    // f 대상 아카이브 지정
    tar xvf kafka_2.12-3.2.0.tgz
    ```

- 카프카 브로커 힙 메모리 설정
    - 카프카 패키지 기본 설정, 힙메모리 1G, 주키퍼 512MB
    → EC2 프리티어 계정 메모리 1G, 따라서 메모리를 적게 변경하는 것 필요.

    ```java
    // Amazon Linux 2 AMI 인스턴스 bash쉘 사용
    vi ~/.bashrc

    // 힙 메모리 설정
    export KAFKA_HEAP_OPTS = "-Xmx400m -Xms400m"

    // 스크립트 파일 수정 후, 수정값 바로 적용
    source ~/.bashrc

    // 정상적으로 변경되었는지 확인
    echo $KAFKA_HEAP_OPTS
    ```

- 카프카 브로커 실행 옵션 설정
    - *주의* 카프카 브로커가 이미 실행중이라면 옵션 변경 후 재시작해야함.

    ```java
    // 카프카 브로커가 클러스터 운영에 필요한 옵션 지정 가능
    vi config/server.properties

    // 카프카 클라이언트를 브로커와 연결할 때 사용
    // 퍼블릭IP와 포트번호를 넣고, 주석제거
    advertised.listeners=PLAINTEXT://(퍼블릭IP):9092
    ```

- 옵션 정보
    
    ```java
     
    broker.id
    
    listeners=PLAINTEXT://your.host.name:9092
    
    SASL_PLAINTEXT, SASL_SSL:SASL_SSL
    
    num.network.threads=3
    
    num.io.threads=8
    
    log.dirs=/tmp/kafka-logs
    
    num.partitions=1
    
    log.retention.hours=168
    
    log.segment.bytes=1073741824
    
    log.retention.check.interval.ms=300000
    
    zookeeper.connect=localhost:2181
    
    zookeeper.connection.timeout.ms=18000
    ```
    
<br>

## 주키퍼, 카프카 실행

- 주키퍼 실행
    - 카프카 실행 **필수** 애플리케이션
    
    ```java
    // 주키퍼 실행
    bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
    
    // jps : jvm 프로세스 확인
    // m : main메서드에 전달된 인자, v : jvm에 전달된 인자 확인(힙메모리, log4j설정)
    jps -vm
    ```
    
- 카프카 브로커 실행 및 로그 확인
    
    ```java
    // 카프카 브로커 실행
    sudo bin/kafka-server-start.sh -daemon config/server.properties
    
    // 주키퍼, 카프카 프로세스 동작 여부 확인
    jps -m
    
    // 로그 확인
    tail -f logs/server.log
    ```
    
<br>

## 로컬 컴퓨터에서 카프카와 통신 확인

1. 로컬 컴퓨터에 서버와 동일한 버전의 카프카 바이너리 패키지 다운
    
    ```java
    wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz
    
    tar xvf kafka_2.12-3.2.0.tgz
    ```
    
2. 카프카 브로커 정보 가져오기
    
    ```java
    bin/kafka-broker-api-versions.sh --bootstrap-server (퍼블릭IP):9092
    ```
    
<br>

## 카프카 커맨드 라인 툴

- 카프카 운영시 다양한 명령어 사용
    - 카프카 클러스터와 연동하여 데이터 이동
    - 토픽, 파티션 개수 변경
- 실행 방법
    - 카프카 브로커 인스턴스에 원격 접속
    - 로컬에서 브로커 9092로 접근
    

### 1. Topic 생성

```java
bin/kafka-topics.sh --create --topic (토픽명) --bootstrap-server (퍼블릭IP):9092
```

### 2. Topic list 확인

```java
bin/kafka-topics.sh --describe --topic (토픽명) --bootstrap-server (퍼블릭IP):9092

bin/kafka-topics.sh --bootstrap-server (퍼블릭IP):9092 --list
```

### 3. Topic 삭제

```java
./bin/kafka-topics.sh --delete --zookeeper (퍼블릭IP):2181 --topic (토픽명)
```

### 4. Topic 내에 Event 생성

```java
bin/kafka-console-producer.sh --topic (토픽명) --bootstrap-server (퍼블릭IP):9092
```
<br>

## 카프카 명령어 모음

```java
# EC2 카프카 설치, 실행
1 chmod 400 test-kafka.pem
2 ssh -i kafka-test.pem ec2-user@(퍼블릭IP)
3 sudo yum install -y java-1.8.0-openjdk-devel.x86_64
4 wget <https://downloads.apache.org/kafka/2.7.0/kafka_2.12-2.7.0.tgz>
5 export KAFKA_HEAP_OPTS="-Xmx400m -Xms400m"
6 vi config/server.properties
7 listeners=PLAINTEXT://:9092
8 advertised.listeners=PLAINTEXT://(퍼블릭IP):9092
9 bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
10 bin/kafka-server-start.sh -daemon config/server.properties
11 tail -f logs/*

# Local(macOS) 설치 및 CLI 실행
12 curl <https://archive.apache.org/dist/kafka/2.5.0/kafka_2.13-2.5.0.tgz> --output kafka.tgz
13 tar -xvf kafka.tgz
14 cd kafka_2.13-2.5.0/bin
15 ./kafka-topics.sh --create --bootstrap-server (퍼블릭IP):9092 --replication-factor 1 --partitions 3 --topic test
16 ./kafka-console-producer.sh --bootstrap-server (퍼블릭IP):9092 --topic test
17 ./kafka-console-consumer.sh --bootstrap-server (퍼블릭IP):9092 --topic test --from-beginning
18 ./kafka-console-consumer.sh --bootstrap-server (퍼블릭IP):9092 --topic test -group testgroup --from-beginning
19 ./kafka-consumer-groups.sh --bootstrap-server (퍼블릭IP):9092 --list
20 ./kafka-consumer-groups.sh --bootstrap-server (퍼블릭IP):9092 --group testgroup --describe
21 ./kafka-consumer-groups.sh --bootstrap-server (퍼블릭IP):9092 --group testgroup --topic test --reset-offsets --to-earliest --execute
22 ./kafka-consumer-groups.sh --bootstrap-server (퍼블릭IP):9092 --group testgroup --topic test:1 --reset-offsets --to-offset 10 --execute

# 카프카 프로듀서 실습 및 실행
23 git clone <https://github.com/AndersonChoi/tacademy-kafka.git>
24 ./kafka-console-consumer.sh --bootstrap-server (퍼블릭IP):9092 --topic test --property print.key=true --property key.separator="-"

# 홈브루 설치
25 /bin/bash -c "$(curl -fsSL <https://raw.githubusercontent.com/Homebrew/install/master/install.sh>)"

# 텔레그래프 설치
26 brew install telegraf

또는 <https://drive.google.com/file/d/1iRbOjRvHpEnzbezjdifRWDTOrrD2iDag/view?usp=sharing

27 telegraf -config telegraf.conf

또는 ./telegraf -config telegraf.conf

# 카프카 활용 실습 토픽 생성
28 ./kafka-topics.sh —create —bootstrap-server (퍼블릭IP):9092 —replication-factor 1 —partitions 5 —topic my-computer-metric

# 카프카 활용 실습 토픽 데이터 확인
29 ./kafka-console-consumer.sh —bootstrap-server (퍼블릭IP):9092 —topic my-computer-metric —from-beginning
30 tail -f *.csv
```
