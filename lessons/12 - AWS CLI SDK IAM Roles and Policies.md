# AWS CLI, SDK, IAM Roles & Policies

## EC2 Instance Metadata
* The Instance Metadata Service (IMDS) lets an EC2 instance retrieve information about itself through the link-local address `169.254.169.254`.
* Instance metadata describes the instance; user data is the launch-time configuration script.
* IMDSv2 requires a short-lived session token before metadata requests, adding protection compared with direct IMDSv1 requests. Prefer IMDSv2 where possible.
* Metadata can reveal the attached role name, but it does not expose the role's IAM policy document.

## CLI and SDK Access
* The AWS CLI is suited to interactive administration and scripts; an AWS SDK lets application code call AWS APIs using a supported programming language.
* The CLI and SDK use the configured credential provider chain and sign AWS requests for you.
* To use MFA-protected credentials with CLI calls, obtain temporary credentials through AWS STS `GetSessionToken` and use the returned session credentials.
* Applications running on AWS should use an attached IAM role instead of embedded long-term access keys.

## API Limits, Retries, and Signing
* AWS APIs have request-rate limits and services also enforce resource quotas. Intermittent throttling is usually handled with exponential backoff and retry; consistently exceeding a quota may require reducing demand or requesting an increase.
* AWS SDKs include retry handling for many API calls. When calling APIs directly, implement the appropriate retry behavior.
* Direct AWS HTTP API requests are authenticated with Signature Version 4 (SigV4), using credentials to sign the request. CLI and SDK calls sign requests automatically.
* Pre-signed URLs carry a time-limited signature in the URL and can grant temporary access to a specific S3 operation.
