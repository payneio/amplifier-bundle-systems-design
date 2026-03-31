# Checkpoint 2: On-Demand Design Knowledge via Skills — Implementation Plan

> **Execution:** Use the subagent-driven-development workflow to implement this plan.

**Goal:** Add 5 skills to the system-design-intelligence bundle so the agent can load specialized design knowledge on demand without paying token cost on every turn.

**Architecture:** Skills are directories under `skills/` containing a `SKILL.md` file with YAML frontmatter + markdown body. They are discovered by the `tool-skills` module wired into our behavior YAML. Four are inline skills (inject knowledge into current context when loaded). One is a fork skill (spawns an isolated subagent that orchestrates parallel adversarial reviewers via the delegate tool).

**Tech Stack:** Amplifier skills system (YAML frontmatter + markdown), tool-skills module for discovery/loading, fork execution for the adversarial-review skill.

**Existing files from Checkpoint 1:**
- `bundle.md` — Root bundle (thin, includes foundation)
- `behaviors/system-design.yaml` — Behavior wiring with context includes
- `context/system-design-principles.md` — Core methodology (6 principles + standing behaviors)
- `context/structured-design-template.md` — 11-section output template
- `context/instructions.md` — Standing orders

---

### Task 1: Wire tool-skills into the behavior YAML

**Files:**
- Modify: `behaviors/system-design.yaml`

**Step 1: Verify no skills are discoverable yet**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: The agent either has no `load_skill` tool available, or shows no skills from our bundle. This confirms we haven't wired skills yet.

**Step 2: Add the tools section to the behavior YAML**

The file `behaviors/system-design.yaml` currently contains:

```yaml
bundle:
  name: system-design-behavior
  version: 0.1.0
  description: |
    System design methodology behavior.
    Loads design principles, structured output template, and standing orders
    into the root session context.

context:
  include:
    - system-design-intelligence:context/system-design-principles.md
    - system-design-intelligence:context/structured-design-template.md
    - system-design-intelligence:context/instructions.md
```

Add a `tools:` section **between** the `bundle:` block and the `context:` block. The complete file should become:

```yaml
bundle:
  name: system-design-behavior
  version: 0.1.0
  description: |
    System design methodology behavior.
    Loads design principles, structured output template, and standing orders
    into the root session context.

tools:
  - module: tool-skills
    source: git+https://github.com/microsoft/amplifier-module-tool-skills@main
    config:
      skills:
        - "@system-design-intelligence:skills"

context:
  include:
    - system-design-intelligence:context/system-design-principles.md
    - system-design-intelligence:context/structured-design-template.md
    - system-design-intelligence:context/instructions.md
```

**Step 3: Create the skills directory with a placeholder**

Create `skills/.gitkeep` (empty file) so the directory exists in git:

```bash
mkdir -p skills && touch skills/.gitkeep
```

**Step 4: Verify the bundle still loads without errors**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Hello, confirm you can see me."
```
Expected: The agent responds normally. No errors about missing skills directory or tool-skills module. The tool-skills module should load even though we have no skills yet.

**Step 5: Commit**
```bash
git add behaviors/system-design.yaml skills/.gitkeep && git commit -m "feat: wire tool-skills module into behavior YAML"
```

---

### Task 2: Create the tradeoff-analysis skill

**Files:**
- Create: `skills/tradeoff-analysis/SKILL.md`

**Step 1: Create the skill file**

Create `skills/tradeoff-analysis/SKILL.md` with this exact content:

```markdown
---
name: tradeoff-analysis
description: "Structured tradeoff analysis methodology — the 8-dimension comparison frame, tradeoff matrix template, and common tradeoff patterns. Use when evaluating design alternatives, comparing technology choices, or when the answer is 'it depends.'"
---

# Tradeoff Analysis

## The Central Question

Every design decision answers this:

> **"What does this optimize for, and what does it sacrifice?"**

If you cannot answer this clearly for a design, the design is not yet understood.

## The 8-Dimension Comparison Frame

Evaluate every design alternative against these fixed dimensions. Do not invent new dimensions — force the analysis into this frame so alternatives are comparable.

| Dimension | Key Question | Watch For |
|-----------|-------------|-----------|
| **Latency** | How fast must it respond? | P50 vs P99 distinction; latency budgets per hop |
| **Complexity** | How many concepts must be held in mind? | Operational complexity vs code complexity — they diverge |
| **Reliability** | What is the acceptable failure rate? | Partial degradation vs total failure; blast radius |
| **Cost** | What are the resource costs now and at scale? | Cost curves that are linear now but exponential later |
| **Security** | What is the attack surface? | Authentication, authorization, data exposure, supply chain |
| **Scalability** | What grows with usage, time, and org size? | The thing that scales worst is the bottleneck |
| **Reversibility** | How hard is it to undo this decision? | Data model choices are least reversible; API contracts are hard; implementation details are easy |
| **Organizational fit** | Does this match the team's actual ability? | A design the team cannot operate is a failed design |

