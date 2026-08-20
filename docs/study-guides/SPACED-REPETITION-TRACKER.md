# 🔁 Spaced Repetition Tracker

**Purpose:** Track concepts you found confusing or got wrong, and review them on a schedule — so weak spots actually stick before exam day, instead of fading a week after you first understood them.

**Why spaced repetition, and why these intervals:** memory decays fastest right after learning something, then decay slows each time you successfully recall it. Reviewing at increasing gaps (1 day → 2 days → 4 days → 7 days) hits each concept right before you'd naturally forget it, which is far more efficient than re-reading everything from scratch. Given the exam is close, intervals here are compressed into a single exam-prep month rather than the traditional multi-month spacing.

## How to use this file

1. When you deep-dive a confusing concept (with Claude, in these notes, or on a practice test), add a row below with today's date as **First Learned**.
2. Check off each review box **only if you could recall/explain it correctly without looking it up.** If you couldn't, don't check it — instead re-read the concept, then restart its interval from Day 1.
3. Review intervals: **Day 1 → Day 2 → Day 4 → Day 7 → Final Review** (the week before your exam).
4. Do a quick pass of this whole table before bed or first thing in the morning — it should take under 10 minutes once entries build up.

---

## Active Review Queue

| Concept | Module | First Learned | Day 1 | Day 2 | Day 4 | Day 7 | Final Review | Notes (what tripped you up) |
|---------|--------|----------------|:-----:|:-----:|:-----:|:-----:|:-------------:|-------------------------------|
| Implicit deny — why "Allow only" beats "Allow + explicit Deny" wording | 02-IAM | | ☐ | ☐ | ☐ | ☐ | ☐ | Q3: option C's wording was misleading — implicit deny isn't something you "use," it's default behavior |
| MFA enforcement distractors — logging (CloudTrail) ≠ enforcement | 02-IAM | | ☐ | ☐ | ☐ | ☐ | ☐ | Q4: bucket policy MFA condition is equally valid to IAM policy condition — same underlying mechanism, different attachment point |
| EC2 instance family letters (C-R-M-T-I-D-H) | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Mnemonic: "Crazy Rabbits Munch Tiny Insects During Hunts" |
| EC2 pricing models — discount ranking | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Spot(90%) > Reserved(72%) > Savings Plans(66%) > On-Demand/Dedicated(0%) |
| Placement Groups — tradeoffs, not just definitions | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Cluster=fast but risky, Spread=safe but limited(7/AZ), Partition=safe+scalable |
| Target Tracking vs Step Scaling — who creates the alarm | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Both use CloudWatch; TT = AWS auto-creates alarm + calculates amount, Step = you do both manually |
| Simple Scaling vs the other two — the missing policy | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Only Simple Scaling uses classic Cooldown; Step/Target Tracking use Instance Warmup instead |
| Auto Scaling Group vs ECS/EKS/Lambda scaling | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | EC2=ASG only. ECS/EKS=two layers (app+servers). Lambda=fully automatic, no ASG concept |
| Lambda + VPC — the internet access tradeoff | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Not attached=free internet, no private VPC access. Attached=private VPC access, but needs NAT Gateway for internet back |
| Reserved vs Provisioned Concurrency | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Reserved=per-function guarantee+cap, free, permanent even if idle. Provisioned=pre-warmed, costs extra, solves cold starts |
| DLQ/Destinations — only for async invocations | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Sync=caller sees error immediately. Async=fails silently without a DLQ/Destination to catch it |
| ELB deregistration delay vs idle timeout | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | Dereg delay(300s default)=finish in-flight requests before removal. Idle timeout(60s default)=close inactive connections |
| Cross-zone load balancing default: ALB vs NLB | 03-Compute | | ☐ | ☐ | ☐ | ☐ | ☐ | ALB=always on, free. NLB=off by default, can cost extra when enabled |
| | | | ☐ | ☐ | ☐ | ☐ | ☐ | |
| | | | ☐ | ☐ | ☐ | ☐ | ☐ | |

*(Add new rows as you work through remaining modules — Storage, Database, Networking, Security, etc.)*

---

## Weak Modules Summary

*(Update as you finish each module — a fast way to see where to focus remaining study time.)*

| Module | Status | Confidence (1-5) | Needs another pass? |
|--------|--------|--------------------|-----------------------|
| 01 - AWS Fundamentals | | | |
| 02 - IAM | In progress | | |
| 03 - Compute | In progress (deep) | | |
| 04 - Storage | Not started | | |
| 05 - Database | Not started | | |
| 06 - Networking | Not started | | |
| 07 - Security | Not started | | |
| 08 - Application Integration | Not started | | |
| 09 - Monitoring | Not started | | |
| 10 - Migration | Not started | | |
| 11 - Analytics | Not started | | |
| 12 - Architecture Patterns | Not started | | |
| 13 - Cost Optimization | Not started | | |

---

## Final Week Protocol

In the last 7 days before your exam:
1. Go through every row in the Active Review Queue that's missing a "Final Review" check.
2. For anything still shaky, re-read that module's `FAST-LEARN.md` (not the full `README.md` — you don't have time for deep re-reads now).
3. Do a full pass of `exam-reviews/quick-reference/ULTRA-QUICK-REFERENCE-CARD.md` the night before.
4. Don't learn brand-new topics in the final 48 hours — only reinforce what's already in this tracker.
