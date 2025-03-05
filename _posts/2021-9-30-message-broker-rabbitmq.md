---
layout: post
title: "Message Broker: RabbitMQ"
description:
date: "2021.09.30"
image: ../assets/images/rabbitmq/rabbitmq.jpg
---

In this article, I will try to provide a simple introduction to **RabbitMQ**. To define it briefly, **RabbitMQ** is an **open-source** message queuing system. Written in the Erlang programming language, **RabbitMQ** was initially developed for messaging systems. However, over time, as application scales grew, request volumes increased, and response times became longer, RabbitMQ started being preferred to prevent service disruptions. **We observe RabbitMQ and its alternatives being used in various high-traffic projects across multiple sectors, including banking, insurance, and e-commerce.**

![RabbitMQ Image](/assets/images/rabbitmq/rabbitmq-two.jpg)

#### How Does RabbitMQ Work?

Similar to Redis, **RabbitMQ follows a Publisher/Subscriber model**. In RabbitMQ, the **Producer acts as the Publisher**, and the **Consumer acts as the Subscriber**. **A RabbitMQ broker operates between the Producer and Consumer**. The data sent by the **Producer** first **reaches** the **Exchange Type** it is connected to within the **broker**.

![Exchange Types](/assets/images/rabbitmq/rabbitmq-three.jpg)

If the binding key defined in the message matches the binding key of a queue, the message is sent to that queue. Queues operate in a **FIFO** (First In, First Out) manner, ensuring that connected Consumers receive messages in this order. Now, let's take a look at RabbitMQ’s different Exchange Types. **Unlike Redis and Kafka**, **RabbitMQ** provides **three** distinct **Exchange Type** solutions:

![RabbitMQ Working Process](/assets/images/rabbitmq/rabbitmq-four.jpg)

#### Direct Exchange

In this model, the message sent by the **Producer** contains a **routing key**. The Direct Exchange routes the message to the queue whose routing key matches, and then the queue delivers the message to its connected **Consumer**.

For example, in the diagram below, the message sent by the **Producer** includes a **routing key**, and it is transmitted using the **Direct Exchange** method. The queue with the matching key **receives** the message and **delivers** it to its linked **Consumer**.

![Direct Exchange Diagram](/assets/images/rabbitmq/rabbitmq-five.jpg)

#### Topic Exchange

With Topic Exchange, the goal is to send the message to one or multiple queues. The message is routed to the appropriate queue(s) based on pattern-matching rules on the routing key.

If **multiple Consumers** are interested in the same type of message, Topic Exchange is the **best approach**. As shown in the diagram, the routing key contains **three different segments**, and the message is being transmitted using **Topic Exchange**. There are two queues that match this pattern. These queues allow flexible pattern matching, except for specifically defined segments. However, the third queue **does not receive** the message because the third segment in its routing pattern (new) **does not match** the message’s **routing key**.

![Topic Exchange Diagram](/assets/images/rabbitmq/rabbitmq-six.jpg)

#### Fanout Exchange

This exchange type is **commonly used for broadcasting messages**. It **does not** use a **routing key**. Instead, any message sent by the **Producer** is delivered to all queues connected to the **Fanout Exchange**, and each queue then delivers the message to its linked Consumer.

As shown in the diagram, since there is **no routing key**, **all queues connected to the Fanout Exchange** receive the message and pass it to their respective **Consumers**.

![Fanout Exchange Diagram](/assets/images/rabbitmq/rabbitmq-seven.jpg)

#### Experimenting with RabbitMQ

If you want to simulate RabbitMQ’s operation online, **you can use** the [tryrabbitmq](http://tryrabbitmq.com/) simulator. You can also install a RabbitMQ broker on your preferred environment and experiment with it. RabbitMQ can run on a **single node** or as a **cluster** across **multiple nodes**.

Unlike systems with a **Master/Slave architecture**, **RabbitMQ does not require** electing a new master if one of the nodes fails. **Instead**, the system continues running without any downtime.

#### Conclusion

If you need a scalable system, **do not require real-time processing**, and want to **minimize server costs**, **RabbitMQ is one of the best options**. In this article, I have provided an overview of RabbitMQ and its key concepts. I hope you found it useful. See you in my next article!
