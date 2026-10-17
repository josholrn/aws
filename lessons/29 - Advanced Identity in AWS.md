# Advanced Identity in AWS

## At a glance

* **STS** issues temporary credentials for time-limited, scoped access instead of distributing long-lived keys.
* Cross-account access depends on both a role's trust policy and its permissions policy.
* Authorization starts with implicit deny; an explicit deny overrides an allow.
* `iam:PassRole` lets a principal configure a service to use a role, while the service's trust policy separately allows it to assume that role.
* Use least privilege, MFA where required, workload-specific roles, and CloudTrail for operational review.

## AWS STS and temporary credentials

AWS Security Token Service (STS) issues temporary credentials for principals that need time-limited AWS access. A session includes an access key ID, secret access key, session token, and expiration; all three credential values are required when making signed API requests. Temporary credentials reduce the risk of distributing long-lived IAM user keys and can be scoped to a specific role and task.

The main STS operations highlighted in this chapter are:

* **`AssumeRole`** returns credentials for an IAM role in the same or another account. For cross-account access, the target role's trust policy must allow the caller, and the role's permissions policy defines what the assumed identity can do.
* **`AssumeRoleWithSAML`** exchanges a SAML identity-provider assertion for role credentials.
* **`AssumeRoleWithWebIdentity`** exchanges a web identity token for role credentials. For web/mobile identity federation, prefer Cognito Identity Pools rather than building directly on this STS operation.
* **`GetSessionToken`** provides session credentials for an IAM user or root identity, including MFA-backed sessions when configured.
* **`GetFederationToken`** returns temporary credentials for a federated user.
* **`GetCallerIdentity`** identifies the IAM user or role whose credentials signed the request.
* **`DecodeAuthorizationMessage`** decodes an encoded authorization failure message when one is returned by an AWS API.

The slides use 15 minutes to one hour as the usual duration range for assumed-role credentials. Role session duration can be configured subject to AWS limits and service behavior; do not confuse a role session with a long-lived access key.

For on-premises workloads, obtain temporary credentials through STS instead of leaving IAM user keys on the server. EC2 instances, Lambda functions, ECS tasks, and CodeBuild projects should each receive a purpose-specific role.

### Cross-account role flow

1. Create a role in the target account.
2. Configure its trust relationship to name the permitted source principal or account.
3. Grant the role only the required actions/resources.
4. Have the caller use `AssumeRole`.

A trust relationship answers **who may assume the role**; the role's permissions answer **what the assumed role may do**. Both sides of that distinction matter when diagnosing `AccessDenied`.

## Authorization evaluation

The simplified evaluation model starts with **implicit deny**:

1. Evaluate all applicable policies.
2. If any applicable policy explicitly denies the request, the result is deny.
3. Otherwise, an applicable allow can authorize it.
4. If nothing allows it, the request remains denied.

An explicit deny overrides an allow. IAM identity policies are attached to users, groups, or roles. S3 bucket policies are resource policies attached to buckets.

For S3 access, an allow from an applicable identity policy or bucket policy can authorize the principal, but an explicit deny in either can block the operation. A role's allow does not override a bucket policy's deny, and a bucket-policy allow does not override an identity-policy deny. Other policy types and service-specific rules can further constrain a request, so this simplified model is not a claim that every AWS authorization decision uses only two policies.

### Policy variables and attachment styles

Policy variables make common access patterns reusable. For example, `${aws:username}` can be used in an IAM policy resource path to give users access to their own `/home/<username>` prefix in S3 without maintaining a separate nearly identical policy for every user. Carefully scope both the object ARN and any bucket-listing permissions; access to a prefix's objects and permission to list a bucket are distinct actions.

Policy attachment style affects maintainability:

* **AWS managed policies** are maintained by AWS and can evolve as services and APIs change; review their scope before attaching them, particularly for privileged users.
* **Customer managed policies** are reusable across principals and can be versioned and centrally updated, making them the preferred reusable policy type in the slide material.
* **Inline policies** have a one-to-one relationship with a principal and are deleted with that principal. Use them only when the coupling is intentional.

## Passing a role to a service

When configuring an AWS service to perform work using a role, the configuring principal often needs `iam:PassRole` (frequently accompanied by `iam:GetRole` to inspect it). Examples include assigning a role to EC2, Lambda, ECS, or CodePipeline.

Passing the role is a setup-time authorization that lets the caller tell the service which role to use; it does **not** grant the service the role's permissions by itself. The service must separately be allowed to assume the role through its trust policy.

Scope `iam:PassRole` to the specific role ARNs and, where practical, use conditions to restrict the destination service. Otherwise, a principal that can pass a powerful role could indirectly cause a service to exercise permissions that principal does not have directly.

## MFA, least privilege, and operational checks

For MFA-protected temporary access, the slides describe `GetSessionToken` plus an IAM policy condition such as `aws:MultiFactorAuthPresent: true`. Ensure the policy requires MFA for the intended actions; merely enabling MFA on an account does not cause every API call to require it.

Follow least privilege:

* Avoid broad `*` permissions.
* Give each workload its own role.
* Enable MFA on the root account.
* Avoid using root credentials for routine work.
* Review CloudTrail records for API activity, including denied requests, to identify attempted misuse and diagnose missing permissions.

For cross-account access, verify both the role trust policy and its permissions, then use STS to obtain an expiring session rather than copying permanent credentials between accounts.
