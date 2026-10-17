# AWS Monitoring, Troubleshooting, and Audit

## At a glance
- CloudWatch metrics and alarms monitor measurements and trigger threshold-based actions.
- CloudWatch Logs, Insights, and subscription filters support log storage, investigation, and near-real-time processing.
- EventBridge routes discrete events, while X-Ray traces requests across application components.
- CloudTrail records API activity for resource-change audits and can detect unusual management activity.
- Select the signal—metrics, logs, traces, events, or audit records—that answers the operational question.

## CloudWatch metrics and alarms
- CloudWatch provides metrics, logs, alarms, and event integrations for operational monitoring.
- Metrics are timestamped measurements grouped by namespace; dimensions identify a particular metric variant, such as an EC2 instance ID or environment.
- A metric can have up to 30 dimensions, and dashboards can combine service metrics for operational views.
- Standard EC2 metrics are published every five minutes.
- Detailed monitoring (an additional-cost option) provides one-minute datapoints and can help an Auto Scaling group react faster.
- EC2 does not publish memory utilization by default; install the CloudWatch agent or publish a custom metric for memory, disk space, or application-specific measurements.
- Custom metrics are sent with `PutMetricData` and can use dimensions to segment values.
- Standard-resolution datapoints have one-minute granularity; high-resolution values can be published at 1-, 5-, 10-, or 30-second intervals at higher cost.
- The API accepts data points up to two weeks in the past and two hours into the future, so correct host time matters.
- CloudWatch alarms evaluate metrics against thresholds and can notify or initiate actions, such as sending to SNS or supporting Auto Scaling.
- A composite alarm combines the states of multiple alarms, useful when an action should depend on a broader condition rather than a single metric.
- EventBridge complements alarms by routing discrete AWS events.

## CloudWatch Logs
### Storage and collection
- A log group usually represents an application or service; log streams contain records from a particular instance, file, container, or function.
- Set retention deliberately: groups may otherwise retain data indefinitely, while configured expiration can range from one day to ten years.
- Logs are encrypted by default and can use a customer-managed KMS key.
- AWS services such as Lambda, ECS, Elastic Beanstalk, API Gateway, CloudTrail, and VPC Flow Logs can publish logs.
- EC2 and on-premises hosts do not send operating-system/application log files automatically; install and configure the CloudWatch agent and grant it the needed IAM permissions.
- The older CloudWatch Logs agent only handles logs; the unified CloudWatch agent can also publish system metrics such as memory.

### Investigation and delivery
- CloudWatch Logs Insights is an interactive query engine for stored logs.
- It discovers fields in supported service and JSON records, then filters, sorts, aggregates, and counts events.
- Queries can be saved and added to dashboards.
- Logs Insights is for analysis, not real-time processing.
- Exporting a log group to S3 can take up to 12 hours to become available, so it is not a near-real-time delivery path.
- Use a subscription filter to stream matching log events to Lambda, Kinesis Data Streams, or Firehose.
- Subscriptions support near-real-time processing and can forward logs across accounts when the destination access policy and IAM role allow it.
- Firehose is a managed path onward to destinations such as S3.

## Event routing and tracing
### EventBridge
- EventBridge (formerly CloudWatch Events) routes matching service, API, or custom events to targets such as Lambda, SNS, and SQS.
- Rules can respond to state changes or CloudTrail-recorded API activity—for example, alert when a security group ingress rule is changed or a table is deleted.
- Use event patterns for discrete events; use a CloudWatch alarm when a metric crossing a threshold is the signal.

### AWS X-Ray
- AWS X-Ray traces an end-to-end application request across services, helping locate latency bottlenecks, errors, exceptions, throttling, and affected callers.
- It supports Lambda, Elastic Beanstalk, ECS, ELB, API Gateway, EC2-hosted applications, and on-premises applications.
- Instrument application code with the X-Ray SDK.
- It can capture downstream AWS service, HTTP(S), database, and queue calls.
- The application or daemon needs IAM permission to publish trace data.
- EC2 deployments commonly run the X-Ray daemon, while supported managed services integrate with it.
- For Lambda, enable active tracing and ensure the execution role can write to X-Ray.
- Beanstalk platforms include the daemon, but the multi-container Docker platform does not provide it by default.

### Trace data
- A segment describes work by a service; subsegments add detail.
- Segments from components are combined into a trace and service map.
- Annotations are indexed key/value fields suitable for filtering; metadata is attached but is not indexed for search.
- Sampling controls volume and cost:
  - The default SDK rule records the first request each second.
  - It samples five percent of additional requests.
- Sampling rules can be changed without application-code changes.

## CloudTrail audit
### Event collection and retention
- CloudTrail records account activity and API calls made through the console, CLI, SDK, or AWS services.
- Event history for management events is available by default.
- Configure a trail to deliver records to S3 or CloudWatch Logs for ongoing storage and analysis.
- CloudTrail is the primary place to investigate who changed or deleted an AWS resource and when.
- A trail can cover all Regions (the usual choice) or one Region.
- Management events record control-plane operations such as changing IAM permissions, creating a subnet, or configuring a trail.
- Trails log management events by default; read and write management events can be selected separately.
- Data events record high-volume operations on resources, such as S3 object reads/writes or Lambda invocations, and are not logged by default.
- Enable the specific data-event selectors needed for auditing.
- Event history provides 90 days of management-event history.
- For longer retention and analysis, configure a trail to store events in S3 and query the data with Athena; deliver to CloudWatch Logs when operational search or alerting is also needed.

### CloudTrail Insights
- CloudTrail Insights analyzes management activity to establish a baseline and detect unusual write patterns, such as bursts of IAM actions, unexpected provisioning, or activity suggestive of service-limit issues.
- Insight findings appear in CloudTrail, can be delivered to S3, and produce EventBridge events for automation.

## Choosing the right signal
| Need | Use |
| --- | --- |
| Service health, utilization, and threshold-based actions | CloudWatch metrics and alarms |
| Application log storage and investigation | CloudWatch Logs and Insights |
| Near-real-time log processing | Subscription filters |
| Following a slow or failing request across application components | X-Ray |
| Auditing API activity and resource changes | CloudTrail |

- For example:
  - Use CloudTrail to identify the principal that changed a security group.
  - Use EventBridge to alert or automate a response to that API event.
  - Use CloudWatch to detect a rise in application errors or latency.
  - Use X-Ray to locate which downstream call is responsible for a slow request.
