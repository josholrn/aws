# Amazon CloudFront

## CDN and Origins
* CloudFront caches content at globally distributed edge locations to reduce latency and origin load; it also integrates with AWS Shield and WAF for protection.
* Origins can be S3, public HTTP services, or private VPC resources such as an ALB, NLB, or EC2 instance. Use Origin Access Control (OAC) and an S3 bucket policy to keep an S3 origin private. VPC origins let CloudFront reach supported resources in private subnets.
* CloudFront caches according to TTL. Invalidation removes selected cached paths before expiry. Separate cache behaviors route path patterns (for example `/api/*`) to different origins/settings; the default behavior is processed last.
* Compared with S3 Cross-Region Replication, CloudFront caches globally and is suited to widely distributed static content. Replication copies objects to selected regions, updates near real-time, and suits low-latency reads in those regions.

## Cache and Request Policies
* A cache key determines whether requests share a cached object. By default it uses the hostname and URL path; add only necessary headers, cookies, or query strings. Including more values creates more cache variants and can reduce hit ratio.
* Cache policies control both the cache key and TTL. Origin request policies forward selected viewer data to the origin without adding it to the key.
* For dynamic or user-specific responses, forward the needed values and ensure caching cannot expose one user's content to another. Separating static and dynamic distributions can improve static cache hits.

## Access Restrictions
* Geo restrictions allow or block viewers by country.
* Signed URLs authorize individual files; signed cookies can authorize multiple files. CloudFront signing can restrict path, expiry, or source IP and still use edge caching. Unlike an S3 presigned URL, it is not an S3 request performed with the signer's IAM credentials.
