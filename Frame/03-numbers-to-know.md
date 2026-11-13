# 03 — Numbers to Know (Estimation)

Sources: free parts of Hello Interview's [Numbers to Know](https://www.hellointerview.com/learn/system-design/core-concepts/numbers-to-know) and [Mastering Estimation](https://www.hellointerview.com/blog/mastering-estimation). The component cheat sheet is Premium, so the "one machine handles about" table below is my own ballpark of commonly quoted figures. Verify before quoting it as fact.

## 1. Problem
"Design Twitter": how do you know whether you need one DB or fifty? A cache? A queue? Gut feeling doesn't answer that, a few quick calculations do. Candidates fail this in two ways: they quote outdated numbers ("shard at 100 GB") and over-engineer, or they guess wildly wrong magnitudes (an internet round trip in microseconds). Estimation exists to justify a design decision, not to show arithmetic.

Every calculation answers one of three questions:
1. **Traffic:** can one machine handle the requests per second?
2. **Storage:** can one machine hold all the data?
3. **Speed:** is the way I fetch data fast enough?

"Yes, one machine is fine" means keep the design simple. "No" tells you what to add: sharding, caching, replicas, CDN.

## 2. Mechanism

### Q1: Traffic
1. **Per day → per second.** 1 day ≈ **100K seconds** (86,400). Divide any per-day number by 100K.

   | Requests/day | Per second |
   |---|---|
   | 1M | ~10 |
   | 100M | ~1,000 |
   | 10B | ~100,000 |

2. **Allow for peaks.** Peak ≈ 2-10× average (use ~5×).
3. **Compare with one machine** (approximate):

   | Thing | One machine handles about |
   |---|---|
   | Postgres/MySQL writes | ~1-10K/sec |
   | Postgres/MySQL simple reads | tens of thousands/sec |
   | Redis | ~100K ops/sec |
   | App server | thousands of req/sec |
   | Kafka broker | ~100K-1M msgs/sec |

**Worked example, Twitter:** 250M DAU × 2 tweets = 500M writes/day → 5K/sec avg, ~25K/sec peak. More than one Postgres node writes, so shard or batch writes. Reads: 100 tweets viewed/user/day = 25B/day → ~250K/sec. No single DB does that, so caches and read replicas. Same system, a read problem and a write problem; that is why "read-heavy or write-heavy?" matters.

### Q2: Storage
Formula: `records × size per record = total bytes`. Add up the fields and round up.

| Thing | Size |
|---|---|
| Text record (tweet, URL row) | 300 B-1 KB, round to 1 KB |
| Medium image | ~100 KB |
| High-res photo | ~1 MB |
| 2-hour movie | ~1 GB |

Units: stay in powers of 1000 (K → M → G → T → P).

**Worked example, Twitter:** 500M tweets/day × 300 B = 150 GB/day ≈ 55 TB/year. One box holds ~60 TB SSD, so a year nearly fits on one machine. "Shard at 100 GB" is badly outdated. The real reasons to shard later: growth every year, write throughput, and replicas/failure domains.

**Does it fit in RAM?** One box has 512 GB-4 TB+ RAM (up to 24 TB). Bitly: 1B URLs × 500 B = 500 GB, so it fits in memory on one big box. 500 TB would not, and you'd need another plan.

### Q3: Speed
| Action | Time |
|---|---|
| Read 1 MB from RAM | 0.25 ms |
| Read 1 MB from SSD | 1 ms (~4× RAM) |
| Read 1 MB from HDD | 20 ms (~20× SSD) |
| Network, same AZ / cross-AZ | <1 ms / 1-2 ms |
| Network, cross-region (CA ↔ Netherlands) | 50-150 ms |

What this teaches: caches exist because RAM beats disk; CDNs exist because 5 sequential cross-region calls ≈ 750 ms; never put a cross-region call in a hot loop.

Other hardware facts: 128+ vCPU, 25-100 Gbps network, S3 effectively unlimited.
Business magnitudes: big social DAU ~1B, Google searches ~100K/sec, Netflix ~100M hours/day, Wikipedia ~100 GB.

## 3. Trade-off
- **Gain:** you prune over-engineering (no premature sharding/caching) and defend every component with a number.
- **Give up:** interview time, and precision. Rounding makes answers off by ~3×; wrong magnitudes hurt, wrong digits don't.
- Modern-hardware optimism can bite if you ignore peak factors, replication cost, or failure domains: "fits on one machine" still needs HA.

## 4. When

### When in the interview
Not upfront. It follows the [delivery framework](../02-delivery-framework/notes.md): do the math at the moment a decision depends on it.

| Phase | Numbers |
|---|---|
| Requirements (0-5 min) | No math. Ask for the scale and write it as a target ("100M new URLs/day, redirect < 100 ms"). |
| Entities and API | None. |
| High-level design | 20-second checks when picking a component: "can one DB take this write rate?" |
| Deep dives | Heaviest use: read/write heavy? need a cache? fits on one node or shard? hot data fits in RAM? |

Triggers: you're about to add a cache, queue, or sharding; you're about to say "one DB is fine"; the interviewer says "how much storage?", "now scale it", or "will it hold at 10×?".

