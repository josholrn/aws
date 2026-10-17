# AWS Elastic Beanstalk

## Managed Application Deployment
* Elastic Beanstalk simplifies application deployment while provisioning familiar AWS resources such as EC2, Auto Scaling, and load balancers. AWS manages capacity, health monitoring, and deployment coordination; developers retain configuration control and pay for the underlying resources.
* An application groups versions and environments. An application version is a packaged code revision; each environment runs one version. Web environments serve requests, while worker environments consume SQS messages and can scale based on queue depth.
* Single-instance environments suit development; load-balanced, multi-instance environments are typical for production.

## Deployment Strategies
* All-at-once is quickest but causes downtime. Rolling updates deploy in batches but temporarily reduce capacity. Rolling with additional batches launches spare instances to preserve capacity.
* Immutable deployment launches a temporary Auto Scaling group and swaps only after the new instances are healthy. It costs more during deployment but supports fast rollback.
* Blue/green uses a separate environment and switches traffic after validation; it is an environment-level approach rather than a direct deployment mode. Traffic splitting sends a small share to the new version for canary validation and can roll back on health failure.

## Configuration and Lifecycle
* The EB CLI supports environment creation, status/health checks, logs, configuration, deployment, and termination—useful in automation.
* `.ebextensions` files in the source bundle configure options and can define CloudFormation resources. Resources tied to the environment may be removed when it is terminated.
* Beanstalk uses CloudFormation underneath. Cloning copies configuration but not database contents. For production, decouple RDS from the environment lifecycle; changing load balancer type also requires creating a new environment and switching the application endpoint.
* A lifecycle policy removes old application versions to stay within the 1,000-version limit; versions currently in use are retained.
