# RabbitMQ

**RabbitMQ** is an open-source message broker that enables applications to communicate with each other asynchronously. It implements the Advanced Message Queuing Protocol (AMQP) and facilitates the sending, receiving, and processing of messages between producers (senders) and consumers (receivers). RabbitMQ acts as an intermediary that decouples systems, allowing them to interact without requiring direct connections.

![image](https://github.com/user-attachments/assets/7ed7f47c-ce10-41c9-bd93-e392918ea528)

**Advantages of RabbitMQ**:
Asynchronous Communication: Enables applications to communicate without waiting for a response, improving overall efficiency and responsiveness.

Decoupling: RabbitMQ decouples producers and consumers, allowing them to operate independently. This is particularly useful in microservices architectures.

Scalability: RabbitMQ supports multiple consumers, allowing you to scale message processing horizontally by adding more consumers to handle large volumes of messages.

Reliability: RabbitMQ ensures message reliability through features like message durability (persisting messages on disk), acknowledgments, and dead-letter queues.

Flexibility: RabbitMQ supports various messaging patterns, including point-to-point, publish/subscribe, and request/reply, making it suitable for different use cases.

Fault Tolerance: RabbitMQ provides built-in mechanisms to handle failures and retries, ensuring that messages are not lost even in the event of consumer or broker failures.

## Installation
Running RabbitMQ as a container in your local environment using Docker is a common setup for development and testing. Below are the step-by-step instructions for pulling the RabbitMQ Docker image and running it as a container locally.

**Step 1: Install Docker**

Ensure that Docker is installed and running on your local machine. You can download Docker Desktop from here and follow the installation instructions.

**Step 2: Pull the RabbitMQ Docker Image**

To pull the RabbitMQ Docker image, you can use the following command in your terminal or command prompt:
`docker pull rabbitmq:management`

The _rabbitmq:management_ tag includes the RabbitMQ Management Plugin, which provides a web-based UI for managing and monitoring RabbitMQ.

**Step 3: Run RabbitMQ Container**

Once the image is pulled, you can run the RabbitMQ container using the following command:

`docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management`

•	-d: Runs the container in detached mode (in the background).

•	--name rabbitmq: Names the container rabbitmq for easy reference.

•	-p 5672:5672: Maps port 5672 on your host to port 5672 in the container. This is the default port for RabbitMQ messaging.

•	-p 15672:15672: Maps port 15672 on your host to port 15672 in the container. This is the port for RabbitMQ's management web UI.

**Step 4: Verify RabbitMQ is Running**

You can verify that the RabbitMQ container is running with the following command:
`docker ps`

This will list all running containers. You should see your RabbitMQ container listed.

**Step 5: Access RabbitMQ Management UI**

Once the container is running, you can access the RabbitMQ management web UI by navigating to: 
_http://localhost:15672/_

![image](https://github.com/user-attachments/assets/fb4f013b-1bc8-4a77-81fc-68587dcebb8b)

•	**Username**: guest   	**Password**: guest

**Step 6: Use RabbitMQ in Your Application**

Now that RabbitMQ is running locally as a container, you can connect to it from your application using the default connection settings:

**Host**: localhost, **Port**: 5672, **Username**: guest, **Password**: guest

## Terminologies
A **Producer**, the application that sends messages to the RabbitMQ server (broker).

A **Consumer**, the application that receives and processes messages from the RabbitMQ server.

The **ConnectionFactory** class is responsible for creating connections to RabbitMQ. It's the starting point when you want to establish communication with the broker.

A **Connection** represents a TCP connection to the RabbitMQ broker. It is a long-lived connection that is established using the ConnectionFactory.

A **Channel** represents a virtual connection within a TCP connection and is used for most of the messaging operations, such as publishing and consuming messages, declaring queues, and bindings.

An **exchange** is a crucial component responsible for routing messages to queues. When a producer sends a message, it first goes to an exchange, which then routes it to the appropriate queue(s) based on routing rules. Exchanges allow for flexible and powerful message routing strategies in a RabbitMQ system.

Types of Exchanges: RabbitMQ provides several types of exchanges, each with different routing behaviors:

•	Direct Exchange: Routes messages to queues based on an exact match between the routing key of the message and the queue binding key.

•	Fanout Exchange: Routes messages to all bound queues indiscriminately, ignoring the routing key. It's useful for broadcasting messages to multiple queues.

•	Topic Exchange: Routes messages to queues based on a pattern match between the routing key and the queue binding key, allowing more complex routing logic (e.g., wildcards).

•	Headers Exchange: Routes messages based on message header values rather than routing keys. It's less common but allows for more granular control.

**Binding**: A binding is the relationship between an exchange and a queue. You bind a queue to an exchange with a binding key, which determines how messages are routed from the exchange to the queue.
•  Routing Key: The routing key is a message attribute used by exchanges to determine how to route the message. The use of the routing key depends on the type of exchange:

•	Direct Exchange: Matches the routing key exactly with the binding key.

•	Topic Exchange: Uses patterns with wildcards (e.g., *.critical).

•	Fanout Exchange: Ignores the routing key.

•	Headers Exchange: Uses headers instead of the routing key.

**Automatic Acknowledgment (autoAck = true)**: The message is acknowledged (and thus removed from the queue) as soon as it is delivered to the consumer, regardless of whether the consumer has successfully processed it. This is faster but riskier, as messages could be lost if the consumer fails.

## Code

Install `RabbitMQ.Client` from Nuget package manager.

```
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

public class RabbitMQTest
{
    public void Send()
    {
        // Create a connection factory
        var factory = new ConnectionFactory()
        {
            HostName = "localhost", // RabbitMQ is running locally
            Port = 5672,
            UserName = "guest",     // Default username
            Password = "guest"      // Default password
        };

        //var factory = new ConnectionFactory()
        //{
        //    UserName = "guest",
        //    Password = "guest",
        //    VirtualHost = "/",
        //    DispatchConsumersAsync = true
        //};

        //// List of RabbitMQ hosts with different ports
        //var endpoints = new List<AmqpTcpEndpoint>
        //{
        //    new AmqpTcpEndpoint("host1", 5672), // RabbitMQ on host1 with default port
        //    new AmqpTcpEndpoint("host2", 5673), // RabbitMQ on host2 with custom port 5673
        //    new AmqpTcpEndpoint("host3", 5674)  // RabbitMQ on host3 with custom port 5674
        //};

        // Create a connection
        using (var connection = factory.CreateConnection())
        {
            // Create a channel
            using (var channel = connection.CreateModel())
            {
                // Declare a queue
                channel.QueueDeclare(queue: "testQueue", // Name of the queue
                                     durable: false,
                                     exclusive: false,
                                     autoDelete: false,
                                     arguments: null);

                string message = "Hello RabbitMQ from Manas!";
                var body = Encoding.UTF8.GetBytes(message);

                // Publish a message to the queue
                channel.BasicPublish(exchange: "",
                                                 routingKey: "testQueue", // Queue name as the routing key
                                                 basicProperties: null,
                                                 body: body);
                Console.WriteLine(" [x] Sent {0}", message);
            }
        }
    }

    public void Receive()
    {
        var factory = new ConnectionFactory()
        {
            HostName = "localhost",
            Port = 5672,
            UserName = "guest",
            Password = "guest"
        };

        using (var connection = factory.CreateConnection())
        {
            using (var channel = connection.CreateModel())
            {
                channel.QueueDeclare(queue: "testQueue",
                                     durable: false,
                                     exclusive: false,
                                     autoDelete: false,
                                     arguments: null);

                var consumer = new EventingBasicConsumer(channel);
                consumer.Received += (model, ea) =>
                {
                    var body = ea.Body.ToArray();
                    var message = Encoding.UTF8.GetString(body);
                    Console.WriteLine(" [x] Received {0}", message);
                };

                channel.BasicConsume(queue: "testQueue",
                                     autoAck: true,
                                     consumer: consumer);

                Console.WriteLine(" Press [enter] to exit.");
                Console.ReadLine();
            }
        }
    }
}
```

Exchange(Direct)

```
channel.ExchangeDeclare(exchange: "direct_logs", type: ExchangeType.Direct);

// Publishing a message with a routing key "error"
string routingKey = "error";
channel.BasicPublish(exchange: "direct_logs",
                     routingKey: routingKey,
                     basicProperties: null,
                     body: messageBody);
//Binding Queues

channel.QueueBind(queue: "errorQueue",
                  exchange: "direct_logs",
                  routingKey: "error");
```

Exchange (Fanout)
```
channel.ExchangeDeclare(exchange: "logs", type: ExchangeType.Fanout);

// Publishing a message (routing key is ignored in fanout)
channel.BasicPublish(exchange: "logs",
                     routingKey: "",
                     basicProperties: null,
                     body: messageBody);
//Binding Queues
channel.QueueBind(queue: "queue1", exchange: "logs", routingKey: "");
channel.QueueBind(queue: "queue2", exchange: "logs", routingKey: "");
```

**Dead Letter Queue (DLQ)**
RabbitMQ supports the concept of a Dead Letter Queue (DLQ), which allows messages that cannot be processed (e.g., due to errors or failures) to be moved to a separate queue for further analysis or retries. This is useful for handling messages that repeatedly fail processing and should not be requeued indefinitely.

Messages can be redirected to a Dead Letter Exchange (DLX) when they are rejected (with BasicNack or BasicReject), expire, or reach their maximum number of allowed requeues.

The DLX can then route these messages to a Dead Letter Queue (DLQ) where you can inspect or handle them further.
Steps to Implement a Dead Letter Queue in RabbitMQ:
Create the Dead Letter Exchange and Queue: Define a DLX and a corresponding DLQ where messages will be routed when they fail.

Bind the Dead Letter Queue to the Dead Letter Exchange: Bind the DLQ to the DLX so that failed messages are automatically routed to the DLQ.

Configure the Original Queue: Specify the Dead Letter Exchange in the configuration of the original queue. This ensures that messages will be redirected to the DLX upon failure.

```
// Create a connection and channel
var factory = new ConnectionFactory() { HostName = "localhost" };
using (var connection = factory.CreateConnection())
using (var channel = connection.CreateModel())
{
    // 1. Declare the Dead Letter Exchange (DLX)
    channel.ExchangeDeclare(exchange: "dead_letter_exchange", type: ExchangeType.Direct);

    // 2. Declare the Dead Letter Queue (DLQ)
    channel.QueueDeclare(queue: "dead_letter_queue", durable: true, exclusive: false, autoDelete: false, arguments: null);

    // 3. Bind the Dead Letter Queue to the Dead Letter Exchange
    channel.QueueBind(queue: "dead_letter_queue", exchange: "dead_letter_exchange", routingKey: "dead_letter_routing_key");

    // 4. Declare the original queue with Dead Letter Exchange settings
    var queueArgs = new Dictionary<string, object>
    {
        { "x-dead-letter-exchange", "dead_letter_exchange" },
        { "x-dead-letter-routing-key", "dead_letter_routing_key" }
    };

    channel.QueueDeclare(queue: "myQueue", durable: true, exclusive: false, autoDelete: false, arguments: queueArgs);

    // Create a consumer with manual acknowledgment
    var consumer = new EventingBasicConsumer(channel);
    
    consumer.Received += (model, ea) =>
    {
        var body = ea.Body.ToArray();
        var message = Encoding.UTF8.GetString(body);
        Console.WriteLine(" [x] Received {0}", message);

        try
        {
            // Process the message here
            ProcessMessage(message);

            // Acknowledge the message after successful processing
            channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
            Console.WriteLine(" [x] Message acknowledged");

        }
        catch (Exception ex)
        {
            // Log the error and reject the message
            Console.WriteLine(" [x] Failed to process message: {0}", ex.Message);

            // Reject the message and route it to the Dead Letter Queue
            channel.BasicNack(deliveryTag: ea.DeliveryTag, multiple: false, requeue: false);
        }
    };

    // Start consuming messages with manual acknowledgment
    channel.BasicConsume(queue: "myQueue", autoAck: false, consumer: consumer);

    Console.WriteLine(" Press [enter] to exit.");
    Console.ReadLine();
}

// Method to process the message
static void ProcessMessage(string message)
{
    // Your message processing logic here
    Console.WriteLine("Processing message: {0}", message);
}

```

## Summary
RabbitMQ is a powerful, flexible, and reliable message broker that facilitates asynchronous communication between applications. It decouples systems, enabling them to operate independently, and supports a wide range of messaging patterns. RabbitMQ is highly scalable and fault-tolerant, making it a popular choice for distributed systems, microservices architectures, and applications that require reliable messaging and event-driven communication.
