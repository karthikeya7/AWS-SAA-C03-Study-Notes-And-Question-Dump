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

### AMI (Amazon Machine Image)

The "golden image" template used to launch an EC2 instance — includes the OS, installed software, and configuration. Every instance you launch is a clone of an AMI.

- **Exam trap:** AMIs are **region-specific**. If you need to launch the same instance setup in another region, you must explicitly **copy the AMI** to that region first — it doesn't exist there automatically.
- Common use: build a "golden image" once (OS + your app + patches), then launch/scale identical instances from it — faster and more consistent than configuring each instance by hand.

**How launching actually works:** an AMI is a frozen template, not something running. On launch, AWS creates a new EBS volume by copying the AMI's root snapshot, then boots a fresh instance from that volume. Every instance from the same AMI is an independent clone — changing a running instance never affects the AMI or other instances launched from it.

### User Data — why not just bake everything into the AMI?

**What it is:** a script (usually `#!/bin/bash`) you paste into the EC2 launch wizard (Advanced Details) that runs automatically **once, at first boot** — used to bootstrap the instance (install software, pull config, register with a service).

**The key question: if you can bake a script into the AMI, why use User Data at all?** Timing is the difference — AMI code is baked in at **image creation time** (frozen, identical every launch). User Data is supplied at **launch time** (fresh, can differ per launch) even from the exact same AMI.

**Analogy:** AMI = a printed recipe book, fixed once printed. User Data = a sticky note handed to the kitchen at the counter for this specific order ("no onions this time"). Reprinting the whole book for every small variation (rebuilding the AMI) is slow; a sticky note per order (User Data) is fast and flexible — same book, endless variations.

**Tradeoff table:**

| | Bake into AMI | User Data |
|---|---|---|
| Boot speed | Fast — nothing to install at runtime | Slower — script runs at every boot |
| Consistency | Guaranteed identical every launch | Depends on the script + any external calls it makes |
| Flexibility | None without rebuilding the AMI | High — change behavior per launch, no image rebuild |
| Iteration speed | Slow (rebuild + redeploy AMI each time) | Fast (edit User Data, relaunch) |
| Best for | Stuff that rarely changes (OS, app code) | Stuff that varies by launch (env config, secrets, registration) |

**Best practice (and shows up on the exam): use both together.** Bake the heavy, unchanging stuff into a "Golden AMI" (OS, app code, dependencies) for fast, consistent boots. Use a small User Data script only for the last-mile, launch-specific bits (pull latest config from Parameter Store, set an env variable, register with a load balancer).

**User Data vs Instance Metadata — don't confuse these:**

| | User Data | Instance Metadata |
|---|---|---|
| What it is | A script **you write** to configure the instance at launch | Info **AWS provides** about the running instance |
| Contains | Your bootstrap commands | Instance ID, IP, security groups, IAM role credentials, etc. |
| Retrieved via | `http://169.254.169.254/latest/user-data` | `http://169.254.169.254/latest/meta-data/` |

**Memory hook:** User Data = what the instance should **do** when it wakes up. Metadata = what AWS knows **about** the instance.

### Instance Metadata Service (IMDSv2) — high-yield security topic

Every EC2 instance can query `http://169.254.169.254/latest/meta-data/` from *inside* the instance to get info about itself (instance ID, IP, security groups, and — critically — **temporary IAM role credentials**).

**The security problem (IMDSv1):** if your app has a vulnerability (e.g. SSRF — Server-Side Request Forgery), an attacker could trick your app into making a request to that metadata URL and steal the instance's IAM credentials, without ever needing direct server access.

**The fix (IMDSv2):** requires a session token (via a `PUT` request) before you can query metadata — a simple GET request (what SSRF attacks typically use) no longer works alone. AWS now recommends and increasingly defaults to **IMDSv2** for this reason.

**Exam tip:** if a question mentions SSRF risk or credential theft via metadata, the answer is almost always "enforce IMDSv2."

### Stop vs Terminate vs Hibernate

| Action | What happens to the root EBS volume | What happens to RAM | Billing while stopped | Use case |
|--------|----------------------------------------|-------------------------|---------------------------|----------|
| **Stop** | Preserved (data intact) | Lost | Billed for EBS storage only, not compute | Pause an instance temporarily, resume later |
| **Terminate** | Deleted by default (unless "Delete on Termination" is unchecked) | Lost | No billing after termination | Permanently done with the instance |
| **Hibernate** | Preserved | **Saved to the root EBS volume** (like a laptop hibernating) | Billed for EBS storage only | Resume instantly with RAM state intact — no need to reboot the OS or reload applications |

