# AWS Data Engineer - Associate (DEA-C01) Study Notes

This is a condensed study note for the AWS Certified Data Engineer - Associate (DEA-C01) exam. It focuses on data ingestion and transformation, data stores, data operations, security, and governance.

For a deeper write-up and scenario-based practice questions, see: https://www.nabilnoh.com/posts/aws-data-engineer-associate-dea-c01-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Data Ingestion and Transformation | 34% |
| Domain 2 | Data Store Management | 26% |
| Domain 3 | Data Operations and Support | 22% |
| Domain 4 | Data Security and Governance | 18% |

The exam has 65 questions, including 50 scored questions and 15 unscored pretest questions. The time limit is 130 minutes, and the passing score is 720 out of 1000 on a scaled score. Question types include multiple choice and multiple response.

## A data-first scenario method

For each scenario, trace the data path in this order:

1. Classify the source and ingestion pattern: batch, stream, CDC, SaaS, or event-triggered files.
2. Identify volume, velocity, schema, ordering, and freshness requirements.
3. Choose the transformation and orchestration boundary.
4. Verify storage format, partitioning, distribution, and query engine.
5. Trace IAM, KMS, Lake Formation, network endpoints, and audit events.
6. Eliminate options that miss a quota or latency target or add needless infrastructure.

The key question is: **Which pipeline, storage format, and governance control moves, transforms, and serves the data at the required latency and scale?**

## Domain 1: Data Ingestion and Transformation

This is the largest domain. It covers getting data into AWS, choosing streaming or batch services, transforming data, and tuning distributed processing.

### Streaming ingestion

| Requirement | Service pattern | Boundary |
| :--- | :--- | :--- |
| Managed delivery to S3, Redshift, OpenSearch, Splunk, or HTTP destinations | Amazon Data Firehose | Serverless delivery with buffering and optional transformation; buffering affects latency. |
| Custom consumers, replay, or strict per-partition ordering | Kinesis Data Streams with KCL, Lambda, or Flink consumers | S3 delivery requires Firehose or a consumer. |
| Near-real-time CloudWatch Logs delivery | Logs subscription filter to Data Firehose | Native integration can avoid a custom log-forwarding fleet. |
| Existing Kafka producers and consumers | Amazon MSK or MSK Serverless | Kafka compatibility can reduce migration changes. |
| Stateful stream processing, joins, and windows | Managed Service for Apache Flink | Use when the stream needs stateful computation rather than pass-through delivery. |

- Firehose buffers records before delivery. Its default S3 buffering interval is 300 seconds, so streaming delivery may still take minutes.
- For custom consumer behavior or replay, use Kinesis Data Streams with an appropriate consumer.
- For Kinesis throttling, check partition-key distribution before adding shards. A low-cardinality key can create a hot shard while others remain underused.
- Choose partition keys that spread writes while preserving any required ordering.

### Batch, SaaS, and database ingestion

| Scenario | Service | Why it fits |
| :--- | :--- | :--- |
| Full database load plus ongoing change data capture | AWS DMS | Moves the initial data and replicates changes to reduce cutover downtime. |
| Database schema conversion | DMS Schema Conversion or AWS SCT, depending on workflow | Schema conversion and data replication are separate responsibilities. |
| SaaS data to S3 or Redshift with little custom code | Amazon AppFlow | Managed connectors, schedules, and event-triggered transfers. |
| Bulk transfer of files or objects | AWS DataSync | Transfers files and objects; use DMS for relational database migration. |
| On-premises file arrival triggering AWS processing | S3 File Gateway with EventBridge and Glue Workflow | Event-driven processing avoids fragile polling and custom watchers. |

For database migrations, separate the **schema** problem from **data movement**: conversion tooling helps with schema and code compatibility, while DMS handles full load and CDC.

### Glue jobs and Spark tuning

