## 목차
- Kafka Streams API
- Q&A

<br>
<br>


## Kafka Streams API

- 카프카 스트림 API를 사용하여 `프로세서 토폴로지`를 작성할 수 있음.
<img src="https://user-images.githubusercontent.com/65100355/182748739-e806edf2-5504-472f-9ea8-8877c3b7e86c.png" width="20%" height="30%">

- 프로세서 토폴로지란?
    - 카프카 스트림 애플리케이션 로직 정의
    - 그래프 : 노드(스트림 프로세서), 에지(스트림)
- 스트림 프로세서?
    
    토폴로지의 노드, 데이터 변환에 사용
    
    - 카프카 스트림 DSL <br>
    `map`, `filter`, `join`, `aggregation`
    - Process API

<br>

## 애플리케이션 코드 내 Kafka Streams 사용

- Topology 정의, 초기화

   ```java
   import org.apache.kafka.streams.KafkaStreams;
   import org.apache.kafka.streams.kstream.StreamsBuilder;
   import org.apache.kafka.streams.processor.Topology;
   
   // Use the builders to define the actual processing topology, e.g. to specify
   // from which input topics to read, which stream operations (filter, map, etc.)
   // should be called, and so on.  We will cover this in detail in the subsequent
   // sections of this Developer Guide.
   
   StreamsBuilder builder = ...;  // when using the DSL
   Topology topology = builder.build();
   //
   // OR
   //
   Topology topology = ...; // when using the Processor API
   
   // Use the configuration properties to tell your application where the Kafka cluster is,
   // which Serializers/Deserializers to use by default, to specify security settings,
   // and so on.
   Properties props = ...;
   
   KafkaStreams streams = new KafkaStreams(topology, props);
   ```

- Kafka Streams 시작

   ```java
   // Start the Kafka Streams threads
   streams.start();
   ```

- Kafka Streams 에러 핸들링

   ```java
   // Java 8+, using lambda expressions
   streams.setUncaughtExceptionHandler((Thread thread, Throwable throwable) -> {
     // here you should examine the throwable/exception and perform an appropriate action!
   })
   ```

- Kafka Streams 종료

   ```java
   // Stop the Kafka Streams threads
   streams.close();
   ```

- Kafka Streams 종료 훅
SIGTERM 정상 종료를 위해 추가하는 것이 권장됨.

   ```java
   // Java 8+
   // Add shutdown hook to stop the Kafka Streams threads.
   // You can optionally provide a timeout to `close`.
   Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
   ```

<br>

## ****Configuring a Streams Application****

- 필수 설정 파라미터
    - [application.id](http://application.id) : 카프카 클러스터 내 식별자
    - bootstrap.servers : 카프카 서버

   ```java
   Properties props = new Properties();
   props.put(StreamsConfig.APPLICATION_ID_CONFIG, "searchlog-streams-application");props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "host:port");
   ```

## Stream DSL

Stream Processor API 위에 DSL 구축

데이터 처리 작업을 간단하게 DSL(Domain Specific Language) 코드로 표현

## ****Creating source streams from Kafka****

데이터 읽는 방법, KStream, KTable.

- KStream
Topic 데이터를 레코드 스트림으로 저장

   ```java
   import org.apache.kafka.common.serialization.Serdes;
   import org.apache.kafka.streams.StreamsBuilder;
   import org.apache.kafka.streams.kstream.KStream;
   
   StreamsBuilder builder = new StreamsBuilder();
   
   KStream<String, Long> wordCounts = builder.stream(
       "word-counts-input-topic", /* input topic */
       Consumed.with(
         Serdes.String(), /* key serde */
         Serdes.Long()   /* value serde */
       );
   ```

- KTable
    - Topic 데이터를 테이블로 저장, UPSERT
    - Topic의 파티션 하위 집합의 데이터로만 채워짐

- GlobalKTable
    - 모든 Topic 파티션의 데이터로 채워짐
    - Serdes 명시 안할 경우, 기본 Serdes 사용

   ```java
   import org.apache.kafka.common.serialization.Serdes;
   import org.apache.kafka.streams.StreamsBuilder;
   import org.apache.kafka.streams.kstream.GlobalKTable;
   
   StreamsBuilder builder = new StreamsBuilder();
   
   GlobalKTable<String, Long> wordCounts = builder.globalTable(
       "word-counts-input-topic",
       Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as(
         "word-counts-global-store" /* table/store name */)
         .withKeySerde(Serdes.String()) /* key serde */
         .withValueSerde(Serdes.Long()) /* value serde */
       );
   ```

## ****Transform a stream****

KStream 변환은 하나 이상의 KStream 개체 생성이 가능하다.

ex) filter, map은 다른 KStream을 생성한다.
      split은 여러개 KStream을 생성할 수 있다

## **Stateless transformations**

상태 비저장 변환

- 처리를 위해 `상태`를 필요로하지 않음
- 스트림 프로세서와 연결된 `상태저장소`가 필요 없음
- `map`, `filter`

## Stateful transformations

상태 저장 변환

   ```java
   KStream<String, String> textLines = ...;
   
   KStream<String, Long> wordCounts = textLines
       .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
       .groupBy((key, word) -> word)
       .count()
       .toStream();
   ```

   `aggregationing`,`joining`,`windowing`















### 카프카 스트림은 어떻게 띄우는 것인가요?
Kafka Streams 라이브러리를 사용하는 모든 Java 애플리케이션은 Kafka Streams 애플리케이션으로 간주됩니다.

### 카프카 스트림이 기존 Consumer, Producer와 다른점이 무엇인가?
