# AWS Security - Specialty (SCS-C03) Study Notes

This is a condensed study note for the AWS Certified Security - Specialty (SCS-C03) exam. It focuses on detection, incident response, infrastructure security, identity and access management, data protection, and security governance.

For a deeper write-up and scenario-based practice questions, see: https://www.nabilnoh.com/posts/aws-security-specialty-scs-c03-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Detection | 16% |
| Domain 2 | Incident Response | 14% |
| Domain 3 | Infrastructure Security | 18% |
| Domain 4 | Identity and Access Management | 20% |
| Domain 5 | Data Protection | 18% |
| Domain 6 | Security Foundations and Governance | 14% |

The exam has 65 questions, including 50 scored questions and 15 unscored pretest questions. The time limit is 170 minutes, and the passing score is 750 out of 1000 on a scaled score. It can include multiple-choice, multiple-response, ordering, and matching questions.

## A security-first scenario method

For each question, evaluate the requirements in this order:

1. Identify the asset, principal, account, network boundary, and data classification.
2. Identify the required verb: prevent, detect, contain, investigate, preserve, or prove.
3. Trace the identity, policy, endpoint, route, service, resource, and encryption path.
4. Decide whether the control must be preventive, detective, corrective, or evidence-producing.
5. Prefer a managed, native integration with the least custom operational work.

The service name is often correct but the boundary is wrong. For example, Inspector finds vulnerabilities, GuardDuty detects threats, Security Hub aggregates findings, and Detective helps investigate relationships. They are complementary rather than interchangeable.

## Domain 1: Detection

This domain covers security telemetry, threat detection, vulnerability management, sensitive-data discovery, and finding aggregation.

### Match the finding to the service

| Service | Primary purpose | Important boundary |
| :--- | :--- | :--- |
| GuardDuty | Threat detection from AWS and workload telemetry | It is not a general vulnerability scanner. |
| Inspector | Continuous vulnerability scanning for EC2, ECR, and Lambda | It finds software and package vulnerabilities, not every active compromise. |
| Macie | Discover and classify sensitive data in S3 | It focuses on S3 data discovery, not application authorization. |
| Detective | Investigate findings with entity relationships and timelines | It supports investigation; it is not the containment mechanism. |
| Security Hub | Aggregate and normalize findings into ASFF | It centralizes findings but does not replace source-service telemetry. |
| Security Lake | Centralize security logs using OCSF | It is a security data lake, not an inline prevention control. |

### CloudTrail, Config, and network telemetry

- CloudTrail management events record control-plane API activity such as changing a bucket policy or disabling a trail.
- CloudTrail data events record object-level or function-level activity such as `PutObject` or Lambda `Invoke`; configure the required resources and Regions explicitly.
- AWS Config records resource configuration and compliance state. It helps answer what a resource looked like and when it changed.
- VPC Flow Logs provide network metadata, not packet payloads. Use a custom format when required fields are missing from the default format.
- Route 53 Resolver query logs help investigate suspicious DNS behavior and possible command-and-control activity.
- Transit Gateway Flow Logs provide visibility into traffic crossing a centralized network hub.

Use this distinction during investigations:

| Question | Best evidence |
| :--- | :--- |
| Who called an AWS API? | CloudTrail |
| What was the resource configuration? | AWS Config |
| Which network connection metadata was observed? | VPC or Transit Gateway Flow Logs |
| Which DNS names were queried? | Route 53 Resolver query logs |
| Is the activity suspicious? | GuardDuty and related findings |

### Centralized findings

For a multi-account organization, use a dedicated Security account as the delegated administrator for supported security services and a separate Log Archive account for protected logs.

- Use Security Hub to aggregate GuardDuty, Inspector, Macie, and other supported findings.
- Use EventBridge to route high-severity findings to SNS, SSM Automation, Lambda, or an approved collaboration target.
- Use Security Lake to centralize security data and retain it with appropriate S3 lifecycle, access, and KMS controls.
- Protect audit records with restricted access, versioning, Object Lock where required, and a separate logging account.
- The management account is an Organizations and billing boundary, not the default home for workloads and security evidence.

## Domain 2: Incident Response

This domain covers containment, evidence preservation, investigation, eradication, recovery, and repeatable response automation.

### Contain before destroying evidence

Use this sequence:

> Detect -> contain -> preserve -> investigate -> eradicate -> improve

For a compromised EC2 instance:

