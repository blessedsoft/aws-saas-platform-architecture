# Operational Considerations

## Monitoring

The platform should be observable from the first production deployment.

- API: request count, latency and error rate
- ALB: unhealthy target count
- ECS: CPU, memory and task count
- RDS: CPU, storage, connections and replica lag if replicas exist
- CloudFront: error rate and cache hit rate

Dashboards should focus on service health, not every possible metric.

## Logging

Application logs should go to CloudWatch Logs.

- Use structured logs where practical to make searching and alerting easier.
- Include request IDs to help trace user issues.
- Set retention periods to control cost.
- Avoid secrets and sensitive personal data to reduce security and privacy risk.

## CI/CD

The deployment pipeline should be boring and repeatable.

- API pipeline: install dependencies, run tests and linting, build the container image, scan the image, push to ECR and update the ECS service.
- Frontend pipeline: install dependencies, run tests and linting, build the React app, upload build artifacts to S3 and invalidate CloudFront entry files.

## Rollback

Rollback should be planned before production launch.

For the API, keep previous ECS task definitions available so the service can be rolled back quickly.

For the frontend, keep previous S3 build artifacts or use versioned deployment paths.

For the database, use migrations carefully. Destructive schema changes should be split into safer steps.

## Runbooks

- API high error rate: diagnose application or dependency failures.
- ECS deployment failure: recover from a bad service rollout.
- Database failover: handle brief connection interruptions.
- Database restore: recover data from backups.
- CloudFront or DNS issue: check public routing and certificate problems.

Runbooks should be simple enough for someone else to follow during an incident.

## Future Improvements

- RDS Proxy: add when connection count becomes a concern.
- Read replicas: add when reporting or read-heavy traffic grows.
- WAF rules: add when public traffic patterns justify extra protection.
- Centralised tracing: add when request paths become harder to diagnose.
- Automated backup restore tests: add after the first production backup process is stable.
- Infrastructure as Code: add as soon as the environment needs repeatable rebuilds.
