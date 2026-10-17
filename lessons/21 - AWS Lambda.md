# AWS Lambda

## At a glance
* Lambda runs short-lived code on demand and scales automatically, subject to concurrency quotas.
* Invocation patterns include synchronous callers, asynchronous event delivery, and polled event source mappings.
* Execution roles authorize AWS API calls; resource-based policies authorize invokers.
* Versions, aliases, layers, concurrency settings, and monitoring shape deployment and operations.
* Remember Lambda’s runtime, duration, storage, environment-variable, and package limits.

## Execution model and economics
* Lambda runs short-lived code on demand without server management. You pay per request and for execution duration, measured in GB-seconds; the free tier includes a monthly request and compute allowance. It scales automatically, but account concurrency quotas still apply.
* Memory is configurable from 128 MB to 10 GB in 1 MB steps. Allocating more memory also allocates more CPU and network capacity:
  * Around 1,792 MB corresponds to one full vCPU, with more CPU available at higher settings.
  * CPU-bound code can benefit from multiple threads.
* The maximum invocation duration is 900 seconds (15 minutes). Use a longer-running compute service such as ECS/Fargate or AWS Batch for workloads that do not fit Lambda’s execution model.
* Supported managed runtimes include Node.js, Python, Java, .NET, PowerShell, and Ruby.
  * A custom runtime or container image must implement the Lambda Runtime API.
  * A Lambda image can be up to 10 GB in ECR; use ECS/Fargate when you need to run an arbitrary container without adapting it to Lambda’s runtime contract.
* `/tmp` is ephemeral execution-environment storage, configurable from 512 MB to 10 GB. It is useful for temporary downloads and processing, not durable/shared data. Lambda can mount EFS for shared persistent file access when configured in a VPC.

## Invocation types and event sources
### Direct and asynchronous invocation
* **Synchronous:** API Gateway, ALB, CLI/SDK, and other direct callers wait for a response. The caller owns retry behavior and should use appropriate backoff.
  * For ALB integration, register the function in a Lambda target group. ALB translates HTTP fields into a JSON event and expects a response containing status, headers, and body.
  * Enable ALB multi-value headers when repeated query parameters or headers must remain arrays.
* **Asynchronous:** S3, SNS, and EventBridge place events on Lambda’s internal queue and return without waiting for function completion.
  * Lambda retries function errors (the course describes three total attempts, with delays between retries); duplicate processing is possible. Make handlers idempotent.
  * Configure retry/age limits and an on-failure destination or DLQ as appropriate.
  * Destinations can capture successful as well as failed async outcomes. A DLQ is primarily a record of the original event, while a destination carries an invocation record.

### Event source mappings
* Lambda polls Kinesis Data Streams, DynamoDB Streams, and SQS, then invokes the function synchronously with batches. The function’s execution role needs read permissions for the source.
* Unlike async invocation DLQs, an SQS source’s DLQ is configured on the queue; mapping failures can also use failure destinations where supported.
* For Kinesis and DynamoDB Streams, mappings read per shard and normally preserve order. A batch failure can cause the batch to be retried and block that shard.
  * Configure retry/record-age limits, batch bisection, or destinations to limit poison-record impact.
  * Parallelization can process up to ten batches per shard while preserving per-partition-key ordering.
* SQS standard queues scale out and do not guarantee order; duplicate delivery can occur even without a function error. Lambda deletes messages after successful processing.
  * Set the queue visibility timeout to at least six times the function timeout as a practical starting point.
  * FIFO preserves order within a message group and scales with active message groups.
* An Event object contains the invoking service’s JSON payload; the Context object describes the invocation and runtime (for example, request ID, function name, and memory limit). Neither is an AWS credential source.

## Permissions, networking, and configuration
* The **execution role** is assumed by the function and authorizes its calls to AWS APIs, including log writes and reads for polled event sources. Use least privilege and, ideally, a role per function. The role does not itself authorize a service to invoke the function.
* A **resource-based policy** grants invoke permission to a service, account, or other principal. For example, S3 needs permission in the function policy to invoke it. Successful access can require both the caller’s identity policy and the function’s resource policy, depending on the caller.
* Environment variables are string key/value configuration that lets code vary by environment without a code change. Their total size is limited to 4 KB.
  * Lambda can encrypt them with its service key or a customer-managed KMS key.
  * Use Secrets Manager or Parameter Store for managed secret lifecycles rather than embedding credentials in code.
* To access private resources, attach Lambda to selected VPC subnets and security groups; Lambda creates elastic network interfaces.
  * A function in a public subnet does **not** receive a public IP.
  * Outbound internet access from private subnets requires a NAT path; VPC endpoints can provide private access to supported AWS services.
* Lambda Function URLs provide a direct HTTPS endpoint for a function. Select `NONE` for public access or `AWS_IAM` to require IAM authorization; the URL is not a replacement for API Gateway features such as API management, request validation, usage plans, or caching.

## Deployment, observability, and limits
* `$LATEST` is mutable. A published version is an immutable snapshot of code and most configuration; aliases such as `dev` and `prod` are mutable pointers to versions.
  * Aliases can split traffic, and CodeDeploy can shift traffic all at once, linearly, or by canary with hooks and alarm-driven rollback.
* A Lambda layer packages shared libraries or data separately from function code. Keep packages lean:
  * ZIP deployment is limited to 50 MB when uploaded directly and 250 MB uncompressed for code and dependencies.
  * Layers count toward the uncompressed limit.
* The default regional concurrency quota is 1,000 and can be raised.
  * Reserved concurrency sets aside/limits capacity for a function; it can protect other functions from a noisy neighbor, but setting it to zero stops invocations.
  * Provisioned concurrency keeps execution environments initialized to reduce startup latency; it incurs cost and is configured against a version or alias.
* Logs go to CloudWatch Logs when the execution role permits log writes. CloudWatch metrics include invocations, duration, errors, throttles, and concurrent executions. X-Ray can trace supported invocation paths.
* Initialize SDK clients and reusable connections outside the handler to reuse the execution environment, but do not rely on it for durable state. Avoid recursive triggers that invoke the same function indefinitely.

## Exam gotchas
* **Exam figures from the slides:** 15-minute maximum execution; 128 MB–10 GB memory; 1,000 default concurrent executions per Region; 4 KB total environment variables; 50 MB direct ZIP upload; 250 MB uncompressed function package; `/tmp` 512 MB–10 GB.
* Quotas and packaging paths can differ by deployment method and change over time, so distinguish a hard runtime limit from an increaseable regional quota.