1. Restrict network access without destroying the instance.
2. Prevent the instance from returning to service through Auto Scaling or a load balancer.
3. Capture EBS snapshots and any required host or memory evidence.
4. Investigate CloudTrail activity, network telemetry, identities, and related findings.
5. Rebuild from a trusted source rather than trusting a modified host.
6. Record the response and improve the control that allowed the incident.

- Security groups are stateful and allow rules only.
- Network ACLs are stateless and support explicit deny rules; allow return traffic and respect rule-number precedence.
- For a compromised IAM user, deactivate access keys and attach a targeted deny policy while preserving the user and audit trail.
- Do not delete the user or terminate the instance before required evidence is captured.
- Use SSM Run Command or an approved forensics workflow instead of opening emergency SSH access.

### Event-driven response automation

The common pattern is:

```text
GuardDuty / Config / CloudTrail finding
                  |
            EventBridge rule
                  |
      SSM Automation or Lambda
                  |
  Isolate -> snapshot -> notify -> record
```

- Prefer an SSM Automation runbook when an AWS-native action already exists.
- Use Lambda when custom orchestration is required, such as identifying the affected Auto Scaling group, deregistering a target, taking snapshots, and writing an incident record.
- Keep response automation narrow and idempotent. A broad rule that terminates every instance with a tag can create a second incident.
- Preserve the original finding, event, action, actor, and timestamps for auditability.

## Domain 3: Infrastructure Security

This domain covers network controls, edge protection, private connectivity, host hardening, and traffic inspection.

### Choose the control by network layer

| Requirement | Appropriate control | Exam boundary |
| :--- | :--- | :--- |
| HTTP/S request filtering | AWS WAF | Supports CloudFront, ALB, API Gateway, AppSync, and other supported resources; not NLB. |
| Layer 3-7 inline inspection | AWS Network Firewall | Requires correct routing and symmetric inspection behavior. |
| Passive packet inspection | VPC Traffic Mirroring and GWLB | Copies traffic to an inspection fleet; it is not inline blocking. |
| Subnet-level explicit deny | Network ACL | Stateless; allow return traffic and respect rule precedence. |
| Instance-level stateful filtering | Security group | Stateful and allow-only; it has no explicit deny rule. |
| Outbound domain restriction | Route 53 Resolver DNS Firewall | Controls DNS queries, not every connection to a hard-coded IP. |

The phrase “block these IP addresses” is not enough to pick a service. Use WAF for HTTP/S resources, NACLs for subnet-level explicit IP denies, and security groups for stateful allow rules.

### Private connectivity and encryption

- PrivateLink provides consumer-initiated access to a specific service through an interface endpoint. It avoids broad VPC routing and can work with overlapping CIDRs.
- VPC peering provides private routing between VPCs but does not support overlapping CIDR ranges.
- Transit Gateway centralizes routing between attachments; route-table association and propagation determine isolation.
- Direct Connect provides dedicated connectivity but does not encrypt traffic by itself.
- A Site-to-Site VPN over a private connection adds IPsec encryption when the requirement explicitly needs encrypted traffic.
- Gateway endpoints provide private S3 and DynamoDB access through route tables and can avoid NAT for same-Region service access.
- Interface endpoints use private ENIs with security groups, endpoint policies, and optional private DNS.

### Host hardening and inspection

- Use SSM Session Manager for administration without public SSH/RDP ingress or bastion hosts.
- Use SSM Run Command for fleet-wide scripts and configuration changes.
- Use EC2 Image Builder to create patched golden AMIs and approved launch-template images.
- Use Inspector for continuous vulnerability scanning of supported compute and package surfaces.
- Use VPC endpoints so private instances can reach required AWS services without unnecessary internet paths.
- Use VPC Flow Logs for accepted/rejected flow metadata.
- Use Traffic Mirroring to send passive traffic copies to an out-of-band inspection fleet.
- Use Network Firewall for inline filtering and TLS inspection when routing is designed correctly.

## Domain 4: Identity and Access Management

This is the largest domain. It covers policy evaluation, federation, delegation, privilege boundaries, and access analysis.

### Policy evaluation checklist

1. An explicit deny wins.
2. SCPs limit principals in member accounts; they do not grant permissions.
3. Resource control policies limit supported resources; they do not replace resource policies.
4. Cross-account access normally requires an allow in the caller's identity policy and the target resource policy.
5. Permission boundaries cap the maximum permissions an identity-based policy can grant.
6. Session policies narrow one STS session.
7. Trust policies allow role assumption; they do not grant the role's service permissions.
8. KMS key policies authorize use of the KMS key and must align with the caller's permissions.