### When NOT to
- Estimating for show: a page of DAU/QPS/storage ending in "so it's big".
- Over-engineering: "10K QPS, let's shard and add Kafka." One DB handles that.
- Exact arithmetic: 323 B becomes 500 B. Round and move on.
- Wrong magnitudes: network calls in microseconds, 1 GB not fitting in memory.

### How to deliver it
Every line: **a number → the decision it drives → what it costs.** Sample answer follows.

## 5. Anchor
Partial. Go has no estimation library, but `time.Duration` constants and `1 << (10*(i+1))` iota size constants mirror the unit math, and `go test -bench` + `unsafe.Sizeof` give real per-op latency and struct sizes: measure, don't guess. A sharded-mutex map is a good intuition for one node's limits.

## Worked answer: URL shortener (interview-delivery format)

**Context/scope:** 100M new URLs/day, ~500 B/row, reads 100× writes (assumed).

**Writes**
- Decision: one Postgres primary. 100M ÷ 100K = ~1K/sec avg, ~5K/sec peak (5×).
- Why: within a single node's range of a few thousand writes/sec.
- Trade-off given up: no horizontal write scaling; revisit at ~10×.

**Reads**
- Decision: Redis in front of the DB, plus read replicas. 100× writes = ~100K/sec avg, ~500K/sec peak.
- Why: one DB does tens of thousands of reads/sec, so 500K can't hit it. Redis ~100K ops/sec per node means ~5-10 cache nodes.
- Trade-off given up: stale cache (fine, URLs rarely change) and extra ops cost.

**Storage**
- Decision: single node for year one; shard by short-code hash before ~year 3; keep replicas.
- Why: 100M × 500 B = 50 GB/day ≈ 18 TB/year ≈ 90 TB over 5 years; one box holds ~60 TB SSD.
- Trade-off given up: sharding makes joins and rebalancing harder.

**Hot data:** hot 20% of a day's URLs ≈ 10 GB, fits easily in one Redis node's RAM.

**Key space:** 7 base-62 chars ≈ 3.5 trillion codes; 10 years at this rate ≈ 365B. 7 is enough.

**If pushed to 10×:** writes ~10K/sec avg, so shard the DB; reads in the millions/sec, so add a CDN/edge cache for popular links.

## Self-test
Answer each one the interview way: **number → decision → what it costs**. Answers are not included; attempt them, then ask for a check.

Use these constants unless a question says otherwise: 1 day ≈ 100K sec · peak = 5× average · one DB node ≈ 5K writes/sec, 30K reads/sec · one Redis node ≈ 100K ops/sec · one box ≈ 60 TB SSD, 1 TB RAM · replication factor 3.

### 1. Storage and sharding (Twitter-like)
**Given:** 250M DAU, each posts 2 tweets/day, 300 B per tweet. Ignore media.
**Compute:**
- a) Tweets per day, and writes/sec (average and peak).
- b) Storage per day, per year, and over 5 years.
- c) Storage per year including replication factor 3.
**Decide:** does the write rate need sharding? Does the storage need sharding? Which one forces it first, and when?
**Cost to state:** what you give up by sharding.

### 2. Traffic and the one-DB question
**Given:** a service gets 50M requests/day, 90% reads and 10% writes.
**Compute:**
- a) Average and peak requests/sec.
- b) Peak reads/sec and peak writes/sec separately.
**Decide:** can one DB node take it? Is a cache needed? A read replica? Say which number made each call.
**Cost to state:** what a cache or replica adds (staleness, ops).

### 3. The outdated-rule question
**Given:** someone says "we must shard once the DB hits 100 GB."
**Explain:**
- a) Why this is outdated (use the one-box capacity numbers).
- b) Name three real reasons that would push you to shard: one about writes, one about storage growth, one about failure/availability.
- c) For each reason, the measurable number that would trigger it.

### 4. URL shortener, full estimate
**Given:** 100M new URLs/day, 500 B per row, reads are 100× writes, redirect latency target < 100 ms.
**Compute:**
- a) Writes/sec (average and peak) and reads/sec (average and peak).
- b) Storage per year and over 5 years.
- c) Redis nodes needed for peak reads.
- d) Will the hot data (say the 20% most-used URLs from the last 30 days) fit in one Redis node's RAM?
- e) Short-code length needed for 10 years (base-62: 62^6 ≈ 57B, 62^7 ≈ 3.5T).
**Decide:** one DB or sharded? Cache or not? Where does a CDN help?
**Cost to state:** one trade-off per decision.

### 5. Latency reasoning
**Given:** a user in India loads a page. The backend is in the US, and the page needs 4 sequential API calls.
**Compute:** total network time using ~150 ms per cross-region round trip.
**Decide:** what do you change to get under 300 ms? (Think CDN, regional deployment, batching calls into one.)
**Cost to state:** what each fix costs (consistency, extra infra, bigger single call).

### 6. Does it fit in RAM?
**Given:** a chat app has 500M users, and you want to keep each user's "last seen" and unread count in memory, 100 B per user.
**Compute:** total memory needed.
**Decide:** one machine or a cluster? What if it were 5 KB per user instead?
**Cost to state:** what in-memory storage costs (durability, cost per GB, restart behaviour).
