# Reliability, Observability & Operations

[⬅ Back to index](../README.md)

Covers: designing for failure, redundancy across AZs and regions, disaster recovery (RPO/RTO), multi-region strategies, cascading failures, chaos engineering, logs/metrics/traces, golden signals, alerting, deployments (blue-green, canary, feature flags), incidents and postmortems, capacity planning.

---

### 1. What does "design for failure" mean?

<details>
<summary>Answer</summary>

Assume every component **will** fail – disks, machines, networks, dependencies, whole zones or regions – and design so the system keeps working (or degrades gracefully):
- **Redundancy** everywhere (no single points of failure).
- **Timeouts, retries with backoff, circuit breakers, bulkheads.**
- **Idempotent** operations so retries are safe.
- **Health checks and automatic failover.**
- **Graceful degradation** of non-critical features.
- **Limit blast radius** – cells, shuffle sharding, staged rollouts.
- Practise failure (game days, chaos experiments).

</details>

### 2. How do availability zones and regions affect design?

<details>
<summary>Answer</summary>

- An **availability zone (AZ)** is one or more isolated data centres within a region, with low-latency links (~1–2 ms) to other AZs.
- A **region** is a geographic area with several AZs; regions are far apart (tens to hundreds of ms).

Baseline for production: run across **at least 3 AZs** (stateless services spread out, databases with synchronous or semi-sync replicas in other AZs). Multi-**region** adds protection from regional outages and lower latency for global users, but brings cross-region data replication, consistency and cost challenges.

</details>

### 3. What are RPO and RTO?

<details>
<summary>Answer</summary>

- **RPO (Recovery Point Objective)** – the maximum acceptable **data loss**, measured in time ("we can lose at most 5 minutes of data").
- **RTO (Recovery Time Objective)** – the maximum acceptable **downtime** ("we must be back within 1 hour").

They drive the disaster-recovery strategy and its cost. RPO ≈ 0 needs synchronous replication; RTO of seconds needs hot standby or active-active.

</details>

### 4. What disaster recovery strategies exist?

<details>
<summary>Answer</summary>

From cheapest/slowest to most expensive/fastest:
1. **Backup and restore** – back up to another region; restore on disaster. RPO hours, RTO hours–days.
2. **Pilot light** – core data replicated, minimal infrastructure running in the DR region; scale up on disaster. RTO tens of minutes.
3. **Warm standby** – a scaled-down but fully working copy runs in the DR region. RTO minutes.
4. **Multi-site active-active** – full capacity in multiple regions serving traffic. RTO near zero, RPO near zero (with appropriate replication), highest cost and complexity.

Whatever you pick, **test failover regularly**.

</details>

### 5. How do you design an active-active multi-region system?

<details>
<summary>Answer</summary>

