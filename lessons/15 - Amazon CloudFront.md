# Amazon CloudFront

## At a glance
- CloudFront caches content at edge locations to reduce viewer latency and repeated origin requests.
- Origins include S3, public HTTP services, and supported private VPC resources.
- Cache keys, cache behaviors, and policies determine what is cached and how requests reach origins.
- Access controls include geo restrictions, signed URLs/cookies, and field-level encryption.
- Delivery options include real-time logs and price classes with differing geographic coverage.

## CDN and origins
- CloudFront is a content delivery network (CDN). It caches objects at edge locations near viewers, improving read latency and reducing repeated requests to the origin.
- Its globally distributed edge network also helps absorb traffic spikes and integrates with AWS Shield and AWS WAF for protection.
- An origin is the source CloudFront fetches from on a cache miss. Supported choices include:
  - An S3 bucket.
  - A public HTTP service, including an S3 static website endpoint or public ALB.
  - A VPC origin targeting supported private ALBs, NLBs, or EC2 instances in private subnets. VPC origins avoid exposing an application directly to the public internet.
- For a private S3 origin, use Origin Access Control (OAC) and a bucket policy that grants the distribution access; do not make the bucket public.
- CloudFront can also be used for uploads to S3 when the origin and request method are configured for it.

### CloudFront caching versus S3 replication
| CloudFront | S3 Cross-Region Replication |
| --- | --- |
| Caches objects across edge locations for global reads. | Copies objects to selected AWS Regions and updates near real time. |
| Cached content refreshes according to TTL or an invalidation. | Useful when readers need regional copies. |
| Provides a global edge cache. | Is not a global edge cache. |

## Caching and behaviors
- Each edge location maintains its own cache.
- The cache key identifies an object; by default it includes the distribution hostname and URL path.
- Add headers, cookies, or query strings to the key only when they genuinely change the response. More key variations create more cache entries and lower the hit ratio.
- A cache policy controls the cache key and TTL:
  - TTL can be set from zero to one year.
  - Origin `Cache-Control` or `Expires` headers can influence caching within the policy's limits.
  - Values added to the cache key are also forwarded to the origin.
- An origin request policy forwards selected viewer headers, cookies, and query strings to the origin without varying the cache key. Use this when the origin needs data for processing but that data should not create separate cached copies.
- **Caution:** Forwarding per-user values while sharing a cache key can expose personalized content. Disable caching or include all response-varying values in the key for user-specific responses.
- When an origin changes, CloudFront does not automatically know that an object is stale. Wait for its TTL or create an invalidation for selected paths. Invalidation is useful for urgent updates, but it is not a substitute for an appropriate TTL strategy.
- Cache behaviors route path patterns to origins and settings—for example, `/images/*` to S3 and `/api/*` to an application origin.
  - Behaviors are evaluated in configured order.
  - The default `*` behavior is the fallback and effectively comes last.
- Separate static and dynamic traffic, or even distributions, when that improves the cache key and hit ratio.
- An origin group has a primary and secondary origin. CloudFront retries the same request against the secondary when the primary returns a configured failure status, improving availability.
- Pair regional S3 origins with replication when regional failover must serve equivalent data.

## Access, security, and delivery options
### Geographic restrictions
- Geo restriction can allow only specified countries or block specified countries.
- The viewer's country is inferred from a third-party Geo-IP database; this is a content-access control, not a precise identity check.

### Signed access
- Signed URLs grant access to an individual file; signed cookies authorize multiple files, useful for a set of protected assets.
- A policy can constrain expiry and optionally viewer IP range.
- Use short expiry for shared paid media and longer expiry only where the content and risk justify it.
- CloudFront signed URLs are validated at the edge and can be used with different origin types while retaining CDN caching.
- A trusted key group is the recommended signer arrangement: the application signs with its private key and CloudFront verifies using the associated public key.
- This differs from an S3 presigned URL, which is an S3 request authorized with the signing IAM principal's credentials and has a limited lifetime.

### Field-level encryption
- Field-level encryption adds protection for selected form fields in POST requests.
- CloudFront encrypts up to 10 configured fields at the edge with a public key; only an application component holding the private key can decrypt them.
- It supplements HTTPS rather than replacing transport encryption.

### Logs and price classes
- Real-time logs stream selected viewer-request fields to Kinesis Data Streams.
- Configure sampling percentage, fields, and cache behaviors to monitor delivery performance without necessarily recording every request.
- Kinesis Data Firehose can then deliver records onward for near-real-time processing or storage.
- Price Classes trade geographic coverage for cost:
  - **All** offers all edge locations.
  - **200** excludes the most expensive locations.
  - **100** uses the least expensive locations.
- A narrower class may increase latency for viewers outside the included locations.
