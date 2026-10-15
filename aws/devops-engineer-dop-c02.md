# AWS DevOps Engineer - Professional (DOP-C02) Study Notes

This is a condensed study note for the AWS Certified DevOps Engineer - Professional (DOP-C02) exam. It focuses on enterprise CI/CD, multi-account Infrastructure as Code, resilient architectures, centralized observability, automated incident response, and security governance.

For a deeper write-up and more detailed notes, see: https://www.nabilnoh.com/posts/aws-devops-engineer-dop-c02-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | SDLC Automation | 22% |
| Domain 2 | Configuration Management and Infrastructure as Code | 17% |
| Domain 3 | Resilient Cloud Solutions | 15% |
| Domain 4 | Monitoring and Logging | 15% |
| Domain 5 | Incident and Event Response | 14% |
| Domain 6 | Security and Compliance | 17% |

The exam has 75 questions, including 65 scored questions and 10 unscored pretest questions. The time limit is 180 minutes, and the passing score is 750 out of 1000 on a scaled score.

## Domain 1: SDLC Automation

This domain covers automated CI/CD pipelines across single-account and multi-account AWS environments.

### Centralized multi-account CI/CD

A common enterprise pattern places CodePipeline and CodeBuild in a central Tooling or DevOps account and deploys into separate development, staging, and production accounts.

- Central S3 artifact buckets require `s3:GetObject` and `s3:PutObject` permissions for target-account execution roles.
- Default AWS-managed S3 keys such as `aws/s3` cannot be shared across accounts. Use a customer managed KMS key with a key policy allowing target roles to perform actions such as `kms:Decrypt` and `kms:GenerateDataKey`.
- Target accounts use deployment roles that trust the tooling account pipeline role through `sts:AssumeRole`.
- CodePipeline execution modes:
  - `QUEUED`: Runs executions sequentially in order.
  - `SUPERSEDED`: Replaces waiting executions with the newest commit.
  - `PARALLEL`: Runs multiple executions concurrently in isolated environments.

### Secrets and automated testing

- AWS Secrets Manager supports dynamic and scheduled rotation and resource-based policies.
- Systems Manager Parameter Store stores configuration and `SecureString` values through identity-based policies.
- CodeBuild can retrieve values through `env.secrets-manager` and `env.parameter-store` in `buildspec.yml`; retrieved values are masked in build logs.
- Pre-build checks include SAST, linting, syntax validation, and dependency checks.
- Build stages should run unit tests and code coverage checks and fail when test commands return a non-zero exit code.
- Staging stages should run integration, acceptance, load, and stress tests before production promotion.

### Artifact and image management

- AWS CodeArtifact centrally manages npm, PyPI, Maven, and NuGet dependencies. Upstream connections cache public packages internally.
- CodeArtifact authorization tokens are temporary and can be retrieved in a build with `aws codeartifact get-authorization-token`.
- Amazon ECR supports basic and enhanced image vulnerability scanning and cross-account or cross-Region image replication.
- EC2 Image Builder automates golden AMI creation using recipes, components, infrastructure configurations, and distribution settings.

### Deployment strategies

| Strategy | Risk and behavior | Common targets |
| :--- | :--- | :--- |
| In-place / rolling | Fast, medium risk; redeploy the previous revision to roll back. | EC2 Auto Scaling, Elastic Beanstalk |
| Blue/green | Validate a separate environment, then switch traffic. | EC2, ECS, Lambda, Elastic Beanstalk |
| Canary | Shift a small percentage, monitor alarms, then continue. | Lambda, ECS, AppConfig |
| Linear | Increase traffic in equal increments over time. | Lambda, ECS |

CodeDeploy lifecycle hooks vary by target:

- EC2 and on-premises: `ApplicationStop`, `BeforeInstall`, `AfterInstall`, `ApplicationStart`, and `ValidateService`.
- ECS: hooks include `AfterAllowTestTraffic`, `BeforeAllowTraffic`, and `AfterAllowTraffic`.
- Lambda: `BeforeAllowTraffic` and `AfterAllowTraffic`.

Use CloudWatch alarms with CodeDeploy deployment groups to trigger automatic rollback when error or latency thresholds are exceeded.

## Domain 2: Configuration Management and Infrastructure as Code

This domain covers declarative infrastructure, multi-account landing zones, governance, and fleet configuration.

### IaC frameworks and safeguards

- AWS CloudFormation uses declarative JSON or YAML templates for AWS resources.
- AWS CDK expresses infrastructure in languages such as TypeScript, Python, Java, and Go, then synthesizes CloudFormation templates.
- AWS SAM extends CloudFormation for serverless functions, APIs, and databases.
- CloudFormation stack policies protect critical resources during stack updates. They do not protect against deletion of the entire stack.
- Use IAM permissions, termination protection, or `DeletionPolicy: Retain` when protecting resources from stack deletion.
- Drift detection identifies changes made outside the template but does not automatically restore the resource.
- AWS Service Catalog publishes approved templates as products in portfolios for controlled self-service deployment.

