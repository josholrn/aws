
# IAM & AWS CLI

## difference between aws identity center vs iam user?
AWS Identity Center (formerly AWS SSO) and IAM Users are both used for identity and access management in AWS, but they operate at different architectural scopes and serve fundamentally different purposes

The single core distinction: IAM Users are local identities tied to a single AWS account, while AWS IAM Identity Center manages centralized human workforce identities across an entire AWS Organization.

## users and groups
* IAM = Identity and Access Management, Global service
* Root account created by default, shouldn’t be used or shared
* Users are people within your organization, and can be grouped
* Groups only contain users, not other groups
* Users don’t have to belong to a group, and user can belong to multiple groups

## IAM: Permissions
* Users or Groups can be assigned JSON documents called policies
* These policies define the permissions of the users
* In AWS you apply the least privilege principle: don’t give more permissions than a user needs

## IAM Policies Structure
* Consists of
    * Version: policy language version, always include “2012-10-17”
    * Id: an identifier for the policy (optional)
    * Statement: one or more individual statements (required)
* Statements consists of
    * Sid: an identifier for the statement (optional)
    * Effect: whether the statement allows or denies access (Allow, Deny)
    * Principal: account/user/role to which this policy applied to
    * Action: list of actions this policy allows or denies
    * Resource: list of resources to which the actions applied to
    * Condition: conditions for when this policy is in effect (optional)

## IAM – Password Policy
* Strong passwords = higher security for your account
* In AWS, you can setup a password policy

## Multi Factor Authentication - MFA
* Users have access to your account and can possibly change configurations or delete resources in your AWS account
* You want to protect your Root Accounts and IAM users
* MFA = password you know + security device you own
* Main benefit of MFA: if a password is stolen or hacked, the account is not compromised

### MFA devices options in AWS
* Virtual MFA device
  * Support for multiple tokens on a single device
    * Google Authenticator (phone only)
    * Authy (phone only)
* Universal 2nd Factor (U2F) Security Key
  * Support for multiple root and IAM users using a single security key
    * YubiKey by Yubico (3rd party)
* Hardware Key Fob MFA Device
  * Provided by Gemalto (3rd party)
* Hardware Key Fob MFA Device for AWS GovCloud (US)
  * Provided by SurePassID (3rd party)

## How can users access AWS ?
* To access AWS, you have three options:
  * AWS Management Console (protected by password + MFA)
  * AWS Command Line Interface (CLI): protected by access keys
  * AWS Software Developer Kit (SDK) - for code: protected by access keys
* Access Keys are generated through the AWS Console
* Users manage their own access keys
* Access Keys are secret, just like a password. Don’t share them
  * Access Key ID ~= username
  * Secret Access Key ~= password

## IAM Roles for Services
* Some AWS service will need to perform actions on your behalf
* To do so, we will assign permissions to AWS services with IAM Roles
* Common roles:
    * EC2 Instance Roles
    * Lambda Function Roles
    * Roles for CloudFormation
* Trusted entity type of roles that allows to perform actions in this account.
  * AWS Service: Allow AWS services like EC2, Lambda or others
  * AWS Account: Allow entities in other AWS accounts belonging to you or a 3rd party
  * Web Identity: Allows users federated by the specified external web identity provider to assume this role.
  * SAML 2.0 Federation: Allow users federated with SAML 2.0 from a corporate directory.
  * Custom trust policy: Create a custom trust policy.


## IAM Security Tools
* IAM Credentials Report (account-level)
    * a report that lists all your account's users and the status of their various credentials
* IAM Access Advisor or "Last Access" (user-level)
    * Access advisor shows the service permissions granted to a user and when those services were last accessed.
    * You can use this information to revise your policies

## IAM Guidelines & Best Practices
* Don’t use the root account except for AWS account setup
* One physical user = One AWS user
* Assign users to groups and assign permissions to groups
* Create a strong password policy
* Use and enforce the use of Multi Factor Authentication (MFA)
* Create and use Roles for giving permissions to AWS services
* Use Access Keys for Programmatic Access (CLI / SDK)
* Audit permissions of your account using IAM Credentials Report & IAM Access Advisor
* Never share IAM users & Access Keys


## Shared Responsibility Model for IAM
* AWS:
  * Infrastructure (global network security)
  * Configuration and vulnerability analysis
  * Compliance validation

* You
  * Users, Groups, Roles, Policies management and monitoring
  * Enable MFA on all accounts
  * Rotate all your keys often
  * Use IAM tools to apply appropriate permissions
  * Analyze access patterns & review permissions


# IAM Section – Summary
* Users: mapped to a physical user, has a password for AWS Console
* Groups: contains users only
* Policies: JSON document that outlines permissions for users or groups
* Roles: for EC2 instances or AWS services
* Security: MFA + Password Policy
* AWS CLI: manage your AWS services using the command-line
* AWS SDK: manage your AWS services using a programming language
* Access Keys: access AWS using the CLI or SDK
* Audit: IAM Credential Reports & IAM Access Advisor
