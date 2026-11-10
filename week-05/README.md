# Week 5: Networking, VPC Deep Dive

## What I built
`training-vpc` (10.0.0.0/16) in us-east-2 with two public subnets, two private subnets, one Internet Gateway, and one NAT Gateway in a single Availability Zone. A bastion host sits in a public subnet, and a private instance sits in a private subnet with no public IP. See `vpc-diagram.png`.

## Traffic path from my laptop to the private instance
1. **Laptop to bastion.** I run `ssh -A -i ~/.ssh/training-key.pem ec2-user@<bastion-public-ip>`. The packet goes over the internet to the VPC's Internet Gateway, which forwards it into the public subnet. The public subnet's route table sends 0.0.0.0/0 to the Internet Gateway, which is what makes the subnet "public." The bastion's security group allows port 22 only from my IP, so nobody else can even reach the login prompt.
2. **Bastion to private instance.** From inside the bastion I run `ssh ec2-user@<private-ip>`. This hop never leaves the VPC: both subnets share the VPC's 10.0.0.0/16 address space, so the local route delivers it. The private instance's security group allows port 22 only from the bastion's security group ID, not from any IP address. That means the only way in is through the bastion.
3. **Agent forwarding.** The `-A` flag forwards my SSH agent through the bastion, so the second hop can use my private key without ever copying `training-key.pem` onto the bastion.

## Why the private instance can still reach the internet
It has no public IP, so nothing on the internet can start a connection to it. For outbound traffic such as `sudo yum update -y`, the private subnet's route table sends 0.0.0.0/0 to the NAT Gateway, which lives in a public subnet with an Elastic IP. The NAT Gateway swaps the private source address for its own public one, sends the request out through the Internet Gateway, and passes the reply back. The connection is only ever started from the inside.

## Security groups vs. NACLs
Security groups are stateful and attached to instances: if an inbound request is allowed, the reply is allowed automatically. The bastion pattern depends on one security group referencing another by ID. NACLs are stateless and sit on the subnet; I left them at the defaults.

## Debug Lab 5.3, in my own words
If the private instance cannot reach the internet, the usual cause is that the private subnet was never associated with the route table that has the NAT route. "Private" is not a property a subnet inherits; routing is explicit, so the fix is VPC → Route tables → the private route table → Subnet associations → add the private subnet.

## Cleanup
Terminated both instances, deleted the NAT Gateway, released its Elastic IP, then deleted the VPC (which removed the subnets and route tables). The NAT Gateway bills about $0.045 per hour plus the Elastic IP, so it was torn down the same day.
