# System Design Interview Questions – Cheat Sheet

A revision guide for system design interviews: **370+ questions with interview-ready answers**, split into 23 topics. It covers the building blocks (caching, databases, sharding, consistency, messaging), 37 full FAANG-style design walkthroughs, mobile system design, low-level design, scenario-based "what would you do if…" questions, AI/LLM systems, and **company-wise questions** from Google, Meta, Amazon, Netflix, Uber, Stripe, Flipkart, Swiggy, PhonePe, Zerodha, Hotstar and more.

Every answer is hidden in a collapsible block, so you can **test yourself first** and then click to check.

> **📈 [Open the interactive study tracker](https://akshayashokcode.github.io/System-Design-Interview-Questions-cheat-sheet/)**: mark questions ✓ Done or ↻ Revise, see your progress per topic, search every question, listen to answers read aloud, and continue where you left off.
>
> **No account needed.** Your progress saves automatically on your device. Want to keep it safe and in sync across your devices? Sign in (optional).

Also see the platform repos: **[Android Interview Questions](https://github.com/AkshayAshokCode/Android-Interview-Questions-cheat-sheet)** · **[iOS Interview Questions](https://github.com/AkshayAshokCode/iOS-Interview-Questions-cheat-sheet)**

---

## How to use this repo

1. **Learn the framework first** ([topic 01](topics/01-interview-framework.md)). Every design answer follows the same steps: requirements → estimates → API → data model → high-level design → deep dives → trade-offs.
2. **Study the building blocks** (topics 02–15). Designs are just these blocks combined.
3. **Practise the designs out loud, timed at 45 minutes** (topics 16–19). Draw the diagram before you open the answer.
4. **Drill the scenarios** (topic 20). Interviewers use them as follow-ups ("what if the cache goes down?").

> Tip: there is no single correct design. Interviewers score how you **clarify, justify with numbers, and discuss trade-offs**. Say "we could do A or B; A costs X, B costs Y; given our requirements I'll pick A" as often as you can.

---

## Topics

### 🧭 The Interview

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 01 | [The Interview Framework](topics/01-interview-framework.md) | 15 | What's scored, the 45-minute flow, requirements, API, data model, deep dives, trade-offs, levels, mistakes |
| 02 | [Back-of-the-Envelope Estimation](topics/02-estimation.md) | 12 | Latency numbers, powers of two, QPS, storage, bandwidth, cache and server sizing, nines |

### 🧱 Building Blocks

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 03 | [Scalability Fundamentals](topics/03-scalability-fundamentals.md) | 16 | Vertical vs horizontal, stateless, latency vs throughput, SLA/SLO/SLI, scaling reads and writes |
| 04 | [Networking & Protocols](topics/04-networking-protocols.md) | 15 | DNS, TCP/UDP, HTTP/1.1/2/3, TLS, REST vs GraphQL vs gRPC, WebSockets/SSE, CDN, pagination |
| 05 | [Load Balancing, Proxies & API Gateways](topics/05-load-balancing-proxies-gateways.md) | 14 | L4 vs L7, algorithms, health checks, consistent hashing, API gateway, service mesh, rate limiting |
| 06 | [Caching](topics/06-caching.md) | 15 | Cache-aside vs write-through/back, eviction, invalidation, stampede, hot keys, Redis vs Memcached |

### 🗄 Data

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 07 | [Databases: SQL, NoSQL & Indexing](topics/07-databases.md) | 17 | SQL vs NoSQL, ACID, NoSQL families, indexes, B-tree vs LSM, NewSQL, zero-downtime migrations |
| 08 | [Replication, Partitioning & Sharding](topics/08-replication-sharding.md) | 16 | Leader-follower, multi-leader, quorums, failover, shard keys, hot spots, rebalancing |
| 09 | [Consistency, Transactions & Consensus](topics/09-consistency-transactions-consensus.md) | 18 | CAP/PACELC, isolation levels, locking, 2PC, sagas, outbox, idempotency, Raft, clocks, CRDTs |
| 10 | [Storage, Search & Specialised Data Systems](topics/10-storage-search-specialized.md) | 15 | Object storage, erasure coding, inverted indexes, geohash/quadtree/H3, time-series, Bloom filters |

### 🔀 Architecture & Operations

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 11 | [Messaging & Event-Driven Architecture](topics/11-messaging-event-driven.md) | 15 | Queues vs pub/sub vs logs, Kafka internals, ordering, DLQs, backpressure, event sourcing, CQRS, CDC |
| 12 | [Microservices & Architecture Patterns](topics/12-microservices-patterns.md) | 18 | Monolith vs microservices, boundaries, retries, circuit breakers, bulkheads, BFF, strangler fig, cells |
| 13 | [Reliability, Observability & Operations](topics/13-reliability-observability.md) | 17 | Multi-AZ/region, RPO/RTO, cascading failures, logs/metrics/traces, canary deploys, incidents |
| 14 | [Security in System Design](topics/14-security.md) | 15 | Sessions vs JWT, OAuth/OIDC, RBAC/ReBAC, encryption, KMS, DDoS, zero trust, GDPR |
| 15 | [Big Data & Stream Processing](topics/15-big-data-streaming.md) | 15 | OLTP vs OLAP, lakehouse, batch vs stream, Spark, Flink, watermarks, Lambda vs Kappa, top-K |

### 🏗 Design Case Studies

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 16 | [Designs – Core Infrastructure](topics/16-designs-core-infrastructure.md) | 11 | URL shortener, rate limiter, ID generator, KV store, distributed cache, crawler, notifications, typeahead, scheduler, message queue, monitoring |
| 17 | [Designs – Social, Media & Collaboration](topics/17-designs-social-media.md) | 12 | News feed, Twitter, Instagram, WhatsApp, YouTube, live streaming, Dropbox, Google Docs, Reddit, search engine, Spotify, Zoom |
| 18 | [Designs – Location, Commerce & Fintech](topics/18-designs-location-commerce.md) | 14 | Uber, Yelp, Google Maps, food delivery, Ticketmaster, Amazon, flash sale, payments, wallet, hotel booking, stock exchange, ad clicks, leaderboard, LeetCode |
| 19 | [Mobile System Design](topics/19-mobile-system-design.md) | 16 | Client architecture, mobile APIs, offline-first sync, real-time, push, caching, background limits, feed/chat/image loader designs |

### ⭐ Interview Rounds & Trends

| # | Topic | Questions | Key areas |
|---|---|---|---|
| 20 | [Scenario-Based Questions](topics/20-scenario-based.md) | 25 | Latency spikes, DB at 95% CPU, 10× traffic, celebrity posts, cache down, region outage, double charges, migrations |
| 21 | [Low-Level Design](topics/21-low-level-design.md) | 16 | Approach, SOLID, patterns, concurrency; parking lot, elevator, LRU cache, rate limiter, vending machine, BookMyShow, Splitwise |
| 22 | [AI/ML Systems & What's New](topics/22-ai-ml-whats-new.md) | 17 | Recommendations, vector search, RAG, ChatGPT design, LLM serving, agents, fraud, moderation, Kubernetes, edge, local-first |
| 23 | [Company-Wise Questions](topics/23-company-wise-questions.md) | 26 | What Google, Meta, Amazon, Microsoft, Apple, Netflix, Uber, Stripe and Indian product companies ask; Gmail, Calendar, view counter, A/B testing, webhooks, UPI, stock ticker, Hotstar |

---

## 🗓 Suggested revision plans

**2-week plan (≈1–2 hours/day)**

| Day | Topics |
|---|---|
| 1 | 01 Interview Framework + 02 Estimation |
| 2 | 03 Scalability + 04 Networking |
| 3 | 05 Load Balancing + 06 Caching |
| 4 | 07 Databases |
| 5 | 08 Replication & Sharding |
| 6 | 09 Consistency, Transactions & Consensus |
| 7 | 10 Storage & Search + 11 Messaging |
| 8 | 12 Microservices + 13 Reliability |
| 9 | 14 Security + 15 Big Data |
| 10 | 16 Core Infrastructure designs (practise 3 out loud) |
| 11 | 17 Social & Media designs (practise 3 out loud) |
| 12 | 18 Location & Commerce designs (practise 3 out loud) |
| 13 | 20 Scenarios + 21 Low-Level Design (or 19 Mobile if you're a mobile engineer) |
| 14 | 23 Company-Wise (your target companies) + 22 AI/ML & What's New + Last-Minute Revision + one full mock interview |

**2 days before:** the framework (01), estimation numbers (02), caching (06), sharding (08), consistency (09), the five most common designs (URL shortener, rate limiter, news feed, chat, Uber), scenarios (20) and the list below.

---

## ⚡ Last-Minute Revision (one-liners)

<details>
<summary>Open the quick revision list</summary>

**Interview flow**
- Requirements (functional + non-functional) → estimates → API → data model → high-level design → deep dives → trade-offs.
- Scope out loud, state assumptions, check in with the interviewer, and keep talking.
- Start simple, then evolve the design as you hit a requirement. Every choice needs a "because".

**Numbers**
- 1 day ≈ 10^5 s. 1M requests/day ≈ 12/s. Peak ≈ 2–3× average.
- Memory ~100 ns, SSD random read ~100 µs, same-DC round trip ~0.5 ms, cross-continent ~100 ms.
- One Postgres primary ≈ 5–10K writes/s; Redis ≈ 100K+ ops/s per node; app server ≈ 1–10K req/s.
- 99.9% = ~43 min down/month; 99.99% = ~4 min/month.

**Scaling**
- Stateless app servers scale horizontally behind a load balancer; state lives in DBs and caches.
- Scale reads with caches, replicas, CDN and denormalisation. Scale writes with sharding, queues, batching and LSM stores.
- Consistent hashing with virtual nodes moves only ~1/N keys when nodes change.
- Shard key: high cardinality, even load, matches the main query, keeps related data together.

**Caching**
- Cache-aside + delete on write is the default. Add TTL jitter.
- Stampede → single-flight / stale-while-revalidate. Penetration → cache nulls / Bloom filter. Hot key → local cache / replicate the key.

**Data & consistency**
- CAP: during a partition, choose consistency or availability. PACELC: otherwise, latency or consistency.
- Strong consistency for money, inventory, bookings, uniqueness; eventual for likes, feeds, counts.
- Replication lag issues: read-your-writes, monotonic reads.
- W + R > N gives quorum overlap.
- Cross-service transactions → sagas with compensations; reliable events → transactional outbox / CDC.
- Retries are inevitable → idempotency keys. Exactly-once = at-least-once + idempotent processing.
- Consensus (Raft) needs a majority: 3 nodes tolerate 1 failure, 5 tolerate 2. Use fencing tokens with locks.

**Messaging**
- Kafka: partitioned append-only log; ordering per partition; consumer groups; replay by offset.
- Retry with backoff, then DLQ. Watch consumer lag. Partition by entity ID for ordering.
- CQRS: separate write model from read models updated by events.

**Reliability**
- Timeouts on every call, retries with backoff + jitter + budgets, circuit breakers, bulkheads, load shedding.
- Multi-AZ by default; multi-region for DR (backup/restore → pilot light → warm standby → active-active).
- Observability: metrics (RED/USE, golden signals), structured logs, distributed traces. Alert on SLO burn.
- Deploy with canaries and feature flags; expand-and-contract for schema changes.

**Classic design patterns**
- Feed: hybrid fan-out (push for normal users, pull for celebrities) + ranking.
- Chat: WebSocket gateways + session registry + messages partitioned by chat_id + push when offline.
- Booking/tickets: conditional updates or holds with TTL; waiting room for spikes.
- Payments: idempotency keys, state machine, double-entry ledger, reconciliation.
- Location: in-memory geo index (geohash/H3), ETA-based matching, atomic driver locks.
- Video: chunked upload → parallel transcoding → adaptive bitrate (HLS/DASH) → CDN.
- Search: inverted index sharded by document, scatter-gather, multi-stage ranking.

</details>
