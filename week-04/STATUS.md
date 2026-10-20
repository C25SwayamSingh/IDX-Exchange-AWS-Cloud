# Week 4 Status (Oct 20, 2026)

## Lab 4.1: S3 bucket with a lifecycle rule
Commands used (region us-east-1):
```bash
aws s3 mb s3://my-training-bucket-swayam
aws s3 cp week-03/runbook.md s3://my-training-bucket-swayam/
aws s3 ls s3://my-training-bucket-swayam/
```
The lifecycle rule `archive-after-30-days` applies to all objects and moves them to Glacier Flexible Retrieval 30 days after creation.

This bucket name matches the Week 2 policy `S3UploaderOnly-swayam`, so it also tests that policy from the other side:
```bash
aws s3 cp week-04/decision-guide.md s3://my-training-bucket-swayam/ --profile s3test   # allowed: PutObject on this bucket
aws s3 ls --profile s3test                                                            # denied: listing buckets is not in the policy
```

## Lab 4.3: DynamoDB table
```bash
aws dynamodb create-table \
  --table-name Properties \
  --attribute-definitions AttributeName=L_ListingID,AttributeType=S \
  --key-schema AttributeName=L_ListingID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
aws dynamodb wait table-exists --table-name Properties
aws dynamodb put-item --table-name Properties \
  --item '{"L_ListingID": {"S": "R100234"}, "L_City": {"S": "Sacramento"}, "L_Keyword2": {"N": "3"}, "L_SystemPrice": {"N": "450000"}}'
aws dynamodb get-item --table-name Properties \
  --key '{"L_ListingID": {"S": "R100234"}}'
```
The handbook asks for a row from the cleaned `rets_property` table from Lab 4.2. Because Lab 4.2 is blocked, this uses the matching row from the Week 0 sample data.

## Lab 4.2: RDS MySQL (blocked)
Lab 4.2 imports `rets_property.sql` into RDS from an EC2 instance in the same VPC, and this account can't launch EC2 yet (see `week-03/STATUS.md`). I'm holding off on creating the RDS instance until I can connect to it, because RDS bills by the hour while it runs.

The raw `rets_property.sql` contains real agents' names, emails, and phone numbers. It stays out of this repo, and the PII columns get dropped right after import (handbook step 44).

## Cleanup
```bash
aws dynamodb delete-table --table-name Properties
aws s3 rm s3://my-training-bucket-swayam --recursive
aws s3 rb s3://my-training-bucket-swayam
```
Deleting the bucket also removes the lifecycle rule. The object is deleted while it is still in Standard, so no Glacier minimum charge applies.
