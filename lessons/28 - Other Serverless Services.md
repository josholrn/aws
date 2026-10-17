# Other Serverless Services

## At a glance

* **Step Functions** coordinates multi-step processes as state machines, with branching, retries, and service integrations.
* **Standard** workflows suit long-running, auditable processes; **Express** workflows suit high-volume, short-lived work.
* **AppSync** provides GraphQL APIs that can combine multiple data sources and deliver real-time updates.
* **Amplify** brings together tools and libraries for building, connecting, testing, and hosting web and mobile applications.

## AWS Step Functions

Step Functions coordinates a multi-step process as a state machine. A state machine definition describes its states, transitions, and data flow. The console presents a visual workflow and execution history for troubleshooting. Workflows can be started through the SDK/API or integrations such as API Gateway and EventBridge. Common examples include order fulfillment and data processing, where the process needs branching, retries, or coordination across multiple services.

### States and tasks

A **Task** state performs work. It can:

* Invoke Lambda.
* Submit an AWS Batch job.
* Run an ECS task and wait for it to finish.
* Interact with DynamoDB.
* Publish to SNS or SQS.
* Start another Step Functions workflow.
* Dispatch work to an Activity worker.

Use a service integration when the workflow can call a supported AWS service directly. A Lambda task is a natural choice when custom code is needed.

Other states shape the workflow without necessarily running application code:

* **Choice** evaluates conditions and selects a branch, with a default branch available.
* **Wait** pauses for a duration or until a specified time.
* **Pass** forwards input to output or adds fixed data.
* **Map** iterates over a collection, while **Parallel** starts independent branches.
* **Succeed** and **Fail** end an execution with the corresponding outcome.

### Failures and recovery

State failures can come from invalid workflow logic, a failed task, or transient infrastructure/network conditions. Put recovery behavior in the state machine where possible, rather than reimplementing orchestration in application code.

* A **Retry** rule is evaluated in order and can match named errors or `States.ALL`.
* Retry settings control the initial `IntervalSeconds`, multiplier `BackoffRate`, and `MaxAttempts` (three by default, or zero to disable retries).
* Built-in error names include `States.Timeout`, `States.TaskFailed`, and `States.Permissions`.
* A **Catch** rule routes a matching error to a named next state. `ResultPath` controls how error details are added to the input passed along that path.
* Catch rules are also evaluated in order.

Avoid retrying non-idempotent work blindly: a repeated payment or other side effect may happen more than once if the service performed it before the workflow observed a failure.

### Callback and Activity tasks

For **callback tasks**, append `.waitForTaskToken` to the integration resource. Step Functions supplies a task token to the external worker or service and suspends the task until that token is returned through `SendTaskSuccess` or `SendTaskFailure`. This pattern suits human approval, third-party systems, and legacy integrations. An SQS-based pattern can put the token in a message for a worker to process and respond to. Do not treat the token as ordinary application data: the external actor must return the matching token to complete that particular task.

**Activity tasks** let a separately running worker poll with `GetActivityTask`, process the received input and token, then report success or failure. Workers can run on EC2, ECS, Lambda, or outside AWS. Configure `TimeoutSeconds` and, where applicable, `HeartbeatSeconds`; workers must send periodic heartbeats to show that long-running work is still active. With a long timeout and regular heartbeats, the slide material describes an Activity task remaining active for up to one year. This is distinct from an Express workflow's much shorter maximum duration.

### Standard and Express workflows

| Property | Standard | Express |
| --- | --- | --- |
| Maximum duration | Up to 1 year | Up to 5 minutes |
| Execution semantics | Exactly-once workflow execution by default | Asynchronous is at-least-once; synchronous is at-most-once |
| Throughput | Over 2,000 executions/second in the slide comparison | Over 100,000 executions/second in the slide comparison |
| History | Execution history retained up to 90 days, or use CloudWatch Logs | CloudWatch Logs |
| Pricing basis | Number of executions and state transitions | Executions, duration, and memory consumption |

Choose **Standard** for longer, auditable workflows where execution history and exactly-once workflow semantics matter, such as coordinating microservices or a process with non-idempotent actions. Choose **Express** for high-volume, short-lived work such as streaming ingestion or mobile backends.

Express asynchronous execution does not wait for a workflow result; synchronous execution can return one immediately. Because Express asynchronous delivery can be at-least-once, make side effects idempotent. The slide comparison gives the indicative throughput above; actual service quotas depend on region and account.

## AWS AppSync

AppSync is a managed GraphQL API service. The API begins with a GraphQL schema that defines the fields clients may request; resolvers connect those fields to data sources.

* A single API can combine data from DynamoDB, Aurora, OpenSearch, HTTP endpoints, and custom Lambda logic.
* Clients can request only the fields they need instead of calling a separate endpoint for each source.
* AppSync can expose real-time updates over WebSockets, including MQTT over WebSockets.
* It supports local data access and synchronization for mobile applications.

For authorization, AppSync supports API keys, IAM, OpenID Connect (OIDC) tokens, and Cognito User Pools. Select the method that matches the client and identity model; an API key is not a substitute for user-level authorization when the API contains protected data.

For a custom domain and HTTPS in the slide-described setup, put CloudFront in front of AppSync. AppSync emits metrics and logs to CloudWatch, which helps diagnose resolver and API behavior.

## AWS Amplify

Amplify is a set of tools and libraries for creating and delivering web and mobile applications, rather than one backend service.

* **Amplify Studio** provides visual app/backend development.
* The **Amplify CLI** guides backend configuration.
* Frontend libraries connect applications to AWS services.
* The tools can use Cognito for authentication, S3 for storage, and AppSync with DynamoDB for data.

### Authentication and data

Amplify authentication builds on Cognito and includes sign-up, sign-in, account recovery, MFA, social sign-in, and prebuilt UI components.

Its DataStore approach uses AppSync, GraphQL, and DynamoDB to support local data, offline use, real-time updates, and synchronization with the cloud.

### Hosting and testing

Amplify Hosting provides a build-and-deploy workflow for modern web applications, with a delivery path through CloudFront. Features include custom domains, pull-request previews, monitoring, redirects, custom headers, and optional password protection.

Hosting can run end-to-end tests in the build pipeline using the test phase in `amplify.yml`; Cypress is the slide-highlighted integration. A failing test can catch a regression before production deployment.

Use Amplify when its integrated application workflow fits the project. The underlying services (such as Cognito, AppSync, and DynamoDB) remain the AWS components providing identity and data capabilities.
