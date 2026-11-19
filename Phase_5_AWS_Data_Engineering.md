# Phase 5 — AWS for Data Engineering (27 Lessons)

> Built from your syllabus sheet **"Cloud Data Engineer" → Core Skills → Option 3 – Amazon Web Services**.
> Goal: be able to **build a working data pipeline on AWS inside an 8-hour hackathon** and explain it in the interview that follows.

---

## 0. How to use this file

1. Read lessons in order. Each lesson has: **Simple idea → Must-know points → Hands-on / code → Hackathon tip**.
2. Do the **hands-on** parts in a real (free-tier) AWS account. Reading alone will not work for a hackathon.
3. The last sections give you a **hackathon blueprint, time plan, error-fix list, and interview Q&A**.

### Syllabus → Lesson map

| Syllabus module | Lessons |
|---|---|
| Basics of AWS | 1–5 |
| Amazon S3 | 6–8 |
| Data Warehousing (Redshift) | 9–10 |
| AWS Glue | 11–12 |
| AWS Glue Studio | 13 |
| Data Lake Architecture | 14 |
| Streaming (Kinesis) | 15–16 |
| Serverless Data Processing (Lambda) | 17–18 |
| Workflow Orchestration (Step Functions) | 19 |
| Apache Spark on AWS (EMR) | 20 |
| Query Services (Athena) | 21–22 |
| Data Governance (Lake Formation) | 23 |
| Data Migration (DMS) | 24 |
| API & Ingestion (API Gateway) | 25 |
| DevOps for Data Engineering | 26 |
| Gen AI on AWS (Bedrock) | 27 (full depth in Phase 6) |
| Capstone | Section B |

### The one-picture mental model

```
 SOURCES            INGEST              STORE (Data Lake)        TRANSFORM         SERVE / ANALYSE        GenAI
 files (CSV)  ──►  S3 upload      ─┐
 APIs         ──►  Lambda/API GW  ─┼─►  S3 RAW  ──►  Glue / EMR / Lambda ──►  S3 CURATED (Parquet) ──► Athena / Redshift ──► Bedrock
 databases    ──►  DMS            ─┤        ▲              ▲                        │
 streams      ──►  Kinesis→Firehose┘       │        Glue Catalog + Lake Formation   │
                                  Orchestration: Step Functions / Glue Workflows / EventBridge
                                  Monitoring: CloudWatch, CloudTrail     Security: IAM, KMS
```

---

# PART A — THE 27 LESSONS

---

## MODULE 1 — BASICS OF AWS

### Lesson 1 — Cloud basics, AWS account structure, Regions & Availability Zones

**Simple idea:** Instead of buying servers, you **rent** them by the hour from Amazon. AWS is a giant rental shop with 200+ services.

**Must-know points**
- **Cloud models:** IaaS (EC2 – you manage OS), PaaS (Glue/RDS – AWS manages most), SaaS (a finished app). Serverless = you only supply code/config, AWS runs the servers (Lambda, Glue, Athena).
- **Shared responsibility model:** AWS secures the *cloud* (hardware, data centers). **You** secure what you put *in* the cloud (IAM, data, encryption, security groups).
- **AWS account structure**
  - **Root user** = the email you signed up with. Has full power. Enable MFA, create no access keys, don't use it daily.
  - **IAM users/roles** = what you actually use.
  - **AWS Organizations** = group many accounts (dev/test/prod) under one bill, apply **Service Control Policies (SCPs)**.
  - Each account has a 12-digit **Account ID** (used in ARNs).
- **Region** = a geographic area (e.g. `ap-south-1` Mumbai, `us-east-1` N. Virginia). Data stays in a region unless *you* move it. Pick a region close to users/data; prices and available services differ.
- **Availability Zone (AZ)** = one or more separate data centers inside a region (e.g. `ap-south-1a`). Spread across AZs for high availability.
- **Edge locations** = CDN points (CloudFront).
- **ARN** (Amazon Resource Name) = unique ID of any resource:
  `arn:aws:s3:::my-bucket/raw/data.csv`, `arn:aws:iam::123456789012:role/GlueRole`
- **Free tier & billing safety:** set a **Budget alert** on day 1. Delete resources after practice (Redshift, EMR, NAT gateways, Kinesis shards cost money while running).

