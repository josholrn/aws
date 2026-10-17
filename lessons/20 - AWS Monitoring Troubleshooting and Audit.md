# AWS Monitoring, Troubleshooting, and Audit

## CloudWatch
* CloudWatch metrics track service measurements in namespaces; dimensions identify metric variants such as an instance or environment. Standard EC2 metrics arrive every five minutes; detailed monitoring provides one-minute data. Memory is not published by default—use an agent or custom metric.
* CloudWatch Logs organizes log groups and streams and supports retention policies and exports. Logs Insights queries and aggregates stored events but is not a real-time processor. For near-real-time delivery, use subscription filters to send matching logs to Lambda, Kinesis Data Streams, or Firehose.
* The unified CloudWatch agent gathers logs and system metrics such as memory from EC2 or on-premises servers. CloudWatch alarms react to metric thresholds; composite alarms combine alarm states. EventBridge routes matching service events to targets such as Lambda or SNS.

## X-Ray and CloudTrail
* X-Ray traces requests across application components to reveal latency and errors. Instrument code with the X-Ray SDK and run a daemon/agent or use an integrated service. Segments describe components; annotations are indexed for filtering, while metadata is not. Sampling controls tracing volume.
* CloudTrail records AWS API activity for auditing who changed resources and when. Event history retains 90 days of management events; trails can deliver logs to S3 and CloudWatch Logs for longer retention and analysis. CloudTrail Insights can identify unusual API activity patterns.
* Use CloudWatch for operational metrics and logs, X-Ray for application request tracing, and CloudTrail for account/API auditing. When investigating a resource change or deletion, start with CloudTrail; use X-Ray to follow a slow or failing request.
