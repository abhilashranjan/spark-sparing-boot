# spark-spring-boot
This is a maven based project, which require some basic infrastructure to run this. I have use below technology or open source project.
<ul>
<li>Java 8</li>
<li>Spring Boot</li>
<li>Apache kafaka</li> 
<li>Apache spark.</li>
</ul>
I have created one simple application in spring boot which will push messages into kafaka topic and spark job will read that message and print on console.
Before going to implementation get ready with infrastructure.



# Apache Kafka

Kafka is very good distributed messaging plateform and it can also use for streaming with latest versions.
Download <a href="https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.1.0/kafka_2.11-0.10.1.0.tgz">apache kafka</a>.
More information on kafa availabe on <a href="https://kafka.apache.org/intro">kafa wiki</a> .
I have created topic with <i><b>partition size 2</b></i> and<i><b>replication factor 2</b></i>.
Before ceating topic we need to run in kafaka in loacal cluster.
<ul>
<li> Run Zookeeper</li>
    <p>First you need to start the zookeeper, it will be used to store the offsets for topics. There are more advanced versions of using where you don't need it but for someone just starting out it's much easier to use zookeeper bundled with the downloaded kafka. Zookeeper starts at 2181 port bydefault.</p>
    <code>$bin/zookeeper-server-start.sh config/zookeeper.properties</code>
    <li>Configuring brokers</li>
    <p>
    Go to your config directory and copy 2 server.property with name <i><b>serevr0.properties</b></i> and <i><b>serevr1.properties</b></i>. 
    modify below content according to the 0 and 1 order of your properties file.
    </br>
    </br>
    <code>
      broker.id=0
      listeners=PLAINTEXT://:9092
      num.partitions=2
      log.dirs=/var/tmp/kafka-logs-{as per your property file order}
    </code>
    </p>
    
    <li>Start kafa server</li>
    <p>
      </br>
      <code>
        $ bin/kafka-server-start.sh config/server0.properties
        $ bin/kafka-server-start.sh config/server1.properties
      </code>
    </p>
    
    <li>Creating a topic</li>
    <p>
      Before producing and consuming messages we need to create a topic.We need to give a reference to the zookeeper. We'll name a topic "votes", topic will have 2 partitions and a replication factor of 2.
      </br>
      <code>
        $ bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic votes --partitions 2 --replication-factor 2
      </code>
    </p>
</ul>


# Apache Spark
Apache spark is used for data processing ana streaming. In this example I have used for the data streaming and then kafa connector which will read data from topic and print.
We have another module kafka -spark  where we are consuming message kafka topic. To run this example you need to run kafka and previous program for pushing message into topics
then prepare spark context in local mode which is already available in program.
    </br>
    <p>
    <code>
     SparkConf conf = new SparkConf()
                    .setAppName("kafka-sandbox")
                    .setMaster("local[*]");
            JavaSparkContext sc = new JavaSparkContext(conf);
    </code>
    </p>

After spark context create stream object and fetch message from kafka topic and stream as an spark RDD.
 
 <p>
    <code>
        JavaPairInputDStream<String, String> directKafkaStream = KafkaUtils.createDirectStream(ssc,
                        String.class, String.class, StringDecoder.class, StringDecoder.class, kafkaParams, topics);      
                        directKafkaStream.foreachRDD(rdd -> {
                    System.out.println("--- New RDD with " + rdd.partitions().size()
                            + " partitions and " + rdd.count() + " records");
                    rdd.foreach(record -> System.out.println(record._2));
                });
    </code>
 </p>


 