## Tradeoff Matrix Template

For comparing N alternatives, fill this matrix:

| Dimension | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| Latency | [rating + note] | [rating + note] | [rating + note] |
| Complexity | ... | ... | ... |
| Reliability | ... | ... | ... |
| Cost | ... | ... | ... |
| Security | ... | ... | ... |
| Scalability | ... | ... | ... |
| Reversibility | ... | ... | ... |
| Org fit | ... | ... | ... |
| **Optimizes for** | [1-line summary] | [1-line summary] | [1-line summary] |
| **Sacrifices** | [1-line summary] | [1-line summary] | [1-line summary] |

Ratings: use qualitative assessments (good/adequate/poor) with a concrete note explaining why. Numeric scores create false precision.

## Common Tradeoff Patterns

These pairs recur across system design. Recognizing the pattern accelerates analysis.

**Consistency vs. Availability** — CAP theorem and its practical implications. Strong consistency requires coordination; eventual consistency allows independent operation. Most systems need consistency for some data and availability for other data — the design question is where to draw the line.

**Simplicity vs. Flexibility** — Simple systems are easy to understand but hard to extend. Flexible systems handle change but are harder to reason about. Prefer simplicity unless you have concrete evidence that flexibility will be needed — not hypothetical future requirements.

**Latency vs. Throughput** — Optimizing for individual request speed often reduces total system throughput (and vice versa). Batching improves throughput but hurts latency. Streaming can sometimes improve both.

**Build vs. Buy** — Building gives control and fit; buying gives speed and maintained infrastructure. The hidden cost of "buy" is operational dependency. The hidden cost of "build" is maintenance burden.

**Centralization vs. Distribution** — Centralized systems are simpler to reason about but create single points of failure and scaling bottlenecks. Distributed systems are resilient but introduce coordination complexity.

**Optimization vs. Observability** — Aggressive optimization (caching, denormalization, precomputation) makes systems faster but harder to debug. Ensure every optimization comes with the monitoring needed to verify it works.

**Safety vs. Speed** — Guardrails (validation, type checking, review processes) slow development but prevent failures. The cost of skipping them is paid later, with interest.

**Present Cost vs. Future Cost** — Technical debt is borrowing from the future. Sometimes that's the right tradeoff — but name it explicitly and track the repayment plan.

## How to Use This Framework

1. **Identify the alternatives.** If you have fewer than 2, you haven't explored enough. If you have more than 5, narrow first.
2. **Fill the matrix.** Force every cell to have a rating and a note. Empty cells mean unexamined risk.
3. **Identify the dominant tradeoff.** Which 1-2 dimensions most differentiate the options? That is where the real decision lies.
4. **State what you are sacrificing.** The recommendation is incomplete without this.
5. **Ask: "What would have to be true for this to be the wrong choice?"** This is the most powerful question in tradeoff analysis. It surfaces hidden assumptions and identifies monitoring signals.
```

**Step 2: Verify the skill appears in the skill list**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: Output includes `tradeoff-analysis` with its description. This confirms the tool-skills module discovers skills from our `skills/` directory.

**Step 3: Verify the skill content loads**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Load the tradeoff-analysis skill and tell me the 8 dimensions in the comparison frame."
```
Expected: The agent loads the skill and lists all 8 dimensions (latency, complexity, reliability, cost, security, scalability, reversibility, organizational fit).

**Step 4: Commit**
```bash
git add skills/tradeoff-analysis/SKILL.md && git commit -m "feat: tradeoff-analysis skill (8-dimension comparison frame)"
```

---

### Task 3: Create the architecture-primitives skill

**Files:**
- Create: `skills/architecture-primitives/SKILL.md`

**Step 1: Create the skill file**

Create `skills/architecture-primitives/SKILL.md` with this exact content:

