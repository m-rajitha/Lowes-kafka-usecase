Implement a Kafka cluster with KRaft:

To implement a Kafka cluster with KRaft (Kafka Raft Metadata mode), follow these step-by-step instructions. KRaft mode enables Kafka to manage metadata without relying on an external ZooKeeper ensemble.

Prerequisites:

Java Development Kit (JDK): Kafka requires Java 17
Apache Kafka: Download the Kafka release 3.0.0 from Apache kafka


Steps:

1.Download Kafka

wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz

Extract Kafka:

tar -xvzf kafka_2.13-3.7.0.tgz
cd kafka_2.13-3.7.0

2. Configure Kafka for KRaft Mode

Create a Cluster ID:

KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
echo $KAFKA_CLUSTER_ID

Format the Log Directories:

bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties

3. Edit the server.properties file to configure the KRaft mode:

Open config/kraft/server.properties:

vi config/kraft/server.properties

Update the following properties:

process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
log.dirs=/tmp/kraft-combined-logs


Process.roles:

In KRaft mode each Kafka server can be configured as a controller, a broker, or both using the process.roles property. This property can have the following values:

If process.roles is set to broker, the server acts as a broker.
If process.roles is set to controller, the server acts as a controller.
If process.roles is set to broker,controller, the server acts as both a broker and a controller.
If process.roles is not set at all, it is assumed to be in ZooKeeper mode.

controller.quorum.voters:

All of the servers in a Kafka cluster discover the quorum voters using the controller.quorum.voters property. This identifies the quorum controller servers that should be used. All the controllers must be enumerated. Each controller is identified with their id, host and port information. For example:

controller.quorum.voters=id1@host1:port1,id2@host2:port2,id3@host3:port3

Every broker and controller must set the controller.quorum.voters property

The node ID supplied in the controller.quorum.voters property must match the corresponding id on the controller servers.


