
# Network Design

  

## VPC Architecture

  

The platform runs in a single VPC distributed across two Availability Zones. Public-facing components are isolated from application and database workloads.

  

## High-Level Architecture

  

```mermaid

flowchart TB

  

Internet((Internet))

  

CloudFront["Amazon CloudFront<br/>CDN / HTTPS"]

  

subgraph VPC["AWS VPC"]

  

subgraph AZA["Availability Zone A"]

  

subgraph PublicA["Public Subnet A"]

ALBA["Application Load Balancer"]

NATA["NAT Gateway"]

end

  

subgraph AppA["Private App Subnet A"]

ECS_A["ECS Fargate<br/>API Tasks"]

end

  

subgraph DataA["Private Data Subnet A"]

RDS_A["RDS PostgreSQL<br/>Multi-AZ"]

end

  

end

  

subgraph AZB["Availability Zone B"]

  

subgraph PublicB["Public Subnet B"]

ALBB["Application Load Balancer"]

NATB["NAT Gateway"]

end

  

subgraph AppB["Private App Subnet B"]

ECS_B["ECS Fargate<br/>API Tasks"]

end

  

subgraph DataB["Private Data Subnet B"]

RDS_B["RDS PostgreSQL<br/>Standby"]

end

  

end

  

end

  

IGW["Internet Gateway"]

  

Internet --> CloudFront

CloudFront -->|HTTPS| ALBA

CloudFront -->|HTTPS| ALBB

  

ALBA -->|HTTP / HTTPS| ECS_A

ALBA -->|HTTP / HTTPS| ECS_B

  

ALBB -->|HTTP / HTTPS| ECS_A

ALBB -->|HTTP / HTTPS| ECS_B

  

ECS_A -->|PostgreSQL :5432| RDS_A

ECS_A -->|PostgreSQL :5432| RDS_B

  

ECS_B -->|PostgreSQL :5432| RDS_A

ECS_B -->|PostgreSQL :5432| RDS_B

  

ECS_A --> NATA

ECS_B --> NATB

  

NATA --> IGW

NATB --> IGW

  

RDS_A <--> RDS_B

  

```

  

## Network Layout

  

```mermaid

flowchart LR

  

subgraph VPC["AWS VPC"]

  

subgraph Public["Public Subnets"]

ALB["Application Load Balancer"]

NAT1["NAT Gateway AZ-A"]

NAT2["NAT Gateway AZ-B"]

end

  

subgraph PrivateApp["Private Application Subnets"]

ECS1["ECS Fargate<br/>AZ-A"]

ECS2["ECS Fargate<br/>AZ-B"]

end

  

subgraph PrivateDB["Private Data Subnets"]

RDS1["RDS PostgreSQL<br/>Primary"]

RDS2["RDS PostgreSQL<br/>Standby"]

end

  

end

  

ALB --> ECS1

ALB --> ECS2

  

ECS1 --> RDS1

ECS2 --> RDS1

  

ECS1 --> NAT1

ECS2 --> NAT2

  

RDS1 <--> RDS2

  

```

  
  

## Traffic Flow

  

```mermaid

flowchart LR

  

User((User))

CF["CloudFront"]

ALB["Public ALB"]

ECS["ECS Fargate API"]

RDS["RDS PostgreSQL"]

  

User -->|HTTPS :443| CF

CF -->|HTTPS| ALB

ALB -->|Application Port| ECS

ECS -->|PostgreSQL :5432| RDS

  

```

  

## Outbound Traffic

  

```mermaid

flowchart LR

  

ECS["ECS Fargate<br/>Private Subnet"]

NAT["NAT Gateway<br/>Public Subnet"]

IGW["Internet Gateway"]

Internet((Internet))

  

ECS -->|Outbound HTTPS| NAT

NAT --> IGW

IGW --> Internet

  

```

  
  
  


## Subnet Design

| Layer | AZ-A | AZ-B | Internet Route |
|---|---|---|---|
| **Public** | ALB + NAT Gateway | ALB + NAT Gateway | Internet Gateway |
| **Application** | ECS Fargate | ECS Fargate | NAT Gateway |
| **Database** | RDS PostgreSQL | RDS PostgreSQL | None |

### Public Subnets
* **Hosted Resources:** Application Load Balancer (ALB), NAT Gateway (NAT GW)
* **Routing:** Public route tables route outbound traffic directly through the **Internet Gateway** for full internet connectivity.

### Private Application Subnets
* **Hosted Resources:** ECS Fargate API tasks
* **Routing:** These subnets have no direct internet exposure. Outbound internet traffic is securely routed through the **NAT Gateway** in the corresponding Availability Zone.

### Private Data Subnets
* **Hosted Resources:** RDS PostgreSQL
* **Routing:** Database subnets are fully isolated with **no route to the internet**. The RDS instances strictly accept database connections only from the ECS application security group.

  

## Security Group Flow

  

```mermaid

flowchart LR

  

CF["CloudFront"]

  

ALBSG["ALB Security Group"]

ECSSG["ECS Security Group"]

RDS_SG["RDS Security Group"]

  

CF -->|HTTPS :443| ALBSG

ALBSG -->|Application Port| ECSSG

ECSSG -->|PostgreSQL :5432| RDS_SG

  

```

  

Security groups should reference other security groups rather than relying on broad CIDR ranges wherever possible.

  

## Security Boundaries

  

*  **CloudFront:** Public entry point and CDN layer.

*  **ALB:** Internet-facing load-balancing boundary.

*  **ECS:** Private application execution layer.

*  **RDS:** Isolated database layer.

*  **NAT Gateway:** Controlled outbound Internet access for private workloads.

*  **Security Groups:** Primary stateful network access control.

*  **NACLs:** Kept simple initially and tightened only where there is a documented requirement.

  

## Network ACLs
 

Network ACLs remain intentionally simple at launch.

  

Security Groups provide the primary workload-level access control because they are stateful and can reference other security groups.

  

NACLs can be introduced or tightened for specific subnet-level security requirements after validating the operational impact.