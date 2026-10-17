# AWS Serverless Application Model

## What SAM adds
* AWS SAM is a framework for describing, building, and deploying serverless applications with concise YAML.
* Its `Transform: AWS::Serverless-2016-10-31` header tells CloudFormation to interpret SAM-specific resources, such as `AWS::Serverless::Function`, `AWS::Serverless::Api`, and `AWS::Serverless::SimpleTable`.
* SAM extends rather than replaces CloudFormation: regular CloudFormation outputs, mappings, parameters, and resources can also be used. The transform expands the shorthand into CloudFormation resources.
* SAM can coordinate Lambda deployments through CodeDeploy, including traffic shifting, pre/post-traffic validation hooks, CloudWatch alarms, and rollback.

## Build and iterate
* `sam build` prepares the application locally; `sam deploy` packages and uploads artifacts, then deploys the generated CloudFormation change set and stack.
* `sam sync` synchronizes a SAM project with AWS. `sam sync --code` can push code-only changes without updating infrastructure through CloudFormation; `--watch` monitors files and synchronizes changes automatically.
* SAM policy templates provide common Lambda permissions, such as S3 read access, SQS polling, or DynamoDB CRUD. Review the resulting permissions and keep them least-privileged.
* For local development, `sam local invoke` runs a function once, `sam local start-lambda` exposes a local Lambda endpoint, and `sam local start-api` runs an API Gateway-like HTTP endpoint. `sam local generate-event` creates sample event payloads for testing.
* `samconfig.toml` can hold environment-specific settings so the same application can be deployed to environments such as development and production.

## Exam distinction
SAM is serverless-focused and declarative (YAML); CDK supports broader infrastructure using programming languages. Both ultimately use CloudFormation for deployment. Use SAM when the scenario emphasizes concise serverless templates, local Lambda/API testing, or streamlined serverless deployment.