**Memory hook:** Stop = pause (keep disk, lose RAM). Terminate = delete (lose everything, unless protected). Hibernate = suspend-to-disk (keep disk AND RAM state) — like closing a laptop lid vs. shutting it down vs. wiping it.

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

### Termination Policy — which instance gets removed when scaling IN?

When the ASG needs to remove an instance, the **default termination policy** decides which one, in this order:
1. Instance in the AZ with the **most** instances (keeps AZs balanced)
2. Within that AZ, the instance using the **oldest launch template/configuration** (removes outdated instances first)
3. If still tied, the instance **closest to its next billing hour** (minimizes wasted partial-hour cost)

You can customize this (e.g. `OldestInstance`, `NewestInstance`, `ClosestToNextInstanceHour`) if the default doesn't fit your use case.

### Instance Refresh

**The problem:** you update your Launch Template (e.g. new AMI with security patches) — but existing running instances **don't automatically get the update**. They keep running on the old template until they happen to be replaced.

**The fix:** Instance Refresh triggers a **rolling replacement** of all instances in the ASG with new ones based on the updated template — gradually, respecting a minimum healthy percentage so you don't cause an outage.

**Exam trigger phrase:** "roll out a new AMI/patch to all instances in an ASG" → Instance Refresh.

### Warm Pools

**The problem:** some applications take a long time to boot (heavy initialization, large software installs) — so when Auto Scaling needs to scale out fast, new instances aren't ready in time to handle the traffic spike.

**The fix:** Warm Pools keep a pool of **pre-initialized instances** in a stopped (or running) state, ready to be put into service almost instantly when scaling out — instead of booting from scratch each time.

### Mixed Instances Policy

Lets ONE Auto Scaling Group launch a **combination** of instance types and purchase options — e.g. a base of On-Demand instances for guaranteed capacity, plus Spot instances layered on top for cheap extra capacity, potentially across several different instance types for better Spot availability.

**Exam trigger phrase:** "cost-optimize an ASG while maintaining some guaranteed baseline capacity" → Mixed Instances Policy (On-Demand base + Spot for the rest).

### Scaling on Custom Metrics (e.g. SQS Queue Depth)

Target Tracking and Step Scaling aren't limited to CPU — they can scale off **any CloudWatch metric**, including custom ones you publish yourself.

**Classic exam scenario:** "Worker instances process jobs from an SQS queue — scale based on how backed up the queue is, not CPU." → Create a custom CloudWatch metric for queue depth (e.g. `ApproximateNumberOfMessagesVisible` divided by number of instances) and use it as the Target Tracking metric.

### Suspending Processes

You can temporarily **pause specific ASG processes** (e.g. `Launch`, `Terminate`, `HealthCheck`, `AlarmNotification`) without deleting the whole ASG — useful during maintenance windows or troubleshooting, so the ASG doesn't fight you by launching/terminating instances while you're working on them.

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

| Type | Layer | Plain English | Best For | Features |
|------|-------|----------------|----------|----------|
| ALB | 7 (HTTP/HTTPS) | Reads the actual request (URL, headers) and routes smartly | Microservices, containers | Path/host routing, Lambda targets |
| NLB | 4 (TCP/UDP) | Just checks IP+port, doesn't inspect content — very fast | Extreme performance | Static IP, millions req/sec |
| GWLB | 3 (Network) | Sends traffic through a security appliance first | Virtual appliances | Firewalls, IDS |
| CLB | 4 & 7 | The old load balancer, does both layers poorly | Legacy — avoid for new work | Not recommended |

**Quick pick:** ALB = smart routing by URL/content. NLB = raw speed, no content inspection. GWLB = routes through security appliances first. CLB = old, avoid for new builds.

### Features
- **Health checks** — pings each target; unhealthy ones stop receiving traffic
- **Sticky sessions** — same user always routed to the same backend instance
- **Cross-zone load balancing** — spreads traffic evenly across all AZs, not just one
- **SSL/TLS termination** — LB handles HTTPS decryption, backend only deals with plain HTTP

### Deregistration Delay (Connection Draining) — default 300s

**The problem:** you're removing an instance from the load balancer (scaling in, or deploying a new version) — but that instance might have requests **in-flight right now**. Yanking it out immediately would cut those requests off mid-response.

**The fix:** the LB stops sending *new* requests to that instance immediately, but waits up to the deregistration delay (default 300s, configurable) for **existing** in-flight requests to finish before fully removing it.

**Exam trigger phrase:** "zero-downtime deployment" or "don't drop in-flight requests during scale-in" → Deregistration delay / connection draining.

### Idle Timeout — default 60s

If a connection between client and LB sits idle (no data sent) longer than this, the LB closes it. Relevant for long-polling or WebSocket connections — you may need to increase this default if your app holds connections open intentionally.

### ALB Target Types

ALB can route traffic to three different kinds of targets:

| Target type | What it points to |
|-------------|------------------------|
| **Instance** | EC2 instances (by instance ID) |
| **IP** | Any IP address — including on-premises servers reachable via VPN/Direct Connect |
| **Lambda** | A Lambda function directly — lets you put a Lambda function behind a standard ALB |

### X-Forwarded-For Header

**The problem:** since the load balancer sits between the client and your app, your app sees every request as coming from the *load balancer's* IP, not the real client's IP.

**The fix (ALB only):** ALB adds an `X-Forwarded-For` header containing the original client IP — your app must read this header if it needs to know the real client IP (e.g. for logging, geo-blocking, rate limiting).

**Note:** NLB doesn't have this problem — it operates at Layer 4 and **preserves the original source IP natively**, so no header trick is needed there.

### SNI Support (ALB)

Lets ALB serve **multiple SSL/TLS certificates on the same HTTPS listener** — so one ALB can host multiple domains (each with its own certificate) without needing a separate load balancer per domain.

### Cross-Zone Load Balancing Defaults Differ by Type

| Type | Default | Cost when enabled |
|------|---------|------------------------|
| **ALB** | Always on, cannot be disabled | Free |
| **NLB** | **Off** by default | Enabling it can incur cross-AZ data transfer charges |

**Exam trap:** don't assume cross-zone load balancing is free/on for NLB just because it is for ALB — it's the opposite default.

---

## 4. AWS Lambda

Run your code without managing any server. Upload a function, AWS runs it only when triggered, and you pay only for the milliseconds it actually executes.

**Analogy:** EC2 is a restaurant — always open, staffed, costing money whether customers show up or not. Lambda is a vending machine — dormant and free when idle, only "does work" the instant someone presses the button.

### Limits

| Limit | Value | What it means in practice |
|-------|-------|-----------------------------|
| Max execution time | 15 minutes | Needs longer? Wrong tool — use EC2/ECS/Batch instead |
| Memory | 128 MB - 10 GB | You choose this; CPU scales automatically with memory |
| /tmp storage | 512 MB - 10 GB | Temporary scratch disk, wiped after the function ends |
| Concurrent executions | 1,000 (default, raisable) | Account-wide cap on simultaneous running copies, not per-function |

### Invocation Types

| Type | How it works | Examples | Analogy |
|------|----------------|----------|---------|
| **Synchronous** | Caller waits for the response | API Gateway, CLI | Asking a question, waiting for the answer |
| **Asynchronous** | Caller fires and moves on; Lambda processes in background with auto-retries | S3, SNS, EventBridge | Dropping a letter in a mailbox |
| **Event Source Mapping** | Lambda actively polls a stream/queue itself | SQS, Kinesis, DynamoDB Streams | Lambda checks the mailbox itself on a schedule |

### Pricing
- **$0.20 per 1M requests** — small flat fee per invocation
- **$0.00001667 per GB-second** — the real cost driver: (memory) × (duration). More memory or longer runtime = more cost.

**Rule of thumb:** cheapest for short, infrequent, event-driven tasks. Constant or long-running workloads usually become cheaper on EC2/Fargate.

### Lambda + VPC

**The problem:** by default, Lambda runs outside your VPC entirely, in an AWS-managed network. It can reach the public internet and public AWS services fine — but it CANNOT reach resources inside your private VPC (e.g. an RDS database in a private subnet), because it's treated like any other outsider.

**The fix:** attach the Lambda function to your VPC. AWS creates an **ENI** (Elastic Network Interface — a virtual network card) for the function inside subnets you choose, giving it a real private IP inside your VPC. Now it can reach RDS, ElastiCache, internal EC2, etc. following normal VPC rules (security groups, subnets).

**Analogy:** your VPC is an office building with badge access. By default, Lambda is a courier standing outside — can deliver to the public mailbox, can't enter private offices. Attaching it to the VPC is like giving that courier an employee badge to walk the internal hallways.

