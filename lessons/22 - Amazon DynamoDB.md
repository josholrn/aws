# Amazon DynamoDB

## Data model and capacity
* DynamoDB is a managed, multi-AZ NoSQL key-value/document database. Design items around access patterns: joins and relational-style aggregations are not its strength. Tables contain items with flexible attributes; an item can be at most 400 KB.
* Choose the primary key at table creation. A partition-key-only key must be unique; a composite partition-plus-sort key is unique as a pair and groups related items under each partition key. Prefer high-cardinality, evenly distributed partition values to avoid hot keys and throttling.
* Provisioned mode sets read and write capacity, with optional autoscaling; excess demand can cause `ProvisionedThroughputExceededException`, for which exponential backoff is appropriate. On-demand mode adjusts to traffic without capacity planning but generally costs more. The slides note that mode changes are limited to once per 24 hours.
* One WCU covers one write per second for an item up to 1 KB (larger items round up). One RCU covers one strongly consistent read/sec of up to 4 KB, or two eventually consistent reads/sec; strong reads consume twice the capacity of eventual reads.

## Access patterns and features
* `GetItem` reads by primary key. `Query` requires a partition-key value and can constrain the sort key; a filter is applied to matching results. `Scan` examines the whole table before filtering, can consume substantial read capacity, and returns at most 1 MB per request; paginate results.
* Batch writes support up to 25 puts/deletes and 16 MB per call; batch reads support up to 100 items and 16 MB. Retry `UnprocessedItems` or `UnprocessedKeys` with backoff.
* Conditional writes protect updates from conflicting changes. Transactions provide all-or-nothing multi-item operations, with about twice the normal read/write capacity cost.
* DynamoDB Streams capture ordered item changes for up to 24 hours and can trigger Lambda. TTL uses a numeric Unix-epoch expiry attribute; deletion is asynchronous and does not consume WCUs. DAX is an in-memory cache for read-heavy workloads.