### Multi-account onboarding and guardrails

- AWS Control Tower and AWS Organizations establish accounts, OUs, and landing-zone governance.
- Service Control Policies define the maximum permissions available in an account or OU. An explicit SCP deny overrides IAM permissions, including permissions held by the account root user.
- Service-managed CloudFormation StackSets deploy stack instances across multiple accounts and Regions and can automatically deploy to new accounts in a targeted OU.

### Fleet management

- Patch Manager applies approved patches using patch baselines and maintenance windows.
- State Manager continuously enforces software, agent, and configuration state.
- Systems Manager Automation runs operational procedures such as resizing, snapshot management, and security remediation.
- AWS AppConfig manages feature flags and runtime configuration separately from code deployments.

## Domain 3: Resilient Cloud Solutions

This domain focuses on highly available, scalable, self-healing, multi-AZ, and multi-Region designs with defined RTO and RPO targets.

### Multi-Region resilience

- Remove single points of failure across global routing, regional load balancing, compute, and data tiers.
- Amazon Aurora Global Database uses storage-level replication across Regions with typically sub-second replication latency. It supports very low RPO and rapid planned or unplanned failover.
- DynamoDB Global Tables provide fully managed active-active multi-Region replication and use last-writer-wins conflict resolution.
- Route 53 Application Recovery Controller (ARC) provides readiness checks and routing controls for regional failover. Safety rules help prevent unsafe or cascading failovers.

### Scaling engines

- EC2 Auto Scaling supports target tracking, step scaling, and predictive scaling.
- ECS capacity providers and Fargate align compute capacity with task demand using metrics such as `CapacityProviderReservation`.
- EKS uses tools such as Cluster Autoscaler and Karpenter for node and workload scaling.

### Disaster recovery strategies

| Strategy | Typical RTO | Typical RPO | Cost |
| :--- | :--- | :--- | :--- |
| Backup and restore | Hours | Hours | Lowest |
| Pilot light | Minutes | Minutes | Low |
| Warm standby | Seconds to minutes | Seconds | Medium |
| Multi-site active-active | Near zero | Sub-second | Highest |

- AWS Elastic Disaster Recovery provides continuous block-level replication into a low-cost staging area for rapid recovery.
- AWS Backup Vault Lock in compliance mode prevents users, including the root account, from deleting backups or shortening retention periods.
- Choose the least expensive strategy that satisfies the required RTO and RPO.

## Domain 4: Monitoring and Logging

This domain covers centralized metrics and logs, operational analysis, dashboards, and distributed tracing.

### CloudWatch Agent and metrics

Hypervisors provide host-level metrics such as CPU, network I/O, and disk operations. Memory usage, swap usage, and filesystem space require the CloudWatch Agent installed on the guest operating system.

The agent can also collect application and OS logs such as `/var/log/syslog` and web server access logs.

### Centralized cross-account logging

- Use CloudWatch Logs subscription filters to stream logs from source accounts.
- Cross-account subscription destinations commonly use Kinesis Data Streams or Kinesis Data Firehose in a central security or compliance account.
- Firehose can deliver logs to S3, OpenSearch, or other destinations.
- Cross-account S3 buckets and Firehose streams should use a customer managed KMS key. Default AWS-managed keys cannot be shared across accounts.

### Analytics tool selection

| Tool | Best fit |
| :--- | :--- |
| CloudWatch Logs Insights | Interactive queries against live CloudWatch log groups. |
| Amazon Athena | SQL queries against historical JSON, CSV, or Parquet files in S3. |
| Amazon OpenSearch Service | Real-time indexing, full-text search, dashboards, and log aggregation. |
| CloudWatch cross-account observability | Query source-account CloudWatch logs centrally without exporting them. |

### AWS X-Ray

- Active instrumentation samples requests, injects tracing headers, and creates service-map nodes. Lambda and API Gateway are common examples.
- Passive instrumentation preserves upstream trace IDs through asynchronous services such as EventBridge, SQS, and SNS.
- Annotations are indexed and searchable.
- Metadata is non-indexed diagnostic context, such as complete JSON payloads.
- Use service maps, traces, segments, and subsegments to identify latency and downstream failures.

## Domain 5: Incident and Event Response

This domain covers event ingestion, automated remediation, orchestration, and failure diagnostics.

### Event routing and orchestration

- AWS Health, CloudTrail, S3, and other services emit events to Amazon EventBridge.
- EventBridge rules match event patterns and target Lambda, SNS, Systems Manager Automation, or Step Functions.
- SNS and EventBridge can fan out events to multiple SQS queues or Lambda functions for independent processing.
- Step Functions orchestrates multi-step recovery workflows with retry and catch handling.

