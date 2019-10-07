

# Start Zeebe and Kafka via Docker Compose

See also https://github.com/zeebe-io/kafka-connect-zeebe/tree/master/examples

This project contains a Docker Compose file to startup a nice set of tools for playing around:

```shell
cd docker
docker-compose up
```

will start:

- [Zeebe](https://zeebe.io), on port `26500` for the client, and port `9600` for monitoring.
    - To check if your Zeebe instance is ready, you can check [http://localhost:9600/ready](http://localhost:9600/ready), 
      which will return a `204` response if ready.
- [Kafka](https://kafka.apache.org/), on port `9092`.
    - [Zookeeper](https://zookeeper.apache.org/), on port `2081`.
- [Kafka Schema Registry](https://docs.confluent.io/current/schema-registry/index.html), on port `8081`.
- [Kafka Connect](https://docs.confluent.io/current/connect/index.html), on port `8083`.
- Monitoring tools
    - [Operate](https://github.com/zeebe-io/zeebe/releases/tag/0.20.0), a [monitoring tool for Zeebe](https://zeebe.io/blog/2019/04/announcing-operate-visibility-and-problem-solving/), on port `8080`.
        - Operate has an external dependency on [Elasticsearch](https://www.elastic.co/), which we'll also run on port `9200`.
    - [Confluent Control Center](https://www.confluent.io/confluent-control-center/), on port `9021`. This will be our tool to monitor the Kafka cluster, create connectors, visualize Kafka topics, etc.

Of course you can customize the Docker Compose file to your needs. This Docker Compose file is also just based on the examples provided by Zeebe and Confluent:

- [Zeebe Docker Compose](https://github.com/zeebe-io/zeebe-docker-compose)
- [CP Docker Images](https://github.com/zeebe-io/zeebe-docker-compose)


# Use Ping Pong Example

See https://github.com/zeebe-io/kafka-connect-zeebe/tree/master/examples/ping-pong

#### Deploy workflow 

```shell
docker-compose -f docker/docker-compose.yml exec zeebe \
  zbctl create instance --variables "{\"name\": \"pong\", \"payload\": { \"foo\": \"bar\"}, \"key\": 1}" ping-pong
```

#### Deploy connectors

If `curl` is not available, you can also use [Control Center](http://localhost:9021) to create the connectors.
Make sure to configure them according to the following properties: [source connector properties](source.json), [sink connector properties](sink.json)

Now create the source connector:

```shell
curl -X POST -H "Content-Type: application/json" --data @examples/ping-pong/source.json http://localhost:8083
```

Next, create the sink connector:

```
curl -X POST -H "Content-Type: application/json" --data @examples/ping-pong/source.json http://localhost:8083
```

#### Create a workflow instance

We can now create a workflow instance:

```shell
docker-compose -f docker/docker-compose.yml exec zeebe \
	zbctl create instance --variables "{\"name\": \"pong\", \"payload\": { \"foo\": \"bar\"}, \"key\": 1}" ping-pong
```

Replace the value of the key variable to change the correlation key.