| Glue feature | Best fit | Better alternative when |
| :--- | :--- | :--- |
| Glue ETL with Apache Spark | Distributed batch or streaming transformations | A tiny single-node utility task is all that is needed. |
| Glue Python Shell | Lightweight Python, API calls, and small tasks | Processing requires distributed compute. |
| Glue Streaming ETL | Micro-batch processing from supported streams such as Kinesis or Kafka | Simple delivery is needed without transformation. |
| Glue DataBrew | Visual, no-code data preparation | Complex programmatic Spark pipelines are needed. |
| Glue Workflows | Glue-centric crawler and job DAGs | Broad cross-service or hybrid orchestration is needed. |

- **Millions of small files:** Use `groupFiles="inPartition"` and `groupSize` to combine input work at read time and reduce task overhead.
- **Reduce output file count:** Use `coalesce(N)` when reducing partitions without a full redistribution.
- **Redistribute data evenly:** Use `repartition(N)` when even distribution is needed and the shuffle cost is acceptable.
- **Many catalog partitions:** Use Glue Data Catalog partition indexes with `catalogPartitionPredicate` to prune partitions before extensive S3 listing.
- **`push_down_predicate`:** Filters partitions in Glue/Spark workflows, but may still require listing candidate prefixes. Use partition indexes for very large partition catalogs.
- **Driver or executor out-of-memory:** Check input-file size, skew, joins, and worker memory before changing worker types.
- **ResolveChoice:** Resolves ambiguous or inconsistent column types in a DynamicFrame.
- **Relationalize:** Flattens nested semi-structured records into relational parent and child tables.
- **Glue Job Bookmarks:** Track previously processed input so recurring jobs can process new data.
- **Glue Data Quality:** Validates completeness, uniqueness, row counts, and allowed values; bookmarks are not data-quality checks.
- **Glue FindMatches:** Links and deduplicates records with imperfect matches.
- **Glue Detect PII transform:** Identifies and masks or redacts supported sensitive data during ETL.

### EMR, storage, and data layout

- **HDFS:** Local storage on EMR core nodes. It is ephemeral when the cluster terminates but can provide fast random I/O for intermediate processing.
- **EMRFS:** Reads and writes S3 data, separating persistent storage from cluster compute.
- **Uniform instance groups:** Support custom automatic scaling and EMR managed scaling.
- **Instance fleets:** Offer flexibility across instance types and Spot capacity; use EMR managed scaling.
- Lambda has a 15-minute maximum invocation runtime. Longer distributed transformations belong on Glue, EMR, or another suitable batch compute service.

| Dimension | Partitioning | Bucketing |
| :--- | :--- | :--- |
| Layout | Separate prefixes or folders for partition values | Fixed number of hash-distributed files |
| Goal | Skip irrelevant partitions for filtered scans | Improve join and aggregation locality |
| Useful keys | Low- or medium-cardinality values such as country and date | Higher-cardinality join keys such as user or order ID |

## Domain 2: Data Store Management

This domain covers data formats, storage classes, Redshift modeling and query options, Glue Data Catalog, Athena, DynamoDB, and OpenSearch.

### File formats and table formats

| Format | Layout | Typical use | Key detail |
| :--- | :--- | :--- | :--- |
| Parquet | Columnar | Athena, Redshift Spectrum, Glue analytics | Efficiently reads selected columns; commonly paired with Snappy. |
| ORC | Columnar | Hive, Presto, Athena, and Glue analytics | Supports columnar compression and predicate filtering. |
| Avro | Row-based | Streaming and schema evolution | Useful for record serialization; less efficient for analytical column scans. |
| CSV and JSON | Text/row-oriented | Raw landing and interoperability | Often scans more data than columnar formats for analytics. |
| Iceberg, Hudi, Delta Lake | Open table formats, often on Parquet | ACID operations, schema evolution, and table-level features | Check engine compatibility and the required feature set. |

For analytical S3 data, columnar formats reduce scanned data when queries select a subset of columns. Keep ingestion format and query format separate when conversion is worthwhile.

