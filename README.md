# Instructions

https://kafka.apache.org/quickstart#
   

# Install Location

D:\Install\Kafka\kafka_2.11-1.0.0


# Start Zoopkeeper

## Unix
bin/zookeeper-server-start.sh config/zookeeper.properties

## Windows
bin/windows/zookeeper-server-start.bat config/zookeeper.properties


# Start the Kafka Server:

## Unix
bin/kafka-server-start.sh config/server.properties

## Windows
bin/windows/kafka-server-start.bat config/server.properties


# Create a Topic

## Unix
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

## Windows
bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test


# List the Topics

## Unix
bin/kafka-topics.sh --list --zookeeper localhost:2181

## Windows
bin/windows/kafka-topics.bat --list --zookeeper localhost:2181


# Send some Messages

## Unix
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

## Windows
bin/windows/kafka-console-producer.bat --broker-list localhost:9092 --topic test


# Start a Consumers

## Unix
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

## Windows
bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning


# Create Multiple Servers

## Unix and Windows (with Gitbash)
cp config/server.properties config/server-1.properties

cp config/server.properties config/server-2.properties


## Edit the following Properties in each:

config/server-1.properties:

	broker.id=1
	listeners=PLAINTEXT://:9093
	log.dir=/tmp/kafka-logs-1
 
config/server-2.properties:

	broker.id=2
	listeners=PLAINTEXT://:9094
	log.dir=/tmp/kafka-logs-2


## Start the 2 new Servers

## Unix
bin/kafka-server-start.sh config/server.properties
bin/kafka-server-start.sh config/server.properties

## Windows
bin/windows/kafka-server-start.bat config/server-1.properties
bin/windows/kafka-server-start.bat config/server-2.properties


# Create a new topic with a replication factor of three:

## Unix
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic

## Windows
bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic


# Now that we have a cluster how can we know which broker is doing what? To see that run the "describe topics" command:

## Unix
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic

## Windows
bin/windows/kafka-topics.bat --describe --zookeeper localhost:2181 --topic my-replicated-topic

	Output (you can see the leader is node 2):
	
	Topic:my-replicated-topic       PartitionCount:1        ReplicationFactor:3    Configs:
	        Topic: my-replicated-topic      Partition: 0    Leader: 2       Replicas: 2,0,1 Isr: 2,0,1

	- "leader" is the node responsible for all reads and writes for the given partition. Each node will be the 
	  leader for a randomly selected portion of the partitions.
	- "replicas" is the list of nodes that replicate the log for this partition regardless of whether they are 
	  the leader or even if they are currently alive.
	- "isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive 
	  and caught-up to the leader.
	
	
# Publish some messages to the new replicated topic

## Unix
bin/windows/kafka-console-producer.bat --broker-list localhost:9092 --topic my-replicated-topic


# Consume messages from the new replicated topic

## Unix
bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic my-replicated-topic --from-beginning


# Kill the leader to see failover in action (leader was 2 above)

Now let's test out fault-tolerance. Broker 2 was acting as the leader so let's kill it:

## Unix
ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
kill -9 7564

## Windows
wmic process where "caption = 'java.exe' and commandline like '%server-2.properties%'" get processid
ProcessId
6016
taskkill /pid 6016 /f


# Now check details about the topic again

## Unix

## Windows
bin/windows/kafka-topics.bat --describe --zookeeper localhost:2181 --topic my-replicated-topic

	Output (you can see the leader has switched from node 2 to slave node 0, node 2 is no longer in the in-sync 
	replica "isr" set:):

	$ bin/windows/kafka-topics.bat --describe --zookeeper localhost:2181 --topic my-replicated-topic
	Topic:my-replicated-topic       PartitionCount:1        ReplicationFactor:3    Configs:
	Topic:my-replicated-topic       Partition: 0    Leader: 0       Replicas: 2,0,1 Isr: 0,1