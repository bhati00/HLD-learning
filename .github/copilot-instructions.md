# HLD Learning — Copilot Instructions

This repo is a personal HLD (system design) interview prep space for a Go backend
developer (~3 YOE). Whenever generating notes, explanations, or content for any
topic in this repo (README topics, `notes.md`, topic write-ups, etc.), apply the
**5-step topic framework** below instead of writing free-form explanations.

## The 5-step topic framework

For any system design topic, structure the explanation in this exact order:

1. **Problem** — What pain point / limitation forced this concept to exist? Start
   here, never with mechanism. No problem statement, no lesson.
2. **Mechanism** — The simplest version of how it actually works. Avoid going
   into implementation minutiae unless asked; one clear mental model is enough.
3. **Trade-off** — What do you gain, and what do you explicitly give up? Every
   topic must end with a trade-off, never presented as a free win.
4. **When** — When to reach for it, and just as important, when NOT to (the
   "when not" half is usually the part people skip and interviewers probe).
5. **Anchor** — Map the concept to something the user already knows as a Go
   developer (stdlib, common packages, or a concurrency pattern they'd
   recognize). Prefer real Go anchors over generic analogies. Reference table:

   | HLD Topic | Go anchor |
   |---|---|
   | Caching / stampede prevention | `singleflight`, `sync.Map`, `groupcache/lru` |
   | Rate limiting | `golang.org/x/time/rate` (token bucket) |
   | Load balancing | `httputil.ReverseProxy`, round-robin over backend slice |
   | Consistent hashing | `groupcache/consistenthash` |
   | Message queues | buffered channels + worker pool goroutines |
   | Circuit breakers | `sony/gobreaker` |
   | Sharding intuition | sharded-mutex map pattern (bucketing a map to cut lock contention) |

   If a topic has no natural Go anchor, say so explicitly rather than forcing
   a weak analogy.

End each topic explanation with 2-3 self-test questions the user could be asked
on that topic (do not answer them yet, just list them — they seed the later
`practice.md` file for that same topic).

## Repo structure

Each foundational topic lives in its own numbered folder, containing exactly
two files — no `questions.md`, no separate `practice-systems/` tree:

```
04-caching/
├── notes.md      ← Layer 1: the 5-step framework explanation for this topic
└── practice.md   ← Layer 2: researched interview questions + model answers
```

## Layer 2: `practice.md` — how to generate it

Only create/fill `practice.md` for a topic when explicitly asked. When asked,
follow this process:

1. **Research** real, commonly-asked interview questions for that topic (the
   core design prompt most tied to it, plus the chained follow-ups
   interviewers actually ask). Use web search — don't invent questions from
   memory alone.
2. **Compile** a clean, deduped list: the primary design prompt (if one
   naturally belongs to this topic, e.g. "design a rate limiter" under scaling
   patterns) plus 3-5 of the most common follow-up questions.
3. **Answer in interview-delivery format** — decision-first, the way a strong
   candidate would actually speak it, never a listed-options essay:

   ```
   ### Q: <question as asked>

   **Context/scope** (1 line — what's assumed): ...

   **Answer**:
   - Decision: <the actual choice made>
   - Why: <reasoning tied to specific constraints, not generic>
   - Trade-off given up: <what you're explicitly not getting>

   **If pushed further**: <one line on how you'd adapt if constraints change>
   ```

   For the primary full-system design prompt specifically (not a follow-up),
   use the 6-step delivery framework instead (Clarify → Estimate → API →
   High-level design → Deep dive → Scale/bottlenecks), condensed into a
   written model answer in that same decision-first style.

## Placement rule for multi-topic systems

Some canonical practice systems (chat, news feed, payments, ride matching)
span multiple topics at once. Default: place the practice question under the
single topic folder that is its primary bottleneck/test (e.g., a chat system's
message-ordering question goes under the messaging topic, not duplicated
across every topic it touches). Only split a system across multiple topic
folders if explicitly asked to.

## Scope notes

- The 5-step framework applies to `notes.md` (Layer 1) only.
- Keep each topic's `notes.md` tight — roughly half a page. The goal is
  judgment and recall, not an essay.
- Never generate `practice.md` content proactively — only when the user
  explicitly asks for that topic's practice questions.