### S3 storage classes and retention

| Access pattern | Candidate | Decision point |
| :--- | :--- | :--- |
| Unknown or changing access | S3 Intelligent-Tiering | Automatic movement can help; consider monitoring and archive-tier behavior. |
| Predictable transition with instant retrieval | Standard followed by lifecycle transition to Standard-IA | A known lifecycle may avoid monitoring cost. |
| Infrequent, reproducible data where single-AZ storage is acceptable | S3 One Zone-IA | Confirm single-AZ availability meets the requirement. |
| Archive with hours-to-days retrieval tolerance | Glacier Flexible Retrieval or Deep Archive | Compare retrieval-time objectives and total cost. |

Choose storage using access frequency, retrieval time, availability, retention, object size, and minimum-duration charges. For deletion prevention, pair the storage class with Object Lock or Vault Lock.

### Redshift modeling and query options

- **KEY distribution:** Hashes rows on a chosen column. Matching distribution keys on large tables joined on that key can colocate rows and reduce network redistribution.
- **ALL distribution:** Copies a small, slowly changing dimension table to each compute node to avoid repeated broadcasts.
- **EVEN distribution:** Spreads rows without a chosen key when there is no clear join key.
- **Sort keys:** Order data on disk so zone maps can skip ranges. They do not replace distribution keys for reducing join data movement.

| Requirement | Redshift feature | Boundary |
| :--- | :--- | :--- |
| Query S3 data without loading it into Redshift storage | Redshift Spectrum | Uses external tables and the Glue Data Catalog. |
| Query live RDS or Aurora operational data | Federated query | Reads supported operational databases; CDC replicates changes into the warehouse. |
| Share live Redshift data across clusters, accounts, or Regions | Redshift Data Sharing | External Spectrum tables are outside native Redshift Data Sharing. |
| Execute a stored procedure | `CALL procedure_name(...)` | `CALL` executes a stored procedure. |
| Schedule recurring warehouse SQL | Query Editor v2 scheduler where suitable | Avoid custom compute when the built-in scheduler meets the need. |

For window functions, `ROW_NUMBER()` assigns a unique sequence even when sort values tie. `RANK()` gives ties the same rank and skips subsequent rank numbers.

### Catalog, Athena, DynamoDB, and OpenSearch

- The Glue Data Catalog is persistent metadata shared by Glue, Athena, EMR, and Redshift Spectrum.
- Crawlers are more likely to produce one coherent table when objects share compatible prefix layout, file format, compression, and schema.
- For deleted S3 partitions, `ALTER TABLE DROP PARTITION` removes stale catalog entries. `MSCK REPAIR TABLE` discovers and adds partitions.
- Use partition indexes and selective partition design to avoid scanning or listing unnecessary data.
- DynamoDB TTL expires eligible items based on a timestamp attribute without application code repeatedly deleting them.
- OpenSearch Service suits low-latency search, log analytics, and dashboards. Use Redshift for relational warehouse workloads.

## Domain 3: Data Operations and Support

This domain focuses on orchestration, monitoring, troubleshooting, data quality, schema evolution, and cost optimization.

### Choose the orchestration tool

| Tool | Best fit | Avoid when |
| :--- | :--- | :--- |
| Step Functions | Multi-step serverless workflows with retries, branches, and service integrations | A simple Glue-only DAG needs no broader orchestration. |
| MWAA | Existing Airflow DAGs, Python operators, or hybrid orchestration | A small AWS-only pipeline could use a serverless workflow. |
| Glue Workflows | Glue crawler and job sequences | Broad cross-service or on-premises orchestration is required. |
| EventBridge | Event-driven and scheduled triggers | The workflow needs long-running state, complex branching, or durable step-by-step status. |

Use EventBridge for event-driven triggers and Step Functions for workflows that need state, retries, or branching. Step Functions can coordinate Glue, Lambda, EMR, and Redshift operations.

### Pipeline monitoring and troubleshooting

