# Advanced Identity in AWS

## AWS STS and temporary credentials
* Security Token Service (STS) issues short-lived credentials instead of requiring long-lived access keys. `AssumeRole` supports role assumption within or across accounts; the role's trust policy must allow the calling principal.
* Federation-specific APIs include `AssumeRoleWithSAML` and `AssumeRoleWithWebIdentity`; Cognito Identity Pools are generally preferred for web/mobile identity federation. `GetSessionToken` can provide MFA-backed session credentials, and `GetCallerIdentity` reports which identity made a request.
* Temporary credentials contain an access key ID, secret access key, session token, and expiration. Grant only the role permissions needed for the task.
* On-premises workloads should obtain temporary credentials through a role rather than keep IAM user keys on the host. Give EC2, Lambda, ECS tasks, and CodeBuild appropriately scoped roles rather than reusing a broad role.

## How authorization is evaluated
* Begin with implicit deny. An explicit deny in an applicable policy overrides any allow; otherwise, an applicable allow grants access, and no allow means deny.
* IAM identity policies attach to users, groups, or roles; S3 bucket policies attach to buckets. An allow from either can authorize an S3 request, but an explicit deny from either prevents it.
* Use policy variables such as `${aws:username}` to create reusable per-user policies—for example, limiting access to a user's own S3 home prefix.
* Managed policies can be reused across principals; inline policies are tied to one principal and are removed with it.

## Role delegation and safe access
* A user who configures a service with a role often needs `iam:PassRole`. This permission allows passing the role to a service; it does not itself grant the service permission to assume it. The role's trust policy must name an allowed service or principal.
* For MFA-protected role/session access, use the appropriate STS flow and an IAM condition such as `aws:MultiFactorAuthPresent`.
* Follow least privilege, enable root MFA, and review API activity—including denied calls—in CloudTrail. For cross-account access, configure both the target role's trust and permissions, then assume it for temporary access.
