# Orchestrating a RabbitMQ cluster using Docker

This code refers to https://medium.com/@franalvarez/orchestrating-a-rabbitmq-cluster-using-docker-42937465439e where you can find my motivations to create this repository.

I have provided two configurations to create containers, to the taste of the user: using `docker run` and `docker-compose`

## Using `docker run`

First of all check `rabbitmq.config` file since you will have to configure your own username and password.

1. Start master node:
```bash
docker run -d \
    --name="rabbit1" \
    --hostname="rabbit1"\
    -e RABBITMQ_ERLANG_COOKIE="secret string" \
    -e RABBITMQ_NODENAME="rabbit1" \
    --volume=$(pwd)/rabbitmq.config:/etc/rabbitmq/rabbitmq.config \
    --volume=$(pwd)/definitions.json:/etc/rabbitmq/definitions.json \
    --publish="4369:4369" \
    --publish="5671:5671" \
    --publish="5672:5672" \
    --publish="15671:15671" \
    --publish="15672:15672" \
    --publish="25672:25672" \
    rabbitmq:3-management
```
2. Start slave #1:
```bash
docker run -d \
    --name="rabbit2" \
    --hostname="rabbit2"\
    -e RABBITMQ_ERLANG_COOKIE="secret string" \
    -e RABBITMQ_NODENAME="rabbit2" \
    --volume=$(pwd)/rabbitmq.config:/etc/rabbitmq/rabbitmq.config \
    --volume=$(pwd)/definitions.json:/etc/rabbitmq/definitions.json \
    --link="rabbit1:rabbit1" \
    rabbitmq:3-management
```

3. Start slave #2:
```bash
docker run -d \
    --name="rabbit3" \
    --hostname="rabbit3"\
    -e RABBITMQ_ERLANG_COOKIE="secret string" \
    -e RABBITMQ_NODENAME="rabbit3" \
    --volume=$(pwd)/rabbitmq.config:/etc/rabbitmq/rabbitmq.config \
    --volume=$(pwd)/definitions.json:/etc/rabbitmq/definitions.json \
    --link="rabbit1:rabbit1" \
    --link="rabbit2:rabbit2" \
    rabbitmq:3-management
```

4. View container logs individually
```bash
docker logs -f <rabbit#>
```
This will display the logs for the chosen container, and follow them just like `tail -f /log/path` would do.

5. Run producer / consumer

The producer / consumer scripts were created as simple Node.js scripts so they can be executed using regular bash script execution syntax.


## Using `docker-compose`
Also you can start the whole clustering using `docker-compose` and a YAML definition file

1. Create a network shared by all containers
```bash
docker network create rabbitmq-cluster
```

2. Start cluster:
```bash
docker-compose up -d
```

3. View logs for all containers
```bash
docker-compose logs -f
```

3. Run producer / consumer for testing

Same as in the section detailing `docker run` you can launch the consumer/producer as regular shell scripts.

For more information please refer to https://medium.com/@franalvarez/orchestrating-a-rabbitmq-cluster-using-docker-42937465439e