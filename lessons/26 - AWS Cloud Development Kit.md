# AWS Cloud Development Kit

## How CDK works
* AWS CDK lets you define cloud infrastructure in familiar languages such as TypeScript/JavaScript, Python, Java, or .NET.
* Infrastructure is composed from constructs. `cdk synth` converts the CDK app into a CloudFormation template; CloudFormation then provisions the stack. Application code and infrastructure can be deployed together.
* CDK supports AWS services broadly, while SAM is more focused on serverless resources and uses YAML. Both use CloudFormation underneath.

## Construct levels
* **L1** constructs map directly to CloudFormation resources (often named `Cfn...`). They expose the underlying properties and require explicit configuration.
* **L2** constructs provide a higher-level, intent-oriented interface with useful defaults and helper methods.
* **L3** constructs, or patterns, combine multiple resources to implement common architectures, such as an API Gateway backed by Lambda or Fargate behind an Application Load Balancer.
* The AWS Construct Library supplies constructs for AWS resources; Construct Hub offers community and third-party constructs as well.

## Commands and environment setup
* `cdk init app` creates a project; `cdk synth` displays its generated template; `cdk diff` compares local and deployed stacks; `cdk deploy` deploys; `cdk destroy` removes stacks.
* Before deployment, run `cdk bootstrap` for each target account-and-region environment. It creates the `CDKToolkit` CloudFormation stack, including staging storage and deployment IAM roles.
* CDK tests can use the Assertions module with frameworks such as Jest or Pytest. Fine-grained assertions check template resources and properties; snapshot tests compare a synthesized template with a saved baseline.
* SAM CLI can locally invoke functions from a CDK app after `cdk synth` produces its CloudFormation template.

## Exam distinction
Choose CDK when infrastructure is best represented with reusable programming-language abstractions or patterns; choose SAM for concise serverless YAML and its local development workflow.
