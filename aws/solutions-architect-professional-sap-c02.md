# AWS Solutions Architect - Professional (SAP-C02) Study Notes

This is a condensed study note for the AWS Certified Solutions Architect - Professional (SAP-C02) exam. It focuses on multi-account governance, hybrid networking, resilient architectures, performance and cost optimization, and migration and modernization.

For a deeper write-up and scenario-based practice questions, see: https://www.nabilnoh.com/posts/aws-solutions-architect-professional-sap-c02-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Design Solutions for Organizational Complexity | 26% |
| Domain 2 | Design for New Solutions | 29% |
| Domain 3 | Continuous Improvement for Existing Solutions | 25% |
| Domain 4 | Accelerate Workload Migration and Modernization | 20% |

The exam has 75 questions, including 65 scored questions and 10 unscored pretest questions. The time limit is 180 minutes, and the passing score is 750 out of 1000 on a scaled score.

## A scenario-first design method

For long questions, evaluate the requirements in this order:

1. Identify hard constraints such as protocol, RPO, RTO, data sovereignty, licensing, and application-change limits.
2. Trace the identity, network, encryption, and data paths end to end.
3. Test the failure path, including capacity, promotion, routing, secrets, certificates, and quotas.
4. Compare operational effort, performance, and total cost.
5. Select the complete design, not just the service that solves one symptom.

Two answers can both be technically possible. The better answer usually satisfies the hard requirement with fewer operational steps and fewer unsupported assumptions.

## Domain 1: Design Solutions for Organizational Complexity

This domain covers AWS Organizations, multi-account governance, identity, shared networking, hybrid connectivity, centralized logging, and cost allocation.

### Organizations, SCPs, and identity

- IAM identity and resource policies grant permissions. Service Control Policies (SCPs) define the maximum permissions available to principals in member accounts; an SCP does not grant access.
- An explicit deny inherited from the organization root or an OU cannot be overridden by an allow in a child account.
- SCPs do not apply to the management account or service-linked roles. Keep workloads out of the management account.
- Use OUs to separate production, non-production, security, and sandbox governance boundaries.
- IAM Identity Center provides workforce access through groups and permission sets. Amazon Cognito user pools and identity pools serve application users and are a different identity boundary.
- Cross-account access requires the caller's permission to assume a role, a trust policy on the target role, and permissions for the assumed role to access the resource.
- Cross-account use of a customer managed KMS key requires authorization in both the caller's IAM policy and the key policy.

### Centralized account and resource management

| Requirement | Suitable pattern | Important boundary |
| :--- | :--- | :--- |
| Deploy a baseline into new and existing accounts | Service-managed CloudFormation StackSets with automatic deployment | Target the correct organization or OUs and configure trusted access. |
| Centralize security administration | Delegated administration for supported security services | Delegation is service-specific, not blanket access to every account. |
| Let application accounts deploy into centrally owned subnets | VPC sharing through AWS RAM | The VPC owner manages the network; participants manage their resources. |
| Connect independently owned VPCs | Transit Gateway shared through AWS RAM | Route-table association and propagation determine isolation. |
| Expose one service without full network connectivity | AWS PrivateLink | The provider exposes an endpoint service; consumers create interface endpoints. |

VPC sharing, Transit Gateway, VPC peering, and PrivateLink solve different problems. Peering does not support overlapping CIDR ranges. PrivateLink is often the best fit for a one-way consumer-to-provider service relationship and does not require full VPC routing.

### Hybrid networking and DNS

- AWS Direct Connect provides a private network path but does not encrypt traffic by itself. Use a VPN over Direct Connect when encryption is required.
- Transit Gateway centralizes connectivity between VPCs and on-premises networks. Control isolation with route-table association and route propagation, not resource tags.
- Use an AWS Site-to-Site VPN for encrypted connectivity over the internet or as backup connectivity.
- Route 53 Resolver inbound endpoints receive DNS queries from on-premises resolvers. Outbound endpoints forward matching AWS queries to on-premises DNS servers.
- Private hosted zones must be associated with the VPCs that need to resolve them. A TGW attachment or VPC peering connection does not automatically share private DNS visibility.
- Troubleshoot in this order: name resolution, route, security controls, then application permissions.

### Centralized security, audit, and cost

