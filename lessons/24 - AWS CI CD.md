# AWS CI/CD

## Pipeline building blocks
* Continuous integration means frequent source changes are automatically built and tested to surface problems early. Continuous delivery automates the path to release, often with multiple environments and optional approval gates.
* CodeCommit hosts private Git repositories and integrates with AWS IAM. CodePipeline orchestrates source, build, test, and deployment actions; stages can run sequential or parallel actions, and manual approvals can pause a release. Stage artifacts are stored in S3 and passed forward.
* CodeBuild is a managed build-and-test service that runs in Docker-based environments and scales without managing build servers. A `buildspec.yml` at the source root defines environment variables, install/pre-build/build/post-build phases, output artifacts, and optional dependency caching. Store secrets in Parameter Store or Secrets Manager instead of plaintext configuration.
* CodeDeploy automates releases to EC2/on-premises hosts, Lambda, and ECS. EC2/on-premises targets require the CodeDeploy agent and an `appspec.yml`. Deployment choices include in-place or blue/green for supported targets; Lambda and ECS support gradual traffic shifting, while ECS uses blue/green. Rollbacks can be manual or triggered by deployment failure or CloudWatch alarms.

## Exam distinctions and operations
* CodePipeline coordinates tools; it does not itself compile or deploy the application. CodeBuild performs builds/tests, while CodeDeploy manages rollout and rollback behavior.
* CodePipeline and CodeBuild can use third-party source/build providers as well as AWS services. CodeArtifact manages software dependencies and can publish package-change events through EventBridge.
* For troubleshooting, inspect pipeline stage/action state and CloudWatch/EventBridge notifications. Check the pipeline service role’s permissions when an action cannot run; CloudTrail records API activity.
* CodeDeploy rollback redeploys the last known-good application revision as a new deployment. For Lambda alias releases, traffic can shift all at once, linearly, or by canary; pre/post-traffic hooks can validate health.
