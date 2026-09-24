# Security

## Security Model

The security model keeps public access at the edge and protects application and data workloads in private subnets.

```plaintext
Internet
   |
   v
CloudFront / ALB
   |
   v
Private ECS tasks
   |
   v
Private RDS database
```

## Network Security

- Users to CloudFront or ALB: HTTPS only.
- ALB to ECS tasks: API port only.
- ECS tasks to RDS: PostgreSQL port only.
- RDS to public internet: no access.

The database must not be publicly accessible.

## IAM

Use least-privilege IAM roles.

- ECS task execution role: pulls images and writes logs.
- ECS task role: provides application AWS permissions.
- CI/CD deployment role: pushes images and updates services.
- Read-only operational role: supports investigation without write access.

Avoid long-lived access keys for application workloads.

## Secrets

Store sensitive values in Secrets Manager or Parameter Store.

- Database password: Secrets Manager
- API signing secrets: Secrets Manager
- Third-party service credentials: Secrets Manager or Parameter Store

Secrets should not be committed to source control, baked into images or exposed in frontend builds.

## TLS

Use HTTPS for all public traffic.

- Frontend domain: ACM
- API domain: ACM

Traffic from the API to the database should also use encrypted connections where supported by the PostgreSQL client and RDS configuration.

## Logging and Auditing

Enable CloudTrail for account-level audit events. Send application logs to CloudWatch Logs.

Logs should be useful, but they should not contain passwords, tokens, payment data or full personal records.
