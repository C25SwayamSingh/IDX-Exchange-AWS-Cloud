# Week 6 Status (Nov 17, 2026)

## Lab 6.1: Auto Scaling group behind an Application Load Balancer (built)
All in us-east-2, using the default VPC. The Week 5 training-vpc and its NAT gateway were torn down the same day, so this lab uses the default VPC's public subnets instead, which avoids the NAT gateway's hourly cost.

- **Launch template** `property-api-template`: Amazon Linux 2023, t3.micro, the Week 3 `bootstrap.sh` as user data, security group `property-api-sg` (TCP 8080).
- **Target group** `property-api-tg`: instance targets, HTTP 8080, health check path `/health`.
- **Application Load Balancer** `property-api-alb`: internet-facing across us-east-2a and 2b, security group `alb-sg` (HTTP 80 from anywhere), listener HTTP:80 forwarding to the target group.
- **Auto Scaling group** `property-api-asg`: desired 2, min 1, max 4, ELB health checks on, target-tracking policy on average CPU at 50%.
- `property-api-sg` also allows TCP 8080 from `alb-sg`, so the load balancer can reach the instances (the Debug Lab 6.3 fix, applied up front).

## Next
- Verify `curl http://<alb-dns>/health` returns `{"status":"ok"}`.
- Lab 6.2: terminate one instance and watch the ASG replace it; run the CPU stress test and capture the scale-out in CloudWatch for `load-test-report.md`.
- Teardown: delete the ASG, then the ALB, then the target group, then the launch template.
