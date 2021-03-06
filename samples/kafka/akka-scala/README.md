# Using Akka Streams with Event Hubs for Apache Kafka Ecosystems (Scala)

One of the key benefits of using Apache Kafka is the number of ecosystems it can connect to. Kafka-enabled Event Hubs allow users to combine the flexibility of the Kafka ecosystem with the scalability, consistency, and support of the Azure ecosystem without having to manage on prem clusters or resources - it's the best of both worlds!

This tutorial will show how to connect Akka Streams to Kafka-enabled Event Hubs without changing your protocol clients or running your own clusters. Azure Event Hubs for Apache Kafka Ecosystems supports [Apache Kafka version 1.0.](https://kafka.apache.org/10/documentation.html)

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) before you begin.

In addition:

* [Java Development Kit (JDK) 1.8+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
    * On Ubuntu, run `apt-get install default-jdk` to install the JDK.
    * Be sure to set the JAVA_HOME environment variable to point to the folder where the JDK is installed.
* [sbt](https://www.scala-sbt.org/download.html)
    * On Ubuntu, follow the directions at [https://www.scala-sbt.org/release/docs/Installing-sbt-on-Linux.html](https://www.scala-sbt.org/release/docs/Installing-sbt-on-Linux.html)
* [Git](https://www.git-scm.com/downloads)
    * On Ubuntu, you can run `sudo apt-get install git` to install Git.

## Create an Event Hubs namespace

An Event Hubs namespace is required to send or receive from any Event Hubs service. See [Create Kafka-enabled Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-create-kafka-enabled) for instructions on getting an Event Hubs Kafka endpoint. Make sure to copy the Event Hubs connection string for later use.

## Clone the example project

Now that you have a Kafka-enabled Event Hubs connection string, clone the Azure Event Hubs repository and navigate to the `samples/kafka/akka-scala` subfolder:

```bash
git clone https://github.com/Azure/azure-event-hubs.git
cd azure-event-hubs/samples/kafka/akka-scala
```

## Akka Streams Producer

Using the provided Akka Streams producer example, send messages to the Event Hubs service.

### Provide an Event Hubs Kafka endpoint

#### Producer application.conf

Update the `bootstrap.servers` and `sasl.jaas.config` values in `producer/resources/application.conf` to direct the producer to the Event Hubs Kafka endpoint with the correct authentication. Note that Akka Streams automatically uses the `application.conf` file to load configurations, so changing it's name or location will break this tutorial unless other steps to load configurations are taken. 

```config
akka.kafka.producer {
    #Akka Kafka producer properties can be defined here


    # Properties defined by org.apache.kafka.clients.producer.ProducerConfig
    # can be defined in this configuration section.
    kafka-clients {
        bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
        sasl.mechanism=PLAIN
        security.protocol=SASL_SSL
        sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

### Run producer from the command line

To run the producer from the command line:

```bash
sbt run
```

The producer will now begin sending events to the Kafka-enabled Event Hub at topic `test` and printing the message it sent to stdout. If you would like to change the topic, change the TOPIC constant in `producer/scala/ProducerMain.scala`.

## Akka Streams Consumer

Using the provided consumer example, receive messages from the Kafka-enabled Event Hubs.

### Provide an Event Hubs Kafka endpoint

#### Consumer application.conf

Update the `bootstrap.servers` and `sasl.jaas.config` values in `consumer/resources/application.conf` to direct the consumer to the Event Hubs Kafka endpoint with the correct authentication. Note that Akka Streams automatically uses the `application.conf` file to load configurations, so changing it's name or location will break this tutorial unless other steps to load configurations are taken. 

```config
akka.kafka.consumer {
    #Akka Kafka consumer properties defined here
    wakeup-timeout=60s

    # Properties defined by org.apache.kafka.clients.consumer.ConsumerConfig
    # defined in this configuration section.
    kafka-clients {
       request.timeout.ms=60000
       group.id=akka-example-consumer

       bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
       sasl.mechanism=PLAIN
       security.protocol=SASL_SSL
       sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

### Run consumer from the command line

To run the consumer from the command line:

```bash
sbt run
```

If the Kafka-enabled Event Hub has events (for instance, if your producer is also running), then the consumer should now begin receiving events from topic `test`. If you would like to change the topic, change the TOPIC constant in `consumer/scala/ConsumerMain.scala`.

Check out the [Akka Streams Kafka Guide](https://doc.akka.io/docs/akka-stream-kafka/current/home.html) for more detailed information on Akka Streams.