```markdown
---
name: architecture-primitives
description: "Catalog of reusable architectural primitives — boundaries, contracts, state machines, queues, caches, consistency models, and more. For each: what it is, when it's right, when it's WRONG. Use when selecting patterns for a design or evaluating whether a pattern fits."
---

# Architecture Primitives

An architect who knows many patterns is less useful than one who knows when each pattern is wrong.

For each primitive: what it is, when to use it, and when it will hurt you.

---

## Boundaries and Ownership

**What it is.** Drawing lines between components so each has a clear owner, clear interface, and can evolve independently.

**When it's right.** When different teams own different parts. When components have different scaling, deployment, or reliability requirements. When you need to limit blast radius.

**When it's wrong.** When the "boundary" creates more cross-boundary coordination than it prevents. When data that changes together is split across boundaries. When the team is small enough that communication overhead exceeds boundary value.

## Contracts and Interfaces

**What it is.** Explicit agreements between components about what they provide and expect — API schemas, message formats, SLAs, error codes.

**When it's right.** At every boundary. Between any components that evolve independently. When multiple consumers depend on the same provider.

**When it's wrong.** Premature contracts between components that are still being designed together. Overly rigid contracts that prevent necessary evolution. Contracts without versioning strategy.

## Source of Truth

**What it is.** For every piece of state, exactly one system is authoritative. All others are caches, replicas, or derived views.

**When it's right.** Always. Every system should be able to answer "where is the source of truth for X?"

**When it's wrong.** Never wrong as a concept — but wrong when applied as "one database for everything." Different data has different truth owners.

## Synchronous vs. Asynchronous Communication

**What it is.** Sync: caller waits for response. Async: caller sends message and continues.

**When sync is right.** When the caller genuinely cannot proceed without the response. User-facing request/response flows. Reads that must be consistent.

**When sync is wrong.** When the caller doesn't need the result to continue. When the called service is slow or unreliable. When you're creating temporal coupling between services that don't need it.

**When async is right.** When work can be deferred. When you need to absorb load spikes. When producer and consumer should be independently deployable and scalable.

**When async is wrong.** When you need an immediate, consistent response. When the added complexity of message delivery, ordering, and failure handling exceeds the benefit.

## State Machines

**What it is.** Modeling entities as having explicit states with defined transitions. An order is "placed → confirmed → shipped → delivered," not a bag of boolean flags.

**When it's right.** When an entity has a lifecycle with distinct phases. When invalid state transitions are a real failure mode. When you need to audit or replay state changes.

**When it's wrong.** When the entity's behavior doesn't meaningfully vary by state. When the state space is too large to enumerate. When you're adding a state machine to something that is really just a CRUD record.

## Queues and Buffering

**What it is.** Decoupling producers from consumers with an intermediate buffer. Smooths out rate differences and absorbs spikes.

**When it's right.** When producer rate exceeds consumer rate temporarily. When consumer failures shouldn't block producers. When you need to prioritize or reorder work.

**When it's wrong.** When the queue grows unboundedly (you've moved the problem, not solved it). When end-to-end latency matters and the queue adds unacceptable delay. When queue failure becomes a single point of failure worse than direct coupling.

## Caches

**What it is.** Storing computed or fetched results closer to the consumer to avoid repeated expensive operations.

**When it's right.** When reads vastly outnumber writes. When the source is slow and staleness is acceptable. When the working set fits in memory.

**When it's wrong.** When cache invalidation is harder than the performance problem it solves. When data changes frequently and staleness is unacceptable. When the cache becomes the de facto source of truth (and you lose the actual source). When you're caching to hide a fixable performance problem.

## Idempotency

**What it is.** Operations that produce the same result whether executed once or multiple times. "Set X to 5" is idempotent; "increment X" is not.

**When it's right.** Any operation that might be retried — API endpoints, message consumers, database migrations, deployment scripts. Critical for at-least-once delivery systems.

**When it's wrong.** Rarely wrong in principle, but the implementation cost varies. Adding idempotency keys to every endpoint when retries never happen is over-engineering.

## Retries and Backoff

**What it is.** Automatically retrying failed operations, with increasing delays between attempts to avoid thundering herds.

**When it's right.** For transient failures (network blips, temporary overload, lock contention). With idempotent operations. With jitter to prevent synchronized retry storms.

**When it's wrong.** For permanent failures (bad input, missing permissions, business rule violations). Without a circuit breaker — retrying a dead service at high rate makes everything worse. Without idempotency — retries can cause duplicate side effects.

## Backpressure

**What it is.** Signaling upstream when downstream is overloaded, so the system slows intake rather than dropping or crashing.

**When it's right.** In any pipeline where producers can outpace consumers. When dropping work is worse than slowing down. When you need graceful degradation under load.

**When it's wrong.** When the producer cannot slow down (external events, user requests that must be accepted). When backpressure propagates to the end user as unacceptable latency. In these cases, you need buffering or load shedding instead.

## Consistency Models

**What it is.** The guarantees a system provides about when writes become visible to reads. Strong consistency: reads always see the latest write. Eventual consistency: reads will eventually see the write.

**When strong consistency is right.** Financial transactions, inventory counts, anything where acting on stale data causes real harm.

**When eventual consistency is right.** Social feeds, analytics, search indexes, caches — where temporary staleness is tolerable and the performance/availability benefits are significant.

**The real question.** Not "which model?" but "which data needs which model?" Most systems need both.

## Observability

**What it is.** The ability to understand a system's internal state from its external outputs — logs, metrics, traces, health checks.

**When it's right.** Always. A system you cannot observe is a system you cannot operate.

**When it's wrong.** When observability becomes so expensive (storage, performance overhead, cognitive load) that it degrades the system it monitors. When you're collecting data nobody looks at. When alerting is so noisy that real signals are lost.

## Blast Radius Isolation

**What it is.** Designing so that failures in one component don't cascade to others. Bulkheads, circuit breakers, separate resource pools, failure domains.

**When it's right.** Between components with different reliability requirements. Around any component that depends on an external service. Around any component with a history of failures.

**When it's wrong.** When the isolation boundaries add more complexity than the failures they prevent. When the components are so tightly coupled that isolating them is pretending they're independent when they're not.
```

