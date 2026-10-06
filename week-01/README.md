# Week 1: Cloud Fundamentals and Account Setup

## What I did
- Turned on MFA for the root user
- Created a zero-spend budget alert
- Created an IAM user, swayam-admin, with AdministratorAccess
- Created an access key for swayam-admin, configured the AWS CLI with it, and verified it with `aws sts get-caller-identity`

## Note on console access
This account type does not offer console passwords for IAM users, so I sign in to the console as root (protected by MFA) and use swayam-admin for all CLI work.

## Root vs IAM vs the Shared Responsibility Model
The root user is the identity created with the account, and it has unlimited power, including over billing and closing the account. Because losing it would be a disaster, I protected it with MFA. I also created an IAM user called swayam-admin and use its access key for my command-line work, so my day-to-day commands never run as root. If that key ever leaked, I could deactivate it without losing the whole account. The Shared Responsibility Model explains who secures what. AWS secures the cloud itself: the data centers, hardware, and network behind its services. I secure what I put in the cloud: my users, permissions, settings, and data. The MFA, budget alert, and separate admin user I set up this week are my side of that agreement.
