Here is the documentation formatted for a Markdown (.md) file:

```markdown
# Implementing a Kafka Cluster with KRaft Mode

This guide provides step-by-step instructions to implement a Kafka cluster using KRaft (Kafka Raft Metadata mode). KRaft mode enables Kafka to manage metadata without relying on an external ZooKeeper ensemble.

## Prerequisites

- **Java Development Kit (JDK)**: Kafka requires Java 8+
- **Apache Kafka**: Download the latest Kafka release from [Apache Kafka](https://downloads.apache.org/kafka/).

## Steps to Implement Kafka with KRaft Mode

### 1. Download Kafka

Download the latest Kafka release using the following command:

```bash
wget  https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz
```

### 2. Extract Kafka

Extract the downloaded tar file:

```bash
tar -xzf kafka_2.13-3.7.0.tgz
cd  kafka_2.13-3.7.0
```

### 3. Configure Kafka for KRaft Mode

#### Create a Cluster ID

Generate a unique Cluster ID:

```bash
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
echo $KAFKA_CLUSTER_ID
```

#### Format the Log Directories

Format the storage directories using the generated Cluster ID:

```bash
bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
```

### 4. Edit the `server.properties` File to Configure KRaft Mode

Open the `server.properties` file for editing:

```bash
nano config/kraft/server.properties
```

Update the following properties:

```properties
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
log.dirs=/tmp/kraft-combined-logs
```

#### Explanation of Properties

- **process.roles**: Defines the roles of the server. It can be set to `broker`, `controller`, or both (`broker,controller`).
  - `broker`: The server acts as a broker.
  - `controller`: The server acts as a controller.
  - `broker,controller`: The server acts as both a broker and a controller.
  - If `process.roles` is not set, the server operates in ZooKeeper mode.

- **controller.quorum.voters**: Identifies the quorum controller servers. All controllers must be listed, each with their id, host, and port information.
  - Example: `controller.quorum.voters=id1@host1:port1,id2@host2:port2,id3@host3:port3`
  - The node ID in `controller.quorum.voters` must match the corresponding id on the controller servers.

- **listeners**: Specifies the protocols and ports that the server will use to communicate.

- **log.dirs**: Defines the directories where Kafka stores its log files.
