# AWS Cloud Development Kit

## At a glance
* CDK defines AWS infrastructure using programming languages and synthesizes it into CloudFormation templates.
* Constructs range from low-level CloudFormation resources to higher-level service abstractions and patterns.
* The CDK CLI supports synthesis, diffing, deployment, and destruction through CloudFormation.
* Bootstrap each account/Region environment before deploying there.
* Test synthesized infrastructure with assertions or snapshots; use SAM CLI for local invocation of a CDK-defined Lambda.

## Define infrastructure as software
* AWS CDK lets developers define cloud infrastructure using familiar languages, including TypeScript/JavaScript, Python, Java, and .NET.
  * The program instantiates resources and reusable abstractions.
  * `cdk synth` synthesizes the app into a CloudFormation JSON/YAML template.
* CloudFormation remains the provisioning engine. CDK does not bypass CloudFormation’s deployment model: stacks are deployed/updated through CloudFormation, and the synthesized template is useful for review, testing, and troubleshooting.
* CDK supports infrastructure across AWS services, including Lambda, API Gateway, DynamoDB, S3, ECS/EKS, and networking. App code and infrastructure can be defined and deployed together, which is especially convenient for functions and containers.
* A **construct** is a component that encapsulates the resources and configuration needed for part of an application. Constructs can be composed into higher-level resources or architecture patterns, reused across stacks/projects, and parameterized with application-specific values.

## Construct levels and libraries
* **L1 (CFN resources):** one-to-one representations of CloudFormation resources, usually named `Cfn...` (such as `CfnBucket`).
  * They expose underlying CloudFormation properties and require detailed configuration.
  * Use them when you need a property not surfaced by a higher-level construct.
* **L2 (intent-based):** higher-level representations of individual AWS services with sensible defaults and convenience methods.
  * For example, an S3 bucket construct can provide helper methods for lifecycle rules and grants, reducing boilerplate while still modeling one resource.
* **L3 (patterns):** opinionated combinations of multiple related resources for a common use case.
  * Examples include an API Gateway REST API backed by Lambda or Fargate behind an Application Load Balancer.
  * Patterns speed up common architectures but still synthesize to ordinary resources that should be reviewed.
* The AWS Construct Library contains constructs for AWS resources. Construct Hub also offers AWS, community, and third-party constructs; assess versions and security/maintenance before adopting external constructs.

## CLI workflow and deployment environments
* `cdk init app` scaffolds a project. Install the CDK CLI (commonly through the `aws-cdk` package) and the language libraries required by the app.
* `cdk synth` renders the template without deploying it; `cdk diff` compares the local synthesized stack with the deployed stack; `cdk deploy` publishes the stack through CloudFormation; `cdk destroy` removes it.
  * Inspect synth/diff output before deployment, especially for replacements, IAM changes, and data deletion.
* **Bootstrapping is per AWS environment**—an account and Region.
  * Run `cdk bootstrap aws://<account>/<region>` before first deployment there.
  * It creates the `CDKToolkit` CloudFormation stack, which includes staging resources such as an S3 bucket and IAM deployment roles.
  * Missing bootstrap resources/roles are a common cause of deployment errors.
* CDK applications can target multiple accounts and Regions, but each target environment needs correct credentials, permissions, and bootstrap state. Bootstrapping provisions the deployment toolkit; it does not deploy the application stack itself.

## Testing and SAM comparison
* Use the CDK Assertions module with a test framework such as Jest or Pytest.
  * Fine-grained assertions test specific synthesized resources/properties (for example, that a bucket has a particular encryption setting).
  * Snapshot tests compare the complete synthesized template to a saved baseline and can detect broad changes, but they need maintenance when intended changes occur.
* SAM is serverless-focused and declarative YAML; CDK covers infrastructure broadly through programming languages and abstractions. Both ultimately use CloudFormation.
  * Choose SAM for concise serverless templates and SAM’s local workflow.
  * Choose CDK when reusable code abstractions, richer logic, or non-serverless resources are central.
* SAM CLI can test a CDK-defined Lambda locally after synthesizing the stack: run `cdk synth` to produce the CloudFormation template, then point `sam local invoke` at that template and the function’s logical ID. This local run does not provision the complete CDK stack.
