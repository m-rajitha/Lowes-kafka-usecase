# Setting Up Kafka Connect in Distributed Mode

This guide provides step-by-step instructions to install and configure Kafka Connect in distributed mode.

## Prerequisites

- A running Kafka cluster
- Java Development Kit (JDK) 8 or higher installed (using JDK 17 - Recommended)
- Kafka Connect binaries (part of the Kafka download package)

Note: In our case , all the pre-requiste are satisfied

## Steps to Set Up Kafka Connect in Distributed Mode

### 1. Start Kafka Servers

Start the Kafka servers:

```bash
# Start Kafka
 sudo systemctl status server1.service
 sudo systemctl status server2.service
 sudo systemctl status server3.service
```

### 2. Create Kafka Connect Distributed Configuration

Create a configuration file for Kafka Connect in distributed mode, `connect-distributed.properties`:

```properties
bootstrap.servers=localhost:9092,localhost:9096,localhost:9097
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
config.storage.topic=connect-configs
offset.storage.topic=connect-offsets
status.storage.topic=connect-statuses
config.storage.replication.factor=3
offset.storage.replication.factor=3
status.storage.replication.factor=3
```

### 3. Create Systemctl Service Files

Create a systemctl service file for each server in `/etc/systemd/system`:

- `connect.service`
  ```bash
  [Unit]
  Description=Apache Kafka connect
  Documentation=http://kafka.apache.org/documentation.html
  Requires=network.target remote-fs.target
  After=network.target remote-fs.target

  [Service]
  Type=simple
  User=root
  Environment=JAVA_HOME=/home/raji/kafka/jdk-17.0.11
  ExecStart=/home/raji/kafka/kafka_2.13-3.7.0/bin/connect-distributed.sh /home/raji/kafka/kafka_2.13-3.7.0/config/connect-distributed.properties

  [Install]
  WantedBy=multi-user.target
  ```

### 4. Create Kafka Connect Topics

Create the topics for storing configuration, offsets, and statuses:

```bash
bin/kafka-topics.sh --create --topic connect-configs --bootstrap-server localhost:9092 --partitions 3 --replication-factor 3
bin/kafka-topics.sh --create --topic connect-offsets --bootstrap-server localhost:9092 --partitions 50 --replication-factor 3
bin/kafka-topics.sh --create --topic connect-statuses --bootstrap-server localhost:9092 --partitions 10 --replication-factor 3
```
  
### 5. Start Kafka Connect in Distributed Mode by Reload Daemon, Enable, and Start Services

Run the following commands for daemon reload, enabling, and starting the connect distributed service:

```bash
systemctl daemon-reload
systemctl enable connect.service  
systemctl start connect.service  
systemctl status connect.service  
```
![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/6e34c20d-dd92-448d-82d0-c6d0156be508)

### 6. Check the list of connect plugins in connect cluster

Do a curl -XGET using the Kafka Connect REST API to get the list of connect plugins

```bash
curl -X GET http://localhost:8083/connector-plugins
```
![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/e6a574d7-84df-4800-ae9d-3d11d1f25a4f)

### 7. Deploying Connectors

#### FileStream Source Connector

Create a configuration file for the FileStream Source Connector, `filestream-source.json`:

```json
{ "name": "file-source-connector",
    "config":
    {
      "topic":"file-stream-topic",
      "connector.class":"FileStreamSource",
      "tasks.max": "1",
      "file": "/home/raji/source.txt"
     }
}
```

Deploy the connector using the Kafka Connect REST API:

```bash
curl -X POST -H "Content-Type: application/json" --data @filestream-source.json http://localhost:8083/connectors
```
![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/75713fd5-8ddc-4b7b-9bab-6a0023e9bb05)

#### FileStream Sink Connector

Create a configuration file for the FileStream Sink Connector, `filestream-sink.json`:

```json
{ "name": "file-sink-connector",
    "config":
    {
      "topics":"file-stream-topic",
      "connector.class":"FileStreamSink",
      "tasks.max": "1",
      "file": "/home/raji/target.txt"
     }
}
```

Deploy the connector using the Kafka Connect REST API:

```bash
curl -X POST -H "Content-Type: application/json" --data @filestream-sink.json http://localhost:8083/connectors
```
![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/796713c4-51d7-4ce2-8dfb-0a92dd13b7a1)


### 8. Verify Connector Deployment

Check the status of the deployed file source connector:

```bash
curl -X GET http://localhost:8083/connectors/file-source-connector/status
```

![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/592b7697-310e-4645-b96f-0697a6adb859)

Check the status of the deployed file sink connector:

```bash
curl -X GET http://localhost:8083/connectors/file-sink-connector/status
```

![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/2efb84c4-f720-405e-b904-0982585b91ab)

### 9. Push data to source file and checking message available in target file 

Before posting data , both files are empty

![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/c42b004b-3a2b-432e-8fcb-a42cc473944f)

Pushed few lines of data to source file

![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/e3a2e820-5861-40bd-a48a-60564986b2b2)

Checking the data in both files

![image](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/e45bf0f9-2d93-4bb8-91bd-6ed5ddebc25c)

### 10. Consuming Messages from Kafka Topic

Consume messages from the Kafka topic to verify that the connector is working:

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic file-stream-topic --from-beginning
```

## Conclusion
Above docuement provides a  overview of setting up Kafka Connect in distributed mode, deploying a connector, and verifying its operation. 
