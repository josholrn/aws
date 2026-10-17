# AWS Lambda

## Execution model and triggers
* Lambda runs code on demand without requiring you to manage servers. Billing is based on invocations and execution duration; allocating more memory also increases available CPU and network capacity.
* Synchronous invokers—including API Gateway, the CLI/SDK, and an Application Load Balancer—wait for the result. The caller handles errors and retries.
* Asynchronous sources such as S3, SNS, and EventBridge enqueue events. Lambda retries failed processing; make handlers idempotent and consider a dead-letter queue (DLQ) or a destination for failed events.
* For streams and queues, configure an event source mapping. Lambda polls the source and invokes the function with batches; duplicate delivery is possible. The execution role must allow reading the source. Stream processing preserves shard order; SQS FIFO ordering is by message group.

## Permissions, configuration, and monitoring
* The execution role grants the function access to AWS services and CloudWatch Logs. A resource-based policy instead grants a service or another account permission to invoke the function.
* Environment variables configure behavior without changing code; encrypt sensitive values with KMS rather than embedding secrets in source.
* Slide limits to remember: 15-minute maximum runtime; 128 MB–10 GB memory; 1,000 default concurrent executions per Region (increaseable); 4 KB environment-variable total; ZIP upload up to 50 MB and uncompressed code plus dependencies up to 250 MB. `/tmp` storage is configurable from 512 MB to 10 GB.
* `$LATEST` is mutable. Published versions are immutable snapshots of code and configuration; mutable aliases can point to versions and split traffic for canary releases. CodeDeploy can automate gradual alias traffic shifts.
* CloudWatch Logs and metrics report executions, duration, errors, concurrency, and throttles. Initialize reusable clients and connections outside the handler, keep packages lean, and avoid recursive invocation.