- **Route users** to the nearest healthy region (GeoDNS/anycast) and keep them sticky to their "home" region.
- **Data**: either partition by home region (each user's data has one writing region; replicate asynchronously for reads/DR), or use a globally distributed database (Spanner, CockroachDB, DynamoDB Global Tables with last-write-wins, Cosmos DB).
- Handle **conflicts** for data written in multiple regions (CRDTs, LWW, or avoiding it by design).
- Keep **headroom**: each region must absorb a failed region's traffic.
- Global services (auth, ID generation, config) must not become a cross-region single point of failure.
- Watch costs of cross-region replication traffic.

</details>

### 6. What are cascading failures and how do you prevent them?

<details>
<summary>Answer</summary>

One failure triggers others: a slow DB makes app threads block → thread pools fill → health checks fail → load balancers remove instances → remaining instances get more load and die → retries multiply traffic. Prevention:
- Timeouts and deadline propagation; **retry budgets**; exponential backoff with jitter.
- Circuit breakers and bulkheads.
- **Load shedding** and admission control instead of queueing indefinitely.
- Health checks that don't depend on shared downstreams.
- Autoscaling with headroom; capacity tests.
- Caches that serve stale data when the source is down.

</details>

### 7. What is chaos engineering?

<details>
<summary>Answer</summary>

Deliberately injecting failures into a system (ideally in production, carefully) to verify it handles them: killing instances (Netflix's Chaos Monkey), adding latency, cutting off a dependency, failing an AZ. Process: define steady state (metrics) → hypothesise it holds → inject the fault with a small blast radius → observe → fix weaknesses. It builds confidence that redundancy and failover actually work. Start with game days in staging.

</details>

### 8. What are the three pillars of observability?

<details>
<summary>Answer</summary>

- **Metrics** – numeric time-series (request rate, error rate, latency percentiles, CPU, queue lag). Cheap to store, great for dashboards and alerts. (Prometheus, Datadog.)
- **Logs** – detailed event records. Use **structured** logs (JSON) with request/trace IDs; centralise them (ELK, Loki, Splunk). Expensive at volume, so sample and set retention.
- **Traces** – a request's journey across services as a tree of **spans** with timings (OpenTelemetry, Jaeger, Zipkin). Essential for finding which hop is slow in microservices.

Increasingly also **profiles** (continuous profiling) and **events** (deploys, config changes) correlated on the same timeline. **OpenTelemetry** is the standard for instrumenting all of them.

</details>

### 9. What are the golden signals, RED and USE?

<details>
<summary>Answer</summary>

- **Four golden signals** (Google SRE): **latency, traffic, errors, saturation**.
- **RED** (for request-driven services): **Rate, Errors, Duration**.
- **USE** (for resources like CPU, disks, pools): **Utilisation, Saturation, Errors**.

Monitor latency as **percentiles** (p50/p95/p99), separately for successful and failed requests (fast errors can hide real latency).

</details>

### 10. How should alerting work?

<details>
<summary>Answer</summary>

- Alert on **symptoms users feel** (SLO burn rate: error rate or latency breaching the objective), not on every cause (high CPU).
- Use **multi-window burn-rate alerts** (fast burn pages someone; slow burn creates a ticket).
- Every page must be **actionable** and have a runbook; otherwise it's noise that causes alert fatigue.
- Route by severity; dashboards for investigation, not alerts for everything.

</details>

### 11. How does distributed tracing work?

<details>
<summary>Answer</summary>

The entry service creates a **trace ID**; every downstream call propagates it (W3C `traceparent` header) and each unit of work records a **span** (start, duration, parent span, tags). A collector assembles spans into a tree, showing exactly where time goes and where errors start. Use **sampling** (head-based: keep x%; tail-based: keep all slow/error traces) to control cost. Also put the trace ID in logs to jump between them.

</details>

### 12. Blue-green vs canary vs rolling deployments?

<details>
<summary>Answer</summary>

- **Rolling** – replace instances gradually, a few at a time. No extra capacity needed; both versions run at once (must be compatible); rollback is another rollout.
- **Blue-green** – run the new version (green) alongside the old (blue), switch traffic at once (load balancer/DNS). Instant rollback by switching back; needs double capacity; DB schema must work with both.
- **Canary** – send a small percentage of traffic (1% → 5% → 25% → 100%) to the new version, compare metrics with the baseline automatically, and roll back if worse. Limits blast radius; the standard at large companies.

All require **backward-compatible** changes (especially DB schemas: expand-and-contract).

</details>

### 13. What are feature flags and why are they important?

<details>
<summary>Answer</summary>

Feature flags turn code paths on or off **at runtime** without deploying. They decouple **deploy** from **release**:
- Ship dark code, then enable for internal users → a % of users → everyone.
- **Kill switches** to disable a misbehaving feature instantly.
- A/B experiments and per-tenant entitlements.

Design: flags are evaluated locally from a cached config pushed by a flag service (LaunchDarkly, Unleash, in-house), with safe defaults if the service is unreachable. Clean up old flags; they accumulate as technical debt.

</details>

### 14. How do you run an incident and a postmortem?

<details>
<summary>Answer</summary>

**During:** declare an incident, assign roles (incident commander, communications, operations), **mitigate first** (roll back, fail over, shed load, disable a feature) before finding the root cause, communicate status regularly to stakeholders and customers.

**After:** a **blameless postmortem**: timeline, impact, root cause(s) and contributing factors (5 whys), what went well, and action items with owners (detection, prevention, mitigation). Track action items to completion; share learnings widely.

</details>

### 15. How do you plan capacity?

<details>
<summary>Answer</summary>

- Measure current peak usage per service and resource; know each service's per-instance limits from **load tests**.
- Forecast growth (trend + known events like sales or launches).
- Keep **headroom** (e.g. run below 60–70% at peak) and **N+1/N+2** for failures, including a whole AZ or region failing over.
- Check every dependency (DB, cache, third-party API quotas), not just app servers.
- Revisit regularly; use autoscaling for elasticity but don't assume it's instant.

</details>

### 16. What is the difference between monitoring and observability?

<details>
<summary>Answer</summary>

**Monitoring** tells you **whether** known things are wrong (predefined dashboards and alerts for expected failure modes). **Observability** is the ability to understand **why** something is wrong, including unknown failure modes, by asking new questions of the telemetry (high-cardinality data, traces, rich events). Mature systems need both: monitoring to detect, observability to diagnose.

</details>

### 17. How do you limit the blast radius of failures and changes?

<details>
<summary>Answer</summary>

- **Cells / shuffle sharding** – each customer depends on a small, mostly unique subset of servers, so one bad customer or node affects few others.
- **Staged rollouts** – one host → one AZ → one region → all, with automated health checks between waves.
- **Feature flags** for gradual enablement.
- **Quotas and rate limits** per tenant (noisy neighbours).
- **Config changes** treated like code deploys (validated, staged, revertible) – many major outages come from config.

</details>

---

[⬅ Previous: Microservices & Architecture Patterns](12-microservices-patterns.md) · [⬅ Back to index](../README.md) · [Next: Security in System Design ➡](14-security.md)
