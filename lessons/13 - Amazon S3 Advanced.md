# Amazon S3 – Advanced

## Storage Classes and Lifecycle
* Choose a storage class to balance access speed, availability, and cost: Standard suits frequent access; IA classes suit less-frequent access; Glacier classes are for archive data with retrieval delays.
* Lifecycle rules automate transitions and expiration. Scope rules by object prefix or tags; they can also clean up old object versions and incomplete multipart uploads.
* Storage Class Analysis helps identify Standard objects that may fit Standard-IA. It is not a real-time report and does not analyze Glacier or One Zone-IA.
* For recoverable deletes, enable versioning: a delete marker hides the current object while older versions remain. Lifecycle rules can transition and eventually expire noncurrent versions.

## Events and Throughput
* S3 event notifications cover operations such as object creation, removal, restore, and replication. They can target Lambda, SQS, or SNS; delivery is usually quick but is not guaranteed to be instantaneous.
* EventBridge offers richer filtering and more destinations, plus archiving and replay. Use it when routing needs are more flexible.
* S3 scales per prefix. Spread high request volumes across prefixes to increase aggregate throughput.
* Multipart upload parallelizes large uploads; it is recommended above 100 MB and required above 5 GB. Transfer Acceleration routes uploads through an edge location. Byte-range GETs can parallelize downloads or fetch only part of an object.

## Object Metadata
* User metadata is supplied at upload as key-value pairs; custom keys use the `x-amz-meta-` prefix and are stored lowercase. Object tags are separate key-value labels useful for access controls and analytics.
* S3 does not provide a search index over metadata or tags. Store searchable object attributes in an external index such as DynamoDB.
