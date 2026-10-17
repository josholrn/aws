# Other AWS Services

## Data, search, and streaming
* Amazon SES sends and receives application email at scale, including transactional, marketing, and bulk messages. It provides delivery/bounce insights and supports SPF and DKIM.
* OpenSearch provides text and partial-match search across fields, complementing databases such as DynamoDB rather than replacing their primary-key access patterns. It can ingest from services such as Kinesis Data Firehose and CloudWatch Logs and includes dashboards.
* Athena runs serverless SQL queries directly against data in S3. It supports common formats including CSV, JSON, ORC, Avro, and Parquet. Columnar formats, compression, partitioned S3 paths, and reasonably large files reduce scanned data and query overhead.
* Amazon MSK is managed Apache Kafka in a VPC; AWS manages broker infrastructure, while Kafka topics and partitions remain central to the model. MSK Serverless provisions and scales capacity. Kinesis Data Streams instead uses shards; both offer at-rest encryption, while Kafka can be configured for plaintext or TLS in transit.

## Certificates, data discovery, and configuration
* AWS Certificate Manager (ACM) provisions and renews TLS certificates for supported AWS integrations. Public certificates are free. AWS Private CA issues certificates trusted within an organization's private PKI rather than by the public Internet.
* Amazon Macie uses pattern matching and machine learning to discover sensitive data, including PII, in S3 and can notify or integrate with EventBridge.
* AWS AppConfig deploys dynamic application configuration independently of code releases. It supports gradual rollout, validation with JSON Schema or Lambda, and rollback triggered by CloudWatch alarms. Use it for feature flags and runtime settings when a code deployment or restart is unnecessary.

## Exam service matches
* Need SQL over files in S3? Consider Athena. Need full-text or partial-match search alongside an operational database? Consider OpenSearch.
* Need managed Apache Kafka? Consider MSK; managed stream shards are the Kinesis Data Streams model.
* Need managed public website certificates? ACM. Need internal certificates or private PKI? Private CA. Need to identify sensitive S3 content? Macie.
