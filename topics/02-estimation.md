# Back-of-the-Envelope Estimation

[⬅ Back to index](../README.md)

Covers: why estimate, latency numbers every engineer should know, powers of two, QPS, storage, bandwidth, cache and server sizing, and worked examples.

---

### 1. Why do back-of-the-envelope estimates matter in a design interview?

<details>
<summary>Answer</summary>

Numbers turn opinions into decisions. A quick estimate tells you whether:
- one database is enough or you need sharding,
- the working set fits in a cache,
- you need a CDN for bandwidth,
- writes must go through a queue.

Rules: round aggressively (powers of 10), state assumptions out loud, and **only estimate what affects the design**. Two to five minutes is plenty. If a number doesn't change any decision, skip it.

</details>

### 2. What latency numbers should every engineer know?

<details>
<summary>Answer</summary>

Approximate orders of magnitude:

| Operation | Time |
|---|---|
| L1 cache reference | ~1 ns |
| Main memory reference | ~100 ns |
| Compress 1 KB (fast codec) | ~2 µs |
| Read 1 MB sequentially from memory | ~10 µs |
| Round trip within a data centre | ~0.5 ms |
| Random read from SSD | ~100 µs |
| Read 1 MB sequentially from SSD | ~0.5–1 ms |
| Disk (HDD) seek | ~10 ms |
| Read 1 MB sequentially from HDD | ~5–20 ms |
| Round trip same region / cross-AZ | ~1–2 ms |
| Round trip US East ↔ West | ~60–70 ms |
| Round trip US ↔ Europe | ~80–150 ms |

Takeaways: memory is ~1,000× faster than SSD random reads, and the network across continents costs ~100 ms, so **cache in memory, avoid cross-region round trips on the hot path, and batch requests**.

</details>

### 3. What powers of two and unit conversions are handy?

<details>
<summary>Answer</summary>

| Power | Approx | Name |
|---|---|---|
| 2^10 | 1 thousand | KB |
| 2^20 | 1 million | MB |
| 2^30 | 1 billion | GB |
| 2^40 | 1 trillion | TB |
| 2^50 | 1 quadrillion | PB |

Time: **1 day ≈ 86,400 s ≈ 10^5 s**. 1 month ≈ 2.5 × 10^6 s. 1 year ≈ 3 × 10^7 s.

Handy shortcuts:
- 1 million requests/day ≈ **12 requests/s**.
- 100 million/day ≈ ~1,200/s. 1 billion/day ≈ ~12,000/s.
- Peak traffic ≈ **2–3× the average** (more for spiky products).

</details>

### 4. How do you estimate QPS?

<details>
<summary>Answer</summary>

`QPS = DAU × actions per user per day ÷ 86,400`, then multiply by a peak factor.

Example – a Twitter-like app:
- 200M DAU, each views the timeline 10 times/day → 2 × 10^9 reads/day → **~23K read QPS**, peak ~50K.
- Each posts 0.5 tweets/day → 10^8 writes/day → **~1.2K write QPS**, peak ~3K.
- Read:write ≈ 20:1 → read-heavy, so caches and precomputed timelines.

Also estimate fan-out: if the average user has 200 followers, 1.2K tweets/s × 200 = **240K timeline inserts/s** if you fan out on write. That number drives the design.

</details>

### 5. How do you estimate storage?

<details>
<summary>Answer</summary>

`storage/day = new items per day × size per item`, then × retention × replication factor.

Example – photo-sharing app:
- 10M photos uploaded/day, average 2 MB after compression → 20 TB/day.
- 5 years ≈ 1,800 days → ~36 PB, × 3 replicas ≈ ~100 PB (or ~1.5× with erasure coding).
- Metadata: 10M × 1 KB = 10 GB/day → small, fits in a sharded database.

Conclusions: blobs go to object storage with tiering (hot → cold), metadata goes in a database, thumbnails are generated in several sizes.

</details>

### 6. How do you estimate bandwidth?

<details>
<summary>Answer</summary>

`bandwidth = QPS × average response size`.

Example – video streaming: 1M concurrent viewers × 5 Mbps (1080p) = **5 Tbps** egress. No single data centre serves that economically, which is why a **CDN with edge caches** (and ISP-embedded caches like Netflix Open Connect) is mandatory.

