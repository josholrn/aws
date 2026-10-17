# AWS Elastic Beanstalk

## At a glance
- Elastic Beanstalk deploys applications while provisioning and managing common AWS infrastructure.
- Applications group environments, configurations, and immutable application versions.
- Deployment strategies trade rollout speed, capacity, availability, and rollback behavior.
- The EB CLI, `.ebextensions/`, and CloudFormation support automation and configuration.
- Environment, load balancer, database, and version lifecycles have important operational caveats.

## Managed application deployment
- Elastic Beanstalk offers a developer-focused way to deploy applications on AWS without manually coordinating common infrastructure.
- It provisions and manages familiar services such as EC2, Auto Scaling groups, load balancers, and optionally RDS.
- Beanstalk handles capacity, health monitoring, instance configuration, scaling, and deployment coordination; developers can still configure the environment and retain control of its underlying resources.
- The Beanstalk service itself has no additional charge, but the AWS resources it provisions are billed normally.
- Choose a supported platform for the application, for example, Java, Node.js, Python, .NET, Go, PHP, Ruby, or Docker.
- Upload a source bundle, commonly a ZIP containing the application and dependency manifest.

### Applications and environments
- An application groups its environments, configurations, and application versions.
- An application version is an immutable packaged code revision.
- Each environment runs one version at a time, allowing independent dev, test, and production environments.
- Web server environments accept application traffic and commonly use an ALB with an Auto Scaling group.
- Worker environments poll an SQS queue and process background jobs; they can scale based on queue depth.
- A web tier can enqueue work so request-serving capacity is not tied to long-running background processing.
- A single-instance environment is useful for development and testing.
- A load-balanced, multi-instance environment across Availability Zones is the normal production shape when high availability is required.

## Deployment strategies
| Strategy | How it deploys | Capacity, availability, and rollback considerations |
| --- | --- | --- |
| **All at once** | Updates all instances together. | Fastest and requires no extra capacity, but the application is unavailable during replacement; best reserved for non-critical development deployments. |
| **Rolling** | Replaces instances in batches. | The application can temporarily run below full capacity and both old and new versions may serve traffic during rollout. Batch size controls the capacity dip and duration; there is no extra instance cost. |
| **Rolling with additional batches** | Launches temporary extra instances to receive updated batches, keeping the original fleet at capacity. | Costs more during rollout and takes longer, then removes the extra capacity. |
| **Immutable** | Launches a temporary Auto Scaling group, deploys the new version there, and switches only when the new instances are healthy. | Avoids downtime and makes rollback quick by discarding the temporary group, but temporarily needs close to double capacity and is the longest/most resource-intensive strategy. |
| **Traffic splitting** | Deploys the new version to a temporary fleet and sends it a small configurable percentage of requests for a period. | Health is monitored, and a failed deployment can trigger automatic rollback; after success, the new instances are moved into the main fleet. This is a canary-style release, not the same as merely replacing all instances at once. |

### Blue/green environment pattern
- Blue/green is an environment-level pattern, not a direct Beanstalk deployment mode.
- Create a separate environment and validate it.
- Optionally send a small share of traffic through Route 53 weighted routing, then switch environment URLs or DNS.
- Keeping the old environment available supports rollback.

## Configuration and lifecycle
### CLI and source configuration
- The EB CLI streamlines automation with commands such as `eb create`, `eb status`, `eb health`, `eb events`, `eb logs`, `eb deploy`, `eb config`, `eb open`, and `eb terminate`.
- Uploading a new ZIP creates an application version; deployment applies it to an environment.
- `.ebextensions/` at the source bundle root contains YAML or JSON `.config` files.
- `option_settings` can encode settings normally configured in the console.
- The files can declare extra CloudFormation resources such as ElastiCache or S3.
- Resources provisioned this way are environment-bound and may be deleted when the environment is terminated.

### Environments, load balancers, and databases
- Beanstalk uses CloudFormation underneath.
- Cloning an environment copies configuration, including environment variables, load balancer setup, and database type, but does not clone RDS data.
- Cloning is useful for creating a similarly configured test environment.
- The load balancer type cannot be changed in-place after environment creation.
- To migrate, create a new environment with the desired load balancer, deploy and validate the application, then switch using a CNAME/URL swap or Route 53 update.
- An RDS database created as part of a Beanstalk environment is convenient for development, but its lifecycle becomes coupled to the environment and termination can endanger the database.
- In production, create RDS independently and pass its connection information to the application.
- Before decoupling an existing setup, take a snapshot and protect the database.

### Application version limits
- An application can hold at most 1,000 versions.
- Configure a lifecycle policy to remove old versions by age or count; versions currently deployed are retained.
- Choose whether to remove their S3 source bundles as well, since retaining the bundles can be important for recovery.
