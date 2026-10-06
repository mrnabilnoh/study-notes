# AWS Developer - Associate (DVA-C02) Study Notes

This is a condensed study note for the AWS Certified Developer - Associate (DVA-C02) exam. It focuses on AWS SDK development, serverless architectures, application security, CI/CD deployment, observability, and troubleshooting patterns.

For a deeper write-up and more detailed notes, see: https://www.nabilnoh.com/posts/aws-developer-dva-c02-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Development with AWS Services | 32% |
| Domain 2 | Security | 26% |
| Domain 3 | Deployment | 24% |
| Domain 4 | Troubleshooting and Optimization | 18% |

The exam has 65 questions, including 50 scored questions and 15 unscored pretest questions. The time limit is 130 minutes, and the passing score is 720 out of 1000 on a scaled score.

## Domain 1: Development with AWS Services

This domain focuses on writing application code with AWS SDKs, building serverless applications, managing data stores, and creating decoupled event-driven architectures.

### AWS Lambda development and lifecycle

- Execution context reuse: Initialize database connections, HTTP clients, and heavy dependencies outside the function handler so warm invocations can reuse them.
- Reserved concurrency: Guarantees a dedicated pool of concurrent executions and caps maximum concurrency to protect downstream systems.
- Provisioned concurrency: Pre-warms execution environments to reduce cold-start latency for latency-sensitive applications.
- VPC integration: Lambda functions connected to private VPC resources create Elastic Network Interfaces (ENIs). Ensure private subnets have enough IP space and appropriate security groups.

### Lambda event source models

| Model | Examples | Behavior |
| :--- | :--- | :--- |
| Synchronous | API Gateway, Application Load Balancer | Returns the function response directly to the caller. |
| Asynchronous | Amazon S3, Amazon SNS | Uses internal queues and retries failed events before sending them to a DLQ or Lambda Destination. |
| Stream or poll-based | DynamoDB Streams, Amazon Kinesis | Processes records in batches and requires batch failure handling such as bisect batch on error. |

### Amazon DynamoDB and data stores

- Partition key selection: Use high-cardinality keys to distribute read and write traffic evenly and avoid hot partitions.
- `Query`: Efficiently retrieves items for a specific partition key, with optional sort key conditions.
- `Scan`: Reads every item in a table and is expensive and slow. Prefer `Query` whenever possible.
- Local Secondary Index (LSI): Uses the same partition key with an alternate sort key and must be created when the table is created.
- Global Secondary Index (GSI): Uses an alternate partition key and sort key and can be created or deleted after table creation.
- Eventually consistent reads: The default mode and consumes 0.5 read capacity units per 4 KB.
- Strongly consistent reads: Consumes 1 read capacity unit per 4 KB and returns the latest data.
- Amazon ElastiCache: General application or relational database caching; a write-through strategy helps avoid stale cache data.
- DynamoDB Accelerator (DAX): Microsecond in-memory caching designed specifically for DynamoDB.

### Event-driven messaging and AI assistance

- Fanout architecture: Publish to one Amazon SNS topic with multiple Amazon SQS subscriptions so independent services receive every event durably.
- Change data capture: Enable DynamoDB Streams and trigger Lambda for downstream processing or external API calls.
- Amazon Q Developer: Provides code completion, code generation, test creation, and security assistance.

## Domain 2: Security

This domain tests authentication, fine-grained authorization, encryption, and secret management.

### Identity and access management

- Never hardcode access keys in source code.
- EC2 applications use instance profiles and IAM roles.
- Lambda functions use execution roles.
- AWS STS `AssumeRole` provides temporary credentials for cross-account access or temporary elevated permissions.
- Attach the required permissions to the existing role when an application receives `AccessDenied`; an EC2 instance profile cannot use multiple roles at the same time.

### Amazon Cognito

- User pools: Handle user registration, login, MFA, and social identity federation. They issue JWT ID, access, and refresh tokens.
- Identity pools: Exchange authenticated user or third-party identity tokens for temporary AWS IAM credentials to access AWS resources directly.

### Encryption and key management

- Server-side encryption (SSE): AWS encrypts data at rest after receiving it. Examples include SSE-S3, SSE-KMS, and SSE-C.
- Client-side encryption: The application encrypts data before sending it to AWS. The AWS Database Encryption SDK can protect selected DynamoDB item attributes.
- Symmetric KMS keys: Used for encryption and decryption operations and envelope encryption with data encryption keys.
- Asymmetric KMS keys: Use RSA or ECC key pairs for signing or encryption without exposing the private key.
- Encryption in transit protects data while it moves between clients, applications, and AWS services.

### Secret and sensitive data management

