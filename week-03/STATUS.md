# Week 3 Status

## Update (Oct 27, 2026): deployed and verified
PropertyLite is running on `property-api-01` (t3.micro, Amazon Linux 2023) in us-east-2. From my laptop, `curl http://<public-ip>:8080/health` returned `{"status":"ok"}` and `/properties/R100234` returned the Sacramento listing. I then created an EBS snapshot of the root volume (Lab 3.2). Screenshots are in this folder. The instance launched without a key pair, so there is no SSH access; the app test uses curl only.

## Update (Oct 20, 2026): cause found
The blocks below came from a region-restriction service control policy (`AdvancedModeRegionRestrictionSecurityControlPolicy`). My account is part of my own AWS Organization (`Swayam-Singh-Team`), and that policy only allows **us-east-2 (Ohio)**. A quick test confirmed it: `aws ec2 describe-key-pairs` is denied in us-east-1 and us-west-2 and works in us-east-2. All labs now run in us-east-2.

## Original status (Oct 13, 2026)
### Ready
- `bootstrap.sh`: the EC2 user-data script. It installs Python, sets up Flask 3.0.3 in a virtual environment, writes PropertyLite and the sample CSV to `/opt/property-api`, and starts the app on port 8080. It was dry-run tested outside AWS with the yum steps skipped: `/health`, `/properties`, `/properties/R100234`, the city filter, and the 404 case all returned the expected responses.
- `runbook.md`: the step-by-step launch, test, snapshot, and cleanup procedure.
- Security group plan: one inbound rule, TCP 8080 from 0.0.0.0/0. No SSH rule, because SSH is not available on this account (see below).

### Blocked
My AWS account is a member of an AWS Organization whose policies block two steps of this lab:
1. **Creating a key pair:** `ec2:CreateKeyPair` fails with an explicit deny from a service control policy. Workaround: launch with "Proceed without a key pair", since the app test uses curl, not SSH.
2. **Choosing an AMI:** in us-east-1, the launch wizard rejects both Amazon Linux 2023 AMIs (kernel-6.18 and kernel-6.1) with "The AMI ID is not valid." No instance can launch until this is resolved.

### Next steps
- Raising both blocks with Tyler Beede (AWS track mentor), including whether to switch to an IDX-managed AWS account.
- Once access is fixed, the launch takes about 10 minutes, because the script and runbook are already done.
