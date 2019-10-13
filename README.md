# Docker image containing MQTT(Mosquitto server), Kafka(Zookeeper cluster), Kafka Connect(middleware)  MongoDb(sink)

send events in MQTT format from MQTT client to Kafka topic (a producer) which will then be read, consume and stored in MongoDb


## To Run

---

**Make sure docker is installed**
```
docker-compose up "docker_compose.yml"
```

**Extract custom/*/* jars to /custom/jars/ . Kafka Connect requires the 3rd party MQTT and Mongodb jars**
**watch the mount points for the volumes stated in docker_compose.yml**

**Update configuration values through Kafka Connector API**
```
curl -d @./connect-mqtt-source.json -H "Content-Type: application/json" -X POST http://localhost:8083/connectors
```


**Run a quick sanity test**
**Publish: **
```
docker run \
-it --rm --name mqtt-publisher --network 04_custom_default \
efrecon/mqtt-client \
pub -h mosquitto  -t "sample_topic" -m "{\"id\":123,\"message\":\"This is a test\"}"
```

**Listen:**
```
docker run \
--rm \
confluentinc/cp-kafka:5.1.0 \
kafka-console-consumer --network 04_custom_default --bootstrap-server kafka:9092 --topic connect-custom --from-beginning
```

**Update configuration values for Mongodb through Kafka Connector API**
```
curl -d @./connect-mongodb-sink.json -H "Content-Type: application/json" -X POST http://localhost:8083/connectors
```

**Mongodb runs on: http://localhost:3000/**

S**end a new test message:**
```
{
    "firstName": "John",
    "lastName": "Smith",
    "age": 25,
    "address": {
        "streetAddress": "21 2nd Street",
        "city": "New York",
        "state": "NY",
        "postalCode": "10021"
    },
    "phoneNumber": [{
        "type": "home",
        "number": "212 555-1234"
    }, {
        "type": "fax",
        "number": "646 555-4567"
    }],
    "gender": {
        "type": "male"
    }
}
```

**Clean up when your done with testing:**
```
curl -X DELETE http://localhost:8083/connectors/mqtt-source
curl -X DELETE http://localhost:8083/connectors/mongodb-sink
```