The effective permission is the result of all applicable identity, resource, organization, boundary, session, and key-policy decisions.

### Delegation and ABAC

To let developers create roles without allowing unrestricted privilege escalation:

- Require an approved permissions boundary during `iam:CreateRole`.
- Allow only the required policy-management actions.
- Deny removal or replacement of the approved boundary.
- Restrict which managed or customer-managed policies can be attached.
- Review actual access with CloudTrail and IAM Access Analyzer.

For attribute-based access control, match principal and resource tags and require request tags at creation time. Tag policies standardize supported tag values, but they do not automatically require every resource to have every tag or grant access.

### Workforce, customer, and third-party identity

| Identity requirement | AWS pattern |
| :--- | :--- |
| Workforce SSO into many AWS accounts | IAM Identity Center with groups and permission sets |
| Federation from an enterprise IdP | SAML 2.0 and SCIM with the appropriate identity source |
| Customer application registration and sign-in | Cognito User Pools |
| Temporary AWS credentials for an authenticated customer | Cognito Identity Pools |
| Third-party cross-account role assumption | STS `AssumeRole` with an external ID |
| Service-to-service access | IAM roles and temporary credentials |

Keep workforce and customer identity separate. IAM Identity Center is for workforce access to AWS accounts and applications. Cognito User Pools authenticate application users, while Cognito Identity Pools can exchange trusted identities for temporary AWS credentials.

### Find and reduce unused access

- IAM Access Analyzer finds external access, analyzes unused access, validates policies, and can generate policies from activity.
- IAM Access Advisor provides service last-accessed information for permission reviews.
- Credential reports show IAM user credential status, MFA, key age, and last-use information.
- CloudTrail provides evidence of what a principal actually called, subject to configured event coverage.

Do not confuse an externally shared resource with a role that has unused permissions; they are different analysis questions.

## Domain 5: Data Protection

This domain covers encryption, key management, secrets, data discovery, immutability, and encryption in transit.

### KMS keys and rotation

| Scenario | Key choice or rule |
| :--- | :--- |
| Basic service encryption without custom key-policy requirements | AWS-owned or AWS-managed key, depending on the service |
| Cross-account sharing or custom key policy | Customer-managed KMS key |
| Temporary delegated access at scale | KMS grants |
| Key use only through an AWS service | `kms:ViaService` condition |
| New key ID on a schedule | Create and transition to new keys; automatic rotation keeps the key ID |
| Regional encryption and failover | Consider a multi-Region key design where supported |

Automatic rotation changes backing key material while retaining the KMS key ID. If the requirement explicitly says “a new key ID every month,” automatic rotation alone is not enough.

### Cross-account encryption authorization

For an encrypted S3 object read across accounts, verify:

1. The caller's IAM policy allows the required S3 and KMS actions.
2. The S3 bucket policy allows the intended principal or organization.
3. The KMS key policy allows the caller account or role to use the key.
4. The VPC endpoint policy and `aws:SourceVpce` condition match the actual path, if endpoint restriction is required.
5. The KMS encryption context and service path do not contradict the request.

An S3 allow is not a KMS allow. A KMS key policy is not a substitute for the caller's missing cross-account identity permission.

### S3, secrets, and certificates

- Use Macie to discover PII and custom sensitive-data patterns in S3.
- Use S3 Batch Operations to process large object sets without application-side downloads.
- Use SSE-KMS when customer-managed key control and auditable key usage are required.
- Use S3 Bucket Keys when reducing KMS request volume is important.
- Enable versioning and Object Lock for WORM retention.
- Use Compliance mode when no principal, including root, may shorten retention or delete protected versions.
- Treat existing and future S3 objects separately; changing a default setting does not repair historical objects.
- Use Secrets Manager for database credentials and API keys requiring native or custom rotation.
- Use SSM Parameter Store `SecureString` for configuration with lower-cost secure storage and manual or custom rotation.
- Use ACM Private CA for internal TLS and mTLS certificates and ACM public certificates for supported public endpoints.
- Use CloudHSM or an appropriate KMS custom key store for specialized key-custody or HSM requirements.
- Do not put secrets in source code, container images, or plaintext environment variables. Retrieve them at runtime using an application role.

