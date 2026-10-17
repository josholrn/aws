# AWS CloudFormation

## Infrastructure as Code
* CloudFormation declaratively provisions and updates AWS resources from YAML or JSON templates. A stack groups the resources created from a template; deleting the stack normally deletes its resources.
* Templates have required `Resources` plus optional parameters, mappings, conditions, and outputs. Resources reference one another so CloudFormation can determine ordering. Use the resource reference documentation for supported properties and types.
* Parameters supply values at deployment time and can validate user-specific input. Mappings hold fixed values chosen from known factors such as region or environment; use `Fn::FindInMap` to look them up.
* Outputs expose useful values such as IDs. Exported outputs can be imported by other stacks, but a stack cannot be deleted while another stack still references its export.

## References and Deployment Safety
* `Ref` returns a parameter value or a resource's physical ID; `GetAtt` returns a resource attribute. Other common functions include `Sub`, `Join`, `Select`, `GetAZs`, and `If`.
* Conditions control whether resources or outputs are created, often based on a parameter or region.
* A failed stack creation rolls back by default; failed updates revert to the last working state. Inspect events for errors. A failed rollback may require repairing resources and continuing the rollback.
* A CloudFormation service role lets the service provision resources on behalf of users; users need `iam:PassRole`. Acknowledging `CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` is required when templates create IAM resources.
* Deletion policies can retain a resource, delete it, or snapshot supported resources such as databases. Choose deliberately for data-bearing resources.
* StackSets deploy stacks across multiple accounts and regions. Use version control and review template changes before deployment; change sets help preview proposed updates.
