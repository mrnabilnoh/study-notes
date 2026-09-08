# AWS Cloud Practitioner (CLF-C02) Study Notes

This is a condensed study note for the AWS Certified Cloud Practitioner (CLF-C02) exam. It focuses on the core cloud concepts, AWS services, security principles, pricing models, and support tools you need to understand and review.

For a deeper write-up and more detailed notes, see: https://www.nabilnoh.com/posts/aws-cloud-practitioner-clf-c02-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Cloud Concepts | 24% |
| Domain 2 | Security and Compliance | 30% |
| Domain 3 | Cloud Technology and Services | 34% |
| Domain 4 | Billing, Pricing, and Support | 12% |

The exam has approximately 65 questions, including 50 scored questions and 15 unscored questions. The time limit is 90 minutes, and the passing score is 700 out of 1000 on a scaled score.

## Domain 1: Cloud Concepts

### Cloud value proposition and economics

- Agility: Provision IT infrastructure within minutes instead of waiting for hardware procurement.
- Elasticity: Automatically scale capacity up or down based on demand.
- Global reach: Deploy applications across AWS Regions and serve users from locations close to them.
- Variable expenses: Exchange upfront capital expenses (CapEx) for operational expenses (OpEx) based on actual consumption.
- Economies of scale: AWS's operating scale helps provide lower pay-as-you-go pricing.

### AWS Cloud Adoption Framework and migration strategies

The six perspectives of the AWS Cloud Adoption Framework (AWS CAF) are:

- Business
- People
- Governance
- Platform
- Security
- Operations

The 7 Rs of cloud migration describe common migration strategies:

| Strategy | Meaning |
| :--- | :--- |
| Rehost | Move an application without changing its core architecture, often called lift and shift. |
| Replatform | Make minor cloud optimizations without changing the application's core code. |
| Repurchase | Replace the existing solution with a different product or SaaS offering. |
| Refactor / Re-architect | Redesign an application to use cloud-native patterns such as serverless services or microservices. |
| Relocate | Move workloads to another platform, such as VMware Cloud on AWS. |
| Retire | Decommission applications that are no longer needed. |
| Retain | Keep an application in its current environment when migration is not appropriate yet. |

### AWS Well-Architected Framework

The six pillars are:

1. Operational Excellence: Run and monitor systems to deliver business value and continuously improve processes.
2. Security: Protect information, assets, and systems through risk assessment and mitigation.
3. Reliability: Ensure a workload performs its intended function correctly and consistently.
4. Performance Efficiency: Use computing resources efficiently to meet system requirements.
5. Cost Optimization: Run systems at the lowest price point without sacrificing performance.
6. Sustainability: Minimize the environmental and carbon impact of cloud workloads.

## Domain 2: Security and Compliance

### AWS Shared Responsibility Model

- Security of the Cloud: AWS protects the underlying infrastructure, physical facilities, data center hardware, networking, and hypervisors.
- Security in the Cloud: Customers configure IAM permissions, encrypt data, patch guest operating systems on services such as EC2, manage security groups, and secure their applications.

Responsibilities vary by service. For example, AWS manages the underlying operating system and hardware for Amazon RDS, while the customer manages database users, schemas, and network access controls.

### AWS security, governance, and compliance services

| Service | Purpose |
| :--- | :--- |
| AWS IAM | Manages identities and access. Apply least-privilege permissions. |
| AWS WAF | Filters HTTP and HTTPS traffic to protect web applications from attacks such as SQL injection and cross-site scripting. |
| AWS Shield | Provides Standard and Advanced protection against distributed denial-of-service (DDoS) attacks. |
| Amazon GuardDuty | Detects threats using signals such as CloudTrail events, VPC Flow Logs, and DNS logs. |
| Amazon Inspector | Scans EC2 instances and container images for software vulnerabilities. |
| Amazon Macie | Discovers, classifies, and helps protect sensitive data stored in Amazon S3. |
| AWS Artifact | Provides official AWS compliance reports and security attestations. |

### Important security practices

