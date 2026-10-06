# AWS CloudOps Engineer - Associate (SOA-C03) Study Notes

This is a condensed study note for the AWS Certified CloudOps Engineer - Associate (SOA-C03) exam. It focuses on monitoring, logging, reliability, operational automation, security guardrails, and VPC networking.

For a deeper write-up and more detailed notes, see: https://www.nabilnoh.com/posts/aws-cloudops-engineer-soa-c03-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Monitoring, Logging, Analysis, Remediation, and Performance Optimization | 22% |
| Domain 2 | Reliability and Business Continuity | 22% |
| Domain 3 | Deployment, Provisioning, and Automation | 22% |
| Domain 4 | Security and Compliance | 16% |
| Domain 5 | Networking and Content Delivery | 18% |

The exam has 65 questions, including 50 scored questions and 15 unscored pretest questions. The time limit is 130 minutes, and the passing score is 720 out of 1000 on a scaled score.

## Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization

This domain focuses on observability, metric analysis, log protection, automated remediation, and performance tuning.

### CloudWatch metrics and the OS-level agent

- Standard CloudWatch EC2 metrics include hypervisor-level data such as CPU utilization, disk I/O, and network traffic.
- OS-level metrics such as memory utilization and disk-space usage require the Unified CloudWatch Agent.
- A system status check failure usually indicates an underlying host, hardware, or network issue. Stop and start the instance to move it to a healthy host.
- An instance status check failure usually indicates an operating system, kernel, memory, or software problem. Reboot the instance or inspect system console output.
- CloudWatch Logs data protection policies scan and redact sensitive information such as PII and PHI. They emit the `LogEventsWithFindings` metric, which can trigger a CloudWatch alarm.

### Automated remediation

- Use Amazon EventBridge rules to match events such as a CloudWatch alarm state change.
- Trigger AWS Systems Manager Automation runbooks directly from EventBridge when custom remediation code is unnecessary.
- Use Lambda provisioned concurrency to pre-warm execution environments and reduce cold-start latency caused by initialization code.
- Prefer managed remediation workflows when they provide the required action with less operational overhead than custom Lambda code.

## Domain 2: Reliability and Business Continuity

This domain covers high availability, disaster recovery, backup and replication, load balancing, and Auto Scaling behavior.

### Recovery objectives

- Recovery Time Objective (RTO): The maximum acceptable duration of downtime before service restoration.
- Recovery Point Objective (RPO): The maximum acceptable amount of data loss, measured in time.
- Lower RTO and RPO requirements generally require faster recovery mechanisms, more replication, and higher cost.

### S3 replication and backup

- S3 Cross-Region Replication requires versioning to be enabled on both the source and destination buckets.
- Configure the replication rule on the source bucket.
- Attach an IAM role with permissions to read from the source and write to the destination.
- Use AWS Backup and service-specific replication or snapshots to support recovery requirements and centralized backup policies.

### Auto Scaling and load balancing

- Auto Scaling lifecycle hooks place an instance into `Terminating:Wait` so cleanup, backup, or data-upload scripts can finish before termination.
- Call `complete-lifecycle-action` after the final action completes.
- Application Load Balancer (ALB): Layer 7 load balancing with HTTP/HTTPS path-based health checks such as `/health`.
- Network Load Balancer (NLB): Layer 4 load balancing for high-throughput TCP or UDP traffic with port-based health checks.
- Target tracking scaling adjusts capacity around a target metric such as average CPU utilization and is a cost-effective choice for unpredictable demand.

## Domain 3: Deployment, Provisioning, and Automation

This domain covers Systems Manager fleet operations, Infrastructure as Code, drift detection, and container deployments.

### AWS Systems Manager

| Feature | Primary operational purpose |
| :--- | :--- |
| Session Manager | Shell access without inbound SSH/RDP ports, public IP addresses, or bastion hosts. |
| Patch Manager | Automates OS patching with patch baselines and maintenance windows. |
| Run Command | Executes administrative commands or scripts across instances at scale. |
| State Manager | Continuously enforces software and configuration compliance. |
| Automation runbooks | Orchestrates multi-step, multi-account operational procedures, approvals, and remediation. |

Use Automation runbooks scheduled through State Manager associations when centralized patching, health checks, approvals, and remediation are required with minimal custom code.

### Infrastructure as Code and drift detection

