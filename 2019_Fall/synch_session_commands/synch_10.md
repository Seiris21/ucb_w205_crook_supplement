### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #10

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

### Before class

* VM running
* create 4 linux command lines: cluster commands & kafka, flask web API server, curl web API test commands, spark
* go through the checklist:

https://github.com/kevin-crook-ucb/ucb_w205_crook_supplement/blob/master/2019_Fall/synch_session_commands/checklist_b4_class_assignments.md


### Project 3 - Understanding User Behavior Project

We will wait and discuss project 3 at the end of class.

### Flask with Kafka and Spark

Set up directory, get docker-compose
```
mkdir ~/w205/flask-with-kafka-and-spark/

cd ~/w205/flask-with-kafka-and-spark/

cp ~/w205/course-content/10-Transforming-Streaming-Data/docker-compose.yml .
```

Spin up the cluster
```
docker-compose up -d
```

Create a topic
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

Should show
```
Created topic "events".
```

Web-app
More informative events
```
cp ~/w205/course-content/10-Transforming-Streaming-Data/game_api_with_json_events.py .
```

Run it
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_json_events.py flask run --host 0.0.0.0
```

Test it by generating events
```
docker-compose exec mids curl http://localhost:5000/

docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

Read from kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```

Should show
```
{"event_type": "default"}
{"event_type": "default"}
{"event_type": "default"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
...
```

Exit Flask with a control-C

Even more informative events
```
cp ~/w205/course-content/10-Transforming-Streaming-Data/game_api_with_extended_json_events.py .
```

Run it
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_extended_json_events.py flask run --host 0.0.0.0
```

Test it - generate events
```
docker-compose exec mids curl http://localhost:5000/

docker-compose exec mids curl http://localhost:5000/purchase_a_sword

docker-compose exec mids curl http://localhost:5000/purchase_a_frog
```

Read from kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```

Should see
```
{"event_type": "default"}
{"event_type": "default"}
{"event_type": "default"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
...
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
...
```

Spark it up
Run a spark shell
```
docker-compose exec spark pyspark
```

Read from kafka
```python
raw_events = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","events").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

Explore our events
```python
events = raw_events.select(raw_events.value.cast('string'))

import json

extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()

extracted_events.show()
``` 

down
```
docker-compose down
```
