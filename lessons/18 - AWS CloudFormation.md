# AWS CloudFormation

## At a glance
- CloudFormation provisions and updates AWS resources declaratively from YAML or JSON templates.
- Templates combine resources with optional inputs, mappings, conditions, and outputs.
- References and intrinsic functions connect resources and stacks.
- Change sets, rollback controls, policies, and permissions help manage stack changes safely.
- Custom resources and StackSets extend CloudFormation to unsupported actions and multi-account/Region deployments.

## Infrastructure as code
- CloudFormation declaratively provisions and updates AWS resources from YAML or JSON templates.
- Describe the desired resources and their configuration; CloudFormation determines dependencies and the order of creation, updates, and deletion.
- A stack groups the resources managed from a template, making repeatable deployments, version control, review, and environment teardown practical.
- A template has a required `Resources` section and may also include:
  - `AWSTemplateFormatVersion`
  - `Description`
  - `Parameters`
  - `Mappings`
  - `Conditions`
  - `Outputs`
- Resource types use identifiers such as `AWS::EC2::Instance`; consult the resource reference for each type's properties and return attributes.
- Most AWS services have resource types, while custom resources cover gaps.
- Templates can be authored in a code editor or Infrastructure Composer and deployed through the console, CLI, or a delivery pipeline.
- A stack is identified by name.
- Template versions should be reviewed and managed in source control; changing infrastructure means submitting an updated template rather than editing the existing stack's resources manually.
- CloudFormation makes it easier to reproduce environments, review infrastructure changes as code, estimate resource cost, and separate ownership into network, shared-service, and application stacks.
- Resources are tagged with stack identifiers, which helps associate cost with a stack.
- Deleting a stack normally deletes its resources unless configured otherwise; do not treat stack deletion as a harmless cleanup when it owns data.

## Inputs, references, and outputs
### Parameters and mappings
- **Parameters** provide deployment-time values that are not known or should not be fixed in the template, such as an instance type or user-specific network ID.
- Parameter types, allowed values, patterns, and bounds can validate inputs and prevent mistakes.
- **Mappings** are fixed lookup tables encoded in the template.
- Use mappings when a known value is determined by a stable factor such as Region, account, or environment—for example, selecting a Region-specific AMI.
- `Fn::FindInMap` retrieves a value.
- Prefer parameters when a value must be chosen by the user rather than looked up from a fixed table.

### Outputs and cross-stack references
- **Outputs** expose useful resource values such as a VPC ID or security-group ID.
- An output can be exported and referenced by another stack with `Fn::ImportValue`, enabling collaboration between stacks.
- An exported value cannot be removed, nor can its exporting stack be deleted, while another stack imports it.
- Exports are scoped to the same account and Region.

### Intrinsic and pseudo parameters
- `Ref` returns a parameter's supplied value or a resource's physical identifier.
- `Fn::GetAtt` reads a resource attribute defined by that resource type, such as an instance's public DNS name.
- Other useful functions include `Fn::Sub` for string substitution, `Fn::Join`, `Fn::Select`, `Fn::GetAZs`, `Fn::Base64`, and condition functions.
- YAML provides shorthand such as `!Ref`.
- Pseudo parameters are available without declaring them, including `AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::StackId`, `AWS::NotificationARNs`, and `AWS::NoValue`.
- Conditions, often based on parameters or mappings, can include or exclude resources and outputs.
- `Fn::If` and logical functions such as `Fn::Equals`, `Fn::And`, `Fn::Not`, and `Fn::Or` express condition logic.

## Safe stack changes and permissions
### Preview and recover from changes
- A change set previews the resource changes CloudFormation expects from an update, helping catch replacement or deletion of important resources before execution.
- Review the change set, especially for stateful resources, before applying it.
- A failed stack creation rolls back and deletes resources by default; rollback can be disabled when retaining resources for troubleshooting is preferable.
- A failed update normally rolls back to the last working state.
- Review stack events to locate the failure.
- If rollback itself fails, correct the underlying issue and continue the rollback with the console or `ContinueUpdateRollback`.

### Roles and capability acknowledgements
- A service role lets CloudFormation create, update, or delete stack resources using the role's permissions, rather than requiring every caller to have direct permissions for each resource.
- A user who supplies the role needs `iam:PassRole`; keep both the role and the ability to pass it least-privileged.
- A template that creates or updates IAM resources requires acknowledging `CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` (the latter when named IAM resources are specified).
- Templates using macros or nested stacks may require `CAPABILITY_AUTO_EXPAND`.
- Missing acknowledgements produce an insufficient-capabilities error; they are an explicit security confirmation, not a grant of IAM permissions.

### Deletion and update protections
- The default deletion behavior is to delete a resource when the stack is deleted or the resource is removed from the template.
- `DeletionPolicy: Retain` preserves a resource outside stack management.
- `Snapshot` takes a final snapshot before deletion for supported resources such as RDS databases and EBS volumes.
- An S3 bucket cannot be deleted while it contains objects unless those objects are first removed.
- Termination protection prevents accidental deletion of a stack.
- Stack policies constrain which resources can be updated during stack updates.
- Once a policy is set, resources are protected by default unless the policy explicitly permits the update; use this to guard a production database against an unintended replacement.
- A stack policy is separate from IAM permissions and termination protection.

## Custom resources and StackSets
### Custom resources
- A custom resource runs provisioning logic during stack create, update, or delete for a resource type or action CloudFormation does not natively support.
- It is declared as `AWS::CloudFormation::CustomResource` or `Custom::...` and backed most commonly by a Lambda function (or an SNS topic).
- `ServiceToken` identifies the handler and must be in the same Region.
- For example, custom logic can empty a non-empty S3 bucket before CloudFormation deletes it.

### StackSets
- StackSets apply a template across multiple AWS accounts and Regions from one operation.
- A StackSet contains stack instances in the targeted account/Region pairs; updating it can update all associated instances.
- In an AWS Organization, centralized deployment can target member accounts, and the administrator or delegated administrator manages StackSets.