Example – an API: 50K QPS × 10 KB = 500 MB/s ≈ 4 Gbps → a handful of servers' NICs; fine.

</details>

### 7. How do you estimate cache size?

<details>
<summary>Answer</summary>

Use the **80/20 rule**: ~20% of items get ~80% of traffic, so cache the hot 20% of a day's reads.

Example – URL shortener: 100M redirects/day, 20% hot → 20M URLs × 500 bytes ≈ **10 GB**. That fits in RAM on one Redis node; use a small cluster for availability and throughput.

Also check throughput: one Redis node does ~100K+ simple ops/s, so 50K QPS is fine on a few nodes.

</details>

### 8. How many servers do you need?

<details>
<summary>Answer</summary>

Rough per-node capacities (very workload-dependent; state that):
- A stateless app server: **~1K–10K requests/s** for simple JSON APIs.
- A relational DB primary: ~**5–10K writes/s** and tens of thousands of simple reads/s with good indexes.
- Redis: ~100K+ ops/s per node.
- Kafka: hundreds of MB/s per broker.

Servers = peak QPS ÷ per-server capacity, plus ~30–50% headroom and N+1 for failures. Example: 50K peak QPS ÷ 5K per server = 10 servers → run ~15 across 3 availability zones.

</details>

### 9. Walk through a full estimate for a URL shortener.

<details>
<summary>Answer</summary>

Assumptions: 100M new URLs/month, read:write = 100:1, keep 5 years.
- **Writes:** 100M ÷ (30 × 86,400) ≈ **40/s**. Peak ~100/s.
- **Reads:** ×100 → **4K/s**, peak ~10K/s.
- **Total URLs:** 100M × 12 × 5 = **6 billion**.
- **Key length:** base62 → 62^7 ≈ 3.5 trillion ≫ 6B, so **7 characters** is plenty.
- **Storage:** 6B × 500 bytes ≈ **3 TB** → fits a sharded KV store easily.
- **Cache:** 20% of daily reads ≈ ~70M × 500 B ≈ 35 GB → a small Redis cluster.

Design consequence: reads dominate → cache + read path optimisation. Writes are tiny → no queue needed.

</details>

### 10. What estimation mistakes should you avoid?

<details>
<summary>Answer</summary>

- Too much precision (computing 23,148 QPS instead of "~23K").
- Forgetting **peak** vs average, or replication factor for storage.
- Mixing bits and bytes in bandwidth (network is quoted in bits/s; 1 Gbps ≈ 125 MB/s).
- Doing maths that never changes a decision.
- Not sanity-checking ("10 PB of text tweets per day? That's wrong; recheck the unit").

</details>

### 11. What are availability "nines" in time?

<details>
<summary>Answer</summary>

| Availability | Downtime/year | Downtime/month |
|---|---|---|
| 99% (two nines) | ~3.65 days | ~7.2 hours |
| 99.9% | ~8.8 hours | ~43 minutes |
| 99.99% | ~53 minutes | ~4.3 minutes |
| 99.999% | ~5.3 minutes | ~26 seconds |

Components **in series** multiply: two 99.9% services both required → 99.8%. **Redundant** components in parallel: 1 − (0.001 × 0.001) = 99.9999% if failures are independent. That's why critical paths keep few hard dependencies and use redundancy.

</details>

### 12. How do you estimate the number of shards or partitions?

<details>
<summary>Answer</summary>

Take the larger of two limits:
- **By storage:** total data ÷ comfortable size per node (e.g. 30 TB ÷ 1 TB = 30 shards).
- **By throughput:** peak writes ÷ writes per node (e.g. 100K/s ÷ 5K/s = 20 shards).

Then add headroom for growth (often round up to a power of two or create many **virtual/logical shards**, e.g. 1,024, mapped onto fewer physical nodes so you can rebalance without re-hashing everything).

</details>

---

[⬅ Previous: The Interview Framework](01-interview-framework.md) · [⬅ Back to index](../README.md) · [Next: Scalability Fundamentals ➡](03-scalability-fundamentals.md)