| Symptom | Metric or evidence | Likely action |
| :--- | :--- | :--- |
| DMS CDC source latency is high | `CDCLatencySource` | Investigate source log capture and source database behavior. |
| DMS changes wait to apply at the target | `CDCIncomingChanges` and target-side latency | Investigate target capacity and apply throughput. |
| Kinesis write throttling with idle shards | Partition-key distribution and shard metrics | Fix hot-key skew before simply adding capacity. |
| Athena query remains queued | Workgroup concurrency and service quotas | Review concurrency demand and capacity options. |
| S3-triggered Lambda repeatedly invokes itself | Bucket notification and output location | Write results elsewhere or use filtered output prefixes. |

- A skewed Kinesis partition key, such as a small set of high-volume facility IDs, can create hot shards. Use a more evenly distributed key while preserving ordering requirements.
- Avoid writing an output object that matches the same S3 bucket notification rule. Use a separate output bucket or carefully designed prefix filters.
- Keep schema compatibility separate from data quality: a message can conform to a valid schema and still contain an invalid business value.
- Glue Schema Registry validates streaming schema compatibility for supported formats such as Avro, JSON Schema, and Protobuf.
- SageMaker Data Wrangler provides visual data preparation and profiling for interactive exploration.

### Cost optimization

- **EBS gp2 to gp3:** gp3 decouples capacity from provisioned IOPS and throughput. Right-size capacity, then explicitly provision required performance; default gp3 performance may not match the old volume.
- **QuickSight SPICE:** For many users viewing a dataset refreshed on a known schedule, import into SPICE and schedule refreshes to avoid repeated Athena scans per dashboard load.
- **Athena:** Use columnar formats, compression, and effective partitioning to reduce scanned bytes and query latency.
- **EMR:** Use transient clusters, managed scaling, Spot task nodes where interruption is acceptable, and S3 for persistent data.
- **Redshift:** Choose Serverless or provisioned capacity based on workload pattern, concurrency, and control requirements.

Cost optimization must preserve the performance target. A smaller gp3 volume with default IOPS may save storage cost but fail if the workload still needs the old volume's IOPS.

## Domain 4: Data Security and Governance

This domain covers encryption, access management, privacy, auditability, network boundaries, and governance.

### KMS authorization and encryption

- For SSE-KMS access, the caller's IAM permissions and the KMS key policy must both authorize the required use, especially across accounts.
- Use a symmetric customer-managed KMS key when Glue Data Catalog encryption requires customer-managed key control.
- Use DSSE-KMS when the requirement specifically calls for two independent layers of server-side encryption.
- Use Secrets Manager for credentials that need managed rotation or multi-Region replication.
- Use KMS grants when a service or principal needs delegated access without repeatedly editing a large key policy.
- For an encrypted cross-account read, trace role identity policy, bucket policy, KMS key policy, endpoint policy if used, and conditions such as `aws:SourceVpce` or `kms:ViaService`.

### Retention and fine-grained data access

| Mechanism | Protection | Boundary |
| :--- | :--- | :--- |
| S3 Object Lock Compliance mode | Strict WORM retention for object versions | No principal, including root, can delete or shorten retention before expiry. |
| S3 Object Lock Governance mode | Retention with authorized bypass | A principal with bypass permission can override retention. |
| S3 Glacier Vault Lock | WORM policy for Glacier vault archives | Complete the Vault Lock process to make the policy immutable. |
| MFA Delete or bucket-policy deny | Helps reduce accidental or unauthorized deletion | Use Object Lock Compliance mode when retention must resist root-level deletion. |

Object Lock requires versioning. Apply retention to existing object versions and set a default for new versions. Storage class selection handles access and retrieval requirements.

Lake Formation applies fine-grained permissions to data:

| Requirement | Lake Formation control |
| :--- | :--- |
| Hide a sensitive column | Column-level data filter |
| Restrict rows using a predicate | Row-level data filter |
| Apply access policy based on database, table, or column attributes | LF-Tag-based access control |

