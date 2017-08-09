---
title: Event driven infrastructure on AWS, Kinesis or SNS+SQS?
tags: design
excerpt: Event driven, it sounds cool and feels fancy, but it could screw up your day if you do it wrong. Given all the solutions available, which is the one to choose?
---

Recently at work, we were designing our new messaging system to broadcast changes to multiple destinations. The aim is to simplify the communication chain and to reduce coupling between applications. It is quite clear to us that an event driven system is the best fit here.

Event driven, it sounds cool and feels fancy, but it could screw up your day if you do it wrong. Keeping that in mind, we started to work on our first puzzle: **The Infrastructure**, *through what are we going to send those events?*

## Managed or not Managed

The answer to this question is not always obvious, given all the solutions available out there. Many people nowadays would possibly consider [Apache Kafka](https://kafka.apache.org/), which is a distributed message streaming system, we did give it a try as well. However, not every team/company has enough capacity to properly set it up and maintain it over time. Especially if you don't have *the system buddy* in your team to support that. It didn't take us too much time to realize this and therefore look for a managed service instead.

This leads us to look at the cloud providers, more precisely AWS, due to its popularity and our company support. AWS has several fully managed messaging services: Kinesis Streams being the closest equivalent to Apache Kafka, simpler solutions like SNS and SQS seem also do the job, [especially when you combine the two](http://docs.aws.amazon.com/sns/latest/dg/SNS_Scenarios.html). It's nice that AWS gives us alternative choices, only our initial question is still left unanswered by now...

## Keep an eye on the Requirements

Before we deep dive into specifics of AWS services, it's better to make our mind about *what is exactly required for the event infrastructure*. Without this, we are pretty much blind in making decisions.

Here are some requirements we came up with our system:

-   It must guarantee that all events added to the system are delivered to the consumer.
-   It must support event broadcasting, i.e. multiple producers and consumers at the same time.
-   It must be highly available 24/7 with very short TTR.
-   It must scale easily both on data volume and on concurrent consumers.
-   It should keep the causal order of events bounded in the same context (e.g. events of the same object).
-   It should not require or require very less manual maintenance effort.
-   It should provide a way to monitor and track events passing through.
-   It should not be painful to work with its API/SDK.

As one could see from this list, the main focus is *reliability* and *scalability*. Now it's time to take a closer look at what AWS could offer.

## Kinesis Streams

[Kinesis Streams](https://aws.amazon.com/kinesis/streams/) provides streaming capability for events. It means each consumer can have its own progress of reading events and can also go back in time read old events as many times as needed.

{% include article_image.html image='article_event_infrastructure_kinesis_streams.png' %}

#### Guaranteed Delivery

Kinesis Streams provides guaranteed delivery on its own, any submitted event is guaranteed to be delivered throughout the [retention period up to 7 days](http://docs.aws.amazon.com/streams/latest/dev/kinesis-extended-retention.html). After the retention period, events are removed automatically from the stream.

#### Event Streaming

By definition, Kinesis Streams supports natively event streaming. Each consumer can have different event access pattern as well as have different consuming pace. An event can be read as many times as needed. The consumer can go back in time to read all past events if wanted as well.

#### Shards (Strict Ordering / Instrumented Scaling / Moderated Maintenance)

Kinesis Streams operates based on the capacity unit called Shard. Events are partitioned and stored in different shards. Within the same shard, a strict event ordering is guaranteed. In order to keep the causality, events within the same context should be assigned to the same partition key up front.

Kinesis Streams doesn't automatically scale based on traffic or data volume. External instructions (either manual or scripted) are needed in order to add more shards, thus scaling up. For this reason, the potential maintenance effort is moderated but not negligible.

#### Limitations

-   No auto-scaling, thus higher maintenance effort.
-   Hard reading limit at 5 transactions per second per shard. Putting more shards won't help since all consumers have to read from all shards anyway. This implies adding more consumers will impact existing consumer's reading rate, so hard to scale up the number of consumers. There is an [interesting article](https://brandur.org/kinesis-in-production#five-reads) talking about this.
-   Possible to have event duplication, so the consumer has to [deal with it](http://docs.aws.amazon.com/streams/latest/dev/kinesis-record-processor-duplicates.html).
-   No event topic, to barely reproduce the similar functionality multiple streams must be employed.
-   Rather complicated low-level SDK, the client must maintain by itself the shard reference and event position cursor. The situation is improved by using high-level SDKs, but then the client is constrained to a given programming model.
-   Kinesis Streams supports event data size up to 1MB.

For more details, see [Kinesis Streams limits](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html).

## SNS + SQS

[Simple Notification Service](https://aws.amazon.com/sns/details/) and [Simple Queue Service ](https://aws.amazon.com/sqs/details/) offer highly scalable messaging capability. By combining both solutions, we can build an automatically scalable system for distributing events. Each consumer can have its own event queue set-up as needed, for instance, to provide guaranteed delivery or to allow events being consumed at a different pace.

{% include article_image.html image='article_event_infrastructure_sns_sqs.png' %}

#### Flexible Delivery Model

One of the key features of this solution is the flexibility of choosing/combining different delivery model per consumer.

-   **Direct SNS Subscription** - consumer gains simplicity, high throughput, and low latency. But no guarantee of delivery can be provided, only [simple delivery retry](http://docs.aws.amazon.com/sns/latest/dg/DeliveryPolicies.html) could be configured.
-   **Consumer SQS Standard Queue** - consumer can benefit from the _guaranteed delivery (see notes below)_ throughout the [retention period up to 14 days](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-message-lifecycle.html). Events may be occasionally delivered out of order. See [best-effort ordering](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html#standard-queues-message-order). Also, duplication of events may occasionally occur. See [at-least-once delivery](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html#standard-queues-at-least-once-delivery).
-   **Consumer SQS FIFO Queue** - Not compatible with SNS topic subscription at this time.

> Amazon doesn't actually state that SNS+SQS provides guaranteed delivery. They only informally suggest that in their [FAQ list](https://aws.amazon.com/sns/faqs/):
<br><br>
**Q: Does Amazon SNS guarantee that messages are delivered to the subscribed endpoint?**
<br><br>
When a message is published to a topic, Amazon SNS will attempt to deliver notifications to all subscribers registered for that topic. Due to potential Internet issues or Email delivery restrictions, sometimes the notification may not successfully reach an HTTP or Email end-point. In the case of HTTP, an SNS Delivery Policy can be used to control the retry pattern (linear, geometric, exponential backoff), maximum and minimum retry delays, and other parameters. **If it is critical that all published messages be successfully processed, developers should have notifications delivered to an SQS queue (in addition to notifications over other transports)**.
{:.quote--info}

#### Highly Scalable Consumers

SNS is designed for distributing messages to a large number of consumers with high throughput. By using SNS as the event distribution frontend, we can easily add more consumers at any point of time without impacting the existing consumers.

By using SQS as additional event distribution backend, individual consumers have the ability to easily scale-out to multiple instances to increase the throughput and ensure higher availability.

#### Auto Scaling (High Performance / Low Maintenance)

Both SNS and SQS are scaled automatically based on traffic, no external instructions are needed.
-    SNS doesn't have any limits on throughput, it could scale up almost infinitely.
-    SQS Standard Queue also support nearly-unlimited throughput.

Thanks to the highly managed environment and auto-scale capability, the long-term maintenance effort is minimized. Only the initial effort to setup additional SQS queues is considerable depending on how many consumers will be.

#### Event Topic

SNS offers topics, so events can be grouped and consumed by different consumers. Each consumer or SQS Standard Queue is free to subscribe to one or more topics that it is interested in. In such way, only a subset of the total events is delivered, thus enable more focused delivery and reduce processing overhead.

#### Easy-to-use API

Both SNS and SQS provide easy-to-use API to work with including a Java SDK. The programming model of the client (producer & consumer) is straightforward with minimum configuration up front. See more for [SNS Java SDK](http://docs.aws.amazon.com/sns/latest/dg/using-awssdkjava.html) and [SQS Java SDK](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/examples-sqs.html).

#### Limitations
-   Non-uniformed guarantee on delivery, highly dependent on chosen delivery scenario.
-   SQS Standard Queue provides best-effort ordering, thus the consumer must deal with out of order events.
-   SQS Standard Queue provides at-least-once delivery, thus the consumer must [deal with event duplications](https://stackoverflow.com/questions/37472129/using-many-consumers-in-sqs-queue).
-   SQS FIFO Queue is not compatible with SNS, thus the consumer cannot benefit from the first-in-first-out ordering and exactly-once delivery.
-   SQS supports event data size up to 256KB.

For more details, see [SNS limits](http://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_sns) and [SQS limits](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-limits.html).

## Comparison and Conclusion

We can see both solutions provide a certain level of support on guaranteed delivery, event broadcasting and event ordering. It seems Kinesis Streams have a serious problem on supporting a lot of consumers due to its hard 5 reads per second limit. SNS+SQS, on the other hand, cannot provide absolute event ordering.

Both solutions are highly available and also scale well. Kinesis Streams, again, has the problem of scaling concurrent consumers and its instrumented sharding model also put it a bit harder to scale than SNS+SQS.

Monitoring can be ensured by using [CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and [CloudWatch](https://aws.amazon.com/cloudwatch/) for the two. They also come with a Java SDK, although we find the one from Kinesis Streams a bit complicated to work with.

After considering all the aspects discussed above, we decided to go with SNS+SQS solution, due to its fairly good coverage on all requirements and its simplicity to use.

I will continue to evaluate these aspects during our implementation and production usage. Hopefully, after a few months, I could come back and write about my findings then.

{% include abbreviations.md %}