- Use an organization CloudTrail trail to record activity across current and future accounts. Configure data events when S3 object-level or Lambda invocation activity is required.
- AWS Config rules detect configuration state; they do not automatically prevent every noncompliant resource from being created.
- Security Hub aggregates security findings, while GuardDuty detects suspicious activity. Use delegated administration where supported.
- Tag policies standardize supported tag values, but do not by themselves require every resource to have every tag.
- Use activated cost allocation tags and Cost Categories for chargeback. A resource tag is not automatically a billing dimension.
- A Cost and Usage Report (CUR) stored in S3 can be queried with Athena and visualized with QuickSight.

## Domain 2: Design for New Solutions

This domain connects workload requirements to secure, scalable, observable, and resilient architectures.

### Choose services from access patterns

| Requirement | Candidate | Qualification |
| :--- | :--- | :--- |
| Key-value access with unpredictable demand | DynamoDB on-demand | Capacity mode does not eliminate hot partition keys. |
| Relational transactions and SQL joins | RDS or Aurora | Writer capacity, read scaling, and connection pooling are separate concerns. |
| Shared Linux files across AZs | Regional EFS | S3 is not a drop-in POSIX filesystem. |
| Windows shares with AD and NTFS permissions | FSx for Windows File Server | Plan deployment type, throughput, and directory integration. |
| Private access to a supplier service | PrivateLink | Avoid unnecessary full VPC-to-VPC connectivity. |
| Short event-driven processing | Lambda | Account for timeout, concurrency, and downstream limits. |
| Long-running container workloads without host management | ECS on Fargate | Use an appropriate queue, retry, and scaling design. |
| Kubernetes compatibility or control requirements | EKS | The operational burden is higher than a managed serverless container option. |

Separate the system of record from its acceleration layer. A cache needs a defined miss, invalidation, consistency, and failure strategy. MemoryDB is intended for a durable in-memory primary database; not every Redis-compatible service has the same durability model.

### Security and network boundaries

- CloudFront with an S3 origin should use Origin Access Control and a bucket policy scoped to the distribution. Viewer authentication is a separate concern.
- ALB supports HTTP/HTTPS features such as host and path routing. NLB supports TCP, UDP, and TLS and can provide static addresses per AZ.
- TLS termination at a load balancer does not encrypt the hop to targets. Use HTTPS target groups when backend encryption is required.
- Interface VPC endpoints provide private access to AWS services and endpoint services. Gateway endpoints are used for S3 and DynamoDB.
- Security groups are stateful; network ACLs are stateless and require return traffic rules.
- Use least-privilege application credentials and customer managed KMS keys when key-policy control, cross-account use, or key rotation requirements demand them.

### Resilience and global traffic

- Recovery Point Objective (RPO) measures acceptable data loss in time. Recovery Time Objective (RTO) measures acceptable recovery time.
- Multi-AZ resilience does not provide recovery from a Region outage.
- Backup and restore has the lowest standing cost but the most recovery work. Pilot light keeps core data services ready. Warm standby keeps a reduced application stack running. Active-active serves production from both Regions.
- Cross-Region recovery must include replicated data, application capacity, secrets, certificates, quotas, observability, and tested promotion and failover procedures.
- Route 53 failover routing selects between endpoints using health checks but does not promote a database or provision capacity.
- Route 53 latency routing chooses a healthy low-latency Region; it does not create capacity.
- Global Accelerator provides static global IP addresses and health-based routing for supported TCP and UDP endpoints.
- CloudFront accelerates HTTP(S) content. Origin failover is not general failover for arbitrary write requests.

## Domain 3: Continuous Improvement for Existing Solutions

This domain evaluates how to improve a live workload without replacing components unnecessarily. Identify the bottleneck before selecting the service.

### Match the improvement to the symptom

| Symptom | Targeted improvement | Boundary |
| :--- | :--- | :--- |
| Lambda creates too many database connections | RDS Proxy and connection reuse | It does not fix expensive SQL or an undersized writer. |
| Reports compete with transactions | Read replicas or Aurora readers | Reads must actually be routed to the readers. |
| S3 traffic crosses NAT in the same Region | S3 gateway endpoint | It does not replace unrelated internet egress. |
| Workers cannot absorb bursts | SQS buffering and backlog-based scaling | Limit concurrency so the downstream system is protected. |
| Terminated instances lose logs | Continuous CloudWatch log shipping or lifecycle hooks | Abrupt termination can still lose data not yet shipped. |

