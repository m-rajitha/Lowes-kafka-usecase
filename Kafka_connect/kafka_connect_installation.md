# Setting Up Kafka Connect in Distributed Mode

This guide provides step-by-step instructions to install and configure Kafka Connect in distributed mode.

## Prerequisites

- A running Kafka cluster
- Java Development Kit (JDK) 8 or higher installed
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
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1
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



### 6. Deploying Connectors

#### FileStream Source Connector

Create a configuration file for the FileStream Source Connector, `filestream-source.json`:

```json
{
  "name": "filestream-source",
  "config": {
    "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "tasks.max": "1",
    "file": "/path/to/input/file.txt",
    "topic": "file-stream-topic"
  }
}
```

Deploy the connector using the Kafka Connect REST API:

```bash
curl -X POST -H "Content-Type: application/json" --data @filestream-source.json http://localhost:8083/connectors
```

### 7. Verify Connector Deployment

Check the status of the deployed connector:

```bash
curl -X GET http://localhost:8083/connectors/filestream-source/status
```

### 8. Consuming Messages from Kafka Topic

Consume messages from the Kafka topic to verify that the connector is working:

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic file-stream-topic --from-beginning
```
