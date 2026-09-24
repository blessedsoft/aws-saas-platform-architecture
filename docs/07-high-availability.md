# High Availability

## Overview

The platform is designed to survive common infrastructure failures inside one AWS Region by using at least two Availability Zones.

This is not a full multi-region active-active design. That would add cost and complexity that the launch platform does not need yet.

## Availability Zone Resilience

- ALB spans public subnets in multiple AZs.
- ECS tasks run in private app subnets across AZs.
- RDS uses Multi-AZ with a standby in another AZ.
- NAT Gateway should be one per AZ when the budget and availability target justify it.

If one AZ has a problem, the ALB can route traffic to healthy tasks in another AZ. RDS Multi-AZ can fail over to its standby.

## Application Recovery

ECS should replace unhealthy tasks automatically.

Health checks should verify that the API process can serve requests, not just that the container is running. A simple `/health` endpoint is enough at launch.

## Database Failover

RDS Multi-AZ handles failover to a standby instance. Applications should be prepared for brief connection interruptions during failover.

- Use connection retry logic to handle brief failover interruptions.
- Avoid long transactions where possible to reduce risk during failover.
- Log database errors clearly to speed up incident diagnosis.
- Recover without manual restarts so ECS can restore healthy service automatically.

## Frontend Availability

CloudFront and S3 provide highly available static asset delivery. Even if the API has an issue, the frontend shell can still load and show a useful error state.

## Operational Expectations

High availability is not only infrastructure. It also needs:

- Alarms catch unhealthy targets and high error rates.
- Runbooks guide common failure response.
- Tested restores prove backups can be used.
- A rollback process helps recover from bad deployments quickly.

The design should be tested with controlled failure scenarios before major launch milestones.
