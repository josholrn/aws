# Amazon API Gateway

## API setup and integrations
* API Gateway exposes managed APIs, commonly as a front end for Lambda. It can also connect to HTTP backends or AWS services, adding controls such as authentication, throttling, caching, and request validation.
* Endpoint choices: edge-optimized routes global clients through CloudFront edge locations while the API remains in one Region; regional is suited to clients in that Region; private is reachable through an interface VPC endpoint and can be restricted with a resource policy.
* Integration modes determine request handling. Mock returns a response without a backend. Lambda proxy forwards the request to Lambda and expects the function to construct the response. HTTP proxy forwards requests and backend responses. Non-proxy HTTP/AWS integrations use mapping templates (VTL) to transform data.
* API changes are not live until deployed to a stage. Stages such as dev and prod have independent configuration and deployment history. Stage variables can select backend URLs or Lambda aliases; stage canaries route a chosen share of traffic to a candidate release.

## Access, limits, and operations
* IAM/SigV4 suits AWS identities; Cognito User Pools validate user tokens; Lambda authorizers handle custom tokens or request parameters and return a cached IAM policy. API keys identify and meter callers when attached to usage plans; they are not a substitute for authentication.
* The slides list a default account-wide throttle of 10,000 requests/second, a soft quota that can be raised. Stage, method, and usage-plan limits can isolate workloads. Throttling returns HTTP 429. API Gateway’s integration timeout is capped at 29 seconds.
* Enable CORS for browser requests from other origins; the preflight response must allow the relevant origin, methods, and headers. CloudWatch provides stage-level logs and latency/error metrics; X-Ray tracing can be enabled.
* REST APIs provide the broad feature set, including usage plans and API keys. HTTP APIs are a lower-cost, lower-latency option with built-in CORS and OAuth/OIDC support, but omit some REST API features.
