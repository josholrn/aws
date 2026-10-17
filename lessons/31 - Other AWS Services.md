# Other AWS Services

## At a glance

* **SES** sends and receives email; **OpenSearch** provides full-text search and analytics; **Athena** runs SQL over S3 data.
* **Kinesis Data Streams** and **MSK** provide different models for streaming records.
* **ACM** and **Private CA** provide public and organization-trusted certificates, respectively.
* **Macie** discovers sensitive data in S3; it does not replace access policies or encryption.
* **AppConfig** validates and gradually deploys runtime configuration separately from application code.

## Email, search, and analytics

### Amazon SES

Amazon Simple Email Service (SES) is a managed service for sending and receiving email at scale. Applications can submit messages through the console, API, or SMTP interface. Common uses include transactional messages, marketing campaigns, and bulk email.

SES provides delivery and bounce statistics, feedback-loop information, and reputation insights to help operators monitor deliverability and spam complaints. Domain authentication can use SPF and DKIM. IP options include shared, dedicated, and customer-owned addresses, which allow different levels of reputation control.

### Amazon OpenSearch Service

OpenSearch provides full-text and partial-match search across document fields, making it a complement to an operational database rather than a replacement for its primary access patterns. For example, a DynamoDB application can keep transactional records in DynamoDB while indexing searchable attributes in OpenSearch.

A common flow uses DynamoDB Streams and Lambda to update the search index. Other ingestion paths include Kinesis Data Firehose, Kinesis Data Streams, CloudWatch Logs, and AWS IoT.

OpenSearch is available as a managed cluster or serverless option and includes OpenSearch Dashboards for visualization. It is not natively a relational SQL database (SQL support can be enabled through a plugin). Security options include IAM/Cognito integration, KMS encryption at rest, and TLS. Use it when users need flexible text search or analytics over fields that are awkward to query through the source database's keys and indexes.

### Amazon Athena

Athena runs serverless SQL queries over data stored in S3, so it is useful for analytics without provisioning a database or cluster. It supports formats including CSV, JSON, ORC, Avro, and Parquet, and is often paired with QuickSight for reports and dashboards. Typical datasets include application exports, VPC Flow Logs, load balancer logs, and CloudTrail data.

The slides give a price example of $5 per TB scanned; verify current rates. Since query cost is tied to scanned data, data layout matters:

* Prefer columnar formats such as Parquet or ORC; a query can read only relevant columns instead of scanning every field.
* Compress files to reduce the amount of data read.
* Partition S3 data using directory keys such as `year=2025/month=10/day=17`, and filter on those partition columns to avoid scanning unrelated data.
* Use reasonably large files (the slides recommend greater than 128 MB) to reduce small-file overhead.

AWS Glue can transform datasets into a query-friendly format such as Parquet or ORC. Athena Federated Query uses data source connectors running on Lambda to query sources beyond S3, such as DynamoDB, relational databases, CloudWatch Logs, and on-premises data; query results can be stored in S3.

## Streaming with Kinesis or Kafka

Amazon MSK is managed Apache Kafka. AWS provisions and manages Kafka brokers (and the supporting ZooKeeper infrastructure described in the slides), while producers and consumers still use Kafka concepts such as topics and partitions.

MSK clusters run in a VPC and can span up to three Availability Zones in the slide's high-availability example. Data persists on EBS volumes for the configured retention period, and MSK recovers from common broker failures. **MSK Serverless** provisions and scales the compute and storage capacity without requiring the customer to size broker capacity.

Kinesis Data Streams and MSK both move streaming records, but they expose different models:

| Kinesis Data Streams | Amazon MSK |
| --- | --- |
| Stream capacity is organized into shards | Kafka data is organized into topics and partitions |
| Shards can be split or merged | Partitions can be added to a topic; they are not merged in the comparison |
| Slide comparison lists a 1 MB record limit | Kafka's default message size is 1 MB and can be configured higher (the slide gives 10 MB as an example) |
| TLS in transit and KMS encryption at rest | PLAINTEXT or TLS in transit; KMS encryption at rest |

Choose MSK when Kafka compatibility, Kafka clients, or an existing Kafka ecosystem is important; choose Kinesis when the managed shard-based stream model fits. Both can feed consumers such as Lambda, EC2, ECS/EKS applications, and stream-processing or ETL services.

Verify current message-size quotas and encryption settings for a real deployment; do not infer that Kafka and Kinesis have identical scaling or consumer semantics.

## Certificates and sensitive-data discovery

### AWS Certificate Manager and Private CA

AWS Certificate Manager (ACM) provisions, deploys, and renews TLS certificates for supported AWS endpoints, including load balancers, CloudFront distributions, and API Gateway. ACM public TLS certificates are free in the slide material.

**AWS Private CA** instead creates private certificate authorities and issues X.509 certificates trusted by an organization's own systems, not automatically by public browsers. Private PKI is useful for internal TLS, device or user authentication, and code signing.

### Amazon Macie

Amazon Macie is a managed data security and privacy service that uses pattern matching and machine learning to discover sensitive information—such as personally identifiable information—in S3. It helps identify and alert on potentially exposed or misplaced sensitive data; findings can integrate with EventBridge for downstream notification and response.

Macie discovers and classifies risk; it is not a replacement for S3 access policies or encryption controls.

## AWS AppConfig

AppConfig validates and deploys dynamic application configuration independently of an application code release. It can serve feature flags, tuning values, or allow/block lists to applications running on EC2, Lambda, ECS, or EKS. Applications can poll for updated configuration and apply it without a full redeployment or restart.

AppConfig supports gradual rollout and rollback if a CloudWatch alarm signals a problem. Validate changes before deployment:

* JSON Schema checks structure and syntax.
* A Lambda validator can apply semantic or business rules.

Configuration sources can include Parameter Store, S3, and Systems Manager documents. Use AppConfig when operators need controlled runtime changes; use a code deployment when the behavior change requires new application logic rather than just new configuration values.
