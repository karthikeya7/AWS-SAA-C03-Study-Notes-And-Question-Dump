# 🎯 Practice Test 4 - Incorrect Areas Study Guide

**Test Performance**: 49/65 (75.38%) - FAIL  
**Date**: March 1, 2026  
**Passing Score**: 72% (≥720/1000)

---

## 📊 Domain Performance Analysis

| Domain | Total | Correct | Incorrect | % Correct |
|--------|-------|---------|-----------|-----------|
| Design Resilient Architectures | 19 | 14 | **5** | 73.7% |
| Design High-Performing Architectures | 22 | 17 | **5** | 77.3% |
| Design Cost-Optimized Architectures | 10 | 6 | **4** | 60.0% ❌ |
| Design Secure Architectures | 13 | 11 | **2** | 84.6% ✅ |
| Design Secure Applications | 1 | 1 | 0 | 100% ✅ |

**Total Incorrect: 16 questions**

---

## 🚨 Critical Weakness Areas

### 1. **Design Cost-Optimized Architectures** (60% - NEEDS FOCUS)
- EC2 Auto Scaling memory metrics
- Redshift snapshot management
- S3 Glacier retrieval tiers  
- Savings Plans strategies
- Mixed instance policies

### 2. **Design Resilient Architectures** (73.7% - NEEDS IMPROVEMENT)
- VPC security group references
- EBS snapshot cross-region copy
- S3 versioning and delete markers
- Auto Scaling lifecycle hooks
- Multi-AZ NAT Gateway architecture

### 3. **Design High-Performing Architectures** (77.3% - REVIEW)
- CloudFormation drift detection
- ALB access logging
- VPC NACLs with Transit Gateway
- Global Accelerator for failover
- Pre-signed URLs for large uploads

---

## 📚 Flashcards for Incorrect Questions

### Question 7: EC2 Auto Scaling Predefined Metrics

**❓ QUESTION:**  
Which metric is NOT available as a predefined option for EC2 Auto Scaling target tracking?

**❌ YOUR ANSWER:** Network Out  
**✅ CORRECT ANSWER:** Memory Utilization

**📖 EXPLANATION:**
- EC2 does not publish memory usage as a native CloudWatch metric
- **Predefined metrics include:**
  - ASGAverageCPUUtilization
  - ASGAverageNetworkIn
  - ASGAverageNetworkOut
  - ALBRequestCountPerTarget

**💡 TO SCALE ON MEMORY:**
1. Install CloudWatch agent on instances
2. Push custom memory metric
3. Create target tracking policy on custom metric

**🔗 Reference:** [Target Tracking Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html)

---

### Question 13: Redshift Snapshot Cost Optimization

**❓ QUESTION:**  
How to reduce rising snapshot-related costs for Redshift without affecting availability?

**❌ YOUR ANSWER:** Increase automated snapshot retention to 35 days  
**✅ CORRECT ANSWER:** Delete unneeded manual snapshots

**📖 EXPLANATION:**
- **Manual snapshots** persist indefinitely until deleted (incur ongoing charges)
- **Automated snapshots** expire per retention setting
- First action: Audit and delete old manual snapshots

**⚠️ PITFALL:** Increasing retention INCREASES cost, doesn't reduce it

**💡 BEST PRACTICE:**
```
1. List all manual snapshots
2. Identify snapshots older than retention policy
3. Delete unnecessary snapshots
4. Set up lifecycle policies for future automation
```

**🔗 Reference:** [Redshift Snapshots](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-snapshots.html)

---

### Question 17: S3 Glacier Flexible Retrieval

**❓ QUESTION:**  
Management requires archived data retrievable within 3-5 hours. Which retrieval tier?

**❌ YOUR ANSWER:** Expedited  
**✅ CORRECT ANSWER:** Standard

**📖 EXPLANATION:**

| Tier | Retrieval Time | Use Case | Cost |
|------|---------------|----------|------|
| **Expedited** | 1-5 minutes | Urgent access | $$$ |
| **Standard** | 3-5 hours | Planned retrieval | $$ |
| **Bulk** | 5-12 hours | Large datasets | $ |