### High-value remediation patterns

For an exposed IAM key such as `AWS_RISK_CREDENTIALS_EXPOSED`:

1. AWS Health sends the event to EventBridge.
2. EventBridge starts a Step Functions state machine.
3. The workflow revokes or deletes the exposed key.
4. Lambda queries CloudTrail for recent activity.
5. Step Functions publishes a report to an SNS topic for security review.

For a non-compliant security group, AWS Config or CloudTrail can trigger EventBridge, which invokes an SSM Automation document to revoke unauthorized ingress such as `0.0.0.0/0` on port 22.

### Troubleshooting and diagnostics

- Use an Auto Scaling lifecycle hook to place an unhealthy instance in `Terminating:Wait` while SSM Run Command uploads logs or memory dumps to S3.
- ECS Fargate sends container `stdout` and `stderr` to CloudWatch Logs with the `awslogs` driver. Inspect ECS service events for placement failures.
- EKS Container Insights and Embedded Metric Format help identify `CrashLoopBackOff`, `OOMKilled`, and abnormal resource usage.
- CloudWatch Synthetics canaries continuously test APIs and user journeys and can detect frontend or endpoint regressions.

## Domain 6: Security and Compliance

This domain covers enterprise identity, preventive and detective controls, data protection, and continuous compliance.

### Identity at scale

- Human users should use IAM Identity Center with SAML 2.0 or SCIM federation instead of long-term IAM access keys.
- EC2, ECS, and Lambda workloads should use IAM roles, instance profiles, and task execution roles.
- On-premises workloads can use IAM Roles Anywhere with X.509 certificates for temporary STS credentials.
- IAM permissions boundaries define the maximum permissions an IAM user or role can receive and help delegate role creation without allowing privilege escalation.
- SCPs are organization-level preventive guardrails. Explicit denies override IAM permissions.

### Automated security controls

- AWS Firewall Manager centrally deploys WAF, Shield Advanced, VPC security group, and Network Firewall policies across organization accounts.
- Amazon Macie discovers and classifies sensitive data in S3.
- AWS Secrets Manager supports automatic rotation for database credentials and other secrets.
- A common response flow is AWS Config or Security Hub finding -> EventBridge -> Systems Manager Automation -> SNS notification.

### Security monitoring matrix

| Service | Primary purpose |
| :--- | :--- |
| AWS CloudTrail | Records management and data API activity; enable log file integrity validation. |
| AWS Config | Records resource configuration and evaluates compliance; can trigger remediation. |
| Amazon GuardDuty | Detects threats using CloudTrail, VPC Flow Logs, DNS, EKS, and S3 signals. |
| Amazon Inspector | Scans EC2, ECR images, and Lambda for vulnerabilities and exposure. |
| AWS Security Hub | Aggregates findings and checks security standards such as CIS benchmarks. |
| IAM Access Analyzer | Identifies public or cross-account resource sharing using policy analysis. |

For cross-account S3 encryption, require SSE-KMS in the bucket policy and use a customer managed KMS key whose key policy grants target account roles `kms:Decrypt`.

For Lambda code authenticity, use AWS Signer with a Lambda Code Signing Configuration and deploy signed packages through tools such as SAM. Encryption alone does not prove code provenance.

## Quick revision checklist

Before the exam, make sure you can explain:

- cross-account CodePipeline roles, artifact buckets, and customer managed KMS keys
- CodePipeline execution modes and CodeDeploy traffic-shifting strategies
- CodeArtifact, ECR scanning and replication, and EC2 Image Builder
- CloudFormation stack policies, deletion protection, and drift detection
- Control Tower, Organizations, SCPs, and service-managed StackSets
- Aurora Global Database, DynamoDB Global Tables, Route 53 ARC, and DR strategies
- target tracking, predictive scaling, ECS capacity providers, and EKS autoscalers
- CloudWatch Agent, centralized cross-account logs, Athena, OpenSearch, and Logs Insights
- X-Ray active versus passive instrumentation and annotations versus metadata
- EventBridge, Step Functions, Systems Manager Automation, and automated remediation
- IAM Identity Center, Roles Anywhere, permissions boundaries, and SCPs
- Firewall Manager, Macie, GuardDuty, Inspector, Security Hub, and Access Analyzer
- why customer managed KMS keys are required for many cross-account encryption patterns

## Final reminder

DOP-C02 requires connecting delivery automation with reliability, operations, and security decisions at enterprise scale. Focus on cross-account deployment patterns, customer managed KMS keys, CloudFormation governance, multi-Region recovery, centralized observability, automated remediation, and the distinction between SCPs, permissions boundaries, and detective controls.
