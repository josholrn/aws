# Other Serverless Services

## AWS Step Functions
* Step Functions model a workflow as a state machine, with a visual execution history. Start executions through SDK/API calls or integrations such as API Gateway and EventBridge.
* A Task state performs work, including invoking Lambda, running Batch jobs or ECS tasks, accessing DynamoDB, publishing to SNS/SQS, or starting another workflow. Choice, Wait, Pass, Map, Parallel, Fail, and Succeed states handle branching, delay, iteration, concurrency, data pass-through, and completion.
* Handle failures in the workflow with `Retry` and `Catch` rather than embedding all recovery logic in application code. Retry settings include matched errors, delay, backoff, and attempt limits; Catch routes exhausted or selected errors to another state.
* A callback task token pauses execution until an external actor or service calls `SendTaskSuccess` or `SendTaskFailure`. Activity workers can poll for work and return the result.
* Standard workflows suit long-running, auditable processes and support exactly-once execution semantics. Express workflows suit high-volume, short-duration workloads; account for their at-least-once asynchronous or synchronous execution modes and make operations idempotent where needed.

## AppSync and Amplify
* AWS AppSync is a managed GraphQL service. A schema describes the API, and resolvers connect fields to data sources such as DynamoDB, Aurora, OpenSearch, HTTP endpoints, or Lambda. It supports real-time updates and mobile offline data synchronization.
* AppSync authorization options include API keys, IAM, OIDC, and Cognito User Pools. CloudFront can provide a custom domain and HTTPS in front of AppSync.
* AWS Amplify provides tools for building and hosting web/mobile applications: Studio for visual development, libraries to connect app front ends to AWS services, a CLI for backend setup, and Hosting for web delivery and deployment.
* Amplify authentication builds on Cognito; its data features use AppSync and DynamoDB, including local data with synchronization. Hosting supports build/deploy workflows, custom domains, and pull-request previews; build-time end-to-end tests can use Cypress.
