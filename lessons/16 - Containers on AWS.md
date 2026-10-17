# Containers on AWS

## At a glance
- Docker images package applications and dependencies; ECR stores images, while ECS and EKS orchestrate workloads.
- Fargate supplies managed compute for ECS tasks or EKS pods without customer-managed hosts.
- ECS task definitions, IAM roles, networking, scaling, and placement determine how tasks run.
- EKS provides managed Kubernetes control planes and several worker-capacity options.
- Choose persistent storage, configuration, and deployment controls to match the workload.

## Images and AWS container services
- Docker packages an application and its dependencies as an image, which can run as a container on different hosts with more consistent behavior.
- Unlike a virtual machine, a container shares the host OS kernel, so multiple containers can use a host's resources without each carrying a separate guest OS.
- Containers suit microservices as well as moving existing applications to AWS.
- Images are built from a Dockerfile and stored in repositories:
  - Docker Hub is a common public registry.
  - Amazon ECR provides private repositories and a public gallery.
  - ECR stores and manages images; it is not a runtime or orchestrator.
  - ECR integrates with ECS, uses IAM for access, and supports image tags, scanning, and lifecycle policies.
  - If a host cannot pull an image, check its IAM permissions and registry authentication.

### Orchestrators and compute
| Service | Role |
| --- | --- |
| ECS | AWS's native container orchestrator. |
| EKS | AWS's managed Kubernetes service, useful when a team already operates Kubernetes or needs its ecosystem and API. |
| Fargate | Serverless compute that can run ECS tasks or EKS pods; it does not replace either orchestrator. |

- With EC2-backed workloads, you operate worker capacity; with Fargate, AWS operates the underlying hosts.

## ECS tasks, roles, and networking
### Clusters and task definitions
- An ECS cluster groups capacity and tasks.
- A task definition is JSON metadata describing how to run one or more containers, including image, CPU and memory, ports, environment, networking, IAM role, and logging.
- A task definition can specify up to 10 containers, enabling tightly coupled patterns such as an application plus a log/metrics sidecar.
- EC2 launch type requires EC2 instances registered to the cluster and running the ECS agent. You manage host provisioning and patching; ECS places and starts tasks.
- Fargate removes host management: specify the task's CPU and memory and ECS runs each task on managed infrastructure.

### IAM roles
- Distinguish the host instance profile from the task role.
- On EC2 launch type, the instance profile supports the ECS agent, for example, pulling ECR images or publishing logs.
- The task role is assigned in the task definition and grants the application its own AWS permissions.
- Use separate least-privilege task roles for different services.

### Load balancing and security groups
- ALB is the usual choice for ECS web traffic.
- With EC2 tasks:
  - Omitting a fixed host port allows dynamic host-port mapping, so several tasks can share an instance.
  - The ALB target group routes to each assigned port.
  - The instance security group must permit traffic from the ALB security group to those ports.
- With Fargate:
  - Each task has its own ENI/private IP.
  - The task security group allows the application port from the ALB.
- NLB is appropriate for high-throughput or performance needs and PrivateLink.
- Classic Load Balancer is legacy and is not supported for Fargate.

### Storage and configuration
- EFS mounts provide persistent shared files across tasks and Availability Zones and work with both EC2 and Fargate.
- Bind mounts share task-local ephemeral data between containers:
  - On EC2, their storage lifetime is tied to the host instance.
  - On Fargate, their storage lifetime is tied to the task.
- Fargate ephemeral storage defaults to 20 GiB and can be configured up to 200 GiB.
- Use an S3-backed file system where supported (Fargate and ECS Managed Instances, not traditional EC2 launch type) when container workloads need file-style access to S3 data.
- Put non-sensitive configuration in environment variables.
- Retrieve sensitive values from Systems Manager Parameter Store or Secrets Manager rather than hardcoding them; S3 environment files can supply bulk settings.

## ECS scaling, deployment, and placement
### Scaling and deployments
- ECS Service Auto Scaling changes the desired number of tasks, commonly using average CPU, memory, or ALB requests per target.
- Scaling policies include:
  - Target tracking, which follows a target metric value.
  - Step scaling, which responds to CloudWatch alarm steps.
  - Scheduled scaling, which handles predictable demand.
- Task scaling is separate from EC2 Auto Scaling:
  - EC2 launch type may also require an Auto Scaling group or an ASG-backed capacity provider to add hosts when cluster capacity is insufficient.
  - Fargate has no host fleet to scale.
- Rolling service deployments use minimum healthy and maximum task percentages to control how many tasks may stop and start during replacement.
- A more permissive maximum lets ECS start replacement tasks before stopping old ones, while a restrictive minimum can reduce capacity or slow the rollout.
- Choose settings that preserve the required availability and fit available capacity.
- EventBridge can run scheduled or event-triggered tasks and can match stopped-task events to notify an operator, for example through SNS.

### Placement on EC2-backed tasks
- ECS placement strategies and constraints apply to EC2-backed tasks, not Fargate.
- ECS first filters instances by CPU, memory, ports, and constraints.
- Strategies are best effort, not guarantees:
  - `binpack` packs tasks onto the least-free capacity to reduce the number of instances.
  - `spread` distributes tasks by a selected attribute such as Availability Zone or instance.
  - `random` chooses arbitrarily.
  - Strategies can be combined.
- `distinctInstance` places tasks on separate instances.
- `memberOf` limits placement using a Cluster Query Language expression.

## EKS
- EKS runs managed Kubernetes control planes; Kubernetes supplies the orchestration API, while ECS uses AWS's own task/service model.
- Choose EKS when portability or an existing Kubernetes operating model matters; choose ECS when AWS-native orchestration is the simpler fit.

### Worker capacity
- EKS worker capacity can use:
  - Managed node groups, where EKS creates and maintains EC2 nodes in an Auto Scaling group.
  - Self-managed nodes registered to the cluster.
  - Fargate pods with no nodes for the customer to maintain.
- EC2 nodes can use On-Demand or Spot capacity.

### Storage and observability
- EKS persistent storage uses Kubernetes StorageClass configuration and a CSI driver.
- Supported options include EBS, EFS (including Fargate use), FSx for Lustre, and FSx for NetApp ONTAP.
- Deploy a separate EKS cluster per AWS Region when workloads need regional deployments.
- CloudWatch Container Insights can collect cluster and workload logs and metrics.
