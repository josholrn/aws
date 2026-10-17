# Containers on AWS

## Container Basics
* Docker packages an application and its dependencies into a portable image. Images are stored in repositories such as Docker Hub or Amazon ECR; ECR supports private images and a public gallery.
* ECS is AWS's container orchestrator. In EC2 launch mode, you manage the EC2 capacity and ECS agent; in Fargate, AWS runs tasks on serverless infrastructure sized by requested CPU and memory.
* EKS is AWS-managed Kubernetes. Fargate can also run EKS pods. ECR stores images but does not run them.

## ECS Configuration and Scaling
* A task definition describes images, CPU/memory, ports, networking, environment, IAM role, and logging. A task role grants application permissions; the EC2 instance profile is for the host/agent. Keep task roles narrowly scoped.
* ALB is the usual load balancer for ECS. EC2 tasks may use dynamic host-port mapping; Fargate tasks have their own network interfaces and private IPs. NLB fits high-throughput or PrivateLink cases.
* ECS service auto scaling adjusts task count using metrics such as CPU, memory, or ALB requests. EC2 launch mode may separately need an Auto Scaling group or capacity provider to add hosts.
* EFS provides persistent shared file storage across tasks and works with EC2 and Fargate. Bind mounts share ephemeral data within a task; their lifetime depends on the underlying launch type.
* ECS placement strategies apply to EC2 tasks: binpack reduces unused host capacity, spread distributes tasks, and random selects arbitrarily. Placement constraints restrict eligible hosts.

## Operations
* Rolling deployments tune minimum healthy and maximum task percentages to balance availability and rollout speed. EventBridge can start scheduled or event-driven tasks, and stopped-task events can trigger alerts.
