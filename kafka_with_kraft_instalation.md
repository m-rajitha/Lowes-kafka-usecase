# Implementing a Kafka Cluster with KRaft Mode

This guide provides step-by-step instructions to implement a Kafka cluster using KRaft (Kafka Raft Metadata mode). KRaft mode enables Kafka to manage metadata without relying on an external ZooKeeper ensemble.

## Prerequisites

- **Java Development Kit (JDK)**: Below mentioned Kafka release requires Java 17
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

### 11. Verify the Setup

To verify the setup, check the Kafka logs and ensure that the servers have started successfully in KRaft mode.
