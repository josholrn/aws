# Amazon S3 – Security

## Encryption
* Server-side encryption protects objects at rest. SSE-S3 uses AWS-managed AES-256 keys and is the default for new objects. SSE-KMS uses KMS keys, giving key-management controls and CloudTrail auditability, but KMS request quotas can constrain very high object traffic.
* DSSE-KMS applies two server-side encryption layers for stricter compliance needs, with additional cost and latency.
* SSE-C keeps key ownership with the customer: S3 does not retain the supplied key, so send it with each request over HTTPS.
* With client-side encryption, the application encrypts before upload and decrypts after download; the customer controls the keys and process.
* HTTPS/TLS protects data in transit. A bucket policy using `aws:SecureTransport` can reject insecure HTTP requests. A policy can also require encryption headers on uploads; policy evaluation occurs before default encryption is applied.

## Access and Browser Controls
* CORS governs browser requests crossing origins (protocol, host, or port differs). Configure the target bucket to allow the requesting origin and required methods/headers; CORS is not a replacement for bucket authorization.
* MFA Delete adds a second factor for permanently deleting object versions or suspending versioning. Versioning must be enabled, and only the root bucket owner can configure MFA Delete.
* S3 access logging records requests, including denied requests, in a separate bucket for auditing. Avoid logging a bucket to itself.
* Presigned URLs grant time-limited access using the permissions of the signer. They are useful for private downloads or uploads without making a bucket public; anyone holding the URL can use it until it expires.
* S3 Access Points provide separate DNS names and policies for different teams or applications accessing the same bucket. A VPC-only access point requires an S3 VPC endpoint whose policy also permits the access.
* S3 Object Lambda can transform an object at retrieval time through Lambda—for example, redact fields or convert a format—without storing a separate transformed copy.