- Enable Multi-Factor Authentication (MFA) on the root user immediately after creating an AWS account.
- Do not use the root user for everyday tasks.
- Apply least-privilege permissions with IAM.
- Encrypt data in transit and at rest.
- Use security groups and network controls to restrict access.

## Domain 3: Cloud Technology and Services

### AWS global infrastructure

- AWS Regions: Geographic areas that contain multiple physically isolated Availability Zones.
- Availability Zones (AZs): One or more discrete data centers with redundant power, networking, and connectivity. AZs are designed to avoid shared single points of failure.
- Edge locations: Points of Presence used by Amazon CloudFront to cache content close to end users and reduce latency.

### Core AWS service matrix

| Category | Services and use cases |
| :--- | :--- |
| Compute | Amazon EC2 for virtual machines, AWS Lambda for serverless execution, Amazon ECS and Amazon EKS for container orchestration, and AWS Fargate for serverless container compute. |
| Storage | Amazon S3 for object storage, Amazon EBS for persistent block storage attached to EC2, Amazon EFS for shared file storage, and Amazon S3 Glacier for low-cost archival. |
| Database | Amazon RDS for managed relational databases, Amazon Aurora for high-performance managed SQL, Amazon DynamoDB for managed NoSQL key-value workloads, and Amazon Redshift for data warehousing. |
| Networking | Amazon VPC for an isolated private network, Amazon Route 53 for DNS, and AWS Direct Connect for a dedicated connection to AWS. |

### Common exam triggers

- Run event-driven code without provisioning or patching servers: AWS Lambda.
- Deploy across isolated data centers within one Region: Availability Zones.
- Run complex analytical SQL queries on a petabyte-scale data warehouse: Amazon Redshift.
- Create an isolated private virtual network: Amazon VPC.
- Provide highly available managed DNS: Amazon Route 53.
- Cache content close to users: Amazon CloudFront.

## Domain 4: Billing, Pricing, and Support

### Compute purchasing options

- On-Demand: Pay by the second or hour without a long-term commitment. Best for short-term or unpredictable workloads.
- Reserved Instances and Savings Plans: Commit for one or three years in exchange for discounted rates. Best for steady-state workloads.
- Spot Instances: Use unused EC2 capacity at discounts of up to 90%. AWS can reclaim capacity with a two-minute interruption notice. Best for stateless and fault-tolerant workloads.
- Dedicated Hosts: Physical servers dedicated to one customer, often used for strict compliance or Bring Your Own License (BYOL) requirements.

### Cost management and technical support tools

- AWS Organizations: Manages multiple accounts and provides consolidated billing, which can help aggregate usage and qualify for volume pricing.
- AWS Cost Explorer: Visualizes, analyzes, and forecasts historical and future cloud usage costs.
- AWS Pricing Calculator: Estimates projected costs before building or deploying workloads.
- AWS Budgets: Tracks costs and usage against defined thresholds and sends alerts.
- AWS Cost and Usage Report: Provides detailed billing and usage data for analysis.

### AWS Support plans

- Basic: Documentation, service health information, and core support resources.
- Developer: Business-hours email support.
- Business: 24/7 phone and chat support.
- Enterprise: Proactive guidance and an assigned Technical Account Manager (TAM).

## Scenario-based revision points

Before the exam, make sure you can identify the best answer for scenarios involving:

- customer responsibilities when using a managed service such as Amazon RDS
- private cloud networks with Amazon VPC
- agility and elasticity in response to changing demand
- Availability Zones for fault tolerance
- the difference between cost estimation and cost analysis tools
- Spot Instances for interruptible batch workloads
- AWS Organizations for consolidated billing
- Enterprise Support for proactive architectural guidance and a TAM
- Amazon Route 53 for managed DNS

## Final reminder

The CLF-C02 exam is not just about memorizing AWS service names. Connect each service to a business requirement: elasticity for changing demand, IAM for least-privilege access, Availability Zones for resilience, and the right pricing model for the workload. Focus on understanding why an answer is correct and identifying the key requirement in each scenario.
