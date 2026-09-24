# Scaling Strategy

## Starting Point

The platform starts with simple horizontal scaling for the API and measured vertical scaling for the database.

The first goal is not to predict every future bottleneck. The first goal is to make each tier observable, replaceable and independently scalable.

## API Scaling

ECS Service Auto Scaling should adjust the number of Fargate tasks based on real workload signals.

- CPU utilisation: when API tasks need more compute capacity
- Memory utilisation: when tasks are under memory pressure
- ALB requests per target: whether each task is receiving too much traffic
- Target response time: whether users are seeing slower responses
- Error rate: whether scaling or application fixes are needed

Recommended launch setup:

- Minimum tasks: 2
- Maximum tasks: set a practical cap to control cost
- Scale out: increase capacity quickly when load rises
- Scale in: reduce capacity slowly to avoid task churn

## Frontend Scaling

The frontend scales mainly through CloudFront caching. S3 and CloudFront can handle large traffic increases with little operational work.

The main tuning points are cache headers and invalidation strategy.

## Database Scaling

Database scaling should be more careful than API scaling because the database is stateful.

Preferred order:

1. Improve queries and indexes.
2. Add connection pooling.
3. Increase RDS instance class.
4. Add read replicas for read-heavy traffic.
5. Split or partition large tables when the data shape supports it.

## Expected Growth

- 10,000 users at launch: two API tasks, one Multi-AZ RDS database, CloudFront in front of S3, basic alarms and dashboards.
- 500,000 users: more API tasks, right-sized RDS, read replicas where needed and a stronger cache strategy.

## Scaling Guardrails

Scaling should be tied to monitoring. Before increasing capacity, check whether the issue is caused by inefficient code, missing indexes, slow external dependencies or poor cache behaviour.
