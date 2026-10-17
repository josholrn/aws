# AWS Integration and Messaging

## At a glance
- Queues buffer work, pub/sub broadcasts events, and streams retain records for processing and replay.
- SQS provides pull-based queues, including standard and FIFO options.
- SNS distributes messages to subscribers and can fan out durably through SQS.
- Kinesis Data Streams retains records for consumers; Data Firehose manages delivery to destinations.
- Match the service to ordering, durability, replay, throughput, and delivery needs.

## Decoupling with queues and topics
- Direct synchronous calls couple the availability and traffic patterns of services: a slow or overloaded downstream component can hold up its caller.
- Asynchronous messaging lets producers and consumers scale independently and buffers spikes.
- Choose a queue for work that should be processed, pub/sub when several subscribers need an event, and a stream when ordered records must be retained and replayed.

## Amazon SQS
### Standard queues and message handling
- **SQS** is a pull-based, fully managed queue.
- Producers send messages through the SDK; consumers poll, process, then explicitly delete messages.
- Messages remain until deleted or they expire.
- A standard queue supports very high/unlimited throughput and an effectively unlimited queue depth, but delivery is at least once and ordering is best effort.
- Consumers must tolerate duplicates and should make processing idempotent.
- Standard SQS message retention defaults to 4 days and can be configured up to 14 days.
- A message can be up to 1,024 KB.
- Receive requests return at most 10 messages at a time; batch send, delete, and visibility operations reduce request overhead.
- SQS supports HTTPS in transit and KMS encryption at rest.
- IAM controls API access, and a queue resource policy can grant cross-account access or allow services such as SNS or S3 to send.

### Visibility timeout and dead-letter queues
- After a consumer receives a message, the visibility timeout hides it from other consumers while processing continues.
- The default is 30 seconds.
- If the consumer does not delete it before timeout, the message becomes visible again and may be processed twice.
- Set the timeout to fit normal processing and use `ChangeMessageVisibility` for unusually long work.
- An excessively long timeout delays recovery after a consumer crash, while a short timeout creates concurrent duplicate work.
- A redrive policy can send repeatedly failing messages to a dead-letter queue (DLQ) after a configured maximum receive count.
- A FIFO source requires a FIFO DLQ and a standard source requires a standard DLQ.
- A DLQ makes poison messages available for diagnosis; after fixing the cause, use redrive to return messages in batches.
- Ensure DLQ retention is long enough to inspect and repair its messages (14 days is a useful setting).

### Polling, delay, and FIFO
- Long polling waits for messages rather than immediately returning an empty result.
- Configure `ReceiveMessageWaitTimeSeconds` at queue or request level from 1 to 20 seconds; 20 seconds is generally preferred to reduce empty polling calls and improve efficiency.
- Short polling returns sooner but can make more requests.
- Delay queues or per-message delays defer when a message becomes available.
- SQS FIFO preserves order within a `MessageGroupId`, which is required.
- Messages in one group are processed sequentially; different groups allow parallelism, with no ordering guarantee across groups.
- Deduplication uses a supplied deduplication ID or a SHA-256 hash of the body when content-based deduplication is enabled, within a five-minute interval.
- This suppresses duplicate sends in that interval; consumers should still handle retry and processing failures safely.
- The course's baseline FIFO throughput is 300 messages/second without batching or 3,000 with batching.

## Amazon SNS
### Topics and subscriptions
- **SNS** is a push-based publish/subscribe service.
- Producers publish once to a topic; each subscription receives a copy.
- Subscribers may include SQS, Lambda, HTTP(S) endpoints, email, SMS/mobile notifications, and Firehose.
- A topic can have up to 12.5 million subscriptions, with a 100,000-topic account limit in the covered quotas.
- SNS subscription filter policies are JSON rules that select messages using attributes (or a configured message-body scope).
- A subscription without a filter receives all topic messages.
- Filtering avoids sending irrelevant events to each consumer.

### Fanout and FIFO topics
- SNS-to-SQS fanout combines broadcast with durable, independently processed queues.
- Each queue can scale and retry at its own pace, and additional subscribers can be added without changing the producer.
- Configure the SQS resource policy to permit the SNS topic to call `SendMessage`; cross-Region SNS delivery to SQS is supported.
- A plain SNS topic is not a durable work queue: use SQS subscriptions when each consumer needs messages persisted for later polling, retries, and independent processing.
- SNS handles fanout; SQS provides per-subscriber buffering.
- S3 notifications have a constraint: one matching event-type and prefix combination can only be configured once.
- To deliver the same kind of S3 event to multiple queue consumers, send it to SNS and fan out to SQS queues (or use EventBridge for more flexible routing).
- SNS FIFO topics add ordering and deduplication by message group and deduplication ID/content, with throughput limitations similar to FIFO queues.
- Pair an SNS FIFO topic with SQS FIFO subscriptions when broadcast, ordering, and deduplication are all required.

## Streaming and delivery
### Kinesis Data Streams
- **Kinesis Data Streams** collects and retains real-time records.
- Consumers can process or replay retained data; records cannot be individually deleted and expire based on retention, configurable up to 365 days.
- Ordering is guaranteed only for records sharing a partition key/shard, not across the full stream.
- Record size is up to 10 MiB in the course material.
- In provisioned mode, select the shard count and scale it manually.
- A shard supports 1 MB/s or 1,000 records/s of writes and 2 MB/s of reads.
- On-demand mode removes shard management, starts with 4 MB/s or 4,000 records/s of write capacity, and scales based on recent observed peaks (the prior 30 days in the covered behavior).
- Kinesis supports HTTPS in transit and KMS encryption at rest.
- KPL helps producers batch/optimize writes; KCL helps build coordinated consumers.
- Standard consumers pull; enhanced fan-out pushes up to 2 MB/s per shard per consumer.

### Amazon Data Firehose
- **Amazon Data Firehose** is a managed delivery service for loading streaming data to S3, Redshift, OpenSearch, supported partners, or custom HTTP endpoints.
- It automatically scales, can transform records with Lambda, convert formats such as Parquet/ORC, compress, and back up delivered or failed data to S3.
- It buffers by size and time, so delivery is near real time rather than immediate.
- Firehose does not provide retained data for consumer replay and does not require custom consumer code.

### Choosing between Streams and Firehose
| Kinesis Data Streams | Data Firehose |
| --- | --- |
| Use when multiple consumer applications need real-time processing, ordering, and replay from retained records. | Use when the goal is managed delivery into an analytics or storage destination. |
| A stream's partition key governs ordering and capacity distribution. | Prioritizes delivery with buffering. |

### Service roles at a glance
- SQS assigns queued work to consumers.
- SNS broadcasts a notification to subscribers.
- Kinesis Data Streams retains an ordered stream for processing and replay.
- Firehose delivers streaming data to destinations.
