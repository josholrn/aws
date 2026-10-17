# AWS Serverless Application Model

## At a glance
* SAM is a CloudFormation extension for describing, building, testing, and deploying serverless applications in concise YAML.
* SAM shorthand resources and policy templates simplify common serverless definitions but still require permission-scope review.
* SAM CLI provides build, deploy, sync, and local invocation workflows.
* SAM can coordinate safe Lambda releases through CodeDeploy settings.
* Choose SAM for a serverless-focused declarative workflow; CDK provides broader programming-language abstractions.

## Template model and resources
* AWS SAM is a framework for describing, building, testing, and deploying serverless applications using concise YAML.
  * It is an extension of CloudFormation rather than a separate provisioning engine: the template is transformed into ordinary CloudFormation resources, and CloudFormation creates/updates the stack.
* Include `Transform: AWS::Serverless-2016-10-31` so CloudFormation recognizes SAM syntax.
  * Common shorthand resources include `AWS::Serverless::Function`, `AWS::Serverless::Api`, and `AWS::Serverless::SimpleTable`.
  * A SAM template can also contain normal CloudFormation resources, parameters, mappings, outputs, and conditions.
* The SAM template ties application code to infrastructure such as Lambda functions, API Gateway APIs, and DynamoDB tables. This makes serverless defaults and integrations quicker to express than writing all underlying resources by hand, while still allowing explicit CloudFormation configuration.
* SAM policy templates provide common Lambda permissions (for example, S3 read, SQS polling, or DynamoDB CRUD).
  * They are convenience templates, not a guarantee of least privilege: check resource scope and remove permissions the function does not require.

## Build, deploy, and synchronize
* `sam build` prepares functions and dependencies locally for deployment or local invocation.
* `sam deploy` packages/uploads code artifacts and deploys the generated CloudFormation change set and stack; `sam package` can be used separately to package and upload artifacts.
  * The normal deployment path therefore includes both code artifacts (typically uploaded to S3) and CloudFormation infrastructure updates.
  * Review change sets for resource replacements or permission changes just as with other CloudFormation deployments.
* `sam sync` synchronizes the project described in the SAM template with AWS. A regular sync can update code and infrastructure.
  * `sam sync --code` applies code-only changes through service APIs, bypassing CloudFormation for speed; it does not replace a CloudFormation deployment when infrastructure/configuration changes.
  * `sam sync --watch` monitors local files. Code-only changes use the fast code sync; template/configuration changes use the infrastructure sync path.
  * `--resource` or `--resource-id` can limit synchronization to a particular function/resource.
* Use `samconfig.toml` to keep environment-specific settings such as stack name, Region, and parameter overrides.
  * For example, `sam deploy --config-env dev` selects a development configuration; the same template can deploy to another environment with distinct settings.

## Local testing and Lambda releases
* `sam local invoke` starts one function invocation with a supplied event and exits.
* `sam local start-lambda` exposes a local Lambda-compatible endpoint for automated tests.
* `sam local start-api` runs a local HTTP endpoint for API-backed functions and can reload function code after changes.
* `sam local generate-event` produces representative event payloads for sources such as S3, API Gateway, SNS, Kinesis, and DynamoDB.
  * Local emulation is useful for handler logic and event shape, but AWS service credentials, networking, and service behavior still need appropriate testing.
  * Pass the intended AWS profile when local code calls AWS.
* SAM can integrate with CodeDeploy for safe Lambda releases.
  * `AutoPublishAlias` publishes a new immutable Lambda version and moves an alias when code changes.
  * `DeploymentPreference` selects `Canary`, `Linear`, or `AllAtOnce`.
  * CloudWatch alarms can roll back unhealthy releases, and pre-/post-traffic hook functions can run tests before and after the shift.

## Choosing SAM or CDK
* SAM is a strong fit when the application is mainly serverless and the desired workflow is declarative YAML, concise Lambda/API/table definitions, local Lambda/API invocation, and streamlined serverless deployment.
* CDK is more general-purpose and defines infrastructure in programming languages.
* Both synthesize CloudFormation and use CloudFormation for durable infrastructure deployment.
* SAM can also use SAM CLI to test a synthesized CDK template.
