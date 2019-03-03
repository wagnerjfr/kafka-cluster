# High availability with Kafka cluster in Docker

## Steps
### 1. Clone the project and cd into the folder
```
$ git clone https://github.com/wagnerjfr/kafka-cluster.git

$ cd kafka-cluster
```
### 2. Create the containers
We are using Docker-Compose to start the containers. Go to the root folder where 'docker-compose.yml' is located and run the below command:
```
$ docker-compose up -d
```
[Optional] You can either open a separate terminal and follow the logs while systems are initializing:
```
$ docker-compose logs -f
```
[Optional] Or check the starting status:
```
$ docker-compose ps
```
P.S: Docker-Compose will create a default Docker network. In my example it's "kafka-cluster_default". Check whether your is the same by running `docker network ls`.

### 3. Add a Kafka topic
Let's add a topic which will have replicas at all the 3 Kafka Brokers (Servers) and with 3 partitions.
```
$ docker run -t --rm --net kafka-cluster_default \
  kafka-ubuntu:latest \
  bin/kafka-topics.sh --create --zookeeper zookeeper:2181 \
  --replication-factor 3 --partitions 3 --topic MyTopic
```
The console should print something like:
```console
Created topic "MyTopic".
```
### 4. Check the distributed partitions
By running the command below, we can see how the partitions are distributed among the Kafka Brokers and who is the leader of each one:
```
$ docker run -t --rm --net kafka-cluster_default \
  kafka-ubuntu:latest \
  bin/kafka-topics.sh --describe --topic MyTopic --zookeeper zookeeper:2181
```
A similar output should appear:
```console
Topic:MyTopic	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: MyTopic	Partition: 0	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3
	Topic: MyTopic	Partition: 1	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	Topic: MyTopic	Partition: 2	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
```
From the output, we see that `Partition: 0` has kafka1 as its leader. `Partition: 1` and `Partition: 2` have kafka2 and kafka3 as their leaders (respectively). The leader handles all read and write requests for the partition while the replicas (followers) passively replicate the leader.

Taking `Partition: 0` as example, we have two more information:
1. `Replicas: 1,2,3`: This shows that `Partition: 0` has replicas at kafka1, kafka2 and kafka3, in our example.
2. `Isr: 1,2,3`: (Isr: in-sync replica) This shows that kafka1, kafka2 and kafka3 are synchronized with the partition's leader.

### 5. Disturbing the cluster
Coming soon.
