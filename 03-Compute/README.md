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

**Car rental analogy** — helps explain *why* each one costs what it does, not just the label:

| Model | Car rental analogy | Discount | Use Case |
|-------|--------------------|----------|----------|
| On-Demand | Renting by the hour, no booking ahead | None (full price) | Short-term, unpredictable |
| Reserved (1-3 yr) | Signing a 1-3 year lease — cheaper because you commit | Up to 72% | Steady state |
| Savings Plans | Flexible subscription — commit to *spend*, not a specific car | Up to 66% | Flexible commitment |
| Spot | Bidding on a used car at auction — cheap, but owner can reclaim anytime | Up to 90% | Fault-tolerant workloads |
| Dedicated | Your own private garage — nobody else's car touches your space | None (pay for isolation, not discount) | Compliance, licensing |

**Memory hook:** "The longer you commit, the more you save — except Spot, where you're gambling instead of committing."

**Ranking by discount (cheapest → priciest): Spot (90%) → Reserved (72%) → Savings Plans (66%) → On-Demand/Dedicated (0%)**
> Mnemonic: "**S**cared **R**enters **S**ave, **O**thers **D**on't." (Spot → Reserved → Savings → On-Demand/Dedicated)

### Placement Groups

Controls the physical arrangement of EC2 instances relative to each other — for either **speed** or **safety**.

| Type | What it does | Tradeoff | Use when | Analogy |
|------|--------------|----------|----------|---------|
| **Cluster** | Packs instances physically close together (same rack, same AZ) → near-zero latency | If that rack/location fails, you could lose them all together | Tightly-coupled HPC, big data, high-performance computing | Seating your whole team at one table — fast to talk, but if the table collapses, everyone falls |
| **Spread** | Spreads instances across different hardware (different racks; max 7/AZ) | Slightly higher latency since they're not physically close | Small number of critical instances where you want to minimize shared-hardware failure risk | Seating key leads at 7 different tables in different rooms |
| **Partition** | Divides instances into isolated partitions (max 7/AZ); instances *within* a partition can be close | None significant — designed for distributed systems | Large distributed apps that understand partitions/racks (Hadoop, Kafka, Cassandra, HDFS) | Splitting your team into separate rooms — close together within a room, isolated between rooms |

**One-line summary:** Cluster = fast but risky (all eggs, one basket). Spread = safe but limited (max 7, one egg per basket). Partition = safe *and* scalable (many baskets, several eggs per basket).

---

## 2. Auto Scaling

Automatically adds/removes EC2 instances based on real demand — never paying for idle servers, never getting crushed by unexpected traffic. Cost note: **Auto Scaling itself is free** — you only pay for the EC2 instances (and attached EBS/ELB) it launches, same as if you'd launched them manually.

### Auto Scaling Group (ASG)

The container/manager that ties everything together — you don't scale individual EC2 instances directly, you create an ASG and it manages a *group* of identical instances for you.

**Core components:**

| Component | What it defines |
|-----------|-----------------|
| **Launch Template** (or older Launch Configuration) | The "recipe" — AMI, instance type, key pair, security groups, user data. Every instance in the group is a clone of this |
| **Min / Desired / Max capacity** | The boundaries — never below Min, never above Max, tries to sit at Desired |
| **VPC + Subnets/AZs** | Which subnets/AZs instances launch into — this is how it spreads instances for high availability |
| **Scaling policies** | The rules for *when* to change capacity (below) |
| **Health checks** | Rules for what counts as "unhealthy" and needs replacing |
| **Lifecycle hooks** | Optional pause points during launch/terminate |
| **Load balancer attachment (optional)** | New instances auto-register with the ALB/NLB; terminated ones auto-deregister |

**What the ASG does continuously, without manual intervention:**
1. **Maintains capacity** — replaces crashed/unhealthy instances automatically, even with zero traffic change (self-healing, not just scaling)
2. **Scales based on policies** — adjusts Desired capacity per the rules below
3. **Distributes across AZs** — spreads instances evenly for fault tolerance
4. **Registers/deregisters with load balancer** — automatic if attached

**Analogy:** the ASG is a **restaurant manager**, not the wait staff. Launch Template = the training manual every new hire follows identically. Min/Desired/Max = "always have at least 2 staff, ideally 4, never more than 10." Scaling policies = the manager's rule book for calling in more staff. Health checks = the manager noticing someone's sick and calling in a replacement — automatically, without you personally hiring/firing each person.

### Scaling Policies
1. **Simple Scaling**: The oldest/basic policy — one CloudWatch alarm triggers one fixed action (e.g. "add 1 instance"), then waits out the Cooldown period before reacting again
2. **Target Tracking**: Maintain metric (e.g., CPU 50%) — like a thermostat, set the target and AWS handles the rest
3. **Step Scaling**: Add/remove based on CloudWatch alarms (e.g., "if CPU > 70%, add 2 instances")
4. **Scheduled**: Time-based scaling — e.g. add capacity every weekday 9am, remove at 6pm
5. **Predictive**: ML-based forecasting — scales *ahead* of demand instead of reacting to it

#### Target Tracking vs Step Scaling (the confusing pair)

**Both use CloudWatch alarms under the hood — the real difference is who sets up the alarm and who decides how much to scale by.**

