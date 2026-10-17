# Amazon Cognito

## At a glance
* User Pools provide user-directory features, authentication, federation, and JWTs for applications.
* Identity Pools exchange supported identity tokens for scoped, temporary AWS credentials.
* The pools can be combined, or User Pool tokens can be used at API Gateway/ALB without direct client AWS access.
* Validate JWT signatures and claims; decoded token content alone is not trusted.
* Guest access remains limited by the unauthenticated IAM role.

## User Pools: application sign-in and tokens
* A **Cognito User Pool** is a managed user directory and authentication service for web/mobile apps.
  * It supports sign-up/sign-in with username or email and password, password reset, email/phone verification, and MFA.
  * It supports federation with social providers or enterprise OIDC/SAML identity providers.
  * Risk-based/adaptive authentication can challenge or block suspicious sign-ins.
* Successful authentication returns JWTs, commonly an ID token with identity claims and an access token for authorized requests (and optionally a refresh token to obtain new tokens).
  * A JWT contains a header, payload, and signature; Base64 encoding is not encryption.
  * The application/API must validate the signature and relevant claims—issuer, expiry, token use, client/audience, and scopes as applicable—before trusting its contents.
* The `sub` claim is a stable user identifier; tokens may also contain attributes such as name, email, phone, and custom claims. Do not accept a user-supplied decoded payload without signature/claim validation.
* A hosted sign-in UI can provide sign-up/sign-in flows and federated login. It can be customized with branding/CSS.
  * For a Cognito custom domain, the ACM certificate must be in `us-east-1`.
  * Configure app clients and callback URLs consistently with the external identity provider.
* User Pool Lambda triggers run synchronously at points in registration/authentication/token flows.
  * Examples include pre-sign-up validation, pre-authentication checks, post-authentication analytics, post-confirmation messages, migration from a legacy user directory, custom/localized messages, and pre-token-generation claim customization.
  * Trigger functions must meet the flow’s response contract and latency expectations.
* API Gateway can validate User Pool tokens as an authorizer before invoking a backend.
* An Application Load Balancer can authenticate users using a User Pool and then forward to a target group, offloading authentication from application code.
  * ALB authentication actions require an HTTPS listener.
  * Configure the user pool, app client, domain, and callback URLs.

## Identity Pools: temporary AWS credentials
* A **Cognito Identity Pool** exchanges a supported identity token for temporary AWS credentials.
  * Identity providers can include Cognito User Pools, social providers (such as Google, Facebook, or Login with Amazon), OIDC/SAML providers, or a custom developer-authenticated identity system.
* Identity Pools can map authenticated users and optional unauthenticated guests to IAM roles.
  * The role’s trust policy must trust the Identity Pool; Cognito obtains credentials through STS.
  * The returned temporary credentials let a web/mobile client call authorized AWS services directly, or the client can use an API.
* Use role mappings/rules and policy variables to scope access to each user’s own objects or records.
  * For example, an S3 object prefix can incorporate a user identity, or DynamoDB access can be constrained to a user-specific partition.
  * Never give a mobile client broad static IAM credentials; use temporary credentials and least-privilege role policies.
* Guest access is unauthenticated, not unrestricted: the guest role must contain only the narrow permissions needed for public/anonymous operations. A user can move from guest to authenticated access after sign-in, subject to the pool’s identity and role configuration.

## Selecting and combining pools
* **User Pool = authentication and application identity:** a user directory, sign-in experience, federation, MFA, triggers, and JWTs. It does not by itself hand the client general AWS service credentials.
* **Identity Pool = AWS authorization credentials:** exchange a verified identity for scoped, temporary IAM credentials. It is useful when browser/mobile clients need direct access to services such as S3 or DynamoDB.
* They are often combined: the user signs in to a User Pool and receives a JWT; the Identity Pool trusts that User Pool as an identity provider and exchanges the token for temporary AWS credentials.
  * Alternatively, pass the User Pool JWT to API Gateway/ALB and keep AWS access entirely on the server side.
* Choose by required outcome, not by whether users are “federated.”
  * If the application needs user-directory features and API tokens, use a User Pool.
  * If clients need temporary AWS credentials, use an Identity Pool.
  * Enterprise/social federation can feed either flow, and a User Pool token can be an Identity Pool identity source.

## Exam gotchas
* User Pool tokens are not AWS credentials; Identity Pool credentials are temporary STS credentials, not application JWTs.
* Validate JWT signatures and claims.
* ALB Cognito authentication needs HTTPS.
* Guest users still receive only the permissions granted to the unauthenticated role.