LF-Tags annotate databases, tables, and columns. Use row-level data filters for restrictions based on column values.

### Secrets, VPC access, and audit

- Store database credentials in Secrets Manager and retrieve them using an IAM role. Use rotation and multi-Region replication when required.
- S3 and DynamoDB gateway endpoints provide private service access through route tables and can avoid NAT processing for matching service requests.
- A Glue VPC connection requires suitable security-group rules so Glue components can communicate, including self-referencing rules in common configurations.
- For Lake Formation audit reporting, enable the relevant CloudTrail data events, deliver trail logs to S3, query them with Athena, and use Athena as the QuickSight data source.
- CloudTrail records API activity; Config records resource configuration. Choose the evidence source that answers the audit question.

## High-value distinctions

| Almost-right answer | Missing detail |
| :--- | :--- |
| “Firehose delivers streaming data to S3 in under 30 seconds by default.” | Buffering controls delivery latency; the default S3 buffer is 300 seconds. |
| “Kinesis Data Streams writes directly to S3.” | Add Firehose or a consumer such as KCL, Lambda, or Flink. |
| “Add shards to fix Kinesis write throttling.” | A hot partition key can leave one shard throttled while others are idle; fix key distribution. |
| “Run `MSCK REPAIR TABLE` after deleting S3 partitions.” | It adds discovered partitions; use `ALTER TABLE DROP PARTITION` to remove stale metadata. |
| “Use `RANK()` to give every Redshift row a unique rank.” | Ties receive the same rank; use `ROW_NUMBER()` for a unique sequence. |
| “Use sort keys to eliminate data movement in a large join.” | Sort keys help block skipping; matching distribution keys reduce join redistribution. |
| “Use Glue Job Bookmarks to validate data quality.” | Bookmarks track incremental processing; use Glue Data Quality rules for validation. |
| “Use Lake Formation tags to filter individual rows.” | LF-Tags apply to databases, tables, and columns; row restrictions use data filters. |
| “The IAM role allows `kms:Decrypt`, so the cross-account read works.” | The KMS key policy must also authorize the request. |
| “Glacier Deep Archive makes data undeletable.” | Object Lock or Vault Lock enforces retention; Deep Archive defines storage and retrieval behavior. |
| “A VPC Lambda needs a NAT gateway to reach S3.” | An S3 gateway endpoint can provide private S3 access without NAT processing charges. |
| “CloudTrail Lake connects directly to QuickSight.” | Use CloudTrail-to-S3 delivery, Athena, and QuickSight for this reporting path. |
| “An EMR instance fleet supports custom automatic scaling.” | Use uniform instance groups when custom automatic scaling is required. |
| “Default gp3 performance matches a high-IOPS gp2 volume.” | Provision required IOPS and throughput explicitly after conversion. |
| “S3 event notifications can directly target an SQS FIFO queue.” | Route S3 events through EventBridge when an SQS FIFO target is required. |

## Final review strategy

At 130 minutes for 65 questions, the average budget is about 2 minutes per question. Use that as a pacing guide, spending more time on long scenarios and moving on when a question is resolved.

1. Read the business and SLA requirement: freshness, throughput, availability, retrieval time, and cost.
2. Classify the data flow: batch, stream, CDC, SaaS integration, or event-triggered files.
3. Choose the transformation and orchestration boundary: Glue, EMR, Lambda, Flink, Step Functions, MWAA, or Glue Workflows.
4. Verify storage and query design: format, compression, partitioning, distribution, file size, and query engine.
5. Trace access and governance: IAM, KMS key policies, Lake Formation filters, VPC endpoints, and audit events.
6. Eliminate options that miss a quota or add unnecessary infrastructure.

Ask: **What is the required data latency, and which part of the proposed architecture actually meets it?** A pipeline is only as fast as its slowest buffer, transformation, and delivery step.
