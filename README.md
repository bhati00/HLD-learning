# HLD Prep — Foundational Vocabulary (Layer 1)

Priority: **P0** = must know cold (comes up in almost every interview) · **P1** = comes up often, needed for trade-off depth · **P2** = situational, lower priority given time constraints.

Status legend: `[ ]` not started · `[~]` in progress · `[x]` done

Practice layer (Layer 2 — full system designs + follow-up questions) lives separately in `practice-systems/` once you're ready to start it. This README only tracks Layer 1.

---

## 1. Requirements & Estimation — **P0**
- [ ] Functional vs non-functional requirements
- [ ] Back-of-envelope estimation (QPS, storage, bandwidth, memory)
- [ ] Read-heavy vs write-heavy classification
- [ ] Latency vs throughput vs availability trade-offs
- [ ] Explicitly scoping out what you won't design

## 2. APIs, Load Balancers, Gateway — **P0**
- [ ] REST vs gRPC vs GraphQL vs WebSockets — when each fits
- [ ] API design basics (endpoints, pagination, idempotency keys)
- [ ] L4 vs L7 load balancing
- [ ] Load balancing algorithms (round robin, least connections, consistent hashing)
- [ ] API Gateway responsibilities (auth, rate limiting, routing)
- [ ] Reverse proxy basics

## 3. Databases — **P0**
- [ ] SQL vs NoSQL — when to use which
- [ ] Indexing (B-tree, hash index, composite index)
- [ ] Normalization vs denormalization
- [ ] ACID transactions
- [ ] NoSQL types (key-value, document, column-family, graph) and use cases
- [ ] Letting query patterns drive schema design

## 4. Caching — **P0**
- [ ] Cache placement (client, CDN, app-level, DB-level)
- [ ] Cache-aside vs write-through vs write-behind
- [ ] Eviction policies (LRU, LFU, TTL)
- [ ] Cache invalidation strategies
- [ ] Distributed caching basics (Redis cluster concepts)
- [ ] Thundering herd / cache stampede problem

## 5. Scaling Patterns — **P0**
- [ ] Horizontal vs vertical scaling
- [ ] Stateless service design
- [ ] Rate limiting algorithms (token bucket, leaky bucket, sliding window)
- [ ] Backpressure and load shedding
- [ ] Autoscaling basics

## 6. Replication & Sharding — **P1**
- [ ] Leader-follower replication
- [ ] Synchronous vs asynchronous replication
- [ ] Quorum reads/writes
- [ ] Sharding strategies (range, hash, directory-based)
- [ ] Consistent hashing
- [ ] Rebalancing and hot partitions

## 7. Consistency & CAP — **P1**
- [ ] CAP theorem
- [ ] PACELC
- [ ] Strong vs eventual consistency
- [ ] Read-your-writes / causal consistency
- [ ] Conflict resolution (last-write-wins, vector clocks, CRDTs)

## 8. Messaging & Async Processing — **P1**
- [ ] Queues vs pub-sub vs streams
- [ ] Delivery guarantees (at-least-once, exactly-once, at-most-once)
- [ ] Idempotency
- [ ] Dead-letter queues and retries
- [ ] Fan-out patterns (push vs pull vs hybrid)

## 9. Reliability & Observability — **P1**
- [ ] Failover and redundancy
- [ ] Retries with backoff, circuit breakers, timeouts
- [ ] Health checks
- [ ] Monitoring: metrics, logs, traces
- [ ] Alerting thresholds (what to alert on, and why)
- [ ] Incident/runbook thinking

## 10. CDN & Edge — **P2**
- [ ] How CDNs work (edge caching, origin pull)
- [ ] Static vs dynamic content delivery
- [ ] Geo-distribution and latency reduction

## 11. Security Basics — **P2**
- [ ] Authentication vs authorization
- [ ] OAuth2/JWT basics
- [ ] Encryption in transit (TLS) vs at rest
- [ ] Basic threat considerations (abuse via rate limits, injection)

---

## Suggested order of attack

Given your time constraint (HLD is second priority after LLD/DSA/Go internals), work top to bottom — P0 topics (1-5) first since they alone cover ~80% of interview ground, then P1 (6-9), then P2 (10-11) only if time remains.