**⚠️ COST TRAP:** Expedited meets requirement but unnecessarily expensive

**💡 DECISION MATRIX:**
- Requirement: 3-5 hours → Standard (perfect match)
- Requirement: < 1 hour → Expedited
- Requirement: 12+ hours okay → Bulk

**🔗 Reference:** [S3 Glacier Retrieval](https://docs.aws.amazon.com/AmazonS3/latest/userguide/restoring-objects-retrieval-options.html)

---

### Question 18: VPC Security Group References

**❓ QUESTION:**  
Backend instances must only accept traffic from frontend on custom port. How configure backend SG?

**❌ YOUR ANSWER:** Use the launch configuration as the source  
**✅ CORRECT ANSWER:** Use the frontend security group ID as source

**📖 EXPLANATION:**
- SG rules can reference other SGs by ID
- Automatically handles instance scale and IP changes
- Tightly couples access to SG membership

**✅ CORRECT CONFIGURATION:**
```
Backend Security Group Inbound Rule:
- Type: Custom TCP
- Port: 8080 (example)
- Source: sg-frontend123 (frontend SG ID)
```

**❌ WRONG APPROACHES:**
- Public subnet CIDR → Allows ANY instance in subnet
- ASG ARN → Not supported as source
- Launch config → Cannot be used in SG rules

**💡 KEY CONCEPT:** SG-to-SG references = dynamic, auto-scaling access control

**🔗 Reference:** [Security Group Rules](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)

---

### Question 19: Cost Optimization - Savings Plans Mix

**❓ QUESTION:**  
Best savings mix for: diverse web tier, stable app tier, RDS databases?

**❌ YOUR ANSWER:** EC2 Instance SP (web) + Compute SP (app and DB)  
**✅ CORRECT ANSWER:** Compute SP (web) + EC2 Instance SP (app) + RDS RI (DB)

**📖 EXPLANATION:**

| Tier | Characteristics | Best Option | Why |
|------|----------------|-------------|-----|
| **Web** | Diverse instance families | Compute Savings Plans | Flexibility across families |
| **App** | Stable instance family | EC2 Instance Savings Plans | Highest discount for commitment |
| **DB** | Managed RDS | RDS Reserved Instances | Separate from EC2 plans |

**⚠️ CRITICAL:** RDS is NOT covered by EC2 Savings Plans - needs RDS RI

**💡 SAVINGS HIERARCHY:**
1. Most flexible: Compute Savings Plans
2. Most savings: EC2 Instance Savings Plans (same family)
3. RDS specific: RDS Reserved Instances

**🔗 Reference:** [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/sp-ris.html)

---

### Question 34: Mixed Instance Policy for Auto Scaling

**❓ QUESTION:**  
How to reduce cost for ASG (min 2, max 12) without risking performance?

**❌ YOUR ANSWER:** Use all t2.micro On-Demand instances  
**✅ CORRECT ANSWER:** On-Demand base=2; above that 20% On-Demand / 80% Spot

**📖 EXPLANATION:**
- **Baseline (2)**: On-Demand for reliability
- **Burst (3-12)**: Mix of 20% On-Demand + 80% Spot for cost savings

**💰 COST COMPARISON:**
```
All On-Demand @ 12 instances = 12x cost
All Spot @ 12 instances = ~1.2x cost (but risky)
Mixed (2 OD + 8 Spot @ 80%) = ~4x cost (balanced)
```

**⚠️ AVOID:**
- All Spot: Interruptions hurt availability
- All On-Demand: No cost savings
- Too small instances: Performance degradation

**💡 BEST PRACTICE:** Protect baseline with On-Demand, optimize burst with Spot

**🔗 Reference:** [Mixed Instance Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-purchase-options.html)

---

### Question 25: CloudTrail Log File Integrity

**❓ QUESTION:**  
How to cryptographically verify CloudTrail logs haven't been tampered with?

**❌ YOUR ANSWER:** Use SSE-KMS encryption for the CloudTrail logs  
**✅ CORRECT ANSWER:** Enable CloudTrail log file integrity validation

**📖 EXPLANATION:**

| Feature | Purpose | Provides Integrity? |
|---------|---------|-------------------|
| **SSE-S3** | Encryption at rest | ❌ No |
| **SSE-KMS** | Key management + encryption | ❌ No |
| **Bucket Policy** | Access control | ❌ No |
| **Log File Integrity** | Tamper detection | ✅ YES |

**✅ HOW IT WORKS:**
1. Generates digest files with SHA-256 hashes
2. Signed with RSA private key
3. Auditors verify with public key
4. Detects modifications, deletions, or tampering

**⚠️ COMMON MISTAKE:** Encryption ≠ Integrity validation

**💡 COMPLIANCE USE:** Required for PCI, HIPAA, SOC 2 audits

**🔗 Reference:** [Log File Integrity](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)

---

### Question 28: Auto Scaling Lifecycle Hooks

**❓ QUESTION:**  
How to run cleanup scripts before instances terminate during scale-in?

**❌ YOUR ANSWER:** Customize the termination policy to copy data before termination  
**✅ CORRECT ANSWER:** Add lifecycle hooks to the Auto Scaling group

**📖 EXPLANATION:**

**Lifecycle Hook Flow:**
```
Scale-In Triggered
    ↓
Instance enters "Terminating:Wait" state
    ↓
Run cleanup scripts (via Lambda/SSM/SNS)
    ↓
Complete lifecycle action
    ↓
Instance terminates
```

**⚠️ WRONG APPROACHES:**
- **Cooldown period**: Controls time between scale actions (not pre-termination)
- **Termination policy**: Chooses WHICH instance (not HOW to terminate)
- **Suspend Terminate**: Blocks ALL scale-in (breaks elasticity)

**💡 USE CASES:**
- Copy logs to S3
- Drain connections
- Snapshot volumes
- Deregister from external services

**🔗 Reference:** [Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks-overview.html)

---

### Question 36: CloudFormation Drift Detection

**❓ QUESTION:**  
Why aren't some property changes detected as drift?

**❌ YOUR ANSWER:** Grant additional Read permissions to drift detection  
**✅ CORRECT ANSWER:** Explicitly set property values (even if defaults) in template

**📖 EXPLANATION:**
- Drift detection compares **declared** properties vs **actual** resources
- **Implicit defaults** are not tracked
- Must explicitly define values to monitor them

**❌ BEFORE (Won't detect drift):**
```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-12345
    InstanceType: t2.micro
    # Monitoring: defaults to false (not tracked)
```

**✅ AFTER (Will detect drift):**
```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-12345
    InstanceType: t2.micro
    Monitoring: false  # Now explicitly tracked
```

**💡 BEST PRACTICE:** Explicitly declare ALL properties you want monitored

**🔗 Reference:** [Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)

---

### Question 48: ALB Access Logs for Client IPs

**❓ QUESTION:**  
How to identify actual client IPs and request details for forensics?

**❌ YOUR ANSWER:** AWS CloudTrail data events for the ALB  
**✅ CORRECT ANSWER:** ALB access logs to Amazon S3

**📖 EXPLANATION:**

| Log Type | Captures | Client IP? | URL Path? | User Agent? |
|----------|---------|------------|-----------|-------------|
| **ALB Access Logs** | HTTP requests | ✅ Yes | ✅ Yes | ✅ Yes |
| **VPC Flow Logs** | Network flows | ✅ Yes | ❌ No | ❌ No |
| **CloudWatch Metrics** | Aggregated stats | ❌ No | ❌ No | ❌ No |
| **CloudTrail** | API calls | ❌ No | ❌ No | ❌ No |

**✅ ALB ACCESS LOG FIELDS:**
- Client IP (x-forwarded-for)
- Request URL and method
- User agent
- Response status codes
- Processing times
- Backend target

**💡 ANALYSIS:** Query logs with Amazon Athena for forensics

**🔗 Reference:** [ALB Access Logs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html)

---

### Question 56: EBS Snapshot Cross-Region Copy

**❓ QUESTION:**  
How to maintain snapshot copy in another region for compliance?

**❌ YOUR ANSWER:** Enable S3 Cross-Region Replication on the snapshot storage bucket  
**✅ CORRECT ANSWER:** Copy the EBS snapshot from us-east-1 to ap-south-1

**📖 EXPLANATION:**
- EBS snapshots are **Regional resources**
- Use native **snapshot copy** feature
- Can re-encrypt during copy (with different CMK)

**✅ CORRECT PROCESS:**
```
1. Select snapshot in source region
2. Actions → Copy
3. Choose destination region
4. Optional: Change encryption key
5. Copy completes → Independent snapshot in new region
```

**❌ WRONG APPROACHES:**
- S3 CRR: Snapshot storage is AWS-managed (can't configure)
- Direct creation: Can't snapshot into different region
- Launch & re-snapshot: Costly and unnecessary

**💡 AUTOMATION:** Use AWS Backup or Lambda + EventBridge

**🔗 Reference:** [Copy Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-copy-snapshot.html)

---

### Question 58: S3 Versioning Delete Marker Recovery

**❓ QUESTION:**  
How to recover accidentally deleted object in versioned bucket?

**❌ YOUR ANSWER:** In a version-enabled bucket, a Delete request only hides the latest version  
**✅ CORRECT ANSWER:** Delete the delete marker that was applied to the object

**📖 EXPLANATION:**

**What Happens on Delete:**
```
Object.pdf (version ABC123)
    ↓ DELETE operation
Object.pdf (delete marker) ← Makes object appear gone
    └─ version ABC123 (still exists!)
```

**Recovery Process:**
```
1. Enable "Show versions" in S3 console
2. Find the delete marker
3. Delete the delete marker
4. Object becomes visible again
```

**⚠️ CONFUSION:** "Restore" option is for Glacier, not versioning

**💡 PREVENTION:** Enable MFA Delete for protection

**🔗 Reference:** [Versioning Delete](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectVersions.html)

---

### Question 38: VPC NACLs with Transit Gateway

**❓ QUESTION:**  
How are NACL rules evaluated for traffic between instances and TGW in different subnets?

**❌ YOUR ANSWER:** When instances and TGW are in different subnets, only inbound rules are evaluated  
**✅ CORRECT ANSWER:** Outbound rules use destination IP; inbound rules use source IP

**📖 EXPLANATION:**

**NACLs are STATELESS** - Must check both directions:

**Instance → TGW:**
- **Outbound NACL** of instance subnet: Checks destination (TGW subnet)
- **Inbound NACL** of TGW subnet: Checks source (instance subnet)

**TGW → Instance:**
- **Outbound NACL** of TGW subnet: Checks destination (instance subnet)
- **Inbound NACL** of instance subnet: Checks source (TGW subnet)

**✅ REQUIRED RULES:**
```
Instance Subnet NACL:
- Outbound: Allow to TGW subnet
- Inbound: Allow from TGW subnet

TGW Subnet NACL:
- Inbound: Allow from instance subnet
- Outbound: Allow to instance subnet
```

**💡 KEY CONCEPT:** Stateless = explicit allow in BOTH directions

**🔗 Reference:** [TGW NACLs](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-nacls.html)

---

### Question 45: Session Storage Services

**❓ QUESTION:**  
Which services are most appropriate for scalable session storage?

**❌ YOUR ANSWER:** Elastic Load Balancing + Amazon ElastiCache  
**✅ CORRECT ANSWER:** Amazon DynamoDB + Amazon ElastiCache

**📖 EXPLANATION:**

| Service | Purpose | Session Storage? |
|---------|---------|-----------------|
| **DynamoDB** | Persistent NoSQL | ✅ Excellent (TTL, auto-scale) |
| **ElastiCache** | In-memory cache | ✅ Excellent (microsecond latency) |
| **ELB** | Load balancing | ❌ No (routing only) |
| **CloudWatch** | Monitoring | ❌ No |
| **Storage Gateway** | Hybrid storage | ❌ No |

**💡 SESSION PATTERN:**
```
User Request
    ↓
ElastiCache (check session) → Miss
    ↓
DynamoDB (fetch session) → Store in cache
    ↓
Return session data
```

**BENEFITS:**
- **ElastiCache**: Ultra-fast reads (microseconds)
- **DynamoDB**: Durable storage + TTL for expiration
- **Together**: Speed + reliability

**🔗 Reference:** [Session Management](https://aws.amazon.com/caching/session-management/)

---

## 📝 Practice Questions - Incorrect Areas Only

### Question 1: EC2 Memory-Based Auto Scaling

A web application experiences memory pressure during peak times. The solutions architect wants to configure Auto Scaling to add instances when average memory utilization exceeds 75%. What is required?

**A.** Configure a target tracking policy on the ASGAverageMemoryUtilization predefined metric  
**B.** Install CloudWatch agent, publish custom memory metric, create target tracking policy on custom metric ✅  
**C.** Use step scaling with CPU utilization as a proxy for memory usage  
**D.** Enable detailed monitoring on EC2 instances to expose memory metrics

> [!success]- 📖 Explanation
> **Correct Answer: B**
>
> **Why:**
> - EC2 does not publish memory metrics by default
> - CloudWatch agent must be installed to collect and publish memory metrics
> - Create target tracking policy on the custom metric namespace
>
> **Setup Process:**
> 1. Install CloudWatch agent on instances (or AMI)
> 2. Configure agent to publish mem_used_percent
> 3. Create target tracking policy: Metric = Custom Memory, Target = 75%
>
> **Common Mistakes:**
> - Assuming memory is a predefined metric
> - Using CPU as proxy (different resource constraints)
> - Detailed monitoring only adds more frequent CPU/network metrics
>

---

### Question 2: Redshift Snapshot Lifecycle Management

A company's Redshift cluster has automated snapshots with 7-day retention and several manual snapshots from past migrations. Snapshot storage costs have increased significantly. What is the FIRST action to reduce costs?

**A.** Reduce automated snapshot retention to 1 day  
**B.** Delete unnecessary manual snapshots ✅  
**C.** Enable snapshot compression  
**D.** Move snapshots to S3 Glacier

> [!success]- 📖 Explanation
> **Correct Answer: B**
>
> **Why:**
> - Manual snapshots persist indefinitely (ongoing cost)
> - Automated snapshots already expire per retention
> - First action: Audit and delete old manual snapshots
>
> **Cost Analysis:**
> ```
> Automated snapshots (7 days): $X/day × 7 = $7X
> Manual snapshots (50 old): $X/snapshot × 50 = $50X
> ```
>
> **Action Plan:**
> 1. List all manual snapshots
> 2. Identify snapshots > 90 days old (or per policy)
> 3. Verify no longer needed (check with stakeholders)
> 4. Delete in batches
> 5. Set up lifecycle automation
>
> **Why Not Others:**
> - A: Reduces backup window (availability risk)
> - C: No such feature (snapshots are already compressed)
> - D: Snapshots already in S3 (no Glacier option)
>

---

### Question 3: S3 Glacier Retrieval Cost Optimization

An application archives logs to S3 Glacier Flexible Retrieval. For compliance, logs must be retrievable within 4 hours. The team currently uses Expedited retrieval. How can they reduce costs?

**A.** Switch to Standard retrieval ✅  
**B.** Use Bulk retrieval with parallel requests  
**C.** Keep Expedited but reduce request frequency  
**D.** Move to S3 Glacier Instant Retrieval

> [!success]- 📖 Explanation
> **Correct Answer: A**
>
> **Retrieval Tier Comparison:**
>
> | Tier | Time | Cost | Use Case |
> |------|------|------|----------|
> | Expedited | 1-5 min | $$$ | Urgent (< 1 hr) |
> | Standard | 3-5 hrs | $$ | Planned (3-5 hrs) |
> | Bulk | 5-12 hrs | $ | Flexible (12+ hrs) |
>
> **Why Standard Fits:**
> - Requirement: 4 hours → Standard delivers in 3-5 hours
> - Expedited is 5-10x more expensive
> - Still meets compliance requirement
>
> **Cost Savings:**
> ```
> Expedited: $0.03/GB + $0.01/1000 requests
> Standard: $0.01/GB + $0.05/1000 requests
> Savings: ~66% on retrieval costs
> ```
>
> **Why Not Others:**
> - B: Bulk may exceed 4-hour window
> - C: Reduces frequency, doesn't solve overpriced tier
> - D: Instant Retrieval for millisecond access (overkill)
>

---

### Question 4: Security Group Cross-Tier Access

A three-tier application has web servers in public subnets and app servers in private subnets. App servers must only accept traffic from web servers on port 8080. Both tiers scale with Auto Scaling. What is the MOST maintainable solution?

**A.** Configure app server SG to allow port 8080 from the public subnet CIDR  
**B.** Configure app server SG to allow port 8080 from web server SG ID ✅  
**C.** Use Network ACLs to restrict port 8080 access  
**D.** Assign Elastic IPs to web servers and whitelist them

> [!success]- 📖 Explanation
> **Correct Answer: B**
>
> **Security Group Rule:**
> ```
> App Server Security Group (sg-app):
> Inbound Rule:
> - Type: Custom TCP
> - Port: 8080
> - Source: sg-web (Web Server SG ID)
> ```
>
> **Why SG-to-SG References:**
> - ✅ Automatically adjusts as instances scale
> - ✅ Works regardless of IP addresses
> - ✅ No need to update rules when instances change
> - ✅ Tightly couples access to SG membership
>
> **Why Not Others:**
> - **A:** Allows ANY instance in public subnet (not just web servers)
> - **C:** NACLs are subnet-level (not instance-level), stateless (complex rules)
> - **D:** Elastic IPs don't scale with Auto Scaling, manual maintenance
>
> **Key Concept:** SG references create dynamic, auto-scaling access control
>

---

### Question 5: Savings Plans Strategy

A company runs workloads on AWS:
- Web tier: Mixed instance families (t3, m5, c5) across regions
- App tier: Stable c5 family in single region
- Database: RDS MySQL Multi-AZ

What is the MOST cost-effective commitment strategy?

**A.** Compute Savings Plans for all workloads  
**B.** EC2 Instance Savings Plans for all workloads  
**C.** Compute SP (web) + EC2 Instance SP (app) + RDS RI (DB) ✅  
**D.** EC2 Instance SP (all EC2) + Compute SP (RDS)

> [!success]- 📖 Explanation
> **Correct Answer: C**
>
> **Savings Strategy Matrix:**
>
> | Workload | Characteristics | Best Option | Discount |
> |----------|----------------|-------------|----------|
> | Web tier | Diverse families/regions | Compute SP | 66% |
> | App tier | Stable c5 family | EC2 Instance SP | 72% |
> | RDS Database | Managed service | RDS Reserved Instance | 69% |
>
> **Why This Mix:**
>
> **Compute Savings Plans (Web):**
> - Flexible across instance families (t3, m5, c5)
> - Portable across regions
> - Good for dynamic/evolving workloads
>
> **EC2 Instance Savings Plans (App):**
> - Highest discount (72%)
> - Locked to c5 family (acceptable for stable tier)
> - Regional commitment
>
> **RDS Reserved Instances (DB):**
> - Separate from EC2 plans
> - Database-specific pricing model
> - Required for managed service discounts
>
> **Critical Mistake:** RDS is NOT covered by EC2/Compute Savings Plans
>
> **Cost Example:**
> ```
> On-Demand Total: $10,000/month
> With Mixed Strategy: $3,800/month (62% savings)
> With Wrong Strategy: $5,200/month (48% savings)
> ```
>

---

### Question 6: Auto Scaling Mixed Instance Policy

An e-commerce application uses Auto Scaling (min=2, max=15). The architect wants to minimize cost while ensuring baseline reliability. Traffic is variable and can spike unpredictably. What is the BEST mixed instance configuration?

**A.** 100% Spot instances (lowest cost)  
**B.** 100% On-Demand instances (highest reliability)  
**C.** On-Demand base=2, above that 80% Spot / 20% On-Demand ✅  
**D.** On-Demand base=0, 50% Spot / 50% On-Demand

> [!success]- 📖 Explanation
> **Correct Answer: C**
>
> **Mixed Instance Strategy:**
> ```
> Capacity Plan:
> - Base (2): On-Demand (always available)
> - Scale 3-15: 80% Spot (cost savings) + 20% On-Demand (reliability buffer)
>
> Example at 10 instances:
> - 2 On-Demand (base)
> - 6 Spot (80% of 8)
> - 2 On-Demand (20% of 8)
> Total: 4 OD + 6 Spot
> ```
>
> **Cost Analysis:**
> ```
> All On-Demand (10): $1000/month
> All Spot (10): $300/month (but risky)
> Mixed (4 OD + 6 Spot): $400 OD + $180 Spot = $580 (42% savings)
> ```
>
> **Why This Works:**
> - ✅ Guarantees 2 instances always available (On-Demand base)
> - ✅ Achieves significant cost savings with Spot
> - ✅ Maintains reliability buffer with 20% On-Demand in burst
> - ✅ Can handle Spot interruptions gracefully
>
> **Why Not Others:**
> - **A:** Spot interruptions can affect entire fleet (no baseline protection)
> - **B:** No cost optimization (pays full price)
> - **D:** No guaranteed baseline (base=0 means all Spot initially)
>
> **Interruption Handling:**
> - Spot interrupts instance → Auto Scaling launches replacement
> - Always maintains desired capacity
> - On-Demand instances absorb load during Spot interruption
>

---

## 🎯 Study Action Plan

### Immediate Focus (Next 3 Days)

**Day 1: Cost Optimization Deep Dive**
- [ ] Review all Savings Plans types and use cases
- [ ] Practice Redshift snapshot management scenarios
- [ ] Master S3 Glacier retrieval tier selection
- [ ] Understand Auto Scaling cost optimization patterns

**Day 2: Resilient Architecture Patterns**
- [ ] Study VPC security group reference patterns
- [ ] Practice EBS snapshot cross-region scenarios
- [ ] Master S3 versioning and delete marker recovery
- [ ] Review Auto Scaling lifecycle hooks

**Day 3: High-Performance Architecture**
- [ ] Understand CloudFormation drift detection
- [ ] Master ALB logging vs Flow Logs vs CloudTrail
- [ ] Study NACL evaluation with Transit Gateway
- [ ] Review Global Accelerator failover mechanisms

### Week 1 Goals
- Take section quizzes on weak domains
- Complete 50+ flashcard reviews daily
- Hands-on labs for lifecycle hooks and mixed instances
- Score 85%+ on Cost Optimization practice questions

### Week 2 Goals
- Retake Practice Test 4
- Target score: 85%+ (55+/65)
- Focus on timing (< 90 minutes)
- Review all flagged questions

---

## 📚 Additional Resources

### AWS Documentation Links
1. [EC2 Auto Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html)
2. [Savings Plans User Guide](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)
3. [S3 Glacier Retrieval Options](https://docs.aws.amazon.com/AmazonS3/latest/userguide/restoring-objects-retrieval-options.html)
4. [VPC Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)
5. [CloudTrail Log Integrity](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)

### Hands-On Labs
1. **Lab 1:** Configure CloudWatch agent for memory-based Auto Scaling
2. **Lab 2:** Implement lifecycle hooks with Lambda cleanup
3. **Lab 3:** Create mixed instance policy with Spot + On-Demand
4. **Lab 4:** Test S3 versioning delete marker recovery
5. **Lab 5:** Set up ALB access logs with Athena queries

---

**Next Review Date**: March 8, 2026  
**Goal**: Achieve 85%+ on retake

