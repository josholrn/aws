# Amazon API Gateway

## At a glance
* API Gateway provides managed REST, HTTP, and WebSocket APIs with multiple backend integration styles.
* Endpoint type, stages, stage variables, and canary deployments control exposure and release.
* IAM, Cognito, and Lambda authorizers serve different authorization needs; API keys meter clients but do not authenticate them.
* CORS, caching, validation, logs, metrics, and tracing support API operations.
* Check API type capabilities, integration timeout, and proxy response formatting when troubleshooting.

## API capabilities and integrations
* API Gateway is a managed front door for REST, HTTP, and WebSocket APIs. It can handle versioning and environments, expose an API backed by Lambda, proxy to an HTTP service, or invoke AWS services directly (for example, starting a Step Functions workflow or sending a message to SQS).
* It adds API-level authentication, throttling, caching, validation, monitoring, and custom-domain support.
* **Lambda proxy (`AWS_PROXY`)** passes the incoming request context—path, method, headers, query parameters, and body—to Lambda without a mapping template. The function owns application logic and must return a correctly shaped response (status code, headers, and body); malformed output commonly results in a 502.
* **HTTP proxy (`HTTP_PROXY`)** forwards the HTTP request to the backend and returns the backend response. It needs no VTL request/response mapping, though headers can be added or transformed where configured.
* **Non-proxy HTTP/AWS integration** requires configuring both integration request and response.
  * Velocity Template Language (VTL) mapping templates can rename or reshape parameters, transform JSON to another format (such as SOAP/XML), add headers, and filter output.
  * This is useful when adapting an existing service without changing its implementation.
* **Mock integration** returns a configured response without invoking a backend, useful for prototypes, testing, or preflight responses.
* OpenAPI can define/import/export API methods, request and integration settings, and generate client SDKs.
* REST API request validation can reject missing/blank required URI/query/header parameters or invalid JSON-schema payloads before the integration is called, returning a 400 and avoiding unnecessary backend work.

## Endpoints, stages, and releases
### Endpoint types
* **Edge-optimized** (default in the course material) routes global clients through CloudFront edge locations to improve latency; the API itself remains in one Region.
  * For a custom domain with this endpoint, the ACM certificate must be in `us-east-1`.
* **Regional** is intended for clients in the API’s Region; you can place your own CloudFront distribution in front for more control.
  * Its custom-domain certificate belongs in the API Region.
* **Private** APIs are reachable through an interface VPC endpoint and can be restricted with an API resource policy, such as to a VPC endpoint or source network.

### Deployment and traffic shifting
* Creating or editing an API does not publish the changes. Deploy to a **stage** (such as `dev`, `test`, or `prod`) to make a deployment callable.
  * Stages have independent settings and deployment history, so a previous deployment can be restored.
* Stage variables act like per-stage configuration and can select an HTTP backend URL, Lambda ARN/alias, or mapping-template input. They are useful to promote one API configuration across environments without duplicating the API.
* Stage canary deployments send a selected percentage to a candidate deployment; canary metrics/logs are separated and stage variables can be overridden.
* A stage variable can also select a Lambda alias whose own CodeDeploy traffic shift is managed independently. This enables testing a backend release without changing the public API URL.

## Authorization, client plans, and CORS
* Use **IAM authorization with SigV4** for AWS callers such as applications running under IAM roles.
  * The principal’s IAM policy must authorize the API method.
  * Resource policies can additionally restrict source IPs, VPC endpoints, or cross-account access.
* A **Cognito User Pool authorizer** validates user-pool tokens for application users.
* A **Lambda authorizer** runs custom authorization logic and returns an IAM policy; its result may be cached, so account for cache lifetime and identity sources when changing permissions.
* **API keys and usage plans** identify/meter API clients and apply per-client throttles and quotas. They are not authentication: do not treat possession of a key as proof of user identity or a security boundary.
  * Configure methods to require a key, deploy the API, create/import keys and a usage plan, then associate keys and stages with the plan.
  * Clients send the key in `x-api-key`.
* The course lists an account-wide default throttle of 10,000 requests/second as a soft, increaseable quota. Configure stage, method, or usage-plan throttles to protect workloads from noisy neighbors. Throttling yields HTTP 429; clients should back off.
* Browser clients on another origin require CORS.
  * The `OPTIONS` preflight and actual response need appropriate `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`.
  * Allowing CORS in the console does not remove the need for the backend/proxy response to return required headers.
* REST APIs expose a broader feature set, including usage plans/API keys and advanced request mapping. HTTP APIs typically offer lower cost/latency with built-in CORS and JWT/OAuth/OIDC authorization, but do not include every REST API capability. Choose based on needed features, not just the API name.

## Caching, monitoring, and limits
* REST API response caching is configured per stage and can be overridden per method.
  * The course gives a default TTL of 300 seconds and configurable range 0–3,600 seconds.
  * Caching reduces backend calls but can serve stale data and incurs cost; choose cache keys carefully and encrypt the cache when needed.
* The full cache can be flushed. A client may request invalidation with `Cache-Control: max-age=0`; require authorization for cache invalidation, otherwise callers may repeatedly evict the cache.
* Stage-level CloudWatch logs can capture request/response details at `ERROR`, `INFO`, or `DEBUG`; avoid logging sensitive payloads unnecessarily.
* Metrics include request count, 4XX/5XX errors, cache hit/miss, latency, and integration latency. Integration latency measures backend time; overall latency includes API Gateway overhead. Enable X-Ray tracing to correlate Gateway and Lambda behavior.

## Exam gotchas and troubleshooting
* The maximum integration timeout in the slides is 29 seconds.
* API Gateway reports 4XX for client issues (e.g., 400 bad request, 403 denied, 429 throttled) and 5XX for server/integration issues.
  * 502 often points to an incompatible Lambda proxy response.
  * 504 indicates integration timeout.
* Check deployment stage, authorizer/resource policy, integration response format, and backend latency when troubleshooting.
