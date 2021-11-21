---
template: blog-post
title: Kinesis optimistions
slug: /Kinesis
date: 2021-11-20 12:08
description: Kinesis aws
---
Amazon Kinesis is a real-time, fully managed, and scalable platform for streaming data on Amazon Web Services.It has multiple functionality, allowing one to perform various tasks – such as ingesting and processing real-time data, and developing custom streaming applications for specific requirements. While the Kinesis family is made up by 4 different services, people tend to use the name Kinesis to refer to  [Amazon Kinesis Data Streams][1].

Amazon Kinesis Data Streams provides a message broker for (near) real time event streaming. In an on-premise scenario, the closest thing to it would probably Apache Kafka. ([What is Apache Kafka?][2])

Although the two share the same underlying principle of representing an [immutable][3] distributed log, the way they achieve it is quite different. From a first user perspective, something that can be noticed is that all connections to Kinesis go via HTTP, while Kafka uses a custom binary TCP protocol - although there is a Kafka implementation that supports HTTP ([Nakadi][4]).

For a full apply by apple comparison, you can have a look at the table from the [IT Cheerup's comparison chart][5]

On a greenfield project in an AWS environment, people would normally pick Kinesis as their event streaming platform, as it's very simple to use and the overhead required to keep it running is very little.

Having said that, a few considerations should be made before starting to use it.

## Using the AWS official libraries for consuming and producing

AWS provides some official libraries to simplify complicated concepts in a messaging system (like message and batching and consumers resharding). The libraries are battle tested and very well documented, but sadly they do not support many languages.

### KPL (Customer producing library)

On the producing side, the KPL library is written in C++ and is bundled via a Java package. Using the C++ distributable with a different language is technically feasible but not supported. ([KPL platforms][6])

An unofficial GoLang implementation of the KPL by Ariel Mashraki is available on [GitHub][7], and it seems to be a relatively healthy project.

The most interesting features provided by the KPL are:
- message batching: the library allows to easily publish events not sharing the same partition key on the same batch
- handling individual event failures: if not all events could be received by Kinesis (aka a partial success condition), the library simplifies dealing with the individual failures
- logging: the library provides automatic logging

In case no KPL library is available or using an unofficial one is not an option, the only alternative is to connect directly to the Kinesis API and implement these features (if needed).

[1]: https://docs.aws.amazon.com/streams/latest/dev/introduction.html "AWS documentation on Amazon Kinesis Data Streams"
[2]: https://www.confluent.io/what-is-apache-kafka "What is Apache Kafka on Confluent"
[3]: https://www.amazon.com/Art-Immutable-Architecture-Management-Distributed/dp/1484259548 "What immutable means in a distributed architecture in the book The Art of Immutable Architecture: Theory and Practice of Data Management in Distributed Systems"
[4]: https://nakadi.io/ "Nakadi: A distributed event bus that implements a RESTful API abstraction on top of Kafka-like queues"
[5]: http://www.itcheerup.net/2019/01/kafka-vs-kinesis/#:~:text=Key%20Concepts%20Comparison "AWS Kinesis vs Kafka comparison: Which is right for you?"
[6]: https://docs.aws.amazon.com/streams/latest/dev/kinesis-kpl-supported-plats.html#:~:text=64-bit%20only.-,Source%20Code,-If%20the%20binaries "KPL support of additional platforms besides Java"
[7]: https://github.com/a8m/kinesis-producer "Golang Kinesis producer source code"
