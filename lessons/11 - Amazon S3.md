# Amazon S3

## Buckets and Objects
* S3 is regional object storage used for backups, archives, static content, software distribution, and data lakes.
* A bucket is created in a Region. Bucket names must satisfy S3 naming rules and the applicable namespace must be unique.
* Objects are stored under keys. A key can include slash-separated prefixes that look like folders, but S3 stores objects rather than a traditional directory tree.
* An object consists of its data and metadata; tags can support organization, access policies, and lifecycle rules. Large uploads use multipart upload.

## Access and Static Websites
* Access can be granted through identity-based IAM policies or resource-based bucket policies. Object and bucket ACLs are legacy alternatives and may be disabled.
* A request must be allowed by the applicable policies and must not be explicitly denied. Use IAM roles for AWS workloads such as EC2 instances that need S3 access.
* Keep S3 Block Public Access enabled unless public access is deliberately required. Bucket policies can grant public reads, cross-account access, or enforce upload requirements.
* S3 website hosting serves static content from a bucket. Publicly accessible website content requires the appropriate public-access configuration and read permissions.

## Versioning and Replication
* Versioning is configured on a bucket. It preserves previous object versions when a key is overwritten and allows recovery from accidental changes or deletes.
* Objects created before versioning was enabled have a `null` version. Suspending versioning does not remove versions already stored.
* Same-Region Replication (SRR) and Cross-Region Replication (CRR) copy objects asynchronously. Enable versioning on both buckets and grant S3 the required permissions.
* Replication normally applies to new objects; use Batch Replication for existing objects or failed replication. Delete-marker replication is configurable; deletes specifying a version ID are not replicated.

## Storage Classes
* **S3 Standard:** frequently accessed data requiring low latency and high availability.
* **S3 Standard-IA:** infrequently accessed data that still needs rapid retrieval, such as backups.
* **S3 One Zone-IA:** lower-cost infrequent-access storage kept in one AZ; use for data that can be recreated.
* **Glacier Instant Retrieval:** archive data that needs millisecond access.
* **Glacier Flexible Retrieval:** archive data with retrieval choices ranging from minutes to hours.
* **Glacier Deep Archive:** lowest-cost long-term archive, with retrieval measured in hours.
* **Intelligent-Tiering** can move data between access tiers as usage changes. Lifecycle rules can transition objects between classes or expire them.
* Durability and availability are different: durability concerns retaining objects; availability varies by storage class.