- Use queue depth or visible backlog per instance for queue-worker scaling; CPU alone can miss I/O-bound backlog growth.
- Delete an SQS message only after successful processing. Size visibility timeout for the work, use a DLQ for poison messages, and make business side effects idempotent.
- Reserved concurrency protects a downstream system and limits a function. Provisioned concurrency keeps execution environments initialized and reduces cold-start latency; it is not a general concurrency ceiling.
- Use CloudWatch metrics and logs, distributed tracing, and CloudTrail API history to distinguish application errors, saturation, dependency latency, and unauthorized changes.

### Cost, storage, and commitments

- Rightsize before buying commitments. Review peak usage, guest-memory metrics, and Compute Optimizer recommendations rather than relying only on averages.
- Compute Savings Plans cover eligible EC2, Lambda, and Fargate usage. Buy commitments against a dependable baseline and use elastic or Spot capacity for variable, interruptible work.
- EC2 Capacity Reservations provide capacity in an Availability Zone; they are not inherently a pricing discount.
- Evaluate storage using access frequency, retrieval time, object size, retention period, request volume, and minimum-duration or retrieval charges.
- Existing S3 objects are not automatically re-encrypted when a new default encryption setting is applied.

## Domain 4: Accelerate Workload Migration and Modernization

This domain covers discovery, migration waves, database conversion, file and server migration, and validating cutover.

### Select the migration tool

| Workload or requirement | Service or pattern | Boundary |
| :--- | :--- | :--- |
| Rehost servers while preserving their configuration | Application Migration Service (MGN) | Test launches, dependencies, cutover, and rollback. |
| Move data between supported database engines | AWS DMS with full load plus CDC | Assess schema and code compatibility; DMS does not convert every stored procedure. |
| Convert database schemas and code | AWS Schema Conversion Tool (SCT) | Resolve unsupported features and validate application behavior. |
| Transfer NFS, SMB, or supported object data | AWS DataSync | It is not a live shared filesystem or database replication service. |
| Provide S3-backed access to file applications | S3 File Gateway | Account for cache behavior and upload latency. |
| Discover servers and application dependencies | Application Discovery Service and Migration Hub | Build migration waves from dependency evidence. |
| Build a migration business case | Migration Evaluator | Validate assumptions against the target architecture. |

### Discovery and migration waves

- Inventory configuration, utilization, running processes, network connections, authentication dependencies, scheduled jobs, shared storage, and licensing constraints.
- Use agent-based discovery when detailed process and network dependency evidence is needed.
- Group tightly coupled applications together when separating them would create unacceptable latency or availability risks.
- A replicated server is not proof that its dependencies, credentials, DNS, storage, or licensing are ready in the target environment.
- Test launches validate boot and application behavior in an isolated environment before production cutover.

### Validate the cutover

For a database migration with ongoing writes:

1. Assess compatibility and convert schema or code where required.
2. Establish private network paths and least-privilege source and target credentials.
3. Run the initial load and maintain CDC with enough source-log retention.
4. Validate representative data, application behavior, and target performance.
5. Quiesce or control source writes, let changes catch up, and switch application endpoints.
6. Monitor the target and use a tested rollback plan that avoids conflicting writers.

For file migrations, validate protocols, AD integration, ACLs, metadata, mount paths, name resolution, and remaining changes. For server migrations, validate boot configuration, attached data, service dependencies, and licenses.

## High-value distinctions

| Almost-right answer | Missing detail |
| :--- | :--- |
| "The bucket policy allows cross-account reads." | SSE-KMS also needs the required caller and key-policy authorization. |
| "The data is replicated to another Region." | Replication may be asynchronous; promotion, readiness, and RPO still need testing. |
| "A FIFO queue provides exactly-once processing." | Deduplication does not guarantee exactly-once business side effects; consumers need idempotency. |
| "CloudFront can fail over any request." | Origin-group failover has supported-method restrictions and is not general write failover. |
| "A default encryption setting protects all existing data." | Existing objects need a separate re-encryption operation if required. |
| "A managed service is always cheapest." | Compatibility, transfer, retrieval, commitments, and operational effort determine total cost. |
| "A DNS failover record completes disaster recovery." | Data promotion, application capacity, secrets, and routing readiness are separate steps. |