- CloudFormation drift detection compares deployed resource properties with the template baseline and identifies manual out-of-band changes.
- Run drift detection on StackSets to inspect resources across accounts and Regions.
- CloudFormation templates should remain the source of truth; update the template rather than relying on manual changes.

### ECS deployments

- Rolling deployment gradually replaces old tasks with new revisions. It is cost-effective and maintains service availability without requiring an additional target group.
- Blue/green deployment uses CodeDeploy and a secondary target group to validate the new version before shifting traffic.

## Domain 4: Security and Compliance

This domain covers identity federation, governance boundaries, KMS permissions, and perimeter protections.

### Identity and governance

- IAM OIDC identity providers allow external CI/CD tools such as GitHub Actions or GitLab to use temporary AWS credentials through `AssumeRoleWithWebIdentity`.
- OIDC avoids storing long-lived IAM access keys in external pipeline systems.
- Service Control Policies (SCPs) apply at the AWS Organizations level and define the maximum permissions available to member accounts.
- An explicit deny in an SCP overrides identity and resource permissions and cannot be bypassed by an account administrator.

### Cross-account KMS access

Cross-account use of a KMS key requires permission in both places:

1. The calling identity's IAM policy
2. The KMS key policy in the account that owns the key

Adding `kms:Encrypt` only to the caller's IAM policy is insufficient if the key policy does not trust that caller or account.

### VPC Block Public Access

VPC Block Public Access in `ingress-only` mode blocks inbound internet access at the account or VPC boundary while preserving outbound connectivity initiated through NAT Gateways. It is a low-maintenance guardrail compared with modifying each subnet, security group, or route individually.

## Domain 5: Networking and Content Delivery

This domain covers VPC routing, private service connectivity, DNS verification, and network access controls.

### VPC endpoints

- Gateway VPC endpoints support Amazon S3 and DynamoDB.
- Gateway endpoints are free and keep traffic on the AWS network without hourly endpoint or per-GB processing charges.
- Interface VPC endpoints use AWS PrivateLink and attach elastic network interfaces to subnets. They support most other AWS services and incur hourly and data-processing charges.

### VPC peering and route tables

Creating and accepting a VPC peering connection does not automatically enable traffic. Add routes in both VPCs so each destination CIDR uses the `pcx-xxx` peering target.

The VPC CIDR blocks must not overlap, and security groups and network ACLs must also allow the traffic.

### Route 53 and mail verification

To authorize Amazon SES to send mail for a custom domain, publish the required SPF information in a Route 53 TXT record, such as:

`v=spf1 include:amazonses.com -all`

### Security groups and network ACLs

- Security groups are stateful, instance-level controls.
- Network ACLs are stateless, subnet-level controls and require explicit inbound and outbound rules.
- When diagnosing connectivity, check routes, security groups, network ACLs, endpoint policies, and DNS configuration.

## Scenario-based revision points

Before the exam, make sure you can identify the best solution for scenarios involving:

- CloudWatch Agent metrics for memory and disk space
- system status checks versus instance status checks
- CloudWatch Logs data protection and the `LogEventsWithFindings` metric
- EventBridge triggering Systems Manager Automation runbooks
- provisioned concurrency for Lambda initialization latency
- RTO and RPO in disaster recovery planning
- S3 replication versioning and IAM prerequisites
- Auto Scaling lifecycle hooks before instance termination
- target tracking policies for variable CPU demand
- Session Manager, Patch Manager, State Manager, and Run Command
- CloudFormation drift detection for StackSets
- rolling versus blue/green ECS deployments
- OIDC federation instead of static CI/CD credentials
- both IAM policy and KMS key policy for cross-account encryption
- SCP explicit denies and organizational guardrails
- VPC Block Public Access in `ingress-only` mode
- gateway versus interface VPC endpoints
- route table entries required on both sides of VPC peering
- ALB versus NLB health checks and traffic layers

## Final reminder

SOA-C03 requires moving beyond service recognition into practical infrastructure operations. Focus on Systems Manager tools, CloudWatch Agent metrics, recovery objectives, S3 replication prerequisites, Auto Scaling lifecycle hooks, CloudFormation drift detection, cross-account KMS policies, and VPC route tables and endpoints. In operational scenarios, prefer the managed AWS solution that satisfies the requirement with the least ongoing maintenance.
