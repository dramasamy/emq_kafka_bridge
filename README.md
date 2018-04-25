emq_kafka_plugin
===================

This is a plugin for the EMQ broker that sends all messages received by the broker to kafka.

Courtesy: SkylineLabs/emqttd_kafka_bridge

Step 1 — Create a User for Kafka
--------------------------------

As Kafka can handle requests over a network, you should create a dedicated user for it. This minimizes damage to your Ubuntu machine should the Kafka server be comprised.

Note: After setting up Apache Kafka, it is recommended that you create a different non-root user to perform other tasks on this server.

As root, create a user called kafka using the useradd command:
```
useradd kafka -m
```
Set its password using passwd:
```
passwd kafka
```

Add it to the sudo group so that it has the privileges required to install Kafka's dependencies. This can be done using the adduser command:

```
adduser kafka sudo
```

Your Kafka user is now ready. Log into it using su:

```
su - kafka
```

Step 2 — Install Java
---------------------

Before installing additional packages, update the list of available packages so you are installing the latest versions available in the repository:

```
sudo apt-get update
```
As Apache Kafka needs a Java runtime environment, use apt-get to install the default-jre package:

```
sudo apt-get install default-jre
```

Step 3 — Install ZooKeeper
--------------------------

Apache ZooKeeper is an open source service built to coordinate and synchronize configuration information of nodes that belong to a distributed system. A Kafka cluster depends on ZooKeeper to perform—among other things—operations such as detecting failed nodes and electing leaders.

Since the ZooKeeper package is available in Ubuntu's default repositories, install it using apt-get.
```
sudo apt-get install zookeeperd
```

After the installation completes, ZooKeeper will be started as a daemon automatically. By default, it will listen on port 2181.

To make sure that it is working, connect to it via Telnet:

```
telnet localhost 2181
```

At the Telnet prompt, type in ruok and press ENTER.

If everything's fine, ZooKeeper will say imok and end the Telnet session.

Step 4 — Download and Extract Kafka Binaries
--------------------------------------------

Now that Java and ZooKeeper are installed, it is time to download and extract Kafka.

To start, create a directory called Downloads to store all your downloads.
```
mkdir -p ~/Downloads
cd ~/Downloads
```
Use wget to download the Kafka binaries.
```
wget wget http://apache.cs.utah.edu/kafka/1.1.0/kafka_2.11-1.1.0.tgz
```

Create a directory called kafka and change to this directory. This will be the base directory of the Kafka installation.

```
mkdir -p ~/kafka && cd ~/kafka
```
Extract the archive you downloaded using the tar command.

```
tar -xvzf ~/Downloads/kafka_2.11-1.1.0.tgz --strip 1
```

Step 5 — Configure the Kafka Server
-----------------------------------

The next step is to configure the Kakfa server.

Open server.properties using vi:
```
vi ~/kafka/config/server.properties
```
By default, Kafka doesn't allow you to delete topics. To be able to delete topics, add the following line at the end of the file:

```
~/kafka/config/server.properties
delete.topic.enable = true
port = 9092
advertised.host.name = localhost
```
Save the file, and exit vi.

Step 6 — Start the Kafka Server
-------------------------------

Run the kafka-server-start.sh script using nohup to start the Kafka server (also called Kafka broker) as a background process that is independent of your shell session.
```
nohup ~/kafka/bin/kafka-server-start.sh ~/kafka/config/server.properties > ~/kafka/kafka.log 2>&1 &
```
Wait for a few seconds for it to start. You can be sure that the server has started successfully when you see the following messages in ~/kafka/kafka.log:

excerpt from ~/kafka/kafka.log
```
[2015-07-29 06:02:41,736] INFO New leader is 0 (kafka.server.ZookeeperLeaderElector$LeaderChangeListener)
[2015-07-29 06:02:41,776] INFO [Kafka Server 0], started (kafka.server.KafkaServer)
```
You now have a Kafka server which is listening on port 9092.

