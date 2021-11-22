---
template: blog-post
title: Kinesis considerations
slug: /KinesisConsiderations
date: 2021-11-20 12:08
description: Kinesis aws
---
# Amazon Kinesis

## What is Amazon Kinesis

Amazon Kinesis is a real-time, fully managed, and scalable platform for streaming data on Amazon Web Services. It has multiple functionality, allowing one to perform various tasks – such as ingesting and processing real-time data, and developing custom streaming applications for specific requirements. While the Kinesis family is made up by 4 different services, people tend to use the name Kinesis to refer to [Amazon Kinesis Data Streams][1].

Amazon Kinesis Data Streams provides a message broker for (near) real time event streaming. In an on-premise scenario, the closest thing to it would probably Apache Kafka. ([What is Apache Kafka?][2])

Although the two share the same underlying principle of representing an [immutable][3] distributed log, the way they achieve it is quite different. From a first time user perspective, something that can be noticed immediately is that all connections to Kinesis go via HTTP, while Kafka uses a custom binary TCP protocol - although there is a Kafka implementation that supports HTTP ([Nakadi][4]).

For an apple to apple comparison, you can have a look at the table from the [IT Cheerup's comparison chart][5].

On a greenfield project in an AWS setting, people would normally pick Kinesis as their event streaming platform, as it's very simple to use and the overhead required to keep it running is very little.

Having said that, a few considerations should be made before starting to use it.

## Using the AWS official libraries for consuming and producing

AWS provides some official libraries to simplify complicated concepts in a messaging system (like message and batching and consumers resharding). The libraries are battle tested and very well documented, but sadly they do not support many languages.

### KPL (Kinesis Producer Library)

On the producing side, the KPL library is written in C++ and is bundled via a Java package. Using the C++ distributable with a different language is technically feasible but not supported. ([KPL supported platforms][6])

An unofficial Golang implementation of the KPL by Ariel Mashraki is available on [GitHub][7], and it seems to be a relatively healthy project.

The most interesting features provided by the KPL are:

- message batching: the library allows to easily publish events not sharing the same partition key on the same batch
- handling individual event failures: if not all events could be received by Kinesis (aka a partial success condition), the library simplifies dealing with the individual failures
- logging: the library provides automatic logging in response to success/failure of the publication of the events

In case no KPL library is available for the technology at hand or using an unofficial one is not an option, the only alternative is to connect directly to the Kinesis API and provide a bespoke implementation of the needed features.

### KCL (Kinesis Consumer Library)

On the consuming side, although the library started out as multiplatform on v1, on the following version it followed the same path of the KPL - meaning that it is available natively only in Java. 

The main difference in this regard, is that it is available via a MultiLangDaemon ([MultiLangDaemon Code][8]) app that allows call from other languages. In order to make this possible, the Java process needs to be running side by side with application calling it ([MultiLangDaemon on AWS docs][9]).

An unofficial Golang implementation of the KCL by VMWare is available on [GitHub][10], and, similarly to the unofficial KPL library, it also seems to be a healthy project.

The functionality provided by the KCL are most advanced than the ones provided by the KCL. Among them:

- load balancing across multiple consumer application instances: the KCL tries to keep the number of shards per consumer balanced
- responding to consumer application instance failures: if a consumer node crashed, the KCL assigns its shard to different nodes
- check pointing processed records: the status of the consumers (meaning which records have been already read and which have not), are automatically persisted in an underlying DynamoDB table by the KCL
- reacting to resharding: if a shard is split into two, or if two shards are collapsed into one, the KCL takes care of assigning the resulting shard to a consumer

It's quite clear that implementing this functionality is no easy feat, and leveraging the existing libraries is usually the best decision.

## On choosing the partition key

A Kinesis `PutRecord` or `PutRecords` request involves one or more pairs `(PartitionKey, Record)` plus the `StreamName`, which is common for all the records in the same request.
The `PartitionKey` will be used to pick an existing shard where to deposit the `Record`.

From these we can derive that the choice of the `PartitionKey` needs to fulfil the following requirements:

- the values of the `PartitionKey` used in the system should be much bigger than the maximum number of shards we plan to use. Failures to do so, will likely leave some shards busier than others
- the partition key should have a distribution close to the uniform distribution, to allow a similar number of records per shard and therefore share the load uniformly among the shards and hence the consumers
- events that need to respect a temporal ordering, should be sent using the same `PartitionKey`, to always end up in the same shard. As a shard is assigned to a single consumer, this will permit they'll be processed by the same consumer (this will also ultimately depend on whether the custom logic implemented in the same consumer respects the message ordering)
- the number of events per second using the same `PartitionKey` need to be considered: each shard can ingest up to 1 MB/s per second or 1000 records per second (whichever come first). So, in case of having spikes of records with the same `PartitionKey` this will likely result in one or more hot shards and ultimately in throttling.

## On batching

Performing one `PutRecords` operation for N records has the following advantages over performing N `PutRecord` operations:

- the latency cost is paid once instead of N times
- GZIP compression (or similar) can be applied on all the records sharing the same partition key. In this scenario a `PutRecords request rather than being represented by the regular:

```json
{
   "Records": [ 
        { 
            "DataForRecordA": blob
            "PartitionKey1": string
        },
        { 
            "DataForRecordB": blob
            "PartitionKey1": string
        },
        { 
            "DataForRecordC": blob
            "PartitionKey1": string
        }
    ],
    "StreamName": string
}
```

would look instead

```json
{
    "Records": [ 
        { 
            "CompressedDataForRecordA,B,C": blob
            "PartitionKey1": string
        }
    ],
    "StreamName": string
}
```

This has two main benefits:

1. The number of records being written in the `PutRecords` request will be equal to the number of `PartionKey` the request used, rather than the number of total records, making it far less likely to hit the 1000 request per second per shard limit. This in turn leads to being able to use fewer shards.
2. The amount of data being transferred and stored in Kinesis will be greatly reduced (probably somewhere in the range 75-90%, depending on the chosen compression algorithm and the nature of the data being compressed), making it far less likely to hit the 1Mb per shard per second limit. This in turn too leads to being able to use fewer shards.

On both counts, using fewer shards also reduces the DynamoDB capacity required by the KCL (if we're using it).

Two final consideration points: 

- once the records are compressed, the custom code on the receiver side will need to decompress the data and demultiplex the requests.
- if a compressed blob ends up weighting more than 1 MB/s it will hit the Kinesis API Limit. To avoid that, when the threshold is met the records need to be grouped onto multiple blobs, and each blob should be compressed and added to the `PutRecords` request separately.

[1]: https://docs.aws.amazon.com/streams/latest/dev/introduction.html "AWS documentation on Amazon Kinesis Data Streams"
[2]: https://www.confluent.io/what-is-apache-kafka "What is Apache Kafka on Confluent"
[3]: https://www.amazon.com/Art-Immutable-Architecture-Management-Distributed/dp/1484259548 "What immutable means in a distributed architecture in the book The Art of Immutable Architecture: Theory and Practice of Data Management in Distributed Systems"
[4]: https://nakadi.io/ "Nakadi: A distributed event bus that implements a RESTful API abstraction on top of Kafka-like queues"
[5]: http://www.itcheerup.net/2019/01/kafka-vs-kinesis/#:~:text=Key%20Concepts%20Comparison "AWS Kinesis vs Kafka comparison: Which is right for you?"
[6]: https://docs.aws.amazon.com/streams/latest/dev/kinesis-kpl-supported-plats.html#:~:text=64-bit%20only.-,Source%20Code,-If%20the%20binaries "KPL support of additional platforms besides Java"
[7]: https://github.com/a8m/kinesis-producer "Golang Kinesis producer source code"
[8]: https://javadoc.io/static/com.amazonaws/amazon-kinesis-client/1.12.0/com/amazonaws/services/kinesis/multilang/MultiLangDaemon.html "MultiLangDaemon source code"
[9]: https://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html#:~:text=interface%20called%20the-,MultiLangDaemon,-.%20This%20daemon%20is "Multilang deamon explained on AWS docs"
[10]: https://github.com/vmware/vmware-go-kcl "Golang Kinesis consumer source code"