**Step 2: Verify the skill appears in the skill list**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: Output includes both `tradeoff-analysis` and `architecture-primitives` with their descriptions.

**Step 3: Verify the skill content loads**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Load the architecture-primitives skill and tell me when caches are the WRONG choice."
```
Expected: The agent loads the skill and explains when caches are wrong (invalidation harder than the problem, frequent changes, cache becomes de facto source of truth, hiding a fixable performance problem).

**Step 4: Commit**
```bash
git add skills/architecture-primitives/SKILL.md && git commit -m "feat: architecture-primitives skill (12 primitives with when-wrong guidance)"
```

---

### Task 4: Create the system-type-web-service skill

**Files:**
- Create: `skills/system-type-web-service/SKILL.md`

**Step 1: Create the skill file**

Create `skills/system-type-web-service/SKILL.md` with this exact content:

```markdown
---
name: system-type-web-service
description: "Domain patterns for web service architecture — API design (REST/GraphQL/gRPC), scaling, data layer, observability, failure modes, and anti-patterns. Use when designing or evaluating a web service, API, or request/response system."
---

# System Type: Web Service

Patterns, failure modes, and anti-patterns for request/response web services.

---

## API Patterns

### REST
**When to use.** Public APIs, browser-facing services, CRUD-heavy domains, when discoverability and cacheability matter.
**When to avoid.** Highly relational data with many nested queries (N+1 fetches). Real-time bidirectional communication. High-throughput internal service-to-service calls where payload efficiency matters.
**Key decisions.** Resource naming, versioning strategy (URL vs header), pagination approach, error format.