**Hackathon tip:** Choose **one region** and use it for everything (S3, Glue, Athena, Lambda must be in the same region or you'll hit "table/bucket not found" errors).

---

### Lesson 2 — IAM: users, groups, roles, policies

**Simple idea:** IAM is the **security guard**. Every action in AWS ("read this file", "start this job") is checked: *Who are you? Are you allowed?*

**Must-know points**
| Term | Meaning |
|---|---|
| **User** | A person/app with long-term credentials (password / access keys) |
| **Group** | A set of users sharing permissions |
| **Role** | An identity with **temporary** credentials that a *service or person assumes* (Lambda, Glue, EC2 use roles) |
| **Policy** | A JSON document listing allowed/denied actions |
| **Trust policy** | On a role: *who may assume this role* (e.g. `lambda.amazonaws.com`) |
| **Permissions policy** | On a role/user: *what it may do* |

- **Policy JSON anatomy:** `Effect` (Allow/Deny) · `Action` (e.g. `s3:GetObject`) · `Resource` (ARN) · optional `Condition`.
- **Rules of evaluation:** everything is **denied by default** → an **explicit Allow** is needed → an **explicit Deny always wins**.
- **Least privilege:** give only what's needed, on only the resources needed.
- **Managed vs inline policies:** AWS-managed (e.g. `AmazonS3ReadOnlyAccess`), customer-managed (yours, reusable), inline (embedded in one identity).
- **Use roles, not access keys, for services.** A Lambda gets its permissions from its **execution role**. An EC2 instance gets them from an **instance profile**.
- **MFA** on root and admin users.

**Least-privilege example — read only one bucket**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::my-data-bucket"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-data-bucket/*"
    }
  ]
}
```
> Note the two different resources: `ListBucket` applies to the **bucket** ARN; `GetObject` applies to **objects** (`/*`). Mixing these up is the #1 beginner mistake.

**Trust policy for a Lambda role**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

**Roles you will create in this course**
| Role | Trusted service | Typical permissions |
|---|---|---|
| Lambda execution role | `lambda.amazonaws.com` | S3 read/write, CloudWatch Logs (`AWSLambdaBasicExecutionRole`) |
| Glue service role | `glue.amazonaws.com` | `AWSGlueServiceRole` + S3 access to your buckets |
| Redshift role | `redshift.amazonaws.com` | S3 read (for COPY), Glue catalog (for Spectrum) |
| Step Functions role | `states.amazonaws.com` | Invoke Lambda, start Glue, publish SNS |
| EMR roles | `elasticmapreduce.amazonaws.com` / `ec2.amazonaws.com` | Cluster + instance profile |

**Test a policy:** IAM **Policy Simulator**, or call the CLI and see if you get `AccessDenied` (HTTP 403).

**Hackathon tip:** 70% of "it doesn't work" = missing IAM permission. Read the error: it names the **action** and **resource** that was denied. Add exactly that.

---

### Lesson 3 — AWS Console and CLI

**Simple idea:** The **Console** is the website with buttons. The **CLI** is the same thing typed as commands — repeatable and scriptable.

**Setup**
```bash
aws configure
# AWS Access Key ID: ...
# AWS Secret Access Key: ...
# Default region name: ap-south-1
# Default output format: json

aws sts get-caller-identity        # "Who am I?" – first command to run when debugging
```
> Never put access keys in code or GitHub. Use `aws configure`, environment variables, or roles.

**Commands you should know by heart**
```bash
# S3
aws s3 mb s3://my-data-bucket-2026              # make bucket
aws s3 ls s3://my-data-bucket-2026/raw/         # list
aws s3 cp orders.csv s3://my-data-bucket-2026/raw/orders/
aws s3 sync ./data s3://my-data-bucket-2026/raw/   # upload only changed files
aws s3 rm s3://my-data-bucket-2026/raw/orders.csv

# Lambda
aws lambda invoke --function-name my-func out.json
aws lambda list-functions

# Glue
aws glue start-crawler --name raw-crawler
aws glue start-job-run --job-name orders-etl
aws glue get-job-run --job-name orders-etl --run-id jr_123

# Athena
aws athena start-query-execution \
  --query-string "SELECT count(*) FROM sales_db.orders" \
  --result-configuration OutputLocation=s3://my-data-bucket-2026/athena-results/

# Logs
aws logs tail /aws/lambda/my-func --follow

# EC2
aws ec2 describe-instances --query "Reservations[].Instances[].[InstanceId,State.Name]"
```
- `--query` uses **JMESPath** to filter JSON; `--output table|json|text` changes format.
- **CloudShell** (icon in console top bar) gives a browser terminal with CLI pre-authenticated — great in a hackathon when your laptop setup fails.
- **Python SDK = boto3** (`pip install boto3`): `boto3.client("s3")`.

---

### Lesson 4 — Monitoring (CloudWatch), Logging (CloudTrail), Error monitoring

**Simple idea:** **CloudWatch** = "how is my stuff *performing* and what did it *print*?" **CloudTrail** = "who *did what* in my account?" (audit).

**CloudWatch**
- **Metrics:** numbers over time (EC2 `CPUUtilization`, Lambda `Errors`, `Duration`, Glue job metrics, Kinesis `IncomingBytes`).
- **Logs:** every Lambda `print()` / Glue driver log lands in a **Log Group** (`/aws/lambda/<name>`, `/aws-glue/jobs/output`) → **Log Streams**. **Logs Insights** lets you query logs with a mini query language.
- **Alarms:** watch one metric against a threshold. States: `OK`, `ALARM`, `INSUFFICIENT_DATA`. An alarm can notify an **SNS topic** (email/SMS) or trigger an action.
- **Dashboards:** graphs of many metrics.
- **EventBridge (CloudWatch Events):** rules that react to events or run on a schedule (`cron(0 2 * * ? *)` = 2 AM UTC daily). Used to **schedule pipelines**.
- EC2 doesn't report **memory/disk usage** by default → install the **CloudWatch agent**.

**Create an alarm (typical exam task):** metric `CPUUtilization` > 85% for 2 of 3 periods of 5 min → SNS topic → your email (confirm the subscription email!).

**CloudTrail**
- Records **API calls** (who, when, from which IP, which resource). Management events are on by default (90-day Event history); create a **Trail** to store logs in S3 long-term.
- Use it for: security investigations, "who deleted this bucket?", compliance.

**Error monitoring checklist (for pipelines)**
1. Lambda: check **CloudWatch Logs** + metric `Errors`; configure a **Dead-Letter Queue (SQS)** or on-failure destination.
2. Glue: job run **status + error message**, driver/executor logs, `Spark UI`.
3. Step Functions: **Execution history** shows which state failed with its input/output.
4. Alarm on failures → SNS email.

**Logging pattern in Python**
```python
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)
logger.info("Processing file %s", key)       # visible in CloudWatch Logs
```

---

### Lesson 5 — EC2 virtual machines & SSH

**Simple idea:** EC2 = a **rented computer** in the cloud. In data engineering you use it for jump boxes, running scripts, small tools — not as the main engine (that's Glue/EMR/Lambda).

**Must-know points**
- **AMI** = template (OS + software), e.g. Amazon Linux 2023, Ubuntu.
- **Instance type** = size: `t3.micro` (free tier eligible, general), `m5` general, `c5` compute, `r5` memory.
- **EBS** = the instance's disk (persists if stopped). **Instance store** = temporary disk.
- **Key pair** = `.pem` file used to SSH. Lose it → can't log in.
- **Security Group (SG)** = **instance-level firewall**, **stateful**, **allow rules only** (default: deny all inbound). Open port **22 (SSH)** only to *your IP*, never `0.0.0.0/0`.
- **NACL** = **subnet-level** firewall, **stateless**, allow + deny rules.
- **VPC** = your private network; **subnet** = slice of VPC in one AZ; **public subnet** has a route to an **Internet Gateway**; **route table** decides where traffic goes; **NAT gateway** lets private subnets reach the internet outward only.
- **Pricing:** On-Demand, Reserved/Savings Plans, **Spot** (cheap, can be interrupted). **Stop** = pay only disk; **Terminate** = deleted.
- **Status checks 2/2:** *System status* (AWS hardware) + *Instance status* (your OS). Both must pass.

**SSH**
```bash
chmod 400 mykey.pem                              # required, else "UNPROTECTED PRIVATE KEY FILE"
ssh -i mykey.pem ec2-user@<public-ip>            # Amazon Linux
ssh -i mykey.pem ubuntu@<public-ip>              # Ubuntu
```
Or use **EC2 Instance Connect / Session Manager** (no keys needed).

**Troubleshooting SSH timeout:** (1) SG allows port 22 from your IP? (2) Instance in public subnet with public IP? (3) Route table has IGW route? (4) NACL not blocking? (5) Instance running with 2/2 checks?

**Give the instance AWS access safely:** attach an **IAM role** (instance profile) → run `aws s3 ls` on the box with no keys.

**Investigate a CPU spike (common question):** SSH in → `top` / `ps aux --sort=-%cpu | head` → find process → check logs in `/var/log` → fix/kill/scale.

---

## MODULE 2 — AMAZON S3

### Lesson 6 — S3: buckets, objects, storage classes

**Simple idea:** S3 = **unlimited folder in the cloud** (really: a key-value store of files). It is the **foundation of every AWS data lake**.

**Must-know points**
- **Bucket**: container; name is **globally unique**, lives in **one region**. **Object** = file + metadata, identified by a **key** (`raw/orders/2026/11/orders.csv`). Max object size **5 TB**; use **multipart upload** for large files.
- **"Folders" are fake** — they're just key prefixes. Prefixes matter for partitioning (Lesson 14, 22).
- **Durability 99.999999999% (11 nines)**; **strong read-after-write consistency**.
- **Storage classes** (cost vs retrieval speed):

| Class | Use for |
|---|---|
| **S3 Standard** | Frequently accessed (default, raw/curated data) |
| **Intelligent-Tiering** | Unknown/changing access; auto-moves between tiers |
| **Standard-IA** | Infrequent access, still millisecond retrieval, 3+ AZ |
| **One Zone-IA** | Infrequent, re-creatable data, 1 AZ only |
| **Glacier Instant Retrieval** | Archive, rare access, ms retrieval |
| **Glacier Flexible Retrieval** | Archive, minutes–hours retrieval |
| **Glacier Deep Archive** | Cheapest, 12–48 h retrieval, compliance archives |

**Create bucket (CLI):** `aws s3 mb s3://yourname-hcl-lake-2026 --region ap-south-1`

**Suggested bucket layout for the hackathon**
```
s3://yourname-lake/
   raw/orders/            ← untouched source files
   curated/orders/        ← cleaned Parquet, partitioned
   trusted/               ← business-ready tables (optional)
   scripts/               ← Glue job .py files
   athena-results/        ← Athena query output
   temp/                  ← Glue/Redshift temp
```

---

### Lesson 7 — Uploading/transferring data, versioning, lifecycle policies

**Uploading & transferring**
- Console drag-and-drop, `aws s3 cp/sync`, SDK (`boto3`), **multipart upload** (parallel parts, resumable), **Transfer Acceleration** (faster long-distance via edge), **DataSync** (on-prem → S3), **Snowball** (physical device for petabytes), **Transfer Family** (SFTP into S3), **DMS** (database → S3).
```python
import boto3
s3 = boto3.client("s3")
s3.upload_file("orders.csv", "yourname-lake", "raw/orders/orders.csv")
s3.download_file("yourname-lake", "raw/orders/orders.csv", "local.csv")
obj = s3.get_object(Bucket="yourname-lake", Key="raw/orders/orders.csv")
text = obj["Body"].read().decode("utf-8")
```

**Versioning**
- Keeps **every version** of an object. Overwrite → new version ID. Delete → adds a **delete marker** (old versions remain, so you can recover).
- Once enabled you can only **suspend**, never fully disable.
- Protects against accidental delete/overwrite; costs extra storage.
- Optional **MFA Delete** for extra protection.

**Lifecycle policies** (automatic cost control) — rules per prefix:
- **Transition**: Standard → Standard-IA after 30 days → Glacier after 90.
- **Expiration**: delete after 365 days; delete **noncurrent versions** after N days; **abort incomplete multipart uploads** after 7 days.
```json
{
  "Rules": [{
    "ID": "raw-to-glacier",
    "Status": "Enabled",
    "Filter": { "Prefix": "raw/" },
    "Transitions": [
      { "Days": 30, "StorageClass": "STANDARD_IA" },
      { "Days": 90, "StorageClass": "GLACIER" }
    ],
    "Expiration": { "Days": 365 }
  }]
}
```
**Other features:** **Replication** (CRR/SRR copies to another bucket/region), **Event Notifications** (Lesson 18), **Object Lock** (WORM).

---

### Lesson 8 — S3 security and access control

**Layers of S3 security (know all)**
1. **Block Public Access** – bucket/account switch that overrides any public policy. **Keep it ON** for data lakes.
2. **IAM policies** (identity-based) – attached to users/roles: "this role can read this bucket".
3. **Bucket policy** (resource-based) – attached to the bucket; also used for **cross-account** access.
4. **ACLs** – legacy; new buckets have ACLs disabled (Object Ownership = *Bucket owner enforced*). Avoid.
5. **Encryption**
   - **At rest:** **SSE-S3** (AWS-managed keys; default for new objects), **SSE-KMS** (your KMS key; audit via CloudTrail, extra control), SSE-C (your own key per request).
   - **In transit:** HTTPS/TLS. Enforce with a bucket policy condition `aws:SecureTransport = false → Deny`.
6. **Pre-signed URLs** – temporary link to one object.
7. **VPC Gateway Endpoint for S3** – keeps traffic on the AWS network (no internet).
8. **Logging/monitoring:** server access logs, CloudTrail data events, **Macie** (finds PII), **Access Analyzer**.

**Bucket policy: deny non-HTTPS**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyInsecureTransport",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": ["arn:aws:s3:::yourname-lake", "arn:aws:s3:::yourname-lake/*"],
    "Condition": { "Bool": { "aws:SecureTransport": "false" } }
  }]
}
```

**Typical hackathon/assessment task:** *"Create a private bucket, upload a report, give a user read-only access to only that bucket, prove other buckets are blocked."* → Block Public Access ON → upload → IAM policy from Lesson 2 → test with CLI/Policy Simulator; a request to another bucket returns **403 AccessDenied**.

---

## MODULE 3 — DATA WAREHOUSING (AMAZON REDSHIFT)

### Lesson 9 — Redshift: cluster setup, loading data (COPY), querying

**Simple idea:** Redshift is AWS's **data warehouse**: a database built for **analytics over huge tables** (not for tiny transactions). It's **columnar** and **MPP (massively parallel processing)**.

**Must-know points**
- **OLTP vs OLAP:** RDS/Aurora = OLTP (many small reads/writes). Redshift = OLAP (few big aggregate queries).
- **Columnar storage:** reads only the columns a query needs + compresses well → fast `SUM(sales)`.
- **Architecture:** **Leader node** (parses SQL, makes the plan) + **Compute nodes** (store data & run in parallel; each split into **slices**).
- **Deployment:** **Provisioned cluster** (choose node type, e.g. RA3) or **Redshift Serverless** (pay per usage in RPUs; easiest for hackathons — use a **namespace + workgroup**).
- Based on PostgreSQL syntax. **Primary/foreign keys are informational only — not enforced.**
- Query via **Query Editor v2** (console), JDBC/ODBC, or **Redshift Data API** (boto3, no connection).

**Cluster/serverless setup steps**
1. Create IAM role (trusted by `redshift.amazonaws.com`) with S3 read (+ Glue read for Spectrum).
2. Create Serverless workgroup or provisioned cluster; attach the role; set admin user/password; choose VPC & security group.
3. Open Query Editor v2 and connect.

**Create table + load with COPY (fastest way in)**
```sql
CREATE TABLE orders (
  order_id     INT,
  customer_id  INT,
  order_date   DATE,
  amount       DECIMAL(10,2),
  country      VARCHAR(30)
);

COPY orders
FROM 's3://yourname-lake/curated/orders/'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftCopyRole'
FORMAT AS CSV
IGNOREHEADER 1
REGION 'ap-south-1';

-- Parquet:
-- COPY orders FROM 's3://.../orders/' IAM_ROLE '...' FORMAT AS PARQUET;
```
- **Never load row-by-row with INSERT** — use COPY (parallel from many files; split big files into multiple files ≈ multiple of slices).
- Errors → `SELECT * FROM stl_load_errors ORDER BY starttime DESC LIMIT 10;` (`sys_load_error_detail` in newer versions).
- Export back to S3: `UNLOAD ('select * from orders') TO 's3://.../out/' IAM_ROLE '...' PARQUET;`

**Query:** normal SQL (joins, window functions, CTEs — all from Phase 2).

---

### Lesson 10 — Redshift: distribution keys, sort keys, optimization, Spectrum, security

**Distribution style** (how rows are spread across nodes — goal: avoid moving data during joins)
| Style | Meaning | Use when |
|---|---|---|
| **AUTO** | Redshift decides (default) | Start here |
| **KEY** | Rows with the same key value go to the same node | Large fact + large dimension joined on that column |
| **ALL** | Full copy on every node | Small dimension tables |
| **EVEN** | Round-robin | No clear join key |

**Sort key** – physical order on disk; lets Redshift skip blocks (zone maps). Choose the column used in `WHERE` ranges (usually **date**). Types: **compound** (default, ordered by columns in sequence) and **interleaved** (equal weight; rarely needed).
```sql
CREATE TABLE sales (
  sale_id BIGINT, customer_id INT, sale_date DATE, amount DECIMAL(10,2)
)
DISTSTYLE KEY DISTKEY (customer_id)
SORTKEY (sale_date);
```

**Performance optimization checklist**
- Right **DIST/SORT** keys (or let AUTO handle it), **column compression encodings** (`ANALYZE COMPRESSION`, AUTO by default).
- **`ANALYZE`** (update stats) and **`VACUUM`** (re-sort, reclaim deleted space) — mostly automatic now.
- Use **COPY** with Parquet/compressed files; avoid `SELECT *`.
- **Materialized views** for repeated heavy aggregations (`REFRESH MATERIALIZED VIEW`).
- **Result caching** (same query returns instantly), **Concurrency Scaling**, **WLM** (workload management/queues).
- Read plans with `EXPLAIN`; watch for `DS_BCAST_INNER` / `DS_DIST_BOTH` (data shuffling = bad).

**Redshift Spectrum – query S3 without loading it**
```sql
CREATE EXTERNAL SCHEMA spectrum_lake
FROM DATA CATALOG DATABASE 'sales_db'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftSpectrumRole'
REGION 'ap-south-1';

SELECT country, SUM(amount)
FROM spectrum_lake.orders          -- table lives in S3, defined in Glue Catalog
GROUP BY country;
```
Use for **cold/huge data** in the lake; join it with local hot tables.

**Security:** run in a **VPC**, private/not publicly accessible; **encryption at rest (KMS) and in transit (SSL)**; **IAM roles** for COPY/UNLOAD instead of keys; DB-level `GRANT/REVOKE` and roles; row-level/column-level security; audit logging to S3/CloudWatch; **Secrets Manager** for credentials.

**Redshift vs Athena (interview favorite):** Redshift = fast, repeated BI queries, you pay for compute; Athena = ad-hoc queries directly on S3, serverless, pay per data scanned.

---

## MODULE 4 — AWS GLUE

### Lesson 11 — Glue: Crawlers, Data Catalog, schema inference

**Simple idea:** **Glue Data Catalog** = the **phone book of your data** (which tables exist, columns, types, S3 location). A **Crawler** = a robot that **looks at your S3 files and writes the phone-book entry automatically**.

**Must-know points**
- **Data Catalog:** a central, Hive-metastore-compatible metadata store. Holds **databases → tables → partitions**. It stores *metadata only* (the data stays in S3). Used by **Athena, Redshift Spectrum, EMR, Glue, Lake Formation**.
- **Crawler workflow**
  1. Create a **database** (`sales_db`).
  2. Create crawler → data source = `s3://yourname-lake/raw/orders/` → IAM role (Glue service role with S3 read) → target database → schedule (on-demand / hourly / daily).
  3. Run crawler → a **table** appears with inferred columns + partitions.
- **Schema inference:** the crawler samples files, detects format (CSV/JSON/Parquet/ORC/Avro), column names & types, and **partition folders** like `year=2026/month=11/` → partition columns.
- **Schema changes:** crawler settings decide whether to *update the table*, *add new columns*, or *ignore*; deleted objects → *mark deprecated* / *delete*.
- **Gotchas**
  - CSV with header: crawler infers headers only if the first row looks different from data; check column names (`col0, col1` = it failed) → fix manually or use a custom CSV classifier.
  - Mixed schemas in one folder → multiple tables / messy table. **One dataset per folder.**
  - Crawler costs money while running (charged per DPU-hour); don't schedule too often.
  - Alternative to crawling: define table by **Athena DDL** (Lesson 21) or `MSCK REPAIR TABLE` / partition projection for new partitions.

**Hackathon tip:** After the crawler runs, go to **Athena** and run `SELECT * FROM sales_db.orders LIMIT 10;` — instant proof your ingestion + catalog works.

---

### Lesson 12 — Glue ETL jobs (PySpark), transformations, scheduling, monitoring

**Simple idea:** A **Glue job** = a **serverless Spark (PySpark) program** that reads from the lake, transforms, and writes back. No cluster to manage.

**Must-know points**
- **Job types:** **Spark** (main one), **Python shell** (small scripts, 1 DPU/0.0625 DPU), **Streaming** (Kinesis/Kafka), **Ray**.
- **DPU** = unit of compute (4 vCPU + 16 GB). Worker types `G.1X`, `G.2X`, etc. Start small (2 workers) in hackathons.
- **DynamicFrame vs DataFrame**
  - **DynamicFrame** = Glue's schema-flexible frame; handles messy columns (`choice` types). Methods: `ApplyMapping`, `ResolveChoice`, `DropNullFields`, `Filter`, `Join`, `Relationalize` (flatten nested JSON).
  - **DataFrame** = normal Spark. Convert freely: `dyf.toDF()` / `DynamicFrame.fromDF(df, glueContext, "name")`. Most people do heavy logic in plain PySpark (what you learned in Phase 4).
- **Job bookmarks:** Glue remembers what it already processed → **incremental loads** (don't reprocess old files). Enable in job details.
- **Glue version** (4.0/5.0) = Spark + Python versions.

**Standard Glue script skeleton**
```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame
from pyspark.context import SparkContext
from pyspark.sql import functions as F

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# 1) READ from the Data Catalog (raw layer)
dyf = glueContext.create_dynamic_frame.from_catalog(
    database="sales_db", table_name="raw_orders",
    transformation_ctx="read_raw"            # needed for bookmarks
)
df = dyf.toDF()

# 2) TRANSFORM (cleaning)
clean = (df
    .dropDuplicates(["order_id"])
    .filter(F.col("order_id").isNotNull() & F.col("customer_id").isNotNull())
    .withColumn("amount", F.col("amount").cast("double"))
    .withColumn("order_date", F.to_date("order_date", "yyyy-MM-dd"))
    .filter(F.col("order_date").isNotNull())
    .fillna({"amount": 0})
    .withColumn("year",  F.year("order_date"))
    .withColumn("month", F.month("order_date"))
)

# 3) WRITE to curated layer as partitioned Parquet
out = DynamicFrame.fromDF(clean, glueContext, "out")
glueContext.write_dynamic_frame.from_options(
    frame=out,
    connection_type="s3",
    connection_options={
        "path": "s3://yourname-lake/curated/orders/",
        "partitionKeys": ["year", "month"]
    },
    format="parquet",
    transformation_ctx="write_curated"
)
job.commit()
```

**Common transformations to practice** (all PySpark from Phase 4): filter, select, rename (`withColumnRenamed`), cast, `dropDuplicates`, `fillna/dropna`, `when/otherwise`, `groupBy().agg()`, joins (`inner`, `left`, `left_anti`), window functions, `explode` for arrays, `to_date`, string cleaning (`trim`, `lower`, `regexp_replace`).

**Scheduling**
- **Triggers:** *Scheduled* (cron), *On-demand*, *Conditional* (run job B when job A succeeds).
- **Glue Workflows:** chain crawler → job → crawler visually.
- **EventBridge** or **Step Functions** (Lesson 19) for event-driven / richer control.

**Monitoring & debugging**
- **Job runs** tab: status (`RUNNING/SUCCEEDED/FAILED`), duration, DPU, error message.
- **CloudWatch Logs:** `/aws-glue/jobs/output` (your `print`), `/aws-glue/jobs/error` (stack traces).
- **Spark UI** (enable "Spark UI" in job details) to see stages/skew.
- Typical failures: `AccessDenied` (role lacks S3 permission) · wrong column type · path doesn't exist · out-of-memory (more workers/G.2X, filter early) · "Table not found" (wrong region/database).
- Alarms: CloudWatch alarm on job failure → SNS.

---

### Lesson 13 — AWS Glue Studio (visual ETL, drag-and-drop, debugging)

**Simple idea:** Glue Studio = **build the same ETL job by drawing boxes**; it generates the PySpark script for you.

**Must-know points**
- **Visual editor:** Source node → Transform nodes → Target node. Click to configure.
- **Common nodes:** *Data source* (S3 / Catalog table / JDBC / Kinesis), *ApplyMapping* (rename/cast columns), *Filter*, *Join*, *Aggregate*, *Drop Duplicates*, *Drop Null Fields*, *SQL Query* (write Spark SQL), *Custom transform*, *Data target* (S3 in Parquet, Catalog table, Redshift).
- **Script tab** shows generated code → you can switch to *"Edit script"* (after that you can't go back to visual).
- **Notebook interface / interactive sessions:** develop code cell by cell with real Spark.
- **Data preview** per node (needs an IAM role + session) to see rows after each step.
- **Job debugging:** *Run → Runs tab → Logs/Error logs*, preview data at each node, check **Job details** (IAM role, workers, bookmark, temp path, script path).
- **Glue Data Quality** rules (e.g. `IsComplete "order_id"`, `ColumnValues "amount" > 0`) can be added as a node.

**When to choose what:** Studio for speed and clarity in a hackathon; hand-written PySpark when logic is complex.

**Quick Studio recipe (5 min):** Source = `raw_orders` (Catalog) → *ApplyMapping* (cast `amount` to double) → *Drop Duplicates* on `order_id` → *Filter* `amount > 0` → Target = S3 `curated/orders/` Parquet (Snappy) + "Create a table in the Data Catalog" → Save → Run.

---

## MODULE 5 — DATA LAKE ARCHITECTURE

### Lesson 14 — Building a data lake on S3, layers, governance basics

**Simple idea:** A **data lake** = a cheap central store (S3) that keeps **all data in raw form**, then **refines it in stages** so analysts get clean, trusted data.

**Layers (your syllabus: Raw, Curated, Trusted)** — same idea as Bronze/Silver/Gold
| Layer | What's inside | Format | Who uses |
|---|---|---|---|
| **Raw** (Bronze) | Exact copy of source (CSV/JSON/XML, API dumps). Never edited, kept for replay | Original | Data engineers |
| **Curated** (Silver) | Cleaned, de-duplicated, typed, standardized | **Parquet**, partitioned | Engineers, analysts |
| **Trusted** (Gold) | Business-ready aggregates, joined/modelled (star schema), DQ-checked | Parquet / Redshift tables | BI, ML, execs |

**Build checklist**
1. S3 bucket(s) with prefixes per layer (and per source: `raw/orders/`).
2. **Partition** by date or another common filter column: `.../order_date=2026-11-19/` (cuts scan cost — Athena/Spark read only needed folders).
3. Convert to **columnar compressed format** (Parquet + Snappy) in curated: smaller + faster than CSV.
4. Avoid the **small-files problem** (thousands of tiny files slow everything) → combine to ~128 MB–1 GB files (`coalesce`/`repartition`, Firehose buffering).
5. Register everything in the **Glue Data Catalog**.
6. Add **governance**: Lake Formation permissions, encryption (KMS), Block Public Access, lifecycle rules, naming conventions, data quality checks, lineage notes.
7. Orchestrate with Step Functions/Glue workflows; monitor with CloudWatch.

**Data lake vs warehouse vs lakehouse:** lake = cheap, any format, schema-on-read; warehouse = structured, fast SQL, schema-on-write; lakehouse = lake + warehouse features (Delta/Iceberg tables, ACID).

**Governance basics:** who can see what (IAM + Lake Formation) · metadata & catalog · data quality · lineage (where data came from) · retention (lifecycle) · PII protection (mask/encrypt, Macie).

**Hackathon tip:** In your final presentation, **draw these three layers**. Interviewers love hearing "Raw is immutable, Curated is Parquet partitioned by date, Trusted is aggregated for analytics."

---

## MODULE 6 — STREAMING (AMAZON KINESIS)

### Lesson 15 — Kinesis Data Streams

**Simple idea:** Kinesis Data Streams = a **conveyor belt for real-time events** (clicks, sensor readings, orders). Producers put records on; consumers take them off within seconds.

**Must-know points**
- **Batch vs streaming:** batch = process data collected over time (hourly/daily). Streaming = process each event as it arrives (low latency).
- **Stream → Shards.** Each shard handles up to **1 MB/s or 1,000 records/s in** and **2 MB/s out**. More shards = more throughput. **On-demand mode** auto-scales; **Provisioned** you set shards.
- **Partition key** decides which shard a record goes to (same key → same shard → ordered per key). Pick a high-cardinality key (e.g. `customer_id`) to avoid a **hot shard**.
- **Retention:** 24 hours default, extendable up to 365 days. Records can be **replayed**.
- **Producers:** SDK/boto3 `put_record(s)`, **KPL**, Kinesis Agent, apps, IoT.
- **Consumers:** **Lambda** (event source mapping), **KCL** apps, **Firehose**, **Managed Service for Apache Flink** (formerly Kinesis Data Analytics), Spark Streaming/Glue streaming. **Enhanced fan-out** gives each consumer dedicated 2 MB/s.
- Delivery is **at-least-once** → make consumers **idempotent** (handle duplicates).
- Record = partition key + sequence number + data blob (≤ 1 MB).

**Producer (Python)**
```python
import boto3, json, time, random
kinesis = boto3.client("kinesis", region_name="ap-south-1")

for i in range(20):
    event = {"order_id": i, "customer_id": random.randint(1, 5),
             "amount": round(random.uniform(10, 500), 2), "ts": time.time()}
    kinesis.put_record(
        StreamName="orders-stream",
        Data=json.dumps(event).encode("utf-8"),
        PartitionKey=str(event["customer_id"]),
    )
```

**Lambda consumer** (records arrive base64-encoded)
```python
import base64, json

def lambda_handler(event, context):
    for rec in event["Records"]:
        payload = json.loads(base64.b64decode(rec["kinesis"]["data"]))
        print("Got:", payload)
        # validate / enrich / write to S3 or DynamoDB
    return {"processed": len(event["Records"])}
```

---

### Lesson 16 — Kinesis Data Firehose, real-time processing, integration with S3/Redshift

**Simple idea:** **Firehose** (now named **Amazon Data Firehose**) = a **fully managed delivery truck**: it takes streaming data and **delivers it to a destination** (S3, Redshift, OpenSearch, Splunk, HTTP endpoint) with **no code and no shard management**.

**Data Streams vs Firehose**
| | Data Streams | Firehose |
|---|---|---|
| Purpose | Custom real-time processing | Load/deliver data |
| Management | You manage shards (or on-demand) | Fully managed, auto-scale |
| Latency | ~sub-second | **Near-real-time** (buffer 60 s+ or size-based) |
| Consumers | Many custom | Fixed destinations |
| Replay | Yes | No |
| Code needed | Yes | Optional Lambda transform |

**Firehose features**
- **Buffering hints:** deliver when buffer hits size (e.g. 5–128 MB) **or** time (60–900 s), whichever first.
- **Transformation:** attach a **Lambda** to clean/enrich records in flight.
- **Format conversion:** JSON → **Parquet/ORC** using a Glue table schema (great for the lake).
- **S3 prefix:** `raw/orders/year=!{timestamp:yyyy}/month=!{timestamp:MM}/` → automatic partitioning.
- **Error output prefix** for failed records.
- **Redshift destination:** Firehose first writes to S3, then runs **COPY** into Redshift.
- **Source** can be: direct PUT (SDK), a Kinesis Data Stream, MSK, CloudWatch Logs.

**Typical real-time architecture**
```
App/IoT ─► Kinesis Data Stream ─► Lambda (validate/enrich) ─► Firehose ─► S3 (Parquet, partitioned) ─► Glue Catalog ─► Athena
                              └─► (optional) Flink for windowed aggregations (e.g. sales per minute)
```
**Real-time processing patterns:** filtering, enrichment, windowed aggregations (tumbling/sliding), anomaly detection, alerting (Lambda → SNS).

**Boto3 to Firehose**
```python
fh = boto3.client("firehose")
fh.put_record(DeliveryStreamName="orders-to-s3",
              Record={"Data": (json.dumps(event) + "\n").encode()})   # newline = one JSON per line
```

---

## MODULE 7 — SERVERLESS DATA PROCESSING (LAMBDA)

### Lesson 17 — AWS Lambda fundamentals

**Simple idea:** Lambda = **run a small function without any server**. Upload code → AWS runs it when an event happens → you pay only for the milliseconds it ran.

**Must-know points**
- **Handler:** entry function `lambda_handler(event, context)`. `event` = input (JSON about what triggered it); `context` = runtime info (remaining time, request ID).
- **Limits:** **max 15 minutes** timeout, memory 128 MB–10,240 MB (CPU scales with memory), `/tmp` storage up to 10 GB, deployment zip 50 MB (250 MB unzipped) / container image 10 GB. Too long/too big → use Glue/EMR.
- **Execution role:** IAM role giving Lambda its permissions (+ CloudWatch Logs).
- **Triggers:** S3, Kinesis, SQS, SNS, API Gateway, EventBridge schedule, DynamoDB streams…
- **Invocation types:** synchronous (API Gateway waits for the answer), **asynchronous** (S3/SNS/EventBridge; Lambda retries **2 times** on failure), poll-based (Kinesis/SQS via *event source mapping*).
- **Cold start:** first invocation after idle takes longer. **Concurrency:** many copies run in parallel (reserved/provisioned concurrency to control).
- **Layers:** shared libraries (e.g. `pandas`, `requests`). AWS provides **AWSSDKPandas layer** (`awswrangler`) for pandas+boto3.
- **Environment variables, DLQ/on-failure destination, versions & aliases.**
- **Good for:** small file validation, light ETL, event glue, API ingestion, orchestration helpers. **Not good for:** heavy Spark-scale transforms (>15 min or GBs).

**Write a first Lambda (Python)**
```python
import json, logging
logger = logging.getLogger(); logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info("Event: %s", json.dumps(event))
    name = event.get("name", "world")
    return {"statusCode": 200, "body": json.dumps({"message": f"Hello {name}"})}
```
Deploy: console → Create function → Python 3.12 → paste → **Deploy** → **Test** with `{"name":"HCL"}` → view logs in CloudWatch.

**Idempotency:** the same event may arrive twice — design so reprocessing doesn't duplicate (e.g. write to a key named from the input file).

---

### Lesson 18 — Event-driven processing: S3 trigger, Kinesis, APIs

**Simple idea:** **Event-driven** = "when X happens, automatically do Y". Upload a file → pipeline starts. No one presses "run".

**Classic pattern (asked in assessments)**
```
Upload CSV to S3 (input bucket) ──ObjectCreated event──► Lambda ──► validates + cleans ──► writes to S3 (output bucket)
                                                         │ fail ─► CloudWatch Logs + SQS DLQ + SNS alert
```
**Setup steps**
1. Create **input** and **output** buckets (**separate** buckets, or separate prefixes with filters).
2. Create Lambda + execution role (S3 `GetObject` on input, `PutObject` on output, CloudWatch Logs).
3. S3 bucket → **Properties → Event notifications** → Event type `s3:ObjectCreated:*` → prefix `incoming/`, suffix `.csv` → destination = Lambda (S3 auto-adds invoke permission, or add resource policy).
4. Upload test file → check logs.

**Lambda code**
```python
import boto3, csv, io, json, urllib.parse
s3 = boto3.client("s3")
OUT_BUCKET = "yourname-lake-clean"

def lambda_handler(event, context):
    for rec in event["Records"]:
        bucket = rec["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(rec["s3"]["object"]["key"])   # keys are URL-encoded!
        body = s3.get_object(Bucket=bucket, Key=key)["Body"].read().decode("utf-8")

        rows = list(csv.DictReader(io.StringIO(body)))
        good = [r for r in rows if r.get("order_id") and r.get("amount")]
        bad  = len(rows) - len(good)

        out_key = "clean/" + key.split("/")[-1]
        buf = io.StringIO()
        w = csv.DictWriter(buf, fieldnames=rows[0].keys()); w.writeheader(); w.writerows(good)
        s3.put_object(Bucket=OUT_BUCKET, Key=out_key, Body=buf.getvalue().encode())

        print(f"{key}: {len(good)} good rows, {bad} rejected")
    return {"status": "ok"}
```
**Danger — recursive triggering:** if Lambda writes into the **same bucket/prefix** that triggers it, it loops forever and costs money. Use a **different bucket** or non-overlapping prefix filters.

**Other event sources**
- **Kinesis → Lambda** (Lesson 15): batch window, batch size, failure handling (bisect batch, on-failure destination).
- **API Gateway → Lambda** (Lesson 25).
- **EventBridge schedule → Lambda** (cron jobs, API polling).
- **SQS → Lambda** (queue buffering for reliability).
- **S3 → EventBridge → Step Functions** (start a full workflow on file arrival).

**Error handling:** `try/except` + log, raise to trigger retry, **DLQ (SQS)** for async failures, alarms on `Errors`, idempotent writes.

---

## MODULE 8 — WORKFLOW ORCHESTRATION

### Lesson 19 — AWS Step Functions, state machines, pipeline automation

**Simple idea:** Step Functions = a **flowchart that runs**. You draw steps (Lambda → Glue → Athena → notify), AWS runs them in order, handles **retries and failures**, and shows you **exactly where** it broke.

**Must-know points**
- Workflow = **State machine** written in **ASL (Amazon States Language, JSON)**; each run = an **execution**.
- **State types**
| State | Purpose |
|---|---|
| `Task` | Do work (invoke Lambda, start Glue job, run Athena query, publish SNS…) |
| `Choice` | If/else branching |
| `Parallel` | Run branches at the same time |
| `Map` | Loop over a list (e.g. each file) |
| `Wait` | Delay |
| `Pass` | Pass/transform data |
| `Succeed` / `Fail` | End states |
- **Standard** (up to 1 year, exactly-once, full history) vs **Express** (≤ 5 min, very high volume, at-least-once). Pipelines → Standard.
- **Service integrations:** `arn:aws:states:::glue:startJobRun.sync` (**.sync** = wait until the job finishes), `...:athena:startQueryExecution.sync`, `...:sns:publish`, `...:lambda:invoke`.
- **Error handling:** **`Retry`** (`ErrorEquals`, `IntervalSeconds`, `MaxAttempts`, `BackoffRate` for exponential backoff) for *transient* errors; **`Catch`** (`ErrorEquals`, `Next`, `ResultPath`) to route *permanent* errors to a cleanup/notify state.
- **Data passing:** `InputPath`, `Parameters`, `ResultSelector`, `ResultPath`, `OutputPath` shape JSON between states. `ResultPath: "$.validation"` keeps the original input and adds the result under a key.
- **Execution history** (console graph) → click the red state → see its input, output, error.
- Needs an **IAM role** trusted by `states.amazonaws.com` allowing the actions it calls.

**Example: validate → (ok?) → Glue ETL → notify; any failure → alert**
```json
{
  "Comment": "Orders ETL pipeline",
  "StartAt": "ValidateFile",
  "States": {
    "ValidateFile": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-south-1:123456789012:function:validate-file",
      "ResultPath": "$.validation",
      "Retry": [{
        "ErrorEquals": ["Lambda.ServiceException", "Lambda.TooManyRequestsException"],
        "IntervalSeconds": 2, "MaxAttempts": 3, "BackoffRate": 2.0
      }],
      "Catch": [{ "ErrorEquals": ["States.ALL"], "ResultPath": "$.error", "Next": "NotifyFailure" }],
      "Next": "IsValid"
    },
    "IsValid": {
      "Type": "Choice",
      "Choices": [{
        "Variable": "$.validation.status",
        "StringEquals": "OK",
        "Next": "RunGlueJob"
      }],
      "Default": "NotifyFailure"
    },
    "RunGlueJob": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun.sync",
      "Parameters": { "JobName": "orders-etl" },
      "ResultPath": "$.glue",
      "Catch": [{ "ErrorEquals": ["States.ALL"], "ResultPath": "$.error", "Next": "NotifyFailure" }],
      "Next": "NotifySuccess"
    },
    "NotifySuccess": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:ap-south-1:123456789012:pipeline-alerts",
        "Message": "Orders pipeline succeeded"
      },
      "End": true
    },
    "NotifyFailure": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:ap-south-1:123456789012:pipeline-alerts",
        "Message": "Orders pipeline FAILED"
      },
      "Next": "PipelineFailed"
    },
    "PipelineFailed": { "Type": "Fail", "Error": "PipelineError", "Cause": "See execution history" }
  }
}
```
**Start it automatically:** EventBridge rule (S3 upload or cron) → *Start execution*.

**Step Functions vs Glue Workflows vs Airflow(MWAA):** Step Functions = general, serverless, great error handling; Glue Workflows = Glue-only chaining; MWAA = managed Airflow for Python DAG fans.

---

## MODULE 9 — APACHE SPARK ON AWS

### Lesson 20 — Spark on EMR (Elastic MapReduce)

**Simple idea:** **EMR** = AWS rents you a **cluster of machines pre-installed with Spark/Hadoop/Hive** when you want **more control than Glue**.

**Must-know points**
- **Node types:** **Primary** (master – coordinates), **Core** (store HDFS data + run tasks), **Task** (compute only, ideal for **Spot** instances).
- **Deployment options:** **EMR on EC2** (classic cluster), **EMR Serverless** (no cluster to manage — submit jobs to an *application*), **EMR on EKS**.
- **Cluster lifecycle:** **Transient** (spin up → run steps → auto-terminate; cheap, best practice) vs **long-running** (interactive, shared).
- **S3 integration:** use `s3://` paths directly (EMRFS) — store data in S3, not HDFS, so you can delete the cluster without losing data.
- **Steps:** units of work added to a cluster (e.g. a `spark-submit`).
- **Cluster management:** resize (managed scaling), instance fleets, bootstrap actions (install libraries), security groups, IAM roles (`EMR_DefaultRole`, `EMR_EC2_DefaultRole`), logs to S3, Spark UI / YARN UI / Ganglia.
- **Glue vs EMR**
  | | Glue | EMR |
  |---|---|---|
  | Management | Serverless | You pick cluster/config (or EMR Serverless) |
  | Control/tuning | Limited | Full (Spark configs, libs, Hive, HBase, Presto) |
  | Best for | Standard ETL, quick start | Big/complex/custom jobs, cost tuning at scale |

**Run a Spark job on EMR**
```bash
# 1) put your script in S3
aws s3 cp etl_job.py s3://yourname-lake/scripts/etl_job.py

# 2) create a small transient cluster that runs the step and terminates
aws emr create-cluster \
  --name "orders-spark" --release-label emr-7.2.0 \
  --applications Name=Spark \
  --instance-type m5.xlarge --instance-count 3 \
  --use-default-roles --auto-terminate \
  --log-uri s3://yourname-lake/emr-logs/ \
  --steps Type=Spark,Name=ETL,ActionOnFailure=TERMINATE_CLUSTER,\
Args=[--deploy-mode,cluster,s3://yourname-lake/scripts/etl_job.py]
```
*(Release label changes over time — pick a current one in the console.)*

**Inside `etl_job.py`** (plain PySpark)
```python
from pyspark.sql import SparkSession, functions as F
spark = SparkSession.builder.appName("orders-etl").getOrCreate()

df = spark.read.option("header", True).csv("s3://yourname-lake/raw/orders/")
out = (df.dropDuplicates(["order_id"])
         .withColumn("amount", F.col("amount").cast("double")))
out.write.mode("overwrite").partitionBy("country").parquet("s3://yourname-lake/curated/orders/")
```
**Cost safety:** always use `--auto-terminate` or set an **auto-termination idle timeout**. A forgotten cluster = surprise bill.

---

## MODULE 10 — QUERY SERVICES (ATHENA)

### Lesson 21 — Amazon Athena: SQL directly on S3

**Simple idea:** **Athena** = write **SQL on files sitting in S3**, no servers, no loading. You pay for **data scanned**.

**Must-know points**
- Engine: Trino/Presto-based SQL (+ Spark option). Reads table definitions from the **Glue Data Catalog**.
- **Query results** are saved to an S3 **output location** (set it first in Settings/workgroup, else you get "no output location" error).
- **Workgroups:** separate users/teams, enforce result location and **per-query data-scan limits** (cost control).
- Tables are **external** (data stays in S3; dropping table doesn't delete data).

**Create a table with DDL (alternative to crawler)**
```sql
CREATE DATABASE IF NOT EXISTS sales_db;

CREATE EXTERNAL TABLE IF NOT EXISTS sales_db.orders_csv (
  order_id    int,
  customer_id int,
  order_date  date,
  amount      double,
  country     string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://yourname-lake/raw/orders/'
TBLPROPERTIES ('skip.header.line.count'='1');
```
**Query it** (everything you learned in SQL phase works)
```sql
SELECT country, COUNT(*) AS orders, ROUND(SUM(amount),2) AS revenue
FROM sales_db.orders_csv
GROUP BY country
ORDER BY revenue DESC;
```
**CTAS — convert CSV → partitioned Parquet in one statement** (a mini-ETL!)
```sql
CREATE TABLE sales_db.orders_parquet
WITH (
  format = 'PARQUET',
  write_compression = 'SNAPPY',
  external_location = 's3://yourname-lake/curated/orders_parquet/',
  partitioned_by = ARRAY['country']
) AS
SELECT order_id, customer_id, order_date, amount, country   -- partition column must be LAST
FROM sales_db.orders_csv
WHERE order_id IS NOT NULL;
```
Also: `INSERT INTO`, views (`CREATE VIEW`), `UNLOAD`, federated queries, JSON functions (`json_extract`, `json_extract_scalar`), `SHOW TABLES`, `DESCRIBE`.

**From Python (needed for GenAI text-to-SQL later)**
```python
import boto3, time
athena = boto3.client("athena")

def run_athena(sql, db="sales_db", out="s3://yourname-lake/athena-results/"):
    qid = athena.start_query_execution(
        QueryString=sql, QueryExecutionContext={"Database": db},
        ResultConfiguration={"OutputLocation": out})["QueryExecutionId"]
    while True:
        st = athena.get_query_execution(QueryExecutionId=qid)["QueryExecution"]["Status"]["State"]
        if st in ("SUCCEEDED", "FAILED", "CANCELLED"): break
        time.sleep(1)
    if st != "SUCCEEDED":
        raise RuntimeError(f"Athena query {st}")
    rows = athena.get_query_results(QueryExecutionId=qid)["ResultSet"]["Rows"]
    header = [c["VarCharValue"] for c in rows[0]["Data"]]
    return [dict(zip(header, [c.get("VarCharValue") for c in r["Data"]])) for r in rows[1:]]
```
(`awswrangler.athena.read_sql_query(sql, database=...)` returns a pandas DataFrame in one line.)

---

### Lesson 22 — Athena: partitioning and performance optimization

**Why:** Athena charges by **bytes scanned**. Less scanned = cheaper + faster.

**The 6 optimizations (memorize)**
1. **Use columnar formats** — Parquet/ORC instead of CSV/JSON (read only needed columns).
2. **Compress** (Snappy/GZIP/ZSTD).
3. **Partition** data by commonly filtered columns (date, country). Query with `WHERE` on the partition column → **partition pruning** (skips other folders).
4. **Select only needed columns** — avoid `SELECT *`.
5. **Right-size files** (avoid thousands of tiny files; ~128 MB+ each).
6. **Use `LIMIT`, filter early, approximate functions** (`approx_distinct`), and put the **largest table first / smaller table on the right** in joins.

**Partitioned table + making partitions visible**
```sql
CREATE EXTERNAL TABLE sales_db.orders_p (
  order_id int, customer_id int, amount double
)
PARTITIONED BY (year string, month string)
STORED AS PARQUET
LOCATION 's3://yourname-lake/curated/orders/';

-- Hive-style folders year=2026/month=11/ → discover them:
MSCK REPAIR TABLE sales_db.orders_p;
-- or add one:
ALTER TABLE sales_db.orders_p ADD PARTITION (year='2026', month='11')
  LOCATION 's3://yourname-lake/curated/orders/year=2026/month=11/';
```
- **Partition projection** (table property) = Athena computes partitions from a pattern → no crawler/`MSCK` needed for date-style partitions; good for many partitions.
- **Too many partitions** (over-partitioning, e.g. by customer_id) hurts. Aim for partitions with at least a few hundred MB.
- **Check cost** after each query: *Data scanned* shown in console. Compare CSV vs Parquet to **show improvement in your demo**.

---

## MODULE 11 — DATA GOVERNANCE

### Lesson 23 — AWS Lake Formation

**Simple idea:** **Lake Formation** = the **permission manager for your data lake**. Instead of writing S3 bucket policies + IAM for every table, you say "Analyst role can read *these columns* of *this table*" in one place.

**Must-know points**
- Built on top of the **Glue Data Catalog** + S3. Integrated with **Athena, Redshift Spectrum, Glue, EMR, QuickSight**.
- Steps: (1) make yourself **Data lake administrator** → (2) **Register the S3 location** (Lake Formation gets a service-linked role to access it) → (3) **Grant permissions** to IAM users/roles on **database / table / column** (`SELECT`, `INSERT`, `DESCRIBE`, `ALTER`…).
- **Fine-grained control:** column-level, **row-level** (data filters/cell filters), e.g. hide `salary` column or show only `country='IN'` rows.
- **LF-Tags (tag-based access control, LF-TBAC):** tag data (`sensitivity=pii`) and grant by tag — scales better than per-table grants.
- **Cross-account sharing** of catalog resources.
- **Blueprints** (ingest from DB/CloudTrail) and **audit** via CloudTrail.
- **Data Catalog governance** = keep tables documented (descriptions, owners), controlled, discoverable.
- **Security** pieces: IAM (identity), S3 encryption + KMS, Lake Formation (data permissions), CloudTrail (audit), Macie (PII discovery).

**Interview line:** "IAM controls *who can call the API*; Lake Formation controls *who can see which rows/columns of which table*."

---

## MODULE 12 — DATA MIGRATION

### Lesson 24 — AWS DMS (Database Migration Service)

**Simple idea:** DMS **copies data from a database into AWS** (and can **keep it in sync**), even while the source is still running.

**Must-know points**
- **Components:** **Replication instance** (or **DMS Serverless**) · **Source endpoint** (e.g. on-prem MySQL/Oracle/SQL Server/PostgreSQL) · **Target endpoint** (RDS, Redshift, **S3**, DynamoDB…) · **Migration task**.
- **Task types**
  1. **Full load** — bulk copy of existing data (this is your *bulk data transfer*).
  2. **CDC (Change Data Capture)** — ongoing replication of inserts/updates/deletes (reads the DB logs) → *continuous replication*.
  3. **Full load + CDC** — most common.
- **Homogeneous** (MySQL→MySQL) vs **heterogeneous** (Oracle→PostgreSQL; use **AWS Schema Conversion Tool / DMS Schema Conversion** for schema/code).
- **DMS → S3:** writes CSV or **Parquet**; CDC files carry an **`Op` column** (`I`, `U`, `D`) so you can rebuild current state downstream. Great for **hydrating the data lake** (raw layer).
- **Table mappings & transformation rules:** include/exclude tables, rename, add columns.
- **Data validation** compares source and target rows.
- **Other bulk transfer tools:** Snowball (offline, TB–PB), DataSync (file servers → S3/EFS), Transfer Family (SFTP).
- **Monitoring:** task status, CloudWatch metrics (`CDCLatencySource/Target`), table statistics.

**Typical pipeline:** `MySQL (on-prem/RDS) → DMS full load + CDC → S3 raw (Parquet) → Glue → curated → Athena/Redshift`.

---

## MODULE 13 — API & INGESTION

### Lesson 25 — API Gateway and ingesting data from APIs

**Two directions**
1. **Ingest FROM external APIs** (most common in hackathons — e.g. pull weather/stocks/COVID/orders JSON, land in S3).
2. **Expose YOUR data/function AS an API** with **Amazon API Gateway**.

**A) Pulling from an API with Lambda + EventBridge**
```python
import boto3, json, datetime, requests     # add 'requests' via a Lambda layer or zip
s3 = boto3.client("s3")

def lambda_handler(event, context):
    resp = requests.get("https://api.example.com/orders", params={"page": 1}, timeout=10)
    resp.raise_for_status()                       # fail loudly on 4xx/5xx
    data = resp.json()
    today = datetime.date.today().isoformat()
    key = f"raw/api_orders/ingest_date={today}/orders.json"
    s3.put_object(Bucket="yourname-lake", Key=key, Body=json.dumps(data).encode())
    return {"records": len(data), "key": key}
```
- Schedule with **EventBridge** (`rate(1 hour)`).
- Real APIs: handle **pagination** (loop until no next page), **rate limits** (retry with backoff on 429), **auth** (keep API keys in **Secrets Manager / SSM Parameter Store**, never in code), **timeouts**, **idempotent file naming**.
- Write **JSON Lines** (one JSON per line) → easier for Athena/Glue.
- Add `requests` with a Lambda layer: `pip install requests -t python/` → zip → publish layer.

**B) API Gateway**
- Types: **REST API** (full features), **HTTP API** (cheaper/simpler), **WebSocket**.
- Concepts: **Resource/route** (`/orders`), **method** (GET/POST), **integration** (Lambda proxy, HTTP, AWS service), **stage** (`dev`, `prod`), **throttling & usage plans/API keys**, **authorizers** (IAM, Cognito, Lambda authorizer), **CORS**, CloudWatch logging.
- **Lambda proxy integration** → Lambda receives the full request, must return `{statusCode, headers, body}`:
```python
import json
def lambda_handler(event, context):
    body = json.loads(event.get("body") or "{}")        # POST JSON payload
    # validate + write to S3 / send to Kinesis here
    return {"statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"received": body})}
```
- Use case: **ingestion endpoint** — client POSTs JSON → API Gateway → Lambda → Kinesis/Firehose/S3.
- Test: `curl -X POST https://<id>.execute-api.ap-south-1.amazonaws.com/dev/orders -d '{"order_id":1}' -H "Content-Type: application/json"`

---

## MODULE 14 — DEVOPS FOR DATA ENGINEERING

### Lesson 26 — CI/CD basics, Infrastructure as Code, deploying pipelines

**Simple idea:** Stop clicking in the console. **Describe infrastructure in files** (IaC) and **auto-deploy** when code changes (CI/CD).

**CI/CD basics**
- **CI (Continuous Integration):** on every `git push` → run **tests/lint** (e.g. `unittest`/`pytest` for your transformation functions).
- **CD (Continuous Delivery/Deployment):** automatically **deploy** (upload Glue scripts to S3, update Lambda, apply infra).
- **Tools:** **GitHub Actions** (most common), **AWS CodeBuild / CodePipeline / CodeDeploy**. (Note: AWS CodeCommit stopped accepting new customers in 2024 — use GitHub.)
- **Environments:** dev → test → prod, config via **environment variables / parameters** (never hard-code bucket names).
- **Branching:** feature branch → pull request → review → merge to main → deploy.

**Infrastructure as Code (intro)**
- **CloudFormation** (AWS native, YAML/JSON **templates → stacks**), **AWS SAM** (shorthand for serverless), **CDK** (write infra in Python/TS), **Terraform** (multi-cloud, HCL; `init → plan → apply → destroy`, keeps **state**).
- Benefits: repeatable, reviewable (in Git), easy teardown (**delete the stack = delete everything → no surprise bills**).

**Minimal CloudFormation (bucket + Lambda role skeleton)**
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Data lake bucket
Parameters:
  BucketName:
    Type: String
Resources:
  LakeBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      VersioningConfiguration:
        Status: Enabled
Outputs:
  BucketArn:
    Value: !GetAtt LakeBucket.Arn
```
```bash
aws cloudformation deploy --template-file lake.yaml --stack-name lake --parameter-overrides BucketName=yourname-lake-2026
aws cloudformation delete-stack --stack-name lake
```

**Minimal Terraform**
```hcl
resource "aws_s3_bucket" "lake" { bucket = "yourname-lake-2026" }
```

**Deploying data pipelines — checklist:** code in Git → unit tests → package (zip/layer) → upload scripts to S3 → deploy Glue job/Lambda/State machine via IaC → run smoke test → monitoring & alarms.

**GitHub Actions snippet (deploy Glue script)**
```yaml
name: deploy-glue
on: { push: { branches: [main] } }
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/gh-deploy   # OIDC, no stored keys
          aws-region: ap-south-1
      - run: aws s3 cp glue/orders_etl.py s3://yourname-lake/scripts/orders_etl.py
```

---

## MODULE 15 — GEN AI ON AWS

### Lesson 27 — Amazon Bedrock, LLM integration, AI-powered data pipelines

*(This lesson is the AWS entry point; **Phase 6 (GenAI)** goes deep. Read both together.)*

**Simple idea:** **Amazon Bedrock** = a **single AWS API to many foundation models** (Amazon Nova/Titan, Anthropic Claude, Meta Llama, Mistral, etc.). No servers, no GPUs, data stays inside your AWS account (not used to train the base models).

**Must-know points**
- **Model access:** in the Bedrock console you may need to enable/request access to a model (and some models are used through **inference profiles**). Copy the exact **model ID/ARN** from the console for your region.
- **Runtime API:** `bedrock-runtime` client. The **Converse API** gives one consistent way to chat with any supported model; `invoke_model` is the lower-level, model-specific way (used for embeddings).
- **Features:** **Playground** (test prompts), **Knowledge Bases** (managed RAG), **Agents**, **Guardrails** (filter harmful content/PII/topics), **Prompt management**, **Model evaluation**, **Fine-tuning/continued pre-training** (advanced).
- Other AWS AI: **Amazon Q** (assistants), **SageMaker** (build/train/host your own models), **Comprehend** (NLP), **Textract** (OCR), **Rekognition** (images).
- **IAM:** the Lambda/EC2/notebook role needs `bedrock:InvokeModel` (and `bedrock:InvokeModelWithResponseStream` if streaming).

**Minimal call with Converse**
```python
import boto3
bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")   # use a region where your model is enabled
MODEL_ID = "<paste model ID or inference-profile ID from the Bedrock console>"

resp = bedrock.converse(
    modelId=MODEL_ID,
    system=[{"text": "You are a concise data analyst."}],
    messages=[{"role": "user", "content": [{"text": "Explain partition pruning in 2 lines."}]}],
    inferenceConfig={"maxTokens": 300, "temperature": 0.2},
)
print(resp["output"]["message"]["content"][0]["text"])
```

**AI-powered data pipeline ideas (what to build in the hackathon's GenAI step)**
1. **Text-to-SQL on Athena:** user asks a question → Bedrock writes SQL from the table schema → you validate & run on Athena → return answer. *(Phase 6, Lessons 5–6)*
2. **Auto-summary of results:** Athena result → Bedrock → "3 key insights" paragraph. *(Lesson 7)*
3. **Data documentation:** feed column names/samples → LLM writes column descriptions into the Glue Catalog.
4. **Data quality explanations:** feed rejected rows → LLM summarises "why rows failed".
5. **Unstructured → structured:** LLM extracts fields (name, amount, date) from text/emails to JSON, store in S3.

---

# PART B — HACKATHON PLAYBOOK

---

## B1. What the HCLTech hackathon / assessment looks like

> Honest note: HCLTech does not publish an official syllabus for this. The points below combine **your syllabus sheet** (the authoritative part) with what candidates have publicly shared about HCL's Cloud Data Engineer assessments and hackathon-style drives (GitHub write-ups, job posts). Formats change between drives — treat it as a guide.

**Pattern seen in candidate write-ups (hands-on, screenshot-based lab assessments):**
| Section | Typical tasks |
|---|---|
| Advanced SQL | Create tables with PK/FK, joins, self-join (employee–manager), subqueries, `DENSE_RANK()`, `LAG()` month-over-month, index/EXPLAIN, duplicates with `GROUP BY … HAVING` |
| Linux | `grep/awk/sort/uniq` log analysis, `chmod/chown`, `top/ps/free/df/find`, a bash backup script with arguments & exit codes |
| PySpark | DataFrame filter/derive/groupBy/orderBy, cleaning (`dropDuplicates`, nulls, types, dates), joins incl. `left_anti`, broadcast join, lazy vs action |
| AWS | **S3 + IAM** least-privilege & private bucket · **EC2 + CloudWatch** alarm + SNS · **Lambda** triggered by S3 · **Glue crawler + ETL job** · **Athena** queries · **Step Functions** state machine with Retry/Catch |
| Cloud theory | (Some drives also include Azure ADLS/ADF/Databricks — your syllabus lets you **choose one cloud**, so go deep on AWS; see the mapping table in B6 as a safety net) |

**Hackathon-style drives for AI/Data roles** have also been described as an **~8-hour, mostly individual, virtual build** followed by a **technical interview on what you built**. The syllabus **Capstone** matches this exactly:

> *"Build an end-to-end pipeline using file/API ingestion, transformation, storage, SQL analytics, and one GenAI use case."*

So prepare to **deliver one working pipeline + one GenAI feature + a clear explanation**.

---

## B2. The reference architecture (build this once at home)

```
 (1) INGEST                (2) STORE RAW          (3) TRANSFORM              (4) STORE CURATED        (5) ANALYSE           (6) GenAI
 CSV upload  ─────────►   s3://lake/raw/   ─►   Glue PySpark job   ─►   s3://lake/curated/   ─►   Glue Crawler     ─►   Bedrock:
 or Lambda pulls API ─►   orders/ …             (clean, type, dedupe,      (Parquet, partitioned)    + Athena SQL          NL → SQL
 (EventBridge schedule)                          derive, join)                                          (revenue by X)        + insights summary

 Orchestrated by Step Functions  |  Alerts via SNS  |  Logs in CloudWatch  |  Permissions via IAM (+ Lake Formation)
```

**Services you'll almost certainly touch:** S3 · IAM · Lambda · Glue (crawler + job) · Athena · CloudWatch · (Step Functions) · (Bedrock). Redshift/Kinesis/EMR/DMS are less likely in 8 hours but **know them for the interview**.

---

## B3. Hour-by-hour plan for an 8-hour hackathon

| Time | What to do |
|---|---|
| **0:00–0:20** | Read the problem twice. Note **inputs, required outputs, deadlines, what to submit**. Pick region. Open Console, CloudShell, and a notes file. Decide the *minimum viable pipeline*. |
| **0:20–0:50** | Create **bucket + folders**; upload sample data to `raw/`. Create **IAM roles** (Glue, Lambda). |
| **0:50–2:00** | **Ingestion:** file upload and/or Lambda API pull. Verify files land in `raw/`. |
| **2:00–2:30** | **Glue crawler** on raw → check table in Athena (`LIMIT 10`). |
| **2:30–4:30** | **Transformation:** Glue job (or Studio) → clean + enrich → Parquet to `curated/` (partitioned). Handle nulls, duplicates, bad dates, types. Log counts (rows in/out/rejected). |
| **4:30–5:15** | **Catalog + SQL analytics:** crawl curated; write 4–6 business queries in Athena (top-N, trends with window functions, joins). Save them. |
| **5:15–6:30** | **GenAI use case** via Bedrock: NL→SQL *or* auto-insight summary on query results. Keep it small and safe (read-only SQL). |
| **6:30–7:15** | **Orchestration + monitoring:** Step Functions (optional), SNS alert, CloudWatch logs; screenshot a successful run. |
| **7:15–7:45** | **Documentation:** README with architecture diagram, steps, assumptions, how to run, screenshots, sample outputs. Export code (Lambda, Glue script, SQL, ASL) to a repo/folder. |
| **7:45–8:00** | **Cleanup + submit:** re-check submission checklist, **delete costly resources**, final test. |

**Golden rules**
- **Get an end-to-end "ugly but working" pipeline first** (by hour ~4), then improve. A half-built perfect pipeline scores less than a complete simple one.
- **Take screenshots as you go** (many HCL assessments require proof).
- **Commit often**; save every script.
- **Time-box** each error to 15 min, then try an alternative (Glue Studio instead of code; CTAS instead of Glue job).
- **Explain what you built** — the interview is "based on hackathon performance".

---

## B4. Fast fixes for common errors

| Error / symptom | Likely cause | Fix |
|---|---|---|
| `AccessDenied` / 403 | Missing IAM permission, or wrong resource ARN (bucket vs `/*`) | Read message → add that action/resource to the role |
| Glue job fails instantly: *Role* error | Role lacks `glue.amazonaws.com` trust or S3 permission | Add trust policy + `AWSGlueServiceRole` + S3 access |
| Athena: *No output location* | Result location not set | Settings → set `s3://…/athena-results/` |
| Athena returns 0 rows | Wrong `LOCATION` (must be the **folder**, not a file), partitions not loaded, header row mismatch | Check path; `MSCK REPAIR TABLE`; `skip.header.line.count` |
| Crawler made weird columns `col0,col1` | Header not detected | Custom classifier or define table via DDL |
| `HIVE_BAD_DATA` / type errors | Column type mismatch (e.g. string in an int column) | Cast in ETL; use string type in raw, convert in curated |
| Lambda **Task timed out** | Default 3 s timeout / big file | Raise timeout (≤15 min) + memory; or use Glue |
| Lambda `No module named 'requests'/'pandas'` | Library not packaged | Add **layer** (AWSSDKPandas) or zip dependencies |
| Lambda loops/cost spike | Writes to same bucket that triggers it | Separate buckets/prefixes |
| S3 event doesn't fire | Wrong prefix/suffix filter, missing Lambda permission, wrong event type | Check notification config + Lambda resource policy |
| Step Functions state fails | Role missing permission, wrong ARN, wrong JSON path | Open **execution history**, look at that state's input/output & error |
| SSH timeout | SG/route/public IP/NACL | See Lesson 5 checklist |
| Bedrock `AccessDeniedException` | Model access not enabled, wrong region, role lacks `bedrock:InvokeModel` | Enable model, switch region, fix IAM |
| Region mismatch (resource "not found") | Bucket/job/table in different regions | Use **one region** everywhere |

---

## B5. Interview Q&A (short, quotable answers)

**S3 / security**
1. *What is S3 and why is it used for data lakes?* — Object store: unlimited, cheap, 11-nines durable, integrates with every analytics service, decouples storage from compute.
2. *How do you secure an S3 bucket?* — Block Public Access, least-privilege IAM/bucket policies, SSE-S3/KMS encryption, HTTPS-only policy, versioning, access logging/CloudTrail, VPC endpoints.
3. *Versioning vs lifecycle?* — Versioning keeps old copies for recovery; lifecycle automates transitions/expiry to save cost.
4. *IAM user vs role?* — User = long-term credentials for a person/app; role = temporary credentials assumed by services/people. Services should always use roles.
5. *What does "explicit deny wins" mean?* — If any applicable policy denies, access is denied even if another allows.

**Glue / Athena / Redshift**
6. *Glue Catalog vs crawler?* — Catalog = metadata store; crawler = tool that infers schema and populates it.
7. *DynamicFrame vs DataFrame?* — DynamicFrame tolerates schema inconsistencies and has Glue transforms; DataFrame is standard Spark; convertible both ways.
8. *What are job bookmarks?* — Glue tracks processed data so reruns only handle new data (incremental ETL).
9. *Glue vs EMR?* — Glue serverless/simple; EMR more control and flexibility for big/custom workloads.
10. *Athena vs Redshift?* — Athena: serverless ad-hoc SQL on S3, pay per scan. Redshift: warehouse for fast repeated BI, pay for compute (or serverless RPUs).
11. *How do you reduce Athena cost?* — Parquet, compression, partitioning, select only needed columns, right file sizes.
12. *Dist key vs sort key?* — Dist key decides data placement across nodes (join efficiency); sort key orders data on disk (range-filter efficiency).
13. *How to load data into Redshift?* — `COPY` from S3 using an IAM role, preferably Parquet/compressed and split into multiple files; avoid row-by-row INSERT.
14. *What is Redshift Spectrum?* — Query S3 data from Redshift through external tables in the catalog, without loading.

**Streaming / serverless / orchestration**
15. *Data Streams vs Firehose?* — Streams: custom real-time processing, shards, replay. Firehose: managed delivery to S3/Redshift etc., near-real-time, no consumers to write.
16. *What is a shard and partition key?* — Shard = unit of capacity (1 MB/s in); partition key maps records to shards and preserves per-key order.
17. *Lambda limits?* — 15-min timeout, ≤10 GB memory, /tmp ≤10 GB; use Glue/EMR beyond that.
18. *How do you make a Lambda pipeline reliable?* — Idempotent writes, retries, DLQ, CloudWatch alarms, separate input/output buckets.
19. *Why Step Functions?* — Visual orchestration with built-in Retry/Catch, branching, parallelism, service integrations and a full execution history.
20. *Retry vs Catch?* — Retry reattempts transient errors with backoff; Catch routes to a fallback state for errors that persist.

**Architecture / ops**
21. *Explain your pipeline end-to-end.* — (Use B2 diagram: ingest → raw → transform → curated Parquet → catalog → Athena → GenAI; orchestrated, monitored, secured.)
22. *Raw vs curated vs trusted?* — Raw immutable copy; curated cleaned/typed/partitioned; trusted business-ready, quality-checked.
23. *Batch vs streaming?* — Batch = scheduled bulk, higher latency, simpler; streaming = event-by-event, low latency, more complex.
24. *DMS full load vs CDC?* — Full load copies existing data; CDC replicates ongoing changes from DB logs.
25. *CloudWatch vs CloudTrail?* — CloudWatch = performance metrics/logs/alarms; CloudTrail = audit of API calls (who did what).
26. *What is IaC and why use it?* — Define infrastructure in code (CloudFormation/Terraform) for repeatability, review, and easy teardown.
27. *How would you handle bad records?* — Validate early, route rejects to a quarantine prefix with reason, log counts, alert if rejection rate exceeds a threshold.
28. *How do you control cost?* — Budgets/alerts, lifecycle policies, Parquet/partitioning, auto-terminate EMR, right-size Glue workers, delete idle Redshift/Kinesis, workgroup scan limits.
29. *Security-in-depth for a data lake?* — IAM least privilege, Lake Formation column/row permissions, KMS encryption, private networking (VPC endpoints), CloudTrail, Macie for PII.
30. *Where does GenAI fit?* — NL→SQL for self-service analytics, auto-summaries of results, documentation/data-quality explanations, RAG over metadata/docs; always validate output and keep it read-only.

---

## B6. Cheat sheets

### Service → one-liner
| Service | One line |
|---|---|
| IAM | Who can do what |
| S3 | Object storage = the data lake |
| EC2 | Rented virtual machine |
| CloudWatch | Metrics, logs, alarms |
| CloudTrail | Audit trail of API calls |
| Glue Crawler / Catalog | Discover schema / metadata store |
| Glue Job / Studio | Serverless PySpark ETL (code / visual) |
| Athena | SQL on S3, pay per scan |
| Redshift | Columnar MPP data warehouse |
| Kinesis Streams | Real-time event pipe (shards) |
| Firehose | Managed delivery to S3/Redshift |
| Lambda | Serverless functions (≤15 min) |
| Step Functions | Workflow/state machine orchestrator |
| EMR | Managed Spark/Hadoop clusters |
| Lake Formation | Fine-grained data-lake permissions |
| DMS | Database migration + CDC |
| API Gateway | Create/secure REST/HTTP APIs |
| EventBridge | Schedules & event routing |
| SNS / SQS | Notifications / queues |
| Secrets Manager | Store keys & passwords |
| CloudFormation / Terraform | Infrastructure as Code |
| Bedrock | Foundation models via API |

### File-format cheat
| Format | Type | Use |
|---|---|---|
| CSV / JSON | Row, text | Raw landing, human-readable |
| **Parquet** | Columnar, compressed | **Curated layer, Athena, Spark** |
| ORC | Columnar | Hive/Athena |
| Avro | Row, schema-embedded | Streaming/Kafka |

### AWS ↔ Azure ↔ GCP name map (in case a question uses another cloud)
| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Object storage | S3 | Blob / ADLS Gen2 | Cloud Storage |
| Identity | IAM | Entra ID + RBAC | IAM |
| Serverless function | Lambda | Azure Functions | Cloud Functions |
| ETL / orchestration | Glue, Step Functions | Data Factory | Dataflow, Composer |
| Warehouse | Redshift | Synapse | BigQuery |
| Spark platform | EMR / Glue | Databricks / Synapse | Dataproc |
| Monitoring | CloudWatch | Azure Monitor | Cloud Monitoring |
| Streaming | Kinesis | Event Hubs | Pub/Sub |
| Secrets | Secrets Manager | Key Vault | Secret Manager |
| GenAI | Bedrock | Azure OpenAI | Vertex AI |

---

## B7. Final checklist before the hackathon

- [ ] AWS account ready, MFA on root, **Budget alert** set, CLI/CloudShell tested
- [ ] I can create a **bucket + IAM role + Lambda + S3 trigger** from memory
- [ ] I can run **crawler → Glue job → Athena** end-to-end in < 45 min
- [ ] I can write a **Step Functions** ASL with Retry/Catch/Choice
- [ ] I can write Athena **CTAS to Parquet** and a partitioned table
- [ ] I can call **Bedrock Converse** from Python (Phase 6)
- [ ] I have a **README template** and **architecture diagram** ready
- [ ] I know where **logs** are for Lambda, Glue, Step Functions
- [ ] I practised **cleanup** (delete Redshift, EMR, Kinesis, NAT, empty buckets)

*Next: open **Phase6_GenAI_for_Data_Engineering.md**.*
