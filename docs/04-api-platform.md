
# API Platform

  

## Overview

  
The Node.js REST API runs as containers on **Amazon ECS Fargate**. Fargate is a good fit here because the API is stateless and can scale horizontally without managing EC2 instances.

```text
ALB
 |
 v
ECS Service
 |
 +--> Fargate Task AZ-A
 |
 +--> Fargate Task AZ-B

```

  

## Container Image

  

The API container image is stored in Amazon ECR.

  

Typical build flow:

  

1. Build the Node.js application.

2. Build the Docker image.

3. Scan the image.

4. Push the image to ECR.

5. Deploy the new image to ECS.

  

Image tags should be traceable to a commit SHA or release number. Avoid deploying from a floating `latest` tag in production.

  

## ECS Service

  

The ECS service should start with at least two tasks across two Availability Zones.

  

- Minimum tasks: 2

- Deployment type: rolling update

- Health checks: ALB target group health check

- Logs: send container logs to CloudWatch Logs

- Secrets: read from Secrets Manager or Parameter Store

  

The service should be stateless. If a task stops, another task should be able to replace it without user-visible data loss.

  

## Application Load Balancer

  

The ALB receives HTTPS traffic and forwards requests to healthy ECS tasks.

  

- HTTPS listener: port 443

- HTTP listener: port 80 redirecting to HTTPS

- Certificate: ACM certificate for the API domain

- Health check: target group endpoint such as `/health`

- Access logs: enable when traffic grows enough to justify the storage cost

  

## Runtime Configuration

  

The API should receive configuration through environment variables and secret references, not files baked into the image.

  

- Database host and name: environment variables

- Database credentials: Secrets Manager

- Allowed origins: environment variables

- Log level: environment variables

- External service endpoints: environment variables or Parameter Store

  

## Failure Handling

  

The API should fail fast if required configuration is missing. ECS can then replace unhealthy tasks instead of running containers that cannot serve traffic correctly.