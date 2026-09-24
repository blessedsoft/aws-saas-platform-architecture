# Database Design

## Overview

The database layer uses Amazon RDS for PostgreSQL with Multi-AZ enabled. This gives the platform a managed relational database with automated backups, patching options and failover support.

```plaintext
ECS API tasks
   |
   v
RDS PostgreSQL
   |
   +--> standby in another AZ
```

## Launch Setup

- Engine: PostgreSQL on RDS
- Availability: Multi-AZ deployment
- Subnets: private subnet group across at least two AZs
- Storage: storage autoscaling enabled
- Backups: automated backups enabled
- Encryption: KMS encryption at rest
- Public access: disabled

The exact instance size should be chosen from expected workload tests, then adjusted after real traffic data is available.

## Access Control

Only the ECS API security group should be allowed to connect to PostgreSQL.

Database credentials should be stored in Secrets Manager. Application users should not share the database admin account. Use a separate application database user with only the permissions it needs.

## Backups

Backups should be enabled from day one.

- Automated backups: retain for the agreed business recovery period.
- Point-in-time recovery: enable it.
- Major schema changes: take a manual snapshot first.
- Restore process: test it on a schedule.

Backups are only useful if restore has been tested.

## Connection Management

At launch, direct connections from the API may be enough. As traffic grows, connection pooling becomes important because PostgreSQL has practical connection limits.

Use RDS Proxy or application-level pooling when these signs appear.

- Task count increases significantly, because more tasks can create too many database connections.
- The API opens many short-lived connections, because connection churn can hurt PostgreSQL performance.
- CPU or connection count rises during spikes, because pooling can smooth pressure on the database.

## Scaling Path

Database scaling should happen in this order:

1. Fix slow queries and missing indexes.
2. Tune connection pooling.
3. Increase instance size.
4. Add read replicas for read-heavy workloads.
5. Consider partitioning for very large tables.
6. Consider sharding only when simpler options are no longer enough.
