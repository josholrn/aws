# Amazon S3

## At a glance
- S3 stores objects in buckets, with keys rather than filesystem directories.
- Access depends on applicable identity, resource, organization, endpoint, and encryption-key controls.
- Versioning and replication support recovery and copying, but do not prevent every form of loss.
- Storage classes and lifecycle rules trade cost against access, restore time, and resilience.

## Buckets and objects
- **Amazon Simple Storage Service (S3)** is highly scalable object storage.
  - Common uses include backups, disaster recovery, archives, static website assets, software distribution, media, data lakes, and application data.
  - S3 integrates with many AWS analytics, logging, and event services.
- A bucket is created in a selected AWS Region.
  - Bucket names are shared across the applicable S3 namespace and must follow current naming rules; historically, standard bucket names were globally unique.
  - AWS has introduced account-regional bucket namespaces in some contexts, so verify the namespace and rules for the bucket type in use.
  - Names are DNS-compatible (for example, lowercase; underscores are not allowed for standard general-purpose buckets), and some prefixes/suffixes are reserved.
- An object is identified by its bucket and key.
  - The key is the full object name, often written as a slash-separated path such as `reports/2026/july.csv`.
  - S3 is object storage, not a hierarchical filesystem: the apparent “folders” are prefixes in keys, although the console can present them as folders.
- An object contains its data (body), system metadata (such as content type and modification information), optional user-defined metadata, optional tags, and, when versioning is enabled, a version ID.
  - The maximum object size and supported upload method depend on current S3 limits; use multipart upload for large files, and verify current limits before designing around exact values.
- Tags are Unicode key-value labels associated with an object and are useful for lifecycle filtering, organization, and some access-control conditions.
  - Metadata and tags are not a general-purpose searchable index; maintain an external index such as DynamoDB when applications need search by object attributes.

## Access control and encryption
- Access may be controlled by identity-based IAM policies and resource-based bucket or access-point policies.
  - ACLs exist for legacy or specialized use, but modern configurations commonly disable ACLs with S3 Object Ownership set to **Bucket owner enforced**.
- An identity's effective access depends on all applicable policy layers, including IAM identity policies, bucket/access-point policies, permissions boundaries, organization controls, and endpoint policies.
  - An explicit deny overrides an allow.
  - Cross-account access generally needs both the resource owner's permission and permission for the caller's identity, along with any organization or encryption-key permissions.
- For an AWS workload such as EC2, use an IAM role rather than embedding long-term access keys.
  - Keep S3 Block Public Access enabled unless public access is a deliberate, reviewed requirement.
  - It can be configured at account and bucket level and blocks common paths by which policies or ACLs could expose data.
- S3 encrypts new objects at rest by default with SSE-S3.
  - Other choices include SSE-KMS/DSSE-KMS, SSE-C, and client-side encryption; see the security lesson for key ownership, request behavior, and trade-offs.
  - TLS protects data in transit.

## Static website hosting
- S3 can serve static website files from a bucket after website hosting is enabled and an index document is configured.
  - Website endpoints use HTTP; use CloudFront in front when HTTPS, a custom domain, caching, or additional edge protections are required.
- Direct public website access requires suitable bucket permissions and Block Public Access settings that do not block the intended public policy.
  - A 403 can indicate missing read permission, a blocked public policy, or a request for an object that does not exist (depending on what the requester is allowed to learn).
- Static website hosting does not run server-side application code. A separate API or compute service is needed for dynamic behavior.

## Versioning and replication
- Versioning is configured per bucket.
  - With versioning enabled, overwriting a key creates a new version instead of destroying the old one; an ordinary delete adds a delete marker that hides the current version.
  - Previous versions can be restored or permanently deleted as permitted.
- Objects that existed before versioning was enabled have a `null` version.
  - Suspending versioning stops creation of new numbered versions but does not erase existing versions.
- Same-Region Replication (SRR) and Cross-Region Replication (CRR) asynchronously copy eligible objects between buckets.
  - Enable versioning on both source and destination, configure an IAM role for S3 replication, and ensure destination policies and encryption keys allow the operation.
  - Buckets may be in separate accounts.
- Replication is useful for regional access, account separation, compliance, log aggregation, or production-to-test copies.
  - By default it applies to new objects after configuration; use S3 Batch Replication for existing objects or failed replication.
- Delete-marker replication is optional.
  - A delete that specifies a version ID is not replicated, helping prevent a permanent source-version deletion from automatically removing the replica.
  - Replication does not automatically chain through multiple replication rules/buckets.

## Storage classes and data resilience
- **S3 Standard** suits frequently accessed data requiring low latency and multi-AZ resilience.
- **S3 Standard-IA** suits infrequently accessed data that still needs rapid retrieval, such as disaster-recovery copies.
  - It has retrieval charges and minimum storage-duration/size billing rules.
- **S3 One Zone-IA** stores data in one AZ and can be appropriate for replaceable or separately replicated data; it is not suitable as the only copy of irreplaceable data.
- **S3 Intelligent-Tiering** automatically moves eligible objects among access tiers as access patterns change, for a monitoring/automation charge.
  - It is useful when access is unpredictable; optional archive tiers have restore behavior, so check which tier is enabled.
- **S3 Glacier Instant Retrieval** is for rarely accessed archives that still require millisecond access.
  - **S3 Glacier Flexible Retrieval** and **S3 Glacier Deep Archive** are archival tiers with restore delays ranging from minutes/hours to longer windows, and retrieval charges.
- Durability and availability are different.
  - Durability concerns the likelihood that data remains intact; availability concerns whether it can be accessed at a given time.
  - S3's commonly quoted eleven-nines durability is a service design figure, not a guarantee that accidental deletion, wrong permissions, or customer-caused loss is impossible.
  - Availability varies by class and service commitments.
- Lifecycle configurations can transition objects between eligible classes, expire them, clean up noncurrent versions, and abort incomplete multipart uploads.

## Caveats and exam gotchas
- Versioning helps recover from accidental changes/deletes, but it is not a backup against every event: a principal with permission to delete versions can permanently remove them.
- Choose a class based on access frequency, required retrieval time, resilience, minimum-duration charges, and restore requirements—not storage price alone.
- Exact service limits, retrieval times, and pricing can vary and change.
