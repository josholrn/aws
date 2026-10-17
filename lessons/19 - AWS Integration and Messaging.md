# AWS Integration and Messaging

## Decoupling with Queues and Topics
* Synchronous application calls couple availability and traffic. Asynchronous messaging lets producers and consumers scale independently.
* SQS is a pull-based queue. Standard queues offer high throughput but at-least-once delivery and best-effort ordering; consumers should tolerate duplicates and delete messages after successful processing.
* A message becomes hidden for the visibility timeout after receipt. Extend it if processing needs more time; if it expires first, another consumer can receive it. Configure a dead-letter queue after a maximum receive count to isolate failing messages, then inspect and redrive them.
* FIFO queues preserve order within each message group and deduplicate within a five-minute interval. Distinct group IDs allow parallel processing while maintaining per-group order.
* SNS is pub/sub: publishers send to a topic and each subscription receives a copy. SNS-to-SQS fanout combines broadcast with durable, independently processed queues. Queue access policies must allow SNS to send.
* SNS subscription filter policies route only matching message attributes to a subscriber. SNS FIFO plus SQS FIFO supports ordered fanout.

## Event Routing and Streams
* S3 event notifications can target Lambda, SNS, or SQS, but one event type/prefix combination cannot be configured repeatedly; use SNS fanout for multiple queue consumers. EventBridge provides more flexible event filtering and destinations.
* Kinesis Data Streams stores real-time records for consumers to process or replay during retention. Ordering is guaranteed within a partition key, not across the entire stream.
* Kinesis Data Firehose is a managed delivery path that buffers/streams data to destinations such as S3; it is for delivery rather than custom consumer-driven replay.
* Choose SQS for work items, SNS for one-to-many notifications, and Kinesis for ordered streaming data.
