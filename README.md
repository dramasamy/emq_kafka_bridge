emqttd_kafka_plugin
===================

This is a plugin for the EMQ broker that sends all messages received by the broker to kafka.

Courtesy: SkylineLabs/emqttd_kafka_bridge

Build the EMQ broker
-------------------

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
{"type":"published","topic":"bbio/bhoruka/publish","client_id":"basicPubSub","username":"test","payload":"{"Hello World!"}","qos":1,"cluster_node":"emq@127.0.0.1","ts":1524625323880}
```	
This is the format in which kafka will receive the MQTT messages

If Kafka consumer shows no messages even after publishing to EMQTT - ACL makes the plugin fail, so please remove all the ACL related code to ensure it runs properly. 
  
License
-------

Apache License Version 2.0


