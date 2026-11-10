# Week 5 Status (Nov 10, 2026)

## Done
- Lab 5.1: created `training-vpc` (10.0.0.0/16) in us-east-2 with the "VPC and more" wizard: 2 AZs, 2 public subnets, 2 private subnets, 1 NAT gateway (single AZ), Internet Gateway, and route tables. Created Oct 27.
- Key pairs now work in us-east-2, so the bastion pattern (Lab 5.2) is possible on this account.
- `vpc-diagram.png` and `README.md` explain the traffic path from my laptop to the private instance through the bastion, and why the private subnet still reaches the internet through the NAT gateway.

## In progress
- Lab 5.2: launching the bastion in a public subnet (SSH from My IP) and the private instance in a private subnet (SSH only from the bastion's security group), then testing `ssh -A` through the bastion.

## Lesson learned
I left the VPC and NAT gateway running between meetings, which uses credits every hour. Going forward, each lab gets built and torn down in the same sitting.
