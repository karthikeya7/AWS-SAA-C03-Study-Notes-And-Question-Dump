# Module 03: Compute Services

## 📚 Overview

AWS compute services for SAA-C03 exam: EC2, Lambda, containers, Auto Scaling, and Load Balancers.

**Exam Weight**: ~20-25%

---

## 🎯 Key Services

1. **Amazon EC2** - Virtual servers
2. **AWS Lambda** - Serverless functions
3. **Auto Scaling** - Automatic capacity management
4. **Elastic Load Balancing** - Distribute traffic
5. **ECS/EKS/Fargate** - Container services
6. **Elastic Beanstalk** - PaaS deployment

---

## 1. Amazon EC2

### Instance Types (Memorize)

| Letter | Stands for | Optimized for |
|--------|-----------|---------------|
| **C** | Compute | High CPU — video encoding, batch processing, gaming servers |
| **R** | RAM (memory) | High RAM — in-memory databases, caching (Redis), big data |
| **M** | Medium/general | Balanced CPU+RAM — most everyday apps, web servers |
| **T** | Burstable ("burs**T**") | Cheap, low baseline CPU that can burst — small websites, dev/test |
| **I / D / H** | I/O, Density, Hard-disk (storage) | High disk throughput — databases needing fast local storage, data warehousing |

**Mnemonic (letters in order C-R-M-T-I-D-H):**
> "Crazy Rabbits Munch Tiny Insects During Hunts"

### Pricing Models
| Model | Discount | Use Case |
|-------|----------|----------|
| On-Demand | None | Short-term, unpredictable |
| Reserved (1-3 yr) | Up to 72% | Steady state |
| Savings Plans | Up to 66% | Flexible commitment |
| Spot | Up to 90% | Fault-tolerant workloads |
| Dedicated | None | Compliance, licensing |

### Placement Groups
- **Cluster**: Low latency, same AZ
- **Spread**: Max 7/AZ, critical apps
- **Partition**: Distributed apps (Hadoop, Kafka)

---

## 2. Auto Scaling

### Scaling Policies
1. **Target Tracking**: Maintain metric (e.g., CPU 50%)
2. **Step Scaling**: Add/remove based on alarms
3. **Scheduled**: Time-based scaling
4. **Predictive**: ML-based forecasting

### Key Concepts
- Min/Desired/Max capacity
- Health checks
- Lifecycle hooks
- Cooldown periods (default 300s)

---

## 3. Elastic Load Balancing

| Type | Layer          | Use Case                  | Features                          |
| ---- | -------------- | ------------------------- | --------------------------------- |
| ALB  | 7 (HTTP/HTTPS) | Microservices, containers | Path/host routing, Lambda targets |
| NLB  | 4 (TCP/UDP)    | Extreme performance       | Static IP, millions req/sec       |
| GWLB | 3 (Network)    | Virtual appliances        | Firewalls, IDS                    |
| CLB  | 4 & 7          | Legacy                    | Not recommended                   |

### Features
- Health checks
- Sticky sessions
- Cross-zone load balancing
- SSL/TLS termination

---

## 4. AWS Lambda

### Limits
- Max execution: **15 minutes**
- Memory: 128 MB - 10 GB
- /tmp storage: 512 MB - 10 GB
- Concurrent executions: 1,000 (default)

### Invocation Types
1. **Synchronous**: API Gateway, CLI
2. **Asynchronous**: S3, SNS, EventBridge
3. **Event Source Mapping**: SQS, Kinesis, DynamoDB Streams

### Pricing
- $0.20 per 1M requests
- $0.00001667 per GB-second

---

## 5. Container Services

### ECS (Elastic Container Service)
**Launch Types:**
- **EC2**: You manage instances
- **Fargate**: Serverless, AWS manages

**Components:**
- Task Definition
- Task
- Service
- Cluster

### EKS (Elastic Kubernetes Service)
- Managed Kubernetes
- Compatible with standard K8s
- $0.10/hour cluster cost

### Fargate
- Serverless containers
- Works with ECS and EKS
- Pay per task

---

## 6. Other Services

### Elastic Beanstalk (PaaS)
- Deploy code, AWS handles infrastructure
- Supports: Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker
- Auto scaling, load balancing included

### AWS Batch
- Managed batch computing
- Automatic resource provisioning
- Uses Spot instances for cost savings

### Lightsail
- AWS's own VPS-style product — simple virtual server with fixed monthly pricing
- Pre-configured templates (WordPress, LAMP, Node.js, etc.)
- Built on EC2 under the hood, but hides the complexity (no VPC/IAM/Auto Scaling config needed)
- Closest AWS equivalent to a cheap third-party VPS (Hostinger, DigitalOcean)

### VPS vs EC2 vs Lightsail

| | VPS (Hostinger, DigitalOcean) | Lightsail | EC2 |
|---|---|---|---|
| **What it is** | Fixed-size virtual server, flat price | AWS's simplified fixed-price VM | Virtual server + full AWS ecosystem |
| **Scaling** | Manual resize | Manual resize (limited) | Auto Scaling — automatic |
| **Pricing** | Flat monthly | Flat monthly (simple tiers) | Pay-per-second; On-Demand/Reserved/Spot |
| **Networking** | Basic | Simplified VPC | Full VPC — subnets, SGs, NACLs, ALB/NLB |
| **High availability** | Single server | Limited | Multi-AZ / multi-region |
| **Best for** | Small sites, side projects | Simple apps wanting AWS without complexity | Production workloads needing scale, HA, integration |

**Memory trick:** VPS = one fixed box, DIY. Lightsail = AWS's "easy mode" VPS. EC2 = a box plugged into the entire AWS platform.

---

## 🎯 Exam Tips

### Decision Tree
```
Need compute?
├─ Run code without servers? → Lambda
├─ Containers?
│  ├─ Without managing servers? → Fargate
│  ├─ With Kubernetes? → EKS
│  └─ AWS orchestration? → ECS
├─ Just deploy code? → Elastic Beanstalk
└─ Full control? → EC2
```

### Common Scenarios
- **Variable workload** → Auto Scaling + Spot
- **Lowest latency networking** → Cluster placement group
- **Content-based routing** → ALB
- **Millions of req/sec, static IP** → NLB
- **Steady 3-year workload** → Reserved Instances
- **Fault-tolerant batch job** → Spot Instances
- **Run code on events** → Lambda
- **Serverless containers** → Fargate

### Key Points
- EC2 instance store = ephemeral (lost on stop)
- EBS = persistent block storage (AZ-specific)
- T instances = burstable CPU
- Spot instances = 2-minute warning before termination
- Lambda max = 15 minutes
- ALB = Layer 7, content routing
- NLB = Layer 4, extreme performance
- Auto Scaling ensures min/max capacity

---

**Estimated Study Time**: 8-10 hours  
**Difficulty**: ⭐⭐⭐

[⬅️ Previous: IAM](../02-IAM/README.md) | [Next: Storage ➡️](../04-Storage/README.md) | [📚 Main](../README.md)

