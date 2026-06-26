# terraform-aws-3-tier

## Architecture

```architecture
                    Internet
                        |
                        ▼
              Application Load Balancer
                        |
                        ▼
                  Target Group
                        |
        ┌───────────────┴───────────────┐
        ▼                               ▼
    EC2 Instance                    EC2 Instance
       (ASG)                           (ASG)
        │                               │
        └───────────────┬───────────────┘
                        ▼
                   Amazon RDS
                  (Private DB)
```



```architecture
[ PUBLIC INTERNET ]
                               │
                               ▼ (Port 80)
               ┌──────────────────────────────────────┐
               │     Application Load Balancer (ALB)  │  <── Tier 1: Presentation (Public Subnets)
               └──────────────────┬───────────────────┘
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼ (Routes to Port 5000)                           ▼ (Routes to Port 5000)
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│     [ Availability Zone A ]     │       │     [ Availability Zone B ]     │
│                                 │       │                                 │
│  ┌───────────────────────────┐  │       │  ┌───────────────────────────┐  │
│  │   Public Subnet (AZ1)     │  │       │  │   Public Subnet (AZ2)     │  │
│  │  - [NAT Gateway]          │  │       │  │  - (Empty/ALB Entry)      │  │
│  └─────────────┬─────────────┘  │       │  └───────────────────────────┘  │
│                │                │       │                                 │
│  ┌─────────────▼─────────────┐  │       │  ┌───────────────────────────┐  │
│  │   Private Subnet (AZ1)    │  │       │  │   Private Subnet (AZ2)    │  │  <── Tier 2: Application (Private)
│  │  - EC2 App Instance 1     │  │       │  │  - EC2 App Instance 2     │  │      (Managed by Auto Scaling Group)
│  └─────────────┬─────────────┘  │       │  └─────────────┬─────────────┘  │
│                │                │       │                │                │
│                │                │       │                │                │
│  ┌─────────────▼─────────────┐  │       │  ┌─────────────▼─────────────┐  │
│  │  Private DB Subnet (AZ1)  │  │       │  │  Private DB Subnet (AZ2)  │  │  <── Tier 3: Data Layer (Isolated)
│  │  - [RDS MySQL DB Server]  │◄─┼───────┼──┤  - (Empty/Backup Route)   │  │      (1 Active Instance inside DB Subnet Group)
│  └───────────────────────────┘  │       │  └───────────────────────────┘  │
└─────────────────────────────────┘       └─────────────────────────────────┘
```



Application logic chahe jitni marzi tiers mein divided ho, AWS Cloud Setup default security zones ke mutabik 3-Tier Network standard (Public ➔ Private App ➔ Private DB) par hi chalta hai.

summary ka tor pr:

App Code: 2-Tier hai (Flask Frontend App + Database).

AWS Cloud Setup: 3-Tier Network hai (Public ALB Layer ➔ Private EC2 Layer ➔ Private RDS Layer).

1. Agar App Code 3-Tier ho (Real Production Apps)
2. 
Real-world production mein app code aise divided hota hai:

Frontend Tier: Angular, React, ya Next.js (jo user ko browser mein browser mein dikhta hai).

Backend/API Tier: Node.js, Python Flask, ya Spring Boot (jo business logic chalata hai).

Database Tier: MySQL, PostgreSQL (jo data store karta hai).

2. Is 3-Tier App Code ke liye AWS Cloud Setup kaisa hoga?
3. 
Jab hum cloud infrastructure design karte hain, toh cloud setup 4-tier nahi banta, balki setup ka Tier 2 (Private Subnet) thoda broad ho jata hai.

Cloud design tab bhi 3-Tier Network Setup hi rehta hai, lekin components aise adjust hote hain:

Tier 1: Presentation Tier (Public Subnets)

Internet-Facing ALB (Load Balancer) ➔ Internet se request leta hai.

Frontend Servers (React/Static Files) ➔ Yahan se user ko UI milti hai (Kayi baar log frontend ko direct S3 + CloudFront par dal dete hain presentation tier ke taur par).

Tier 2: Application / Logic Tier (Private Subnets)

Internal ALB ➔ Ek chota internal load balancer jo frontend se request le kar backend ko deta hai.

Backend App Servers (Node.js/Flask via ASG) ➔ Jo asli business logic aur queries process karte hain. Yeh layer internet se bilkul hidden hoti hai.

Tier 3: Data Tier (Deep Private Subnets)

Database Nodes (RDS/Aurora) ➔ Jo sirf aur sirf Tier 2 ke backend servers se baat karte hain.

