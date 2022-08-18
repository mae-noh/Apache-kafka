# Kafka Connector
> 커넥터를 실행하기 위해서는 커넥트에 플러그인을 추가해야한다. <br>
> *커넥터 자체로는 실행할 수 없다. <br>
> 이 문서는 **분산 모드 커넥트에 싱크커넥터를 추가**하여 실행하는 예제이다. <br>


<br>

#### 목차
- 싱크 커넥터 jar 파일 생성, jar 파일 이동
- 분산 모드 커넥트 설정 파일 수정
- 분산 모드 커넥트 실행

<br>
<br>

## 싱크 커넥터 jar 파일 생성, jar 파일 이동
커넥터를 플러그인으로 추가하기 위해서는 커넥터 자바파일을 압축해야함.

- `build.gradle`에 추가 후, gradle의 `clean` -> `jar`

  ```
  jar {
      from {
          configurations.runtimeClasspath.collect {
              it.isDirectory() ? it : zipTree(it)
          }
      }

      duplicatesStrategy = DuplicatesStrategy.EXCLUDE
  }
  ```
- 추출한 jar 파일은 카프카 커넥터가 참조할 수 있는 디렉토리로 옮겨서 사용한다. (프로젝트 build > libs > .jar)
- 배포된 카프카 커넥트 디렉토리에 plugins 디렉토리 생성하여 jar파일을 옮긴다.
<br>

## 분산 모드 커넥트 설정 파일 수정
*카프카는 `독립형`, `분산형` 선택하여 실행이 가능하다.<br>*
*카프카의 노드에 카프카 워커들을 실행하여 모든 정보를 카프카에 저장한다.*
```
vi config/connect-distributed.properties

bootstrap.servers=<호스트>:9092
plugin.path=~/kafka_{버전}/plugins (위에서 생성한 plugins 디렉토리 경로)
```

## 분산 모드 커넥트 실행
```
bin/connect-distributed.sh config/connect-distributed.properties
```

## 카프카 커맨드 라인 툴

- Connector 목록 조회
  ```
  curl -X GET "http://<호스트(카프카)>:8083/connectors"
  ```

- Connector 생성
  ```
  curl -L -X POST '<호스트(카프카)>:8083/connectors' 
  -H 'Content-Type: application/json' 
  --data-raw '{
      "name": "<커넥터명>",
      "config": {
          "connector.class": "<커넥터클래스> 예)com.pipeline.searchlogsinkconnector.ElasticSearchSinkConnector",
          "topics": "<토픽명>",
          "es.host": "<호스트(ES)>",
          "es.port": "<포트번호>",
          "es.index": "<인덱스명>"
      }
  }'
  ```

- Connector 삭제
  ```
  curl -X DELETE "http://<호스트(카프카)>:8083/connectors/{connector_name}
  ```

- 특정 Connector 상태 조회
  ```
  curl -X GET "http://<호스트(카프카)>:8083/connectors/{connector_name}/status"
  ```

- Connector 일시정지
  ```
  curl -X PUT "http://<호스트(카프카)>:8083/connectors/{connector_name}/pause"
  ```

- Connector 재시작
  ```
  curl -X POST "http://<호스트(카프카)>:8083/connectors/{connector_name}/restart"
  ```

<br>

## 카프카 브로커 Topic 커맨드 라인 툴

- Topic list 확인
  ```
  bin/kafka-topics.sh --bootstrap-server <호스트(카프카)>:9092 --list
  ```

- Topic 내 데이터 확인(신규)
  ```
  bin/kafka-console-consumer.sh --bootstrap-server <호스트(카프카)>:9092 --topic <토픽명>
  ```

- Topic 내 데이터 확인(전체)
  ```
  bin/kafka-console-consumer.sh --bootstrap-server <호스트(카프카)>:9092 --topic <토픽명> --from-beginning
  ```
