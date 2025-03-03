# RabbitMQ Tutorial

<img src="bunny.jpg" width="150" height="150" style="border-radius: 50%;">

RabbitMQ is an open-source message broker that allows applications to communicate asynchronously using messages. This tutorial will guide you through setting up RabbitMQ and using it in a simple producer-consumer model.

## Key Concepts in RabbitMQ

### Producer
A producer is an application that sends messages to RabbitMQ. It creates messages and publishes them to an exchange, which then routes them to the appropriate queue(s).

### Consumer
A consumer is an application that receives messages from a queue in RabbitMQ. It processes the messages as they arrive and acknowledges them once processed.

### Queue
A queue is a buffer that stores messages until they are processed by consumers. Queues allow message decoupling between producers and consumers.

### Exchange
An exchange is responsible for routing messages from producers to queues based on predefined rules. RabbitMQ supports different types of exchanges:
- **Direct Exchange**: Routes messages to queues based on an exact routing key match.
- **Fanout Exchange**: Routes messages to all bound queues, ignoring routing keys.
- **Topic Exchange**: Routes messages to queues based on pattern-matching in routing keys.
- **Headers Exchange**: Routes messages based on header attributes instead of routing keys.

### Binding
A binding is a link between a queue and an exchange, defining how messages should be routed.

### Acknowledgment
Consumers must acknowledge messages to RabbitMQ to confirm successful processing. If not acknowledged, RabbitMQ can re-deliver the message.

### Prefetch Count
Defines how many messages a consumer can handle at a time before acknowledging previous messages. Useful for optimizing resource usage.

## Prerequisites

- Install Docker on your system: [Docker Installation Guide](https://docs.docker.com/get-docker/)
- Basic knowledge of messaging concepts.
- Install Python and `pika` library (for this tutorial, we will use Python as an example).

```sh
pip install pika
```

## Step 1: Running RabbitMQ in Docker

Start a RabbitMQ container with the management plugin enabled:

```sh
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

Then, access the dashboard at: `http://localhost:15672/` (default login: guest/guest).

## Step 2: Creating a Producer (Publisher)

Create a Python script `producer.py` to send messages to a queue.

```python
import pika

# Establish connection with RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('rabbitmq'))
channel = connection.channel()

# Declare a queue
channel.queue_declare(queue='hello')

# Publish a message
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello, RabbitMQ!')

print(" [x] Sent 'Hello, RabbitMQ!'")
connection.close()
```

### Dockerfile for Producer

Create a `Dockerfile` in the producer directory:

```dockerfile
FROM python:3.9
WORKDIR /app
COPY producer.py /app/
RUN pip install pika
CMD ["python", "producer.py"]
```

## Step 3: Creating a Consumer (Receiver)

Create another script `consumer.py` to receive messages from the queue.

```python
import pika

# Establish connection with RabbitMQ server
connection = pika.BlockingConnection(pika.ConnectionParameters('rabbitmq'))
channel = connection.channel()

# Declare the same queue
channel.queue_declare(queue='hello')

# Callback function to process messages
def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")

# Subscribe to the queue
channel.basic_consume(queue='hello',
                      on_message_callback=callback,
                      auto_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

### Dockerfile for Consumer

Create a `Dockerfile` in the consumer directory:

```dockerfile
FROM python:3.9
WORKDIR /app
COPY consumer.py /app/
RUN pip install pika
CMD ["python", "consumer.py"]
```

## Step 4: Running the Example with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
  producer:
    build: ./producer
    depends_on:
      - rabbitmq
  consumer:
    build: ./consumer
    depends_on:
      - rabbitmq
```

## Step 5: Running Everything

1. Ensure Docker is running.
2. Start everything using Docker Compose:
   ```sh
   docker-compose up --build
   ```

You should see the consumer receiving the message sent by the producer.