### GraphQL
**When to use.** Multiple client types needing different data shapes from the same backend. Complex, nested data relationships. When frontend teams need to iterate independently from backend.
**When to avoid.** Simple CRUD APIs. Server-to-server communication. When caching at the HTTP layer is important (GraphQL's POST-based model breaks HTTP caching). When the team doesn't have GraphQL operational expertise.
**Key decisions.** Schema-first vs code-first, query complexity limits, N+1 resolution strategy (DataLoader pattern), authorization model.

### gRPC
**When to use.** Internal service-to-service communication. When payload size and serialization speed matter. When you want strongly-typed contracts with code generation. Streaming use cases.
**When to avoid.** Browser clients (requires grpc-web proxy). When human readability of requests matters for debugging. When the team lacks protobuf experience. Public APIs (tooling ecosystem is smaller).
**Key decisions.** Proto file organization, backward compatibility discipline, deadline propagation, load balancing (L7 required for HTTP/2).

## Scaling Patterns

**Horizontal scaling.** Add more instances behind a load balancer. Requires stateless services (or externalized state). The default approach for web services. Watch for: session affinity requirements, connection pool exhaustion at the database, cache consistency across instances.

**Vertical scaling.** Bigger machines. Simpler than horizontal but has a ceiling. Right for: databases, in-memory workloads, and when horizontal scaling's coordination cost exceeds the performance benefit.

**Autoscaling.** Scale instance count based on metrics (CPU, request rate, queue depth). Essential for variable load. Watch for: cold start latency, scaling lag, minimum instance counts for availability, cost runaway from misconfigured scaling policies.

**CDN and edge caching.** Serve static and cacheable dynamic content from edge locations. Dramatically reduces latency and origin load. Watch for: cache invalidation complexity, cache poisoning, TTL tuning, varying content by headers (accept-language, authorization).

**Read replicas.** Offload read traffic from the primary database. Watch for: replication lag causing stale reads, read-after-write consistency requirements, connection routing complexity.

## Data Layer Patterns

**RDBMS (PostgreSQL, MySQL).** Default choice for structured, relational data. Strong consistency, ACID transactions, mature tooling. Scales vertically well; horizontal scaling requires sharding (hard) or read replicas (easier).

**Document stores (MongoDB, DynamoDB).** When data is naturally document-shaped, schema varies per record, or you need horizontal scaling without sharding complexity. Watch for: lack of joins, transaction limitations across documents, query patterns that don't match the data model.

**Key-value stores (Redis, Memcached).** Caching, session storage, rate limiting, leaderboards. Extremely fast for simple access patterns. Watch for: data loss on restart (unless configured for persistence), memory limits, using it as a primary datastore when it's a cache.

**Search engines (Elasticsearch, OpenSearch).** Full-text search, log aggregation, analytics on semi-structured data. Watch for: operational complexity, eventual consistency, write amplification, cluster sizing that's hard to change later.

## Observability

**Structured logging.** JSON logs with consistent fields (request_id, user_id, service, timestamp, level). Enable correlation across services. Avoid: unstructured log lines, logging sensitive data, excessive log volume without sampling.

**Distributed tracing.** Propagate trace IDs across service boundaries to reconstruct request paths. Essential when requests span multiple services. Use OpenTelemetry for vendor-neutral instrumentation.

**Metrics.** RED method for services (Rate, Errors, Duration). USE method for resources (Utilization, Saturation, Errors). Define SLOs before choosing what to measure.

**SLOs and SLIs.** Define service level objectives in terms of measurable indicators (latency P99 < 200ms, error rate < 0.1%). SLOs drive alerting, capacity planning, and error budgets. Without SLOs, you're guessing about reliability.

## Common Failure Modes

- **Cascading failures.** One slow service causes callers to queue up, exhausting their resources. Mitigation: timeouts, circuit breakers, bulkheads.
- **Connection pool exhaustion.** Database or HTTP connection pools fill up under load. Mitigation: pool sizing, connection timeouts, backpressure.
- **Thundering herd.** Cache expiry causes all instances to hit the backend simultaneously. Mitigation: jittered TTLs, request coalescing, cache warming.
- **Retry storms.** Clients retry failed requests, multiplying load on an already-stressed system. Mitigation: exponential backoff with jitter, retry budgets, circuit breakers.
- **Memory leaks.** Gradual memory growth leading to OOM kills. Mitigation: memory limits, health checks, regular restarts (if you can't find the leak).
- **Dependency failures.** External services go down. Mitigation: timeouts, fallbacks, graceful degradation, feature flags.

## Anti-Patterns

- **Distributed monolith.** Microservices that must deploy together, share databases, or make synchronous calls in long chains. You got the complexity of distribution without the benefits.
- **God service.** One service that does everything. Split by domain boundary, not by arbitrary size targets.
- **Chatty interfaces.** Many small API calls where one well-designed call would do. Increases latency, error surface, and complexity.
- **Shared mutable state.** Multiple services writing to the same database tables. Define ownership or accept the coupling.
- **Premature microservices.** Splitting into services before understanding domain boundaries. Start with a well-structured monolith; extract services when you have evidence they need independent scaling or deployment.
- **Ignoring cold starts.** Assuming services are always warm. New deployments, autoscaling events, and restarts all serve cold traffic.
```

**Step 2: Verify the skill appears in the skill list**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: Output includes `tradeoff-analysis`, `architecture-primitives`, and `system-type-web-service`.

**Step 3: Commit**
```bash
git add skills/system-type-web-service/SKILL.md && git commit -m "feat: system-type-web-service skill (API, scaling, data, observability patterns)"
```

---

### Task 5: Create the system-type-event-driven skill

**Files:**
- Create: `skills/system-type-event-driven/SKILL.md`

**Step 1: Create the skill file**

Create `skills/system-type-event-driven/SKILL.md` with this exact content:

```markdown
---
name: system-type-event-driven
description: "Domain patterns for event-driven and message-based systems — pub/sub, event sourcing, CQRS, sagas, delivery guarantees, schema evolution, and failure modes. Use when designing or evaluating systems built around events, messages, or asynchronous workflows."
---

# System Type: Event-Driven

Patterns, guarantees, and failure modes for event-driven and message-based architectures.

---

## Core Patterns

### Publish/Subscribe
**What it is.** Producers publish events to a topic; consumers subscribe and receive independently. Decouples producers from consumers.
**When to use.** Fan-out notifications, audit logging, updating multiple read models, cross-domain integration where the producer shouldn't know about consumers.
**When to avoid.** When you need a synchronous response. When ordering across topics matters. When the number of consumers is always exactly one (point-to-point is simpler).

### Event Sourcing
**What it is.** Store state as a sequence of immutable events rather than current-state snapshots. Current state is derived by replaying events.
**When to use.** Audit requirements (financial, regulatory). When you need to reconstruct historical state. When different consumers need different projections of the same data.
**When to avoid.** CRUD-heavy domains with simple state. When event schema evolution is harder than the audit benefit. When the team has no experience with eventual consistency patterns. When query patterns require complex joins across aggregates.

### CQRS (Command Query Responsibility Segregation)
**What it is.** Separate the write model (commands) from the read model (queries). Each is optimized for its access pattern.
**When to use.** When read and write patterns are drastically different (many reads, few writes, or vice versa). When you need different data shapes for different consumers. Pairs naturally with event sourcing.
**When to avoid.** Simple domains where read and write models are essentially the same. When the consistency lag between write and read models is unacceptable. When the operational complexity of maintaining two models exceeds the benefit.

### Saga Pattern
**What it is.** Coordinate a multi-step business process across services using a sequence of local transactions with compensating actions for rollback.
**When to use.** Business processes that span multiple services and need all-or-nothing semantics without distributed transactions.
**When to avoid.** When a single database transaction suffices. When the compensating actions are harder to implement correctly than the forward actions.

### Choreography vs. Orchestration
**Choreography:** Each service listens for events and decides independently what to do next. No central coordinator. Good for simple flows with few steps. Becomes opaque when flows are complex — the process is implicit in the event chain.
**Orchestration:** A central coordinator explicitly directs each step. Easier to understand, monitor, and modify complex flows. The orchestrator is a single point of failure and a coupling point.
**The real question:** How many steps? Fewer than 4: choreography is simpler. More than 4: orchestration is easier to reason about and debug.

## Delivery Guarantees

**At-most-once.** Fire and forget. Message may be lost. Use for: metrics, non-critical notifications, telemetry where gaps are acceptable.

**At-least-once.** Messages are retried until acknowledged. Messages may be delivered more than once. Use for: anything where losing a message is worse than processing it twice. **Requires idempotent consumers.**

**Exactly-once.** Each message processed exactly once. In distributed systems, this is achieved through at-least-once delivery + idempotent processing (not through the broker alone). Use for: financial transactions, inventory updates, anything where duplicates cause real harm.

**The practical reality:** Most systems use at-least-once delivery with idempotent consumers. Exactly-once semantics from the broker (e.g., Kafka transactions) have significant performance and complexity costs.

## Idempotency in Consumers

Every consumer in an at-least-once system must handle duplicate messages safely.

**Strategies:**
- **Idempotency key.** Store processed message IDs. Skip duplicates. Watch for: unbounded storage of processed IDs — add TTL-based cleanup.
- **Natural idempotency.** Design operations to be inherently idempotent. "Set balance to $100" is idempotent; "add $10 to balance" is not.
- **Optimistic concurrency.** Use version numbers or ETags. Reject stale updates. Works well with event sourcing.
- **Deduplication window.** Accept that duplicates outside the window may be processed twice. Appropriate when the cost of occasional duplicates is low.

## Schema Evolution

Events are contracts. They must evolve without breaking consumers.

**Backward compatible changes (safe):**
- Adding optional fields with defaults
- Adding new event types (existing consumers ignore them)

**Breaking changes (dangerous):**
- Removing fields
- Renaming fields
- Changing field types
- Changing the semantics of existing fields

**Strategies:**
- **Schema registry.** Validate compatibility at publish time. Prevents breaking changes from reaching consumers.
- **Versioned events.** `OrderPlaced.v1`, `OrderPlaced.v2`. Consumers subscribe to the version they understand. Producer may need to emit both during migration.
- **Consumer-driven contracts.** Consumers declare what fields they need. Breaking changes are detected before deployment.
- **Upcasting.** Transform old events to new schema at read time. Keeps the event store immutable while evolving the consumer's view.

## Dead Letter Queues

Messages that repeatedly fail processing go to a dead letter queue (DLQ) rather than blocking the main queue or being silently dropped.

**When to use.** Always in production systems. The alternative is losing messages or blocking processing.

**What to include.** Original message, error details, retry count, timestamp, source topic. Enough context to diagnose and replay.

**Operational requirements.** Monitoring on DLQ depth. Alerting when messages arrive. A replay mechanism to reprocess after fixing the consumer bug. Regular review — a growing DLQ is a symptom, not a solution.

## Common Failure Modes

- **Poison messages.** A single malformed message crashes the consumer repeatedly. Mitigation: retry limits + DLQ. Never retry infinitely.
- **Consumer lag.** Consumers fall behind producers. Mitigation: monitoring lag metrics, autoscaling consumers, partitioning for parallelism.
- **Ordering violations.** Messages arrive out of order. Mitigation: partition by entity ID (same entity, same partition), sequence numbers, reordering buffers.
- **Duplicate processing.** At-least-once delivery causes side effects to happen twice. Mitigation: idempotent consumers (see above).
- **Backpressure collapse.** Producers overwhelm consumers and the message broker. Mitigation: producer rate limiting, consumer scaling, broker capacity planning.
- **Split brain.** Competing consumers process the same message differently due to state inconsistency. Mitigation: single-partition-per-consumer assignment, leader election.

## Anti-Patterns

- **Event soup.** Publishing events for everything without a clear schema or purpose. Leads to undiscoverable, unmaintainable event flows.
- **Using events for synchronous queries.** Publishing an event and immediately polling for the result. Use request/response if you need a synchronous answer.
- **Fat events.** Putting entire entity state in every event. Events should carry what changed, not everything. Consumers that need full state should maintain their own projection.
- **Ignoring ordering.** Assuming messages arrive in order when the broker doesn't guarantee it for your partitioning scheme.
- **No dead letter strategy.** Silently dropping or infinitely retrying failed messages. Both are data loss — one is obvious, the other is subtle.
- **Shared event bus as integration layer.** Using one event bus for all services with no ownership model. Becomes a shared mutable dependency worse than a shared database.
```

**Step 2: Verify the skill appears in the skill list**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: Output includes all 4 skills created so far: `tradeoff-analysis`, `architecture-primitives`, `system-type-web-service`, `system-type-event-driven`.

**Step 3: Commit**
```bash
git add skills/system-type-event-driven/SKILL.md && git commit -m "feat: system-type-event-driven skill (pub/sub, sourcing, sagas, guarantees)"
```

---

### Task 6: Create the adversarial-review fork skill

**Files:**
- Create: `skills/adversarial-review/SKILL.md`

**Step 1: Create the skill file**

This is a **fork skill** — it spawns an isolated subagent that orchestrates 5 parallel review perspectives via the delegate tool. Follow the exact pattern from `related-projects/amplifier-bundle-skills/skills/code-review/SKILL.md`.

Create `skills/adversarial-review/SKILL.md` with this exact content:

```markdown
---
name: adversarial-review
description: "Adversarial review of a system design from 5 critical perspectives — SRE, security, staff engineer, finance, and operator. Produces a unified risk assessment. Use after a design is drafted to stress-test it before committing."
context: fork
disable-model-invocation: true
user-invocable: true
model_role: critique
---

# Adversarial Design Review

You are conducting an adversarial review of a system design. Your job is to find flaws, not to praise. You are a critic, not a collaborator.

## Input

The design to review is provided via `$ARGUMENTS`. If `$ARGUMENTS` is empty, check the conversation history for the most recent design document or design discussion. If you cannot find a design to review, respond with:

```
Provide the design to review, or reference a design document.

Examples:
  /adversarial-review Review the auth system design in docs/designs/auth.md
  /adversarial-review [paste or describe the design]
```

## Phase 1: Understand the Design

Before launching reviewers, read and understand the design:
1. If a file path is referenced, read it.
2. Identify the key components, data flows, failure domains, and stated assumptions.
3. Note what the design explicitly addresses and what it is silent on.

## Phase 2: Launch Five Review Agents in Parallel

Use the delegate tool to launch all five agents concurrently in a single message. Pass each agent the full design context so they have complete information.

### Agent 1: SRE Perspective

You are a senior SRE reviewing this design. Your question: **"How does this fail in production?"**

Evaluate:
- What are the failure modes? What is the blast radius of each?
- Where are the single points of failure?
- What happens during partial outages (one dependency down, not all)?
- How does this degrade under load? Is there graceful degradation or cliff-edge failure?
- What are the recovery procedures? Can they be automated?
- What monitoring and alerting is needed? What SLOs would you set?
- What does the on-call runbook look like for this system?

### Agent 2: Security Reviewer Perspective

You are a security reviewer. Your question: **"What is the abuse path?"**

Evaluate:
- What is the attack surface? Where does untrusted input enter?
- What are the authentication and authorization boundaries? Where are they weakest?
- What data is sensitive? How is it protected at rest, in transit, and in logs?
- What happens if a single component is compromised? How far can an attacker move laterally?
- Are there rate limiting, abuse prevention, and input validation gaps?
- What third-party dependencies introduce supply chain risk?
- What regulatory or compliance requirements does this design need to satisfy?

### Agent 3: Staff Engineer Perspective

You are a staff engineer. Your question: **"Where is the hidden complexity?"**

Evaluate:
- What looks simple but is actually hard to implement correctly?
- Where are the implicit assumptions that will break as the system evolves?
- What coupling exists that isn't visible in the architecture diagram?
- Where will the team spend most of their debugging time?
- What abstractions are at the wrong level — too concrete or too abstract?
- Is the design testable? Can components be tested in isolation?
- What technical debt is being introduced, and is it intentional?

### Agent 4: Finance Owner Perspective

You are a finance/cost owner. Your question: **"What cost curve appears later?"**

Evaluate:
- What are the variable costs that scale with usage (compute, storage, bandwidth, API calls)?
- Are any costs superlinear — growing faster than usage?
- Where are the cost cliffs (free tier limits, pricing tier jumps, reserved capacity thresholds)?
- What operational costs are hidden (team time, on-call burden, vendor management)?
- Is there vendor lock-in? What is the switching cost if pricing changes?
- What is the cost of the simplest credible alternative? Is the additional spend justified?
- What monitoring is needed to detect cost anomalies early?

### Agent 5: Operator Perspective

You are the person who operates this at 2am. Your question: **"What becomes painful at 2am?"**

Evaluate:
- Can this be deployed with zero downtime? What is the rollback procedure?
- How long does it take to diagnose a problem? Are logs, metrics, and traces sufficient?
- What manual intervention is required during normal operation? During incidents?
- How do you verify the system is healthy after a deployment or incident?
- What happens when upstream dependencies change their API or behavior?
- Is the configuration manageable? How many knobs exist, and are the defaults safe?
- What documentation does the operator need that doesn't exist yet?

## Phase 3: Synthesize Risk Assessment

Wait for all five agents to complete. Then produce a unified risk assessment:

### Critical Risks
Issues that could cause outages, data loss, security breaches, or cost blowouts. These must be addressed before the design is approved.

### Significant Concerns
Issues that increase operational burden, technical debt, or long-term cost. Should be addressed or explicitly accepted with reasoning.

### Observations
Minor issues, suggestions for improvement, or things to monitor. Not blocking.

### What the Design Gets Right
Briefly acknowledge what is well-designed. Critics who only find flaws lose credibility.

### Recommended Next Steps
The top 3-5 actions to take before proceeding with implementation, ordered by risk reduction.
```

**Step 2: Verify the skill appears in the skill list**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills."
```
Expected: Output includes all 5 skills: `tradeoff-analysis`, `architecture-primitives`, `system-type-web-service`, `system-type-event-driven`, `adversarial-review`. The adversarial-review description should mention "5 critical perspectives."

**Step 3: Commit**
```bash
git add skills/adversarial-review/SKILL.md && git commit -m "feat: adversarial-review fork skill (5-perspective parallel critique)"
```

---

### Task 7: Verify all skills and test adversarial-review

**Files:**
- None (verification only)

**Step 1: Verify all 5 skills are discoverable**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "Use the load_skill tool with list=true to show all available skills. List each skill name and its description."
```
Expected: All 5 skills listed with descriptions:
- `tradeoff-analysis` — structured tradeoff analysis methodology
- `architecture-primitives` — catalog of reusable architectural primitives
- `system-type-web-service` — domain patterns for web service architecture
- `system-type-event-driven` — domain patterns for event-driven systems
- `adversarial-review` — adversarial review from 5 critical perspectives

**Step 2: Verify an inline skill loads and is useful**

Run:
```bash
amplifier run --bundle ./bundle.md --mode single "I'm choosing between PostgreSQL and DynamoDB for an e-commerce order service. Load the relevant skills and help me analyze the tradeoff."
```
Expected: The agent loads `tradeoff-analysis` and/or `system-type-web-service` skills. The response uses the 8-dimension comparison frame and discusses tradeoffs concretely (consistency vs scalability, relational joins vs document access patterns, etc.).

**Step 3: Test the adversarial-review fork skill**

Run:
```bash
amplifier run --bundle ./bundle.md "Use the adversarial-review skill to review this design: A REST API service using PostgreSQL for user auth. JWT tokens for sessions. Single region deployment. Redis for rate limiting. Nginx reverse proxy. No message queue — all operations are synchronous."
```
Expected: The adversarial-review skill spawns a subagent in fork context. That subagent launches 5 parallel review agents (SRE, security, staff engineer, finance, operator). The final output is a unified risk assessment with critical risks, significant concerns, and observations. Key risks that should appear:
- SRE: single region = no failover, synchronous-only = cascading failure risk
- Security: JWT invalidation challenges, Redis as SPOF for rate limiting
- Staff engineer: synchronous-only will require async later (hidden complexity)
- Finance: single region is cheap now but disaster recovery gap
- Operator: single region deployment = no rolling deploys across regions

This step may take a minute or two since it spawns multiple agents. If the fork execution or delegation fails, check that the bundle has access to the delegate tool (it should via foundation). The most common issue would be the skill not being recognized as user-invocable — verify the frontmatter has `user-invocable: true`.

---

### Task 8: Final commit and structure verification

**Files:**
- Delete: `skills/.gitkeep` (no longer needed — directory has real content)

**Step 1: Remove the .gitkeep placeholder**

```bash
rm skills/.gitkeep
```

**Step 2: Verify the final directory structure**

Run:
```bash
find . -not -path './.git/*' -not -path './related-projects/*' -not -path './docs/*' -not -name '.gitignore' | sort
```

Expected output:
```
.
./behaviors
./behaviors/system-design.yaml
./bundle.md
./context
./context/instructions.md
./context/structured-design-template.md
./context/system-design-principles.md
./skills
./skills/adversarial-review
./skills/adversarial-review/SKILL.md
./skills/architecture-primitives
./skills/architecture-primitives/SKILL.md
./skills/system-type-event-driven
./skills/system-type-event-driven/SKILL.md
./skills/system-type-web-service
./skills/system-type-web-service/SKILL.md
./skills/tradeoff-analysis
./skills/tradeoff-analysis/SKILL.md
```

**Step 3: Verify git status is clean (except for .gitkeep removal)**

```bash
git status
```

Expected: Only `skills/.gitkeep` should show as deleted (and possibly untracked plan files in `docs/`). No unexpected changes.

**Step 4: Final commit**

```bash
git add -A && git commit -m "chore: checkpoint 2 complete — 5 design skills with tool-skills wiring"
```

**Step 5: Verify commit log**

```bash
git log --oneline -8
```

Expected: The checkpoint 2 commits should appear in order:
```
<hash> chore: checkpoint 2 complete — 5 design skills with tool-skills wiring
<hash> feat: adversarial-review fork skill (5-perspective parallel critique)
<hash> feat: system-type-event-driven skill (pub/sub, sourcing, sagas, guarantees)
<hash> feat: system-type-web-service skill (API, scaling, data, observability patterns)
<hash> feat: architecture-primitives skill (12 primitives with when-wrong guidance)
<hash> feat: tradeoff-analysis skill (8-dimension comparison frame)
<hash> feat: wire tool-skills module into behavior YAML
<hash> docs: checkpoint 1 implementation plan
```