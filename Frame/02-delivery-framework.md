# 02 — Delivery Framework

## 1. Problem
Mid-level candidates rarely fail on knowledge; they fail to **deliver a working system** in 45 minutes (the opaque "time management" feedback). Without a fixed path you drown in detail early, layer on complexity too soon, and never finish an end-to-end design.

## 2. Mechanism
A linear sequence with time budgets. Always have a "next step" to fall back on when nervous.

| Step | Time | What you produce |
|---|---|---|
| Requirements | ~5 min | Functional ("users should be able to…", top 3 only) + non-functional (top 3-5, quantified, e.g. "feed < 200ms", "100M DAU", AP over CP) |
| Core entities | ~2 min | Bullet list of nouns (User, Tweet, Follow). Draft only; fields come later |
| API | ~5 min | Default REST, plural-noun resources; user identity from the auth token, never the body |
| Data flow (optional) | ~5 min | Numbered steps; only for data-processing systems (e.g. crawler) |
| High-level design | ~10-15 min | Boxes/arrows, built endpoint by endpoint; note state changes; **simple** design that meets functional reqs. Call out caches/queues verbally, then move on |
| Deep dives | ~10 min | Harden against non-functional reqs, edge cases, bottlenecks, interviewer probes |

Estimation: skip upfront; do math mid-design only if it changes a decision (e.g. one min-heap vs sharded).

NFR checklist: CAP, environment constraints, scalability (read/write ratio, bursts), latency, durability, security, fault tolerance, compliance.

## 3. Trade-off
- **Gain:** guaranteed complete system, clear communication, no getting lost.
- **Give up:** flexibility and depth early. Simple HLD first means it is knowingly "woefully inefficient" until deep dives; skipping upfront estimation risks missing a scale-driven decision if you forget to do it later.
- Seniority shifts deep dives: juniors get probed, seniors must lead them (without talking over the interviewer).

## 4. When
- **Use:** any open-ended "design X" prompt, product-style especially.
- **Don't:** treat it as a rigid script. Skip data flow if the flow is short; skip estimation if it won't change the design; don't spend 10 min on perfect requirements or a full schema; don't add caches/queues in HLD before the core works.

## 5. Anchor
No natural Go stdlib/package anchor; this is a process, not a component. Loose mental parallel only: get a `main()` that compiles and passes the happy path first, then optimize (profile → fix) — HLD = working skeleton, deep dives = optimization pass.

## Self-test
1. Your HLD is done at minute 20 and the interviewer says nothing. How do you pick what to deep dive on?
2. Why is "the system should be low latency" a weak non-functional requirement, and how would you rewrite it for a search service?
3. When is upfront capacity estimation worth doing, and when is it wasted time?
