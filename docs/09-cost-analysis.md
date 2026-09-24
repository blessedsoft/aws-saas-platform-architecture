# Cost Analysis

## Cost Goal

The launch target is to keep the platform near or under $500 per month during low traffic, while still using a production-ready architecture.

Actual cost depends on traffic, data size, region, instance choices and log volume, so this document should be treated as a planning guide rather than an exact bill.

## Main Cost Drivers

- ECS Fargate: CPU, memory and number of running tasks
- RDS PostgreSQL: instance size, Multi-AZ, storage and backups
- NAT Gateway: hourly charge and data processing
- CloudFront: data transfer and requests
- S3: storage, requests and data transfer to CloudFront
- CloudWatch: logs, metrics, alarms and retention

## Launch-Sized Setup

A cost-conscious launch setup can start with:

- API: two small ECS Fargate tasks
- Database: one Multi-AZ RDS PostgreSQL instance sized from load testing
- Frontend storage: one S3 bucket for build assets
- CDN: CloudFront distribution for frontend delivery
- Logging: CloudWatch log retention limits
- Outbound network: NAT Gateway kept minimal, with one per AZ when availability needs justify it

The largest recurring costs will usually be RDS, NAT Gateway and always-on compute.

## Optimisation Ideas

- Early: right-size Fargate CPU and memory from observed usage.
- Early: set CloudWatch log retention instead of keeping logs forever.
- Early: cache frontend assets aggressively in CloudFront.
- Early: use S3 lifecycle rules for old build artifacts.
- Early: review NAT Gateway traffic before adding more outbound dependencies.
- Later: use Savings Plans for steady compute usage.
- Later: use Reserved Instances for stable RDS usage.
- Later: size read replicas based on actual read load.

## What Not To Optimise Too Early

Do not remove Multi-AZ from the production database just to reduce cost unless the business accepts the availability risk.

Do not add complex architecture only because it appears cheaper in theory. Operational time is also a cost.
