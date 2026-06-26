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