## Domain 6: Security Foundations and Governance

This domain covers the shared responsibility model, multi-account security architecture, compliance evidence, and organization-wide controls.

### A secure multi-account baseline

```text
Management Account     -> Organizations, billing, Control Tower only
Log Archive Account    -> CloudTrail, Config, VPC Flow, WAF logs
Security Account       -> Delegated GuardDuty, Security Hub, Macie, Inspector
Network Account        -> Transit Gateway, DNS, Direct Connect
Workloads OU           -> Production, staging, and development accounts
```

- The management account should not become the workload or log archive account.
- Restrict deletion and administrative access in the Log Archive account.
- Use the Security account to administer supported security services.
- Operate workloads under OU guardrails and separate production, non-production, and sandbox accounts.

### Preventive, detective, corrective, and evidence controls

| Control type | Examples | What it does |
| :--- | :--- | :--- |
| Preventive | SCP, RCP, declarative policy, permission boundary, key policy | Blocks or limits an action before or during authorization |
| Detective | Config rule, Config aggregator, Security Hub standard, CloudTrail analysis | Finds a state or action that violates a requirement |
| Corrective | SSM Automation, EventBridge workflow, Lambda response | Changes a resource or contains an incident after detection |
| Evidence | Audit Manager, CloudTrail, Config history, AWS Artifact | Supports investigation, audit, or compliance evidence |

When a question says “must not be possible,” choose a preventive control. When it says “identify noncompliant resources,” choose a detective control. When it says “automatically fix,” choose a corrective workflow.

### Central security services

- Use Security Hub Central Configuration or supported delegated-administrator patterns to standardize security settings.
- Use AWS Config aggregators for read-only cross-account visibility.
- Use Config conformance packs to deploy related compliance rules.
- Use Firewall Manager for centralized WAF, Shield, security-group, Network Firewall, and DNS Firewall policies where supported.
- Use service-managed CloudFormation StackSets with automatic deployment for baselines in current and future OU accounts.
- Use AWS RAM for supported resource sharing, such as VPC subnets; do not assume every AWS resource can be shared.

### Compliance and shared responsibility

| Service | Purpose |
| :--- | :--- |
| AWS Artifact | AWS compliance reports, certifications, and agreements |
| AWS Audit Manager | Collect evidence for an organization's audit framework |
| AWS Config | Configuration history and compliance evaluation |
| AWS Well-Architected Tool | Review workload design against architectural pillars |
| AWS Trusted Advisor | Checks across cost, performance, security, and service limits depending on support |

AWS is responsible for security of the cloud. The customer remains responsible for security in the cloud, with the exact split varying by service. A managed service reduces operational responsibility; it does not eliminate IAM, data classification, configuration, or access-control decisions.

## High-value distinctions

| Almost-right answer | Missing detail |
| :--- | :--- |
| “The bucket policy allows the read.” | SSE-KMS also requires the correct caller and key-policy permissions. |
| “A security group can block this IP.” | Security groups allow traffic only; use a NACL or WAF when explicit deny behavior is required. |
| “Config detects the problem.” | Detection does not prevent creation or automatically remediate without a configured workflow. |
| “Security Hub sends directly to SNS.” | Use EventBridge to route findings to notifications or automation. |
| “Delete the compromised IAM user.” | Deactivate credentials and preserve identity and audit evidence first. |
| “Direct Connect encrypts traffic.” | Add a VPN when the requirement explicitly needs IPsec encryption. |
| “Automatic KMS rotation creates a new key every month.” | Automatic rotation keeps the KMS key ID; a new ID needs a key-transition design. |
| “The default S3 retention protects old objects.” | Existing object versions may need explicit retention treatment. |
| “A multi-account aggregator manages every account.” | Aggregation provides visibility; delegated administration or StackSets provide management or deployment. |

## Final review strategy

At 170 minutes for 65 questions, the average budget is about 2 minutes 37 seconds per question.

1. Identify the asset and owner.
2. Underline the requirement verb: prevent, detect, contain, investigate, preserve, or prove.
3. Trace the principal, policy, endpoint, route, service, resource, and key.
4. Check whether the solution is cross-account, cross-Region, or organization-wide.
5. Eliminate answers that use a real service outside its supported boundary.
6. For multiple response, verify that every selected option is required and complete.

The most useful question to ask is:

> What single requirement would make this answer wrong?

If the architecture only works after changing the scenario, it is not the correct answer.
