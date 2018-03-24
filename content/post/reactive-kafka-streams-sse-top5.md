---
title: "Kafka Streams + SSE = Realtime web app"
date: 2017-12-30T07:29:59+03:30
draft: false
tags: [Kafka,Streams,SSE]
---
Last week I decided to dirty my hands with **Kafka Streams**. I wanted to write simple application with Kafka Streams more 
interesting than **World Count**. I decided to write a program that calculates top 5 using Kafka Streams. Imagine you have written
a game, and you want to display top 5 users. You also need that calculating and displaying top 5 to be real time. You need
whenever a user has hit a record, it automatically displays user record. This is how application look like:
{{< fluid_imgs
	"center|https://raw.githubusercontent.com/mostafa-asg/KStreams/master/asset/KafkaStreams.png|Screenshot"
>}}

We can harness the Kafka streams reactive nature and [Server-Sent Events](https://www.w3schools.com/html/html5_serversentevents.asp)
to build real time web application.Let's dive into the implementation.

#### Implementation details
First of all you need a topic for storing each record. Whenever a user got a score, whether it is a record or not, you store
this record to this topic. This topic act a single [source of truth](https://www.confluent.io/blog/messaging-single-source-truth/):
```
./kafka-topics.sh --create --topic scores --replication-factor 2 --partitions 1 --zookeeper localhost:2181
```
The only point is that I set partitions to one, because I wanted to maintain ordering of messages. You can change replication
factor to any number you want. Besides this topic, we need another topic for storing *High Scores*. This topic stores each user
score if and only if, user has hit a record. Here is defination:
```
./kafka-topics.sh --create --topic high-scores --replication-factor 2 --partitions 1 --zookeeper localhost:2181 --config cleanup.policy=compact
```
The only difference moreover topic name is that we set *cleanup.policy=compact*. This is necessary because we want Kafka
 stores all messages in this topic, and deletes a record only when new record with the same key arrived. So if a user hit a
  record, Kafka automatically delete his last record from this topic. For more information see 
  [Log compacction](http://kafka.apache.org/documentation/#compaction)
  
  #### Kafka Streams
  To find out top score for each user using *Kafka streams*, we can write ([HighScoreStreams.java
](https://github.com/mostafa-asg/KStreams/blob/master/src/main/java/com/github/HighScoreStreams.java)):
  ```
  KStream<String, Integer> scores = builder.stream("scores", Consumed.with(Serdes.String(), Serdes.Integer()));
  KTable<String, Integer> highScores = builder.table("high-scores", Consumed.with(Serdes.String(), Serdes.Integer(),new FailOnInvalidTimestamp(), Topology.AutoOffsetReset.EARLIEST));

  scores.leftJoin( highScores , (v1,v2) -> {

      //this is the first time user has submitted his/her score, so there is no record for this user in 'high-scores' topic
      if(v2 == null){
          return v1;
      }

      //user has hit his/hre record
      if( v1 > v2 ) {
          return v1;
      }

      //this value is not a record, we return null but we don't send null values to 'high-scores' topic
      return null;

  })
  .filter( (k,v) -> v != null ) //filter null values
  .to( "high-scores" , Produced.with(Serdes.String(),Serdes.Integer()));
  ```
  I used **builder.stream** for **scores** topic but **builder.table** for **high-scores** beacuse I only want the last record
  of a user, not a history of his records. With this code, **high-scores** topic contains the high score of all users.
  
  #### Top 5
  For calculating top 5, I wrote a simple Kafka consumer ([HighScores.java](https://github.com/mostafa-asg/KStreams/blob/master/src/main/java/com/github/HighScores.java)), 
  that constantly read from **high-scores** topic, and add high scores to a ArrayList with maximum size of 5. Whenever this
  list changes, I send it through SSE to the clients([HttpServer.java](https://github.com/mostafa-asg/KStreams/blob/master/src/main/java/com/github/HttpServer.java)):
  ```
  get("/highScores", (req, res) -> {

  res.header("Content-Type", "text/event-stream");
  res.header("Cache-Control", "no-cache");
  PrintWriter writer = new PrintWriter(res.raw().getOutputStream());

  HighScores highScores = new HighScores();
  Iterator<List<Score>> iterator = highScores.iterator();
  while (iterator.hasNext()) {
      List<Score> scores = iterator.next();

      writer.write("data: " + toJson(scores) + "\n\n");
      writer.flush();
  }

  return "";
});
  ```
  #### Client side
  On client side([home.html](https://github.com/mostafa-asg/KStreams/blob/master/src/main/resources/home.html)), I use **EventSource** to stablish a connection to SSE server and update DOM, whenever a new event has arrived:
  ```
  $(document).ready(function () {
    var source = new EventSource("/highScores");
    source.onmessage = function (event) {
        var data=JSON.parse(event.data);
        document.getElementById("tblHighScores").innerHTML = "";
        for(var i = data.length-1; i >= 0; i--) {
            document.getElementById("tblHighScores").innerHTML += "<tr><td>" + data[i].time + "</td><td>" + data[i].user + "</td><td>" + data[i].score + "</td></tr>";
        }
    };
   });
  ```
  That's it. You can find the full source code [here](https://github.com/mostafa-asg/KStreams).
