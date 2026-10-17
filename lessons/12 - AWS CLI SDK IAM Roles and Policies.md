# AWS CLI, SDK, IAM Roles & Policies

## At a glance
- IMDS supplies instance information and temporary instance-role credentials; prefer IMDSv2.
- CLI and SDK clients use credential providers, sign requests, and commonly retry transient errors.
- Prefer temporary workload credentials over long-lived access keys.
- SigV4 authenticates most AWS API requests; presigned URLs provide time-limited access to a specific operation.

## EC2 instance metadata
- The **EC2 Instance Metadata Service (IMDS)** is a link-local endpoint through which an instance can retrieve information about itself, such as its instance identity, networking details, and attached instance-profile role credentials.
  - The familiar endpoint is `http://169.254.169.254/`; metadata paths are under `/latest/meta-data`.
- Metadata is information about the running instance; **user data** is the launch-time configuration or script made available to the instance.
  - Neither is the same as an IAM policy document.
- An instance can discover the name of its attached IAM role through metadata, but IMDS does not reveal that role's IAM policy.
  - Applications normally obtain temporary role credentials through the credentials endpoint and use them through the SDK/CLI credential chain.
  - Credentials are refreshed automatically by supported providers.
- IMDSv1 permits direct metadata requests.
  - IMDSv2 requires a session token: the client first makes a `PUT` request to obtain a short-lived token and then includes that token in subsequent metadata requests.
  - Enforcing IMDSv2 reduces exposure to some server-side request forgery (SSRF) paths. Prefer IMDSv2 and restrict unnecessary access to IMDS from applications and containers.
- Metadata credentials are powerful because they represent the instance role. Grant the role only the permissions the workload needs and avoid exposing metadata responses to untrusted callers.

## CLI, SDKs, and credentials
- The **AWS Command Line Interface (CLI)** is useful for interactive administration, diagnostics, and shell automation.
- An **AWS Software Development Kit (SDK)** is used by application code to call AWS APIs using a language-specific library (for example, Python's `boto3`/`botocore`, Java, JavaScript, .NET, Go, or Ruby).
- CLI and SDK clients build API requests, obtain credentials through a credential provider chain, and normally handle request signing and common retries.
  - The chain can consider explicit configuration, environment variables, shared profile files, and workload credentials such as EC2 instance roles; exact precedence depends on the tool and version.
- On AWS, prefer temporary credentials from an attached IAM role (instance profile for EC2, execution/task roles for other compute) over long-lived access keys stored in source code or on disk.
  - Outside AWS, use a supported federation or SSO flow when available.
  - Never commit access keys or log secret credentials.
- Use an SDK when an application needs to call AWS as part of its runtime logic—for example, a Lambda function writing to DynamoDB.
  - Use the CLI for operator-driven commands and scripts rather than shelling out from application code where an SDK is available.
- MFA-protected CLI access uses temporary credentials.
  - One flow is STS `GetSessionToken`, supplying the MFA device serial number and one-time code, then using the returned access key, secret key, and session token.
  - These are temporary session credentials and must be provided together.
  - For role-based access, MFA is commonly enforced during role assumption with STS `AssumeRole` according to the trust policy and organization setup.

## API limits, quotas, and retries
- AWS services enforce request-rate limits (throttling) and resource quotas (for example, the number of resources or available compute capacity).
  - These are distinct: an API may throttle bursts even when the account is below a resource quota.
- When throttling or transient network/service errors occur intermittently, use retries with exponential backoff, often including jitter to prevent clients from retrying in synchrony.
  - AWS SDKs provide retry behavior for many errors; review each SDK's retry mode and configure it appropriately.
  - Direct API clients must implement retry logic themselves, while avoiding retries for non-retryable validation or authorization errors.
- Persistent throttling suggests the workload needs smoother request rates, partitioning, batching, caching, or a quota increase.
  - Service Quotas provides quota visibility and, for supported quotas, programmatic increase requests; other limits may require the process documented by the service.

## SigV4 request signing
- Most AWS API requests must be authenticated.
  - **Signature Version 4 (SigV4)** uses the access key identifier and secret key to create a cryptographic signature over request details, date, service, and Region.
  - AWS verifies the signature and uses the caller's permissions to authorize the request.
- SDK and CLI clients sign requests automatically when using credentials.
  - If making raw HTTP API calls, implement the currently required signing method correctly, including canonical request construction, date/Region/service scope, and signed headers.
- Signatures can be carried in an HTTP `Authorization` header or represented in query parameters for a presigned request.
  - A presigned S3 URL is a time-limited signature for a specific operation and object; the recipient can use it without receiving the signer's raw credentials.
- Some S3 requests to public resources may be anonymous, but private access still requires authorization.

## Caveats and exam gotchas
- Do not rely on specific request rates or account defaults from older training slides—limits are service-, Region-, and account-dependent and may change.
- Do not assume an unsigned request is acceptable for a protected resource, and do not hand-build signatures when an AWS SDK can safely do it.
