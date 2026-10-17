# AWS Security and Encryption

## At a glance

* Encryption can protect data in transit, at rest, or before it reaches a storage service.
* **KMS** manages keys and supports envelope encryption for data larger than its 4 KB direct-encryption limit.
* **Parameter Store** and **Secrets Manager** address configuration and secrets with different rotation and management capabilities.
* **CloudHSM** provides dedicated HSM hardware; **Nitro Enclaves** isolate sensitive processing on compatible EC2 instances.
* ACM, CloudWatch Logs, and CodeBuild have certificate, encryption, and secret-handling integrations.

## Encryption models

Choose the encryption boundary to match the threat model:

* **In transit:** TLS encrypts data between communicating endpoints, as with HTTPS. Certificates help authenticate the endpoint and establish the protected connection; encryption in transit is not the same as access control.
* **Server-side encryption at rest:** a service encrypts data after receiving it and decrypts it when returning it to an authorized caller. The service manages or uses the encryption key.
* **Client-side encryption:** the client encrypts before upload and decrypts after download. The storage service holds ciphertext and should not be able to recover plaintext on its own. Client-side encryption can use envelope encryption.

## AWS Key Management Service

AWS KMS provides managed cryptographic keys and integrates with IAM and services such as EBS, S3, RDS, and Systems Manager. IAM and key policies control access; CloudTrail records key use. Do not embed plaintext secrets in source code or templates.

### Key types and management

KMS keys are either:

* **Symmetric** (AES-256): one key is used for encryption and decryption. This is the common type for AWS service integrations. The plaintext key material is not exposed to callers; use KMS APIs to perform cryptographic operations.
* **Asymmetric** (RSA or ECC): a public/private pair supports encryption/decryption or signing/verification. The public key can be downloaded; the private key remains protected by KMS. This suits parties that need a public key but cannot call KMS.

AWS-owned and AWS-managed keys (named like `aws/service-name`) are maintained by AWS; customer-managed keys give customers more control over policy and lifecycle. The slides list AWS-owned/managed keys as free, customer-managed keys at $1 per key per month, and API calls at $0.03 per 10,000; verify current pricing before designing to cost. They describe annual rotation for AWS-managed keys, optional rotation for customer-managed keys, and manual rotation for imported key material.

### Key policies, sharing, and quotas

A KMS key policy is essential: it establishes who may administer or use the key. The default policy grants the account root principal access, allowing delegation through IAM. A custom policy can separate administration from cryptographic use and enable cross-account access; the owning account's key policy and the consuming account's IAM permissions must both allow the use.

KMS keys are regional. Copy an encrypted EBS snapshot to another Region by re-encrypting it under a key there. For cross-account sharing:

1. Authorize the recipient in the customer-managed key policy.
2. Share the snapshot.
3. Have the recipient copy and re-encrypt it under its own key before creating a volume.

Cryptographic operations (`Encrypt`, `Decrypt`, `GenerateDataKey`, `ReEncrypt`, etc.) share request quotas, including calls AWS makes for SSE-KMS. On `ThrottlingException`, use exponential backoff or request an increase. Slide examples vary by Region: symmetric 5,500, 10,000, or 30,000 requests/second; asymmetric 500 for RSA and 300 for ECC. These are region/key-type-dependent examples, not universal constants.

### Encrypting data larger than 4 KB

KMS `Encrypt` and `Decrypt` directly process at most 4 KB of data. For larger payloads, use **envelope encryption** rather than sending the complete object through KMS:

1. Call `GenerateDataKey` using a KMS key. KMS returns a plaintext data-encryption key (DEK) for immediate local use and an encrypted copy of that DEK.
2. Encrypt the large payload locally with the plaintext DEK.
3. Store the ciphertext together with the encrypted DEK; discard the plaintext DEK from memory when no longer needed.
4. To decrypt, send the encrypted DEK to KMS `Decrypt`, then use the returned plaintext DEK locally to decrypt the payload.

`GenerateDataKeyWithoutPlaintext` returns only an encrypted data key for later use; `GenerateRandom` returns random bytes. The AWS Encryption SDK implements envelope encryption and stores the encrypted key with the ciphertext. Its data-key caching reduces KMS calls but trades off security; bound reuse by age, bytes, and message count.

### S3 Bucket Keys

For S3 SSE-KMS, a Bucket Key can reduce KMS requests and costs by up to 99% in the slide comparison. It reduces KMS interactions (and related CloudTrail events) but objects remain SSE-KMS encrypted.

## Secrets and configuration

### Systems Manager Parameter Store

**Systems Manager Parameter Store** holds hierarchical, versioned configuration and secrets, protected by IAM. KMS encryption is optional and available for `SecureString`. Parameters can be retrieved by path, notify through EventBridge, and integrate with CloudFormation. Advanced-tier policies include expiration and expiration/no-change notifications.

The slide's tier comparison is:

| Parameter tier | Count per account/Region | Value size | Policies | Storage cost in the slide |
| --- | ---: | ---: | --- | --- |
| Standard | 10,000 | 4 KB | No | No additional charge |
| Advanced | 100,000 | 8 KB | Yes | $0.05 per advanced parameter per month |

Treat those as course-era figures and verify current quotas and pricing. Advanced parameters may also incur applicable API charges.

### AWS Secrets Manager

**Secrets Manager** stores secrets encrypted with KMS and supports rotation through Lambda, including integrations for RDS/Aurora, Redshift, and DocumentDB. Choose it when managed rotation is needed. Secrets can replicate across Regions and a replica can be promoted to standalone.

Parameter Store is simpler and often lower-cost, but has no built-in rotation in the slide comparison; a Lambda/EventBridge workflow can implement it. Parameter Store can also retrieve a Secrets Manager secret.

CloudFormation **dynamic references** (`ssm`, `ssm-secure`, or `secretsmanager`) resolve external values during stack operations, avoiding plaintext in templates. For RDS/Aurora, either let the database service manage the master secret and rotation or reference and link an existing secret. CodeBuild environment variables should likewise reference a secret store.

## Dedicated hardware and isolated processing

### AWS CloudHSM

**AWS CloudHSM** provides dedicated HSM hardware in a VPC: AWS provisions it, while the customer manages HSM users and keys through client software. The slides identify FIPS 140-2 Level 3, multi-AZ availability, and no free tier.

Choose CloudHSM for dedicated hardware or direct key administration. KMS is managed, multi-tenant, IAM-integrated, and can use CloudHSM as a custom key store for supported integrations.

### AWS Nitro Enclaves

**AWS Nitro Enclaves** isolate sensitive processing in a constrained virtual machine on a compatible Nitro EC2 instance. An enclave has its own kernel and allocated CPU/memory, but no external networking, persistent storage, or interactive access; it communicates with its host through a local channel.

Build an Enclave Image File and launch it with Nitro CLI. Attestation can restrict KMS access to authorized enclave code, reducing host access to plaintext for workloads such as payment processing or private-key operations.

## Certificates, logs, and build secrets

AWS Certificate Manager (ACM) provisions and renews free public TLS certificates for supported integrations such as load balancers, CloudFront, and API Gateway. AWS Private CA issues X.509 certificates trusted within an organization's PKI, not by the public Internet, for internal endpoints, devices, or signing.

Associate a KMS key with a CloudWatch Logs group using the API: `create-log-group` for a new group or `associate-kms-key` for an existing one (not through the console, per the slides).

Configure CodeBuild's VPC settings for private resources, and reference secrets from Parameter Store or Secrets Manager instead of placing plaintext in build configuration.