| Service | Best fit |
| :--- | :--- |
| AWS Systems Manager Parameter Store | Stores strings, lists, and KMS-encrypted `SecureString` values. Standard parameters offer a free tier and rotation is usually manual or custom. |
| AWS Secrets Manager | Stores sensitive secrets such as database credentials and API keys, with native or custom Lambda rotation and cross-account sharing. |

## Domain 3: Deployment

This domain covers packaging, Infrastructure as Code, API Gateway configuration, and automated delivery pipelines.

### Infrastructure as Code and packaging

- AWS SAM extends CloudFormation for serverless resources such as `AWS::Serverless::Function`, `AWS::Serverless::Api`, and `AWS::Serverless::SimpleTable`.
- Common SAM CLI commands include `sam build`, `sam local invoke`, `sam package`, and `sam deploy`.
- `appspec.yml` belongs at the root of the application source bundle for CodeDeploy deployments.
- AWS AppConfig manages, validates, and deploys runtime configuration without requiring an application redeployment or restart.

### Amazon API Gateway

- Lambda proxy integration passes the raw HTTP request, including headers, query parameters, and body, to the Lambda event handler.
- Lambda custom integration uses request and response mapping templates, commonly written in VTL.
- Stage variables provide environment-specific values for stages such as `dev`, `test`, and `prod`.
- Reference a stage variable in an integration URI as `${stageVariables.variableName}`. This can route one API endpoint to different Lambda aliases.

### CI/CD tools

| Service | Purpose |
| :--- | :--- |
| AWS CodeBuild | Runs build and test commands defined in `buildspec.yml`. |
| AWS CodeDeploy | Deploys code to EC2, ECS, or Lambda using instructions in `appspec.yml`. |
| AWS CodePipeline | Orchestrates source, build, test, and release stages. |

### Deployment strategies

- Canary: Shift a small percentage of traffic to the new version, monitor it, then route the remainder.
- Linear: Increase traffic to the new version in equal increments over time.
- Blue/green: Validate a separate environment and switch traffic from the old version to the new version.

## Domain 4: Troubleshooting and Optimization

This domain focuses on distributed tracing, logs and metrics, error handling, and performance optimization.

### Distributed tracing with AWS X-Ray

- Service map: Visualizes microservices, downstream HTTP calls, and AWS resources to identify latency and fault origins.
- Segment: Documents activity for a computing node such as EC2 or Lambda.
- Subsegment: Documents detailed operations such as HTTP requests, SQL queries, or DynamoDB calls.
- Annotations: Indexed key-value pairs that can be used to filter traces.
- Metadata: Non-indexed diagnostic data and debug payloads attached to traces.

### Metrics, logging, and observability

- CloudWatch Logs Insights: Queries, filters, and analyzes structured logs across log groups.
- CloudWatch Embedded Metric Format (EMF): Writes structured JSON logs that CloudWatch extracts into metrics asynchronously.
- Lambda logging: The execution role needs permissions such as `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents`, commonly provided by `AWSLambdaBasicExecutionRole`.
- Use application health checks, readiness probes, and CloudWatch alarms with SNS notifications to detect and communicate failures.

### Error handling and refactoring

- Exponential backoff with jitter: Retries transient errors and throttling with increasing, randomized delays to avoid thundering herd problems.
- Lambda asynchronous failures: Configure DLQs or Lambda Destinations to retain or route failed events.
- Step Functions activities: Use `HeartbeatSeconds` to detect stalled workers and configure retry policies such as `Retry.MaxAttempts`.
- Optimize by reusing connections, selecting appropriate concurrency settings, using efficient DynamoDB access patterns, and moving repeated work to caches.

## Scenario-based revision points

Before the exam, make sure you can identify the best solution for scenarios involving:

- DynamoDB Streams triggering Lambda after item changes
- reusing database connections outside a Lambda handler
- ElastiCache write-through caching when stale reads are unacceptable
- SNS-to-SQS fanout for independent and durable consumers
- IAM roles instead of hardcoded credentials
- Cognito user pools versus identity pools
- client-side encryption with the AWS Database Encryption SDK and symmetric KMS keys
- Secrets Manager rotation for third-party API keys
- SAM packaging and the correct location of `appspec.yml`
- API Gateway stage variables routing to Lambda aliases
- canary, linear, and blue/green deployment strategies
- AWS X-Ray service maps and annotations for filtering traces
- CloudWatch Logs Insights and EMF for application observability
- exponential backoff with jitter for throttling and transient errors

## Final reminder

DVA-C02 requires more than high-level service recognition. Focus on Lambda execution lifecycles, DynamoDB access and index design, IAM permissions, API Gateway and Lambda deployment patterns, CI/CD mechanics, and distributed tracing. In troubleshooting questions, identify the specific root cause before choosing the AWS service or configuration that addresses it.
