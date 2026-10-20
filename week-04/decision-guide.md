# Week 4: Storage Decision Guide

| Service | What it stores | Use this when... | PropertyLite example |
|---|---|---|---|
| **S3** | Objects (files) in buckets, reached over HTTP | you need cheap, durable storage for files of any size, such as backups, logs, exports, or images, and nothing needs to mount it like a drive | runbooks, logs, and backups of data exports |
| **EBS** | Block storage: a virtual hard drive attached to one EC2 instance in one Availability Zone | a single server needs a fast, persistent disk for its operating system, app, or a self-managed database | the root drive of the PropertyLite EC2 server |
| **EFS** | A shared network file system that many instances can mount at the same time | several servers need to read and write the same files at once | a shared folder if PropertyLite ran on several servers behind a load balancer |
| **RDS** | A managed relational database (MySQL, PostgreSQL, and others) | the data has a fixed structure with relationships, and you need SQL queries, joins, and transactions without running the database server yourself | the `rets_property` table in MySQL |
| **DynamoDB** | A managed NoSQL key-value and document database | you look items up by a known key at any scale, need very fast responses, and don't need joins | fast lookups of one listing by `L_ListingID` |

## Week 4 concepts in plain words
- **S3 storage classes:** Standard is for files you read often. Standard-IA costs less to store but charges a fee to read, so it suits files you rarely open. Glacier is the cheapest archive tier, and getting data back takes minutes to hours.
- **Lifecycle rules:** automatic actions on a schedule, such as moving files to Glacier after 30 days or deleting them after a year. Glacier Flexible Retrieval has a 90-day minimum storage charge, so files deleted sooner still bill for 90 days.
- **Versioning:** S3 keeps every version of a file, so an accidental overwrite or delete can be undone.
- **RDS Multi-AZ:** RDS keeps a standby copy of the database in a second Availability Zone and switches to it automatically if the first one fails.
- **DynamoDB partition key:** the attribute DynamoDB uses to decide where each item is stored. When it is the only key, it must be unique for every item, which is why `L_ListingID` works well.
- **DynamoDB capacity modes:** on-demand (`PAY_PER_REQUEST`) bills per read and write, which suits small or unpredictable traffic. Provisioned mode bills for a fixed read and write capacity that you set ahead of time.
