# Amazon Route 53

## DNS and Route 53
* DNS translates human-readable domain names into IP addresses using a hierarchy of root, top-level-domain (TLD), and authoritative name servers.
* Route 53 is AWS's managed authoritative DNS service. It can also register domains and check the health of public endpoints.
* A hosted zone contains the DNS records for a domain:
  * Public hosted zones answer queries from the internet.
  * Private hosted zones resolve names within associated VPCs.
* A DNS record defines a name, type, value, routing policy, and (usually) a TTL.

## DNS Record Types
* `A` maps a name to an IPv4 address; `AAAA` maps it to IPv6.
* `CNAME` maps a subdomain to another hostname. It cannot be used at the zone apex (the root of the domain).
* `NS` records identify the name servers for a hosted zone.
* A TTL controls how long resolvers cache a response. Short TTLs allow changes to propagate sooner but increase DNS query volume; long TTLs reduce queries but retain old answers longer.

## Alias Records
* Route 53 Alias records point a DNS name to supported AWS resources, such as load balancers, CloudFront distributions, API Gateway, S3 website endpoints, and Global Accelerator.
* Alias records can be used at the zone apex, unlike CNAME records; they are free and Route 53 follows changes to the target resource's addresses.
* Alias records use `A` or `AAAA` and do not have a configurable TTL. EC2 instance DNS names are not supported Alias targets.

## Routing Policies
* **Simple:** return one value, or return multiple values from which the client chooses. It does not support health checks.
* **Weighted:** distribute DNS responses according to relative weights; useful for gradual releases or distributing traffic across regions.
* **Latency-based:** answer with the endpoint expected to provide the lowest network latency.
* **Failover:** use a health-checked primary and return the secondary endpoint when the primary is unhealthy.
* **Geolocation:** select an answer based on the user's geographic location; configure a default answer for unmatched locations.
* **Geoproximity:** direct users toward resources based on geographic distance; bias values expand or shrink a resource's influence. It requires Traffic Flow.
* **IP-based:** map client CIDR ranges to designated endpoints.
* **Multi-value:** return up to eight healthy records. This provides basic DNS-level selection, not load-balancer functionality.
* These policies control DNS answers, not the routing of application traffic itself.

## Health Checks and Domain Setup
* Route 53 health checks can monitor public endpoints, combine child checks, or evaluate CloudWatch alarms. CloudWatch alarms can represent private-resource health that external checkers cannot reach.
* Endpoint checks support HTTP, HTTPS, and TCP. Permit health-checker traffic through endpoint firewalls and network controls.
* A domain registrar and DNS provider can be different services. To use Route 53 for a domain registered elsewhere, create a hosted zone and update the registrar's name-server records to Route 53's name servers.
