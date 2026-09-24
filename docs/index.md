# AWS SaaS Platform Architecture

This documentation describes a practical AWS architecture for a SaaS application starting at 10,000 users and growing toward 500,000 users.

## Start Here

- [Overview](./readme.md) - summary architecture and design decisions
- [Architecture](./01-architecture.md) - overall AWS service choices
- [Network Design](./02-network-design.md) - VPC, subnets, routing and security groups
- [Frontend Design](./03-frontend-design.md) - S3 and CloudFront setup
- [API Platform](./04-api-platform.md) - ECS Fargate, ECR and ALB
- [Database Design](./05-database-design.md) - RDS PostgreSQL and high availability
- [Scaling Strategy](./06-scaling-strategy.md) - application and database scaling path
- [High Availability](./07-high-availability.md) - AZ failure and recovery approach
- [Security](./08-security.md) - IAM, TLS, secrets and network boundaries
- [Cost Analysis](./09-cost-analysis.md) - launch cost drivers and optimisation
- [Deployment Plan](./10-deployment-plan.md) - step-by-step implementation order
- [Operations](./11-operational-considerations.md) - monitoring, CI/CD, runbooks and future improvements
