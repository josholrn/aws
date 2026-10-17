# AWS CI/CD

## At a glance
* CI automates feedback on frequent changes; CD automates the validated path toward release, with or without production approval gates.
* CodePipeline orchestrates source, build/test, and deployment actions, passing artifacts stored in S3.
* CodeBuild builds and tests; CodeDeploy manages rollouts and rollback; CodeArtifact stores packages.
* Deployment strategies and troubleshooting differ for EC2/on-premises, Lambda, and ECS.
* A successful build is not the same as a deployed release.

## Continuous integration and delivery
* **Continuous integration (CI)** means developers frequently push small changes to version control and an automated build/test system checks them promptly. Fast feedback surfaces defects early and keeps changes easier to isolate.
* **Continuous delivery (CD)** automates the validated path toward release so software can be deployed reliably and often.
  * Pipelines may deploy automatically or pause at manual approval gates before production.
  * Continuous delivery does not necessarily mean every change is automatically released to users.
* A typical flow is source → build → test → deploy → optional load testing/provisioning. AWS services can be combined with third-party tools; no single AWS service performs every stage.

## Source, orchestration, and build
### Source and pipeline orchestration
* **CodeCommit** provides managed private Git repositories and IAM-controlled access.
  * Git users authenticate using configured SSH keys or HTTPS credentials; IAM policies authorize repository operations.
  * Repositories are encrypted at rest with KMS and in transit.
  * It integrates with CodeBuild and other CI tools.
* **CodePipeline** is the visual workflow orchestrator.
  * Source providers include CodeCommit, S3, ECR, GitHub/Bitbucket and others.
  * Build/test actions can use CodeBuild or external tools; deployment actions can use CodeDeploy, CloudFormation, ECS, Elastic Beanstalk, S3, and more.
  * It can also invoke Lambda or Step Functions.
* A pipeline contains stages, each with sequential and/or parallel actions. Manual approval can pause a stage.
  * Actions hand off artifacts; pipeline artifacts are stored in S3 and passed to downstream stages.
  * The pipeline service role needs permission for each configured provider and action.

### Building with CodeBuild
* **CodeBuild** is a managed, Docker-based build/test service that scales build environments without managing build servers.
  * It accepts supported source providers and prepackaged or custom build images.
  * Supported languages include Java, Ruby, Python, Go, Node.js, Android, .NET Core, and PHP.
* `buildspec.yml` normally lives at the source root.
  * Its `env` section can define plaintext variables or retrieve values from Systems Manager Parameter Store/Secrets Manager; avoid placing secrets in plaintext variables.
  * Phases include `install`, `pre_build`, `build`, and `post_build`.
  * `artifacts` selects output files, and `cache` can preserve dependencies in S3.
* CodeBuild logs can go to CloudWatch Logs or S3; metrics and EventBridge events support monitoring and notifications.
* CodeBuild charges for build compute time. IAM controls what the build can access; KMS can protect artifacts and a VPC configuration can provide access to private network resources.
* CodeBuild itself builds/tests; it does not orchestrate the entire pipeline or deploy releases.

## Release and rollback with CodeDeploy
* **CodeDeploy** automates application rollouts to EC2/on-premises servers, Lambda functions, and ECS services.
  * Deployment instructions are described by an `appspec.yml`; for Lambda, the AppSpec identifies the function version/alias and lifecycle hook functions.
  * EC2/on-premises hosts require the CodeDeploy agent, IAM access to retrieve bundles, and an agent-supported deployment setup.
* EC2/on-premises deployments can be **in-place** (update existing instances) or **blue/green** (provision a separate fleet, then switch traffic).
  * In-place speed can be all-at-once, half-at-a-time, one-at-a-time, or custom; faster rollouts can reduce availability during deployment.
  * Lifecycle hooks can run verification actions.
* For an Auto Scaling Group, in-place deployment updates current instances and newly launched instances receive the revision.
  * Blue/green creates a new group with copied settings.
  * An Elastic Load Balancer is required for traffic switching, and you can choose how long to retain the old fleet.
* **Lambda** deployments shift an alias between versions all-at-once, linearly (increase by a percentage every interval), or by canary (small initial share, then full traffic).
  * Pre/post-traffic Lambda hooks can validate the release, and CloudWatch alarms can initiate rollback.
* **ECS** CodeDeploy releases use blue/green with an ALB and two target groups. Traffic can shift all-at-once, linearly, or by canary between the blue/green task sets.
* A rollback deploys a previously successful application revision as a **new deployment**; it does not rewind history in place.
  * Rollback can be manual or triggered by deployment failure or CloudWatch alarm thresholds.
  * This distinction matters when investigating what revision is currently running.

## Packages, troubleshooting, and exam distinction
* **CodeArtifact** stores and distributes software dependencies/packages for tools such as Maven, Gradle, npm, pip, NuGet, and twine.
  * Developers and CodeBuild can retrieve dependencies from it.
  * Package-version changes can emit EventBridge events to trigger notifications or rebuild/redeploy pipelines.
  * Resource policies can authorize another account to read repositories.
* To diagnose a pipeline, inspect its stage/action execution state and provider logs.
  * An action that cannot run commonly indicates missing permissions in the pipeline service role or provider role.
  * EventBridge can notify on failed pipelines/cancelled stages; CloudTrail audits API activity.
  * Build failures belong in CodeBuild logs; traffic-shift or rollback failures belong in CodeDeploy deployment details.
* **Exam distinction:** CodePipeline coordinates; CodeBuild builds/tests; CodeDeploy controls release and rollback; CodeArtifact manages packages.
* Artifacts passed between pipeline stages are S3-backed, and a successful build is not the same as a deployed release.
