# Canary Deployment Using AWS Application Load Balancer

## Project Overview
Designed and implemented a production-style canary deployment strategy on AWS using an Application Load Balancer (ALB). The setup gradually introduced a new application version by routing 10% of incoming traffic to a canary target group while maintaining 90% traffic on the stable production environment.

The project demonstrates weighted routing, health checks, access log validation, and a safe rollback mechanism commonly used in real-world deployments.

---

## Architecture Overview

### Key Components
- Custom VPC with public and private subnets across two Availability Zones
- Bastion Host in a public subnet for secure SSH access
- Two EC2 instances running Version 1 (stable application)
- One EC2 instance running Version 2 (canary application)
- Application Load Balancer deployed in public subnets
- Two target groups:
  - v1 target group (stable)
  - v2 target group (canary)
- Weighted ALB listener rule:
  - 90% traffic to v1
  - 10% traffic to v2
- S3 bucket for storing ALB access logs
- Internet Gateway and NAT Gateway for connectivity

### Architecture Diagram
![ALB Canary Deployment Architecture](alb-canary-architecture.png.png)

---

## Canary Deployment Flow
1. Client requests reach the Application Load Balancer.
2. ALB listener distributes traffic using weighted routing:
   - 90% forwarded to Version 1 target group
   - 10% forwarded to Version 2 target group
3. Target groups route traffic to EC2 instances running different application versions.
4. ALB access logs are stored in Amazon S3 to verify real traffic distribution.
5. Rollback can be triggered instantly by resetting weights to 100% Version 1.

---

## Networking & Security Design
- Application servers are deployed only in private subnets.
- Public access is limited to the ALB and Bastion Host.
- Bastion Host provides controlled SSH access to private EC2 instances.
- Security Groups restrict traffic:
  - ALB allows inbound HTTP (80)
  - Private EC2 instances allow inbound traffic only from ALB
  - SSH allowed only from Bastion Host

Check the Step by Step Implemenation here - ![Documenation](STEP-BY-STEP_IMPLEMENTATION (2).pdf)
---

## Traffic Validation Using ALB Access Logs
- ALB access logging enabled and delivered to an S3 bucket.
- Logs were analyzed to inspect `target_group_arn` values.
- Majority of requests routed to the stable target group.
- Minority of requests routed to the canary target group.
- Distribution closely matched the configured 90/10 weighting.

This provided backend confirmation of correct canary behavior.

---

## Rollback Strategy
- Listener rules were updated to route:
  - 100% traffic to Version 1
  - 0% traffic to Version 2
- Rollback completed without downtime.
- Verified through browser testing and access log inspection.

---

## Challenges and Solutions

| Challenge | Resolution |
|--------|-----------|
| ALB access log permission errors | Added correct S3 bucket policy for log delivery |
| Difficulty analyzing raw access logs | Downloaded logs locally and inspected request routing fields |
| Verifying traffic distribution | Used both browser testing and access log validation |

---

## What This Project Demonstrates
- Canary deployment strategy using weighted routing
- Practical use of ALB target groups and listener rules
- Multi-AZ, secure AWS networking design
- Traffic observability using ALB access logs
- Safe rollout and instant rollback capability
- Production-style deployment practices

---

## Future Improvements
- Add CloudWatch metrics and alarms for canary monitoring
- Automate deployment using Terraform
- Introduce HTTPS with ACM
- Extend rollout strategy to blue-green deployments
