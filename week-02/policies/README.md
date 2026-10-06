# Week 2: IAM and Security Foundations

## S3UploaderOnly-swayam
This policy allows exactly two actions: s3:PutObject, which uploads a file, and s3:GetObject, which downloads one. Both apply only to objects inside a single bucket, my-training-bucket-swayam. The /* at the end of the resource ARN means "every object inside this bucket." Without it, the ARN would point at the bucket itself, and uploads would be denied. The policy grants nothing else, so the user cannot list buckets, delete files, or use any other service. I scoped it this way to follow least privilege: s3-test-user only needs to upload and read files in one place, so that is all it can do. If its keys ever leaked, the damage would stay inside that one bucket.

## How I tested it
I attached the policy to a user called s3-test-user and ran `aws s3 ls --profile s3test`. AWS returned AccessDenied, because listing buckets is a separate action (s3:ListAllMyBuckets) that the policy never allows. That denial is the correct, expected result.

## IAM Access Analyzer
I created an external access analyzer, which continuously checks whether anything in my account is shared with other accounts or the public. It reported 0 active findings.
