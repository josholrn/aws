# Amazon Route 53

## At a glance
- Route 53 is AWS's authoritative DNS service, with hosted zones, health checks, and routing policies.
- Hosted zones can answer public queries or resolve names privately from associated VPCs.
- TTLs and routing policies affect DNS answers and caching; they do not route application packets.
- Health checks can influence supported routing records, but checks and DNS caching add failover delay.

## DNS and hosted zones
- The **Domain Name System (DNS)** maps human-readable names to values clients can use to connect, commonly IPv4 or IPv6 addresses.
  - It is hierarchical: root servers delegate to top-level domains (such as `.com`), which lead to authoritative servers for a registered domain and its subdomains.
- A **domain registrar** handles domain registration; a **DNS provider** hosts the records that answer queries. These can be different providers.
  - To move DNS hosting to Route 53 while keeping a third-party registrar, create a public hosted zone and update the registrar's name-server (`NS`) delegation to the Route 53 name servers.
- Route 53 is AWS's managed authoritative DNS service. It also offers domain registration, health checks, and DNS routing policies.
  - Authoritative means the domain owner controls the records; it does not mean Route 53 forwards application traffic.
- A **hosted zone** is a container for records for a domain and its subdomains:
  - **Public hosted zones** answer public DNS queries for internet-facing names.
  - **Private hosted zones** resolve names only from associated VPCs. They are useful for internal service names and can be associated with multiple VPCs, subject to account/Region association setup.
- A DNS record includes a 
  * `Domain/subdomain Name` – e.g., example.com
  * `Record Type` – e.g., A or AAAA
  * `Value or Target` – e.g., 12.34.56.78
  * `Routing Policy` – how Route 53 responds to queries
  * `TTL` – amount of time the record cached at DNS Resolvers
- Common types: `A` (IPv4), `AAAA` (IPv6), `CNAME` (another DNS name), and `NS` (name servers).
- Other supported types include `MX`, `TXT`, `CAA`, `PTR`, and `SRV`.

## TTLs and alias records
- A **time-to-live (TTL)** tells recursive resolvers how long they may cache a DNS answer.
  - A long TTL reduces repeat queries but keeps old answers cached longer after a change.
  - A short TTL makes changes take effect sooner at the cost of more queries.
  - Cached answers can persist until their TTL expires, so DNS changes are not necessarily instantaneous.
- A `CNAME` maps a non-apex name, such as `www.example.com`, to another hostname.
  - DNS standards do not allow a CNAME at a zone apex (`example.com`) because the apex must also contain records such as `NS` and SOA.
- A Route 53 **Alias** record is AWS's extension for pointing a name at supported AWS resources or another record in the same hosted zone.
  - It uses `A` and/or `AAAA`, works at the zone apex, and tracks changes to the target resource's addresses.
  - Supported targets include Elastic Load Balancing, CloudFront, API Gateway, Elastic Beanstalk, S3 static website endpoints, Global Accelerator, and certain VPC endpoints.
  - A regular EC2 instance DNS name is not an Alias target.
- Alias records do not have a user-configurable TTL; the target's TTL behavior is used.
  - For supported AWS-resource aliases, Route 53 does not charge for the alias query itself; this does not make all Route 53 features or queries free.

## Routing policies
- Routing policies determine which DNS answer Route 53 returns. They influence where a client connects after DNS resolution; they are not application load balancers and do not route packets.
- **Simple:** Returns one record or multiple values in the answer.
  - With multiple values the client chooses; this is not health-aware load balancing.
  - Simple routing does not use health checks, and an Alias configuration targets a single AWS resource.
- **Weighted:** Assigns relative weights to same-name, same-type records and distributes DNS responses proportionally.
  - Weights need not sum to 100.
  - A zero-weight record receives no traffic when other records have nonzero weights; if all weights are zero, they are treated equally.
  - Useful for canaries, gradual migration, or splitting traffic among Regions.
  - DNS caching means observed client traffic may not exactly match the configured percentages. Health checks can be associated with records.
- **Latency-based:** Returns the endpoint associated with the AWS Region expected to have the lowest latency from the querying user.
  - Useful for multi-Region applications where responsiveness matters; geography does not dictate the answer.
  - Health checks can exclude unhealthy endpoints.
- **Failover:** Active-passive routing returns the primary while its health check is healthy and the secondary when it is unhealthy. Use it for disaster recovery, with a secondary endpoint ready to serve traffic.
- **Geolocation:** Selects a record using the location of the DNS query (continent, country, or US state).
  - Useful for localization, compliance-based content delivery, or regional experiences.
  - Add a default record for queries that do not match a configured location.
- **Geoproximity:** Routes based on the relative locations of users and resources.
  - A bias from `-99` to `+99` shrinks or expands a resource's effective geographic area.
  - AWS resources use their Region; non-AWS resources use latitude and longitude.
  - This policy requires Route 53 Traffic Flow.
- **IP-based:** Maps caller CIDR ranges (organized into CIDR collections) to designated endpoints. It can optimize for known network populations, such as sending a particular ISP's users to a nearer service.
- **Multi-value answer:** Can return up to eight healthy records for a query.
  - It offers health-aware DNS answers, but it is not a substitute for a load balancer and does not guarantee that clients select a particular result.

## Health checks
- Health checks can monitor public endpoints, combine other checks, or evaluate CloudWatch alarms.
  - Endpoint checks support HTTP, HTTPS, and TCP.
  - HTTP checks can validate a response code and optionally search response text.
  - Configure firewalls to allow the Route 53 health-checker addresses.
- Public health-checking systems cannot directly reach endpoints inside a private VPC.
  - For private-resource health, publish an appropriate metric and alarm in CloudWatch, then have a Route 53 health check evaluate that alarm.
  - Calculated health checks combine up to 255 child checks with AND, OR, or NOT logic and can define how many children must pass.
- Health status can be associated with supported routing records to stop returning unhealthy destinations.

## Caveats and exam gotchas
- Health checks and DNS caching introduce detection and failover delay; they are not instantaneous application-level failover.
- Historic slide examples include specific hosted-zone prices, health-check intervals, and DNS availability claims. Pricing and service terms change, so consult current Route 53 documentation for exact values rather than relying on fixed course-era numbers.