**The classic exam trap — the tradeoff:** once attached to your VPC, Lambda **loses automatic internet access**. Whether it can still reach the internet depends on subnet setup, exactly like EC2:

| Subnet setup | Can reach VPC resources (e.g. RDS)? | Can reach internet? |
|--------------|----------------------------------------|------------------------|
| Not attached to VPC (default) | No | Yes |
| Private subnet + NAT Gateway | Yes | Yes |
| Private subnet, no NAT Gateway | Yes | No |

**Exam scenario:** "Lambda needs to query a private RDS database AND call a third-party API." → Attach to VPC (private subnet) **and** ensure that subnet routes through a NAT Gateway.

**Historical note (still shows up in questions):** VPC-attached Lambdas used to have noticeably slower cold starts due to ENI creation time. AWS has since optimized this — but exam questions sometimes still test the older assumption.

### Concurrency: Reserved vs Provisioned

**The shared problem both solve:** all Lambda functions in an account share ONE pool of 1,000 concurrent executions by default. If Function A gets a traffic spike and eats all 1,000, Function B gets throttled too — even with completely normal traffic of its own. "Noisy neighbor" problem.

| Type | Problem it solves (plain English) | What it does | Cost | When to use |
|------|--------------------------------------|-----------------|------|----------------|
| **Reserved Concurrency** | Stops one noisy function from starving others — carves out a guaranteed, capped slice just for one function | Guarantees AND caps a fixed number of the 1,000 slots for one function — others can never steal them, but this function can never exceed them either | Free — just an allocation from the existing pool | Critical functions that must never be starved (e.g. payment processing). Or the reverse: cap a function so it can't overwhelm something downstream (e.g. limit DB connections) |
| **Provisioned Concurrency** | Eliminates cold starts — the delay when Lambda has to build a fresh execution environment from scratch | Pre-warms and keeps N execution environments running and ready in advance, so requests hit an already-initialized environment instantly | Costs extra continuously — you pay for the warm capacity even when idle | Latency-sensitive, user-facing workloads (e.g. an API behind a mobile app) that can't tolerate cold-start delay, especially before a known traffic spike |

**Analogies:** Reserved Concurrency = reserving 100 VIP parking spots in a shared lot — always available for the VIP, but the VIP also can never use more than 100. Provisioned Concurrency = a valet who already has your car running and pulled up before you arrive, instead of only starting the search for your keys once you show up (cold start).

**Important: Reserved Concurrency is per-function, not shared across functions.** Each function's reserved number is its own private slice — 56 functions with reserved concurrency do NOT share one combined pool. Example:
```
Account total: 1,000
Function A reserved = 200  → 200 exclusively for A
Function B reserved = 100  → 100 exclusively for B
Unreserved pool left = 1,000 - 200 - 100 = 700  → shared only by functions WITHOUT reserved concurrency set
```

**Also: the reservation is permanent, not usage-based.** Function A's 200 slots are removed from the shared pool the moment it's configured — even if A is completely idle and never invoked. It doesn't "give back" capacity when unused, because the whole point is a guarantee that's always ready. AWS always keeps at least 100 unreserved for the rest of the account.

### Dead Letter Queues (DLQ) & Destinations

Only relevant for **async** invocations (S3, SNS, EventBridge triggers). If the function fails after automatic retries, the failed event needs somewhere to go:

- **DLQ (legacy)** — routes failed events to an SQS queue or SNS topic
- **Destinations (newer, preferred)** — routes to SQS, SNS, Lambda, or EventBridge, and can also capture *successful* invocations, not just failures — more detail than a DLQ

### Retry Behavior by Invocation Type

| Type | Retry behavior |
|------|-------------------|
| Synchronous | No automatic retries — the caller must handle failure itself |
| Asynchronous | Automatic retries (default 2) with backoff, then routes to DLQ/Destination |
| Event Source Mapping | Keeps retrying until success or the data expires (e.g. Kinesis/DynamoDB retention window, SQS visibility timeout) |

### Versions & Aliases

- **Version** — an immutable snapshot of your function's code + config at a point in time
- **Alias** — a named pointer to a version (e.g. `prod` → v3), used for traffic shifting — e.g. send 90% of traffic to v1 and 10% to v2 for canary testing. Common in blue/green deployments, often paired with CodeDeploy.

### Deployment Package Limits
- **50 MB** zipped (direct console/CLI upload)
- **250 MB** unzipped
- **10 GB** via container image (for larger dependencies)

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

