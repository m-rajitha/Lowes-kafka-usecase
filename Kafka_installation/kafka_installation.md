# Implementing a Kafka Cluster with KRaft Mode

This guide provides step-by-step instructions to implement a Kafka cluster using KRaft (Kafka Raft Metadata mode). KRaft mode enables Kafka to manage metadata without relying on an external ZooKeeper ensemble.

## What is RAFT ?
The Raft protocol is a consensus algorithm designed to manage a replicated log in distributed systems.Raft uses a leader-based approach where a single leader is responsible for managing the replication and ensuring that all followers have the same log entries, providing a clear structure for maintaining consistency.

## Why Kafka uses RAFT with KRAFT ?
- Moving to KRAFT simplifies Kafka architecture by removing zookeeper out of system
- Time to recover metadata during start is faster with KRAFT compared to Zookeeper
- Scaling of partitions - Zookeeper is bottleneck in scaling partitons in metadata maintanence , where as KRAFT is desinged to handle large number of partitions


## Prerequisites

- **Java Development Kit (JDK)**: Below mentioned Kafka release requires Java 8 + (Recommended Java 17)
- **Apache Kafka**: Download the latest Kafka release from Apache Kafka(Currently Apache Kafka v 3.7.0)
  
## Steps to Implement Kafka with KRaft Mode

### 1. Setting up Installation Directory for Kafka

Create a directory for Kafka:

```bash
mkdir /kafka
```

### 2. Installing JDK 17 and Extracting It

Download and extract JDK 17:

```bash
curl -O https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
tar -xvzf jdk-17_linux-x64_bin.tar.gz
```

### 3. Setting up Java Home Variable

Set the `JAVA_HOME` variable:

```bash
export JAVA_HOME=/home/raji/kafka/jdk-17.0.11
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
```

### 4. Activate the Environment Variable

Use the `source` command to activate the environment variable and check the Java version:

```bash
source .bash_profile
java -version
```

### 5. Downloading and Extracting Kafka

Download and extract Kafka:

```bash
curl -O https://dlcdn.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz
tar -xzf kafka_2.13-3.7.0.tgz
cd kafka_2.13-3.7.0
```

### 6. Configuring Broker Properties

Edit the `server.properties` file to configure KRaft mode for each broker. Note that the configuration below is for local installation and is not recommended for critical environments:

```properties
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093,2@localhost:9094,3@localhost:9095
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
log.dirs=/data1
```

**Note:** `node.id`, `listeners`, and `log.dirs` should be unique for each broker. For production cluster `controller roles` should be assigned separately .

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

- **node.id**: The node ID associated with the roles this process is playing when `process.roles` is non-empty. This is required configuration when running in KRaft mode.

### 7. Generate UUID for Kafka Cluster ID

Generate a unique Cluster ID:

```bash
/kafka/kafka_2.13-3.7.0/bin/kafka-storage.sh random-uuid
```

Example output:

```
Q7P2Fv62Riisu9n9jB9Wrg
```

### 8. Add Cluster ID Metadata to Data Directory

Format the storage directories using the generated Cluster ID for each broker:

```bash
/kafka/kafka_2.13-3.7.0/bin/kafka-storage.sh format -t Q7P2Fv62Riisu9n9jB9Wrg -c ../config/kraft/server1.properties
/kafka/kafka_2.13-3.7.0/bin/kafka-storage.sh format -t Q7P2Fv62Riisu9n9jB9Wrg -c ../config/kraft/server2.properties
/kafka/kafka_2.13-3.7.0/bin/kafka-storage.sh format -t Q7P2Fv62Riisu9n9jB9Wrg -c ../config/kraft/server3.properties
```

### 9. Create Systemctl Service Files

Create a systemctl service file for each server in `/etc/systemd/system`:

- `server1.service`
  ```bash
  [Unit]
  Description=Apache Kafka server (broker)
  Documentation=http://kafka.apache.org/documentation.html
  Requires=network.target remote-fs.target
  After=network.target remote-fs.target

  [Service]
  Type=simple
  User=root
  Environment=JAVA_HOME=/home/raji/kafka/jdk-17.0.11
  ExecStart=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-start.sh /home/raji/kafka/kafka_2.13-3.7.0/config/kraft/server1.properties
  ExecStop=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-stop.sh

  [Install]
  WantedBy=multi-user.target
  ```
- `server2.service`
  ```bash
  [Unit]
  Description=Apache Kafka server (broker)
  Documentation=http://kafka.apache.org/documentation.html
  Requires=network.target remote-fs.target
  After=network.target remote-fs.target

  [Service]
  Type=simple
  User=root
  Environment=JAVA_HOME=/home/raji/kafka/jdk-17.0.11
  ExecStart=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-start.sh /home/raji/kafka/kafka_2.13-3.7.0/config/kraft/server2.properties
  ExecStop=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-stop.sh

  [Install]
  WantedBy=multi-user.target
  ```
- `server3.service`
  ```bash
  [Unit]
  Description=Apache Kafka server (broker)
  Documentation=http://kafka.apache.org/documentation.html
  Requires=network.target remote-fs.target
  After=network.target remote-fs.target

  [Service]
  Type=simple
  User=root
  Environment=JAVA_HOME=/home/raji/kafka/jdk-17.0.11
  ExecStart=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-start.sh /home/raji/kafka/kafka_2.13-3.7.0/config/kraft/server3.properties
  ExecStop=/home/raji/kafka/kafka_2.13-3.7.0/bin/kafka-server-stop.sh

  [Install]
  WantedBy=multi-user.target
  ```


### 10. Reload Daemon, Enable, and Start Services

Run the following commands for daemon reload, enabling, and starting the services:

```bash
systemctl daemon-reload
systemctl enable server1.service  # Repeat for server2.service and server3.service
systemctl start server1.service   # Repeat for server2.service and server3.service
systemctl status server1.service  # Repeat for server2.service and server3.service
```

![server1-service-running](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/d2c2598f-1166-4ce0-b69c-8c6cbdd648a5)
![systemctl-services-running](https://github.com/m-rajitha/Lowes-kafka-usecase/assets/142714131/9b1ed154-f236-4bf7-b2a0-28fec3d33ce2)

### 11. Verify the Setup

To verify the setup, kafka logs have been validated and ensure that the servers have started successfully in KRaft mode.

## Conclusion
Above docuement provides a  overview of setting up Kafka with KRAFT mode, and verifying its operation. 