| | Target Tracking | Step Scaling |
|---|---|---|
| You configure | One target value (e.g. "CPU = 50%") | Alarm thresholds + exact scale amount per threshold |
| Creates the CloudWatch alarm | AWS does it automatically | You create it manually |
| Decides how many instances to add | AWS's algorithm calculates it | You decide, in steps |
| Control | Low — simple, hands-off | High — custom, graduated response |

**Example — Step Scaling manually defined:**
```
Alarm: CPU > 60% for 5 min → trigger
60-70% → add 1 instance
70-80% → add 2 instances
> 80%  → add 4 instances
```

**Rule of thumb:** Target Tracking = "set it and forget it" (use this by default). Step Scaling = only when you need different-sized reactions for different severity levels.

### Key Concepts

**Min/Desired/Max capacity** — the boundaries: never fewer than Min, never more than Max, tries to stay at Desired.

**Health checks** — checking itself is automatic once the ASG is running, but you choose the source:

| Type | What it checks | Enabled by default? |
|------|----------------|----------------------|
| EC2 status checks | Is the instance/hardware itself running | Yes — always on by default |
| ELB health checks | Is the app actually responding correctly (e.g. HTTP 200 on `/health`) — deeper than "is the VM alive" | No — must explicitly enable if behind a load balancer |
| Custom health checks | Your own script/Lambda reports health via `SetInstanceHealth` API | No — fully manual |

Once configured: instance fails check → marked Unhealthy → ASG automatically terminates it and launches a replacement to restore Desired capacity. No manual action needed. You must, however, choose which check type and (for ELB checks) build the actual health endpoint in your app — AWS doesn't invent that for you.

**Lifecycle hooks** — a pause button during instance launch or termination, giving you time to run custom actions before the instance goes live (or before it's destroyed). **Opt-in, not automatic** — by default there's no pause.
- Instance enters `Pending:Wait` (launch) or `Terminating:Wait` (termination)
- Your custom logic runs (e.g. Lambda/script — install software, run smoke tests, drain connections, back up logs)
- You call `CompleteLifecycleAction` to continue, or it times out (default 1 hour) and proceeds anyway
- Example: don't let a new instance receive traffic until it's downloaded the latest build and passed a smoke test

**Cooldown periods (default 300s)** — after a scaling activity, further scaling triggers are ignored for this duration, to prevent "flapping" (scale-out drops CPU → immediate scale-in → CPU rises again → loop).

**Important: Cooldown does NOT apply equally to all policies:**

| Policy | Uses classic Cooldown? | Uses instead |
|--------|--------------------------|---------------|
| Simple Scaling | **Yes** — only this policy type uses it | — |
| Step Scaling | No | **Instance Warmup** |
| Target Tracking | No | **Instance Warmup** |
| Scheduled / Predictive | Not applicable | Directly sets capacity — no alarm to cool down from |

Cooldown = a blunt global pause (freezes *all* scaling activity for the duration). Instance Warmup = a smarter alternative that just excludes newly-launched instances from metric calculations until they're contributing real load data, instead of freezing everything. Both have automatic defaults — no manual setup required to get reasonable behavior, though both can be tuned.

### Does Auto Scaling Apply to ECS, EKS, Lambda Too?

**Short answer: EC2 Auto Scaling Groups (above) are only for EC2.** Other services scale differently:

| Service | How it scales |
|---------|----------------|
| **EC2** | Auto Scaling Group (ASG) — what's covered above |
| **ECS** | Scales *task count* (Service Auto Scaling). If using EC2 (not Fargate), the underlying EC2 nodes also scale via a normal ASG |
| **ECS + Fargate** | Only scales task count — no servers to manage at all |
| **EKS** | Scales *pods* (Horizontal Pod Autoscaler) and *worker nodes* (Cluster Autoscaler/Karpenter) separately |
| **Lambda** | Fully automatic, no configuration — scales per request instantly, no Min/Max/ASG concept at all |

**One-line memory hook:** EC2 = you manage the ASG. ECS/EKS = scaling happens at two levels (app + servers). Lambda = no servers, so nothing to scale manually — it just happens.

---

## 3. Elastic Load Balancing

The traffic cop that sits in front of your instances/containers and spreads incoming requests across them, so no single server gets overwhelmed. "Layer" = how deep it looks into your traffic (OSI model).

| Type | Layer          | Use Case                  | Features                          |
| ---- | -------------- | ------------------------- | --------------------------------- |
| ALB  | 7 (HTTP/HTTPS) | Microservices, containers | Path/host routing, Lambda targets |
| NLB  | 4 (TCP/UDP)    | Extreme performance       | Static IP, millions req/sec       |
| GWLB | 3 (Network)    | Virtual appliances        | Firewalls, IDS                    |
| CLB  | 4 & 7          | Legacy                    | Not recommended                   |

**Quick pick:** ALB = smart routing by URL/content. NLB = raw speed, no content inspection. GWLB = routes through security appliances first. CLB = old, avoid for new builds.

### Features
- **Health checks** — pings each target; unhealthy ones stop receiving traffic
- **Sticky sessions** — same user always routed to the same backend instance
- **Cross-zone load balancing** — spreads traffic evenly across all AZs, not just one
- **SSL/TLS termination** — LB handles HTTPS decryption, backend only deals with plain HTTP

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

