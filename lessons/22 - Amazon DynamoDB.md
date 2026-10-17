# Amazon DynamoDB

## At a glance
* DynamoDB is a managed, multi-AZ NoSQL key-value/document database whose tables support flexible item attributes.
* Choose primary keys and indexes around known access patterns and evenly distributed traffic.
* Capacity, consistency, and query/scan behavior determine throughput, latency, and cost.
* Streams, TTL, transactions, DAX, and global tables support operational and application patterns.
* Remember that filters do not avoid evaluated read capacity, TTL is asynchronous, and batch operations can partially succeed.

## Data model and key design
* DynamoDB is a fully managed, multi-AZ NoSQL key-value/document database. A table contains flexible items; attributes need not have the same schema on every item.
  * Attribute types include scalar values, lists/maps, and sets.
  * Maximum item size is 400 KB.
* Decide the table’s primary key at creation.
  * A **partition key** alone must be unique.
  * A **composite key** combines partition and sort keys; the pair is unique, and items sharing a partition-key value are grouped and ordered by sort key.
  * This supports patterns such as all games for a user, or a user’s records over a time range.
* A partition key is hashed to distribute data and throughput. Prefer high-cardinality values with evenly distributed traffic.
  * Low-cardinality or skewed keys—such as a language value dominated by English—or a single popular “hot” key can overload a partition even when aggregate table capacity appears sufficient.
  * Write sharding (adding a random or calculated suffix) spreads hot writes, but reads may then need to query multiple shards and combine results.
* Model for known access patterns rather than expecting relational joins or arbitrary aggregations. Keep large binary objects in S3 and store their object key/URL and searchable metadata in DynamoDB; the 400 KB item cap makes DynamoDB unsuitable for large media payloads.

## Capacity, consistency, and throttling
* **Provisioned mode** specifies read and write capacity, with optional Application Auto Scaling.
  * Burst capacity may temporarily absorb demand, but exceeding available throughput returns `ProvisionedThroughputExceededException`.
  * Retry with exponential backoff and design partition keys to distribute load.
  * Mode changes are limited to once per 24 hours according to the course slides.
* **On-demand mode** (the slides describe it as the default) avoids explicit RCU/WCU planning and charges for request units used. It suits unpredictable or low-forecast workloads but can cost more than well-utilized provisioned capacity.
  * It scales automatically, not without limits: account/table quotas and sharp traffic increases can still cause throttling.
* One WCU supports one write/second for an item up to 1 KB. Larger writes round up to 1 KB units (e.g., a 4.5 KB write consumes five WCUs per write/second).
* One RCU supports one strongly consistent read/second of up to 4 KB, or two eventually consistent reads/second. Read units round up in 4 KB blocks; strong reads use twice the capacity of eventual reads.
* Eventually consistent reads are the default and may briefly return a value preceding a recent write.
  * Set `ConsistentRead=true` for a strongly consistent read where supported (including `GetItem`, `Query`, and `Scan` on a table or LSI).
  * GSI reads do not support strong consistency. Global tables replicate across Regions and are eventually consistent across Regions.
* DynamoDB charges/capacity are affected by item size and operation type, not just item count. Projection expressions reduce returned attributes but do not by themselves reduce the read capacity consumed for evaluated data. DAX can reduce read load for cacheable read-heavy workloads.

## Read/write operations and indexes
### Operations and query patterns
* `GetItem` retrieves by primary key. `PutItem` creates or completely replaces an item with that key; `UpdateItem` changes selected attributes and can create an item if missing. `DeleteItem` can delete conditionally. `UpdateItem` can perform atomic counters.
* `Query` requires equality on the partition key and can constrain the sort key with comparisons, `BETWEEN`, or `begins_with`. It can target a table, LSI, or GSI.
  * A `FilterExpression` applies to matching items after they have been read, so filtering does not avoid the capacity cost of evaluating them.
  * Query pages are limited to 1 MB; continue with the returned pagination key.
* `Scan` examines the entire table/index before applying filters. It is expensive and usually a poor substitute for a designed query.
  * A scan returns up to 1 MB per page; paginate.
  * Parallel scan uses multiple segments for speed but increases consumed throughput and should be capacity-controlled.
* Conditional writes (`ConditionExpression`) make create/update/delete conditional on current values—for example, `attribute_not_exists` to avoid overwriting an existing key, or a version attribute for optimistic locking. This is different from `FilterExpression`, which filters read results.
* `BatchWriteItem` handles up to 25 put/delete operations (up to 16 MB total); it does not support updates. `BatchGetItem` reads up to 100 items and 16 MB.
  * A partial batch may return `UnprocessedItems` or `UnprocessedKeys`; retry only those with exponential backoff.

### Secondary indexes
* A **Local Secondary Index (LSI)** keeps the table’s partition key but offers an alternate sort key.
  * It must be created with the table; a table can have up to five LSIs.
  * It uses the table’s capacity and supports strongly consistent reads.
  * Projected attributes may be `KEYS_ONLY`, `INCLUDE`, or `ALL`.
* A **Global Secondary Index (GSI)** has an alternate partition key and optional sort key, enabling queries by other attributes.
  * It can be created or changed after the table exists, has its own provisioned read/write capacity (or on-demand request usage), and supports eventual consistency only.
  * An under-provisioned or hot GSI can throttle writes to the base table, so size and distribute index capacity and keys carefully.

## Streams, transactions, caching, and operations
* DynamoDB Streams record ordered item-level changes for up to 24 hours. Choose `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, or `NEW_AND_OLD_IMAGES`.
  * Records are not backfilled when streams are enabled.
  * An event source mapping polls shards and invokes Lambda with batches; the Lambda role must permit stream reads.
  * Use streams for near-real-time side effects, derived tables, notifications, search indexing, analytics, or replication.
* TTL automatically expires items using a numeric Unix-epoch timestamp attribute.
  * Deletion is asynchronous (may take days), uses no WCUs, and expired-but-not-yet-deleted items can still appear in reads and queries, so filter them if they must not be shown.
  * TTL deletions also appear in Streams.
* Transactions provide ACID all-or-nothing operations across multiple items/tables: `TransactGetItems` and `TransactWriteItems`.
  * They consume about twice the normal read/write capacity because DynamoDB prepares and commits each item.
  * Use them when a multi-item invariant is essential, rather than for every independent write.
* DAX is a managed, highly available in-memory DynamoDB-compatible cache, useful for microsecond-latency reads and hot keys.
  * The slides list a five-minute default cache TTL and clusters of up to ten nodes.
  * It caches individual items and query/scan results; ElastiCache is more flexible for application-managed objects and aggregation results.
  * DAX does not make a write-heavy access pattern or an unsuitable key design go away.
* Global tables provide multi-Region, multi-active replication. Backups and point-in-time recovery support recovery; encryption at rest uses KMS and transport is protected with TLS. IAM controls access, and VPC endpoints allow private access without traversing the public internet. DynamoDB Local supports local development and tests.

## Exam gotchas
* Query is key-directed; Scan is table-wide. A filter does not reduce read capacity for scanned/evaluated data.
* GSI capacity problems can throttle base-table writes; an LSI shares table capacity.
* TTL is eventual, not an exact timer.
* A batch can partially succeed, and retries must use the returned unprocessed keys/items.
