# AWS Security and Encryption

## Encryption choices
* **In transit**: TLS/HTTPS protects data between client and service. **At rest**: the storage service encrypts data and uses the key. **Client-side**: the client encrypts before upload and controls decryption.
* KMS manages keys, uses IAM for authorization, and logs usage in CloudTrail. Symmetric keys are common in AWS service integrations; asymmetric keys support public/private-key encryption or signing.
* KMS key policies are essential: they define key use and administration, including cross-account access. AWS-managed keys are maintained by AWS; customer-managed keys allow customer-defined policies and rotation.
* KMS `Encrypt` and `Decrypt` handle up to 4 KB. For larger data, envelope encryption uses `GenerateDataKey` to return a plaintext data key for local encryption and an encrypted copy to store with the ciphertext. The AWS Encryption SDK implements this pattern and can cache data keys to reduce KMS calls.
* S3 Bucket Keys reduce KMS requests for SSE-KMS. Handle KMS throttling with exponential backoff.

## Secrets and configuration
* Parameter Store holds hierarchical, versioned configuration and secrets with IAM access control. KMS encryption is optional for `SecureString`; advanced parameters can use expiration and notification policies.
* Secrets Manager encrypts secrets with KMS and supports rotation, often with Lambda and database integrations. Prefer it when automated rotation is needed; Parameter Store is a simpler, lower-cost option.
* CloudFormation dynamic references retrieve values from Parameter Store or Secrets Manager without embedding plaintext in templates. CodeBuild environment variables can reference those stores rather than contain plaintext secrets.

## Higher-assurance options
* CloudHSM provides dedicated hardware where customers manage keys and HSM users; KMS is a managed, multi-tenant service. Choose CloudHSM when dedicated hardware or direct key control is required.
* Nitro Enclaves isolate sensitive processing from the parent EC2 instance, without external networking or interactive access. Attestation can restrict access to authorized enclave code.
* ACM provisions and renews public TLS certificates for supported integrations such as load balancers, CloudFront, and API Gateway. Macie discovers sensitive data such as PII in S3; CloudWatch Logs groups can be associated with KMS keys for encryption.
