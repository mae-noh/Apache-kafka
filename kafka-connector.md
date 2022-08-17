# Kafka Connector

## Connectors

- Connector 생성
```
curl -L -X POST '192.168.0.105:8083/connectors' 
-H 'Content-Type: application/json' 
--data-raw '{
    "name": "file-source-connector",
    "config": {
        "connector.class": "com.pipeline.searchlogsinkconnector.ElasticSearchSinkConnector",
        "topics": "web-log",
        "es.host": "192.168.0.107",
        "es.port": "9201",
        "es.index": "web-log"
    }
}'
```

- Connector 삭제
```
curl -X DELETE "http://localhost:8083/connectors/{connector_name}
```

- Connector 목록 조회
```
curl -X GET "http://localhost:8083/connectors/"
```

- Connector 상세 정보 조회

- Connector config 조회

- 특정 Connector



## 

- Topic list 확인
```
bin/kafka-topics.sh --bootstrap-server 192.168.0.105:9092 --list
```

- Topic 내 데이터 확인
```
bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.105:9092 --topic web-log
```
-