Step 7 — Test the Installation
------------------------------

Let us now publish and consume a "Hello World" message to make sure that the Kafka server is behaving correctly.

To publish messages, you should create a Kafka producer. You can easily create one from the command line using the kafka-console-producer.sh script. It expects the Kafka server's hostname and port, along with a topic name as its arguments.

Publish the string "Hello, World" to a topic called TutorialTopic by typing in the following:
```
echo "Hello, World" | ~/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kafka > /dev/null
```
As the topic doesn't exist, Kafka will create it automatically.

To consume messages, you can create a Kafka consumer using the kafka-console-consumer.sh script. It expects the ZooKeeper server's hostname and port, along with a topic name as its arguments.

The following command consumes messages from the topic we published to. Note the use of the --from-beginning flag, which is present because we want to consume a message that was published before the consumer was started.

```
~/kafka/bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic kafka --from-beginning
```

If there are no configuration issues, you should see Hello, World in the output now.

The script will continue to run, waiting for more messages to be published to the topic. Feel free to open a new terminal and start a producer to publish a few more messages. You should be able to see them all in the consumer's output instantly.

When you are done testing, press CTRL+C to stop the consumer script.

Build the EMQ broker
---------------------

Note: EMQ 2.3.6 depends on erlang 19; you may have to execute following commands to setup the dev environment. 

```
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
sudo dpkg -i erlang-solutions_1.0_all.deb
sudo apt-get update
sudo apt-get install esl-erlang=1:19.3.6
sudo apt-get install make 
sudo apt-get install build-essential checkinstall
```

1. clone emq-relx project
```	
git clone https://github.com/emqtt/emq-relx.git
```
2. Add DEPS of the plugin in the Makefile
```
DEPS += emq_kafka_bridge
dep_emq_kafka_bridge = git https://github.com/dramasamy/emq_kafka_bridge.git master
```
3. Add load plugin in relx.config
```
{emq_kafka_bridge, load},
 ```
4. Build
```
cd emq-relx && make
```  
Configuration
----------------------
You will have to edit the configurations of the bridge to set the kafka Ip address and port.

Edit the file emq-relx/_rel/emqttd/etc/plugins/emq_kafka_bridge.config
```
[
  {emqttd_kafka_bridge, [{values, [
	  %%edit this to address and port on which kafka is running
      {bootstrap_broker, {"127.0.0.1", 9092} },
	  %% partition strategies can be strict_round_robin or random
      {partition_strategy, strict_round_robin},
      %% Change the topic to produce to kafka. Default topic is "kafka". It is on this topic that the messages will be sent from the broker to a kafka consumer
	  {kafka_producer_topic, <<"kafka">>}
    ]}]}
].
```

Start the EMQ broker and load the plugin 
-----------------
1) cd emq-relx/_rel/emqttd
2) ./bin/emqttd start
3) ./bin/emqttd_ctl plugins load emq_kafka_bridge

Test
-----------------
Send an MQTT message on a random topic from an MQTT client to you EMQ broker.

The following should be received by your kafka consumer :
```	
{"type":"published","topic":"telemetry/publish","client_id":"basicPubSub","username":"test","payload":"{"Hello World!"}","qos":1,"cluster_node":"emq@127.0.0.1","ts":1524625323880}
```	
This is the format in which kafka will receive the MQTT messages

If Kafka consumer shows no messages even after publishing to EMQTT - ACL makes the plugin fail, so please remove all the ACL related code to ensure it runs properly. 
  
License
-------

Apache License Version 2.0

Thanks
------
This project is based on the code of:

Erlang MQTT Broker [EMQTTD](https://github.com/emqtt/emqttd)
Helpshift Ekaf [ekaf](https://github.com/helpshift/ekaf)
SkylineLabs [emqttd_kafka_bridge](https://github.com/SkylineLabs/emqttd_kafka_bridge)


