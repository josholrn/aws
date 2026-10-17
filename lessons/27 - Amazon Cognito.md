# Amazon Cognito

## User Pools: authenticate application users
* A Cognito User Pool is a managed user directory for web and mobile applications. It supports sign-up and sign-in, password reset, email/phone verification, MFA, and federation with social, OIDC, or SAML identity providers.
* Successful authentication returns JSON Web Tokens (JWTs). Verify a token's signature and validity before trusting its claims; the payload can include user attributes and identifiers.
* User Pools can provide a hosted sign-in interface and invoke Lambda triggers during authentication or registration—for example, to validate requests, customize claims, migrate users, or tailor messages.
* API Gateway and Application Load Balancers can use User Pool authentication. For ALB authentication actions, configure an HTTPS listener; the load balancer can authenticate before forwarding requests to the application.

## Identity Pools: issue AWS credentials
* A Cognito Identity Pool exchanges supported identities for temporary AWS credentials. Identity sources may include User Pools, social providers, OIDC/SAML providers, or a custom developer-authenticated identity system.
* Identity Pools can also allow guest access. They map authenticated and unauthenticated users to IAM roles, and rules or policy variables can scope what a particular user may access.
* The temporary credentials let a client access AWS services directly or access them through an API. Grant only the permissions the client needs; Identity Pools obtain credentials through STS.

## Choose the right pool
* **User Pool = authentication**: manage users, validate sign-in, and issue tokens.
* **Identity Pool = authorization to AWS resources**: exchange identity information for scoped AWS credentials.
* They can be combined: authenticate a person with a User Pool, then use an Identity Pool to grant temporary, policy-controlled access to AWS resources.
* For social or enterprise federation, select the pool based on the required result: application tokens and user-directory features call for a User Pool; temporary AWS credentials call for an Identity Pool.
