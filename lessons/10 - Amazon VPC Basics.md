# Amazon VPC – Basics

## VPCs and Subnets
* A VPC is a logically isolated network in one AWS Region. Subnets partition its IP space and each subnet belongs to one Availability Zone.
* Public or private describes the subnet's routes, not an automatic property of an instance. Route tables determine where subnet traffic can go.
* An Internet Gateway (IGW) attached to a VPC enables internet connectivity for subnets with an appropriate route and public addressing.
* A NAT Gateway lets instances in private subnets initiate outbound internet connections without accepting unsolicited inbound internet connections.

## Network Controls and Connectivity
* Security groups are stateful allow-lists attached to network interfaces or instances. Rules can refer to IP ranges or other security groups.
* Network ACLs apply at the subnet boundary. They are stateless, support allow and deny rules, and use IP ranges; return traffic needs its own rule.
* VPC Flow Logs record IP traffic metadata for VPCs, subnets, or network interfaces to help troubleshoot connectivity. Logs can be delivered to CloudWatch Logs, S3, or Kinesis Data Firehose.
* VPC peering privately connects two VPCs, but their CIDR ranges cannot overlap and peering is not transitive.
* VPC endpoints provide private paths from a VPC to supported AWS services. Gateway endpoints are for S3 and DynamoDB; interface endpoints use network interfaces and support many other services.
* Site-to-Site VPN connects networks over an encrypted connection through the public internet. Direct Connect provides a dedicated private network connection and usually takes longer to provision.

## Architecture Context
* A common multi-AZ design places load balancers and application instances in public subnets, and databases and other data services in private subnets.
* A typical three-tier application uses Route 53 and a load balancer at the entry point, an Auto Scaling application tier, and RDS plus ElastiCache in the data tier.
* For the Developer exam, understand the purpose and scope of VPC components and how they affect application connectivity.
