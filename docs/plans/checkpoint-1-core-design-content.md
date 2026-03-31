# Checkpoint 1: Core Design Content — Implementation Plan

> **Execution:** Use the subagent-driven-development workflow to implement this plan.

**Goal:** Create a working `system-design-intelligence` Amplifier bundle that loads system design methodology into root context and produces structured design output.

**Architecture:** Thin bundle on foundation — includes `amplifier-foundation` for all infrastructure (tools, orchestrator, hooks, agents), adds our own context files establishing system design methodology. A behavior YAML wires context into the root session. The bundle.md is the entry point with YAML frontmatter (includes, session config) and a markdown body with `@mentions` for context injection.

**Tech Stack:** Amplifier bundle system (YAML frontmatter + markdown), `amplifier run --bundle ./bundle.md` for testing.

---

### Task 1: Create bundle.md skeleton

**Files:**
- Create: `bundle.md`

**Step 1: Create the bundle entry point**

Create `bundle.md` with this exact content:

```markdown
---
bundle:
  name: system-design-intelligence
  version: 0.1.0
  description: Systems design methodology for agentic development — structured design output with tradeoff analysis, multiscale reasoning, and failure mode coverage.

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: system-design-intelligence:behaviors/system-design
---

# System Design Intelligence

You have access to a systems design methodology that produces structured, rigorous architectural output.

@system-design-intelligence:context/system-design-principles.md
@system-design-intelligence:context/structured-design-template.md
@system-design-intelligence:context/instructions.md

---

@foundation:context/shared/common-system-base.md
```

**Step 2: Commit**

```bash
git add bundle.md && git commit -m "feat: bundle.md skeleton with foundation include"
```

---

### Task 2: Create behavior YAML skeleton

**Files:**
- Create: `behaviors/system-design.yaml`

**Step 1: Create the behavior file**

Create `behaviors/system-design.yaml` with this exact content:

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

This follows the exact pattern from `related-projects/amplifier-bundle-superpowers/behaviors/superpowers-methodology.yaml` — a behavior YAML with `bundle` metadata and `context.include` entries. For Checkpoint 1, we only need context wiring. Agents, modes, skills, hooks, and tools will be added in later checkpoints.

**Step 2: Commit**

```bash
git add behaviors/system-design.yaml && git commit -m "feat: behavior YAML with context wiring"
```

---

### Task 3: Create placeholder context files and verify bundle loads

**Files:**
- Create: `context/system-design-principles.md`
- Create: `context/structured-design-template.md`
- Create: `context/instructions.md`

**Step 1: Create minimal placeholder files**

These need to exist so the bundle resolves all `@mentions` and `context.include` references without errors.

Create `context/system-design-principles.md`:
```markdown
# System Design Principles

Placeholder — will be populated with core design methodology.
```

Create `context/structured-design-template.md`:
```markdown
# Structured Design Template

Placeholder — will be populated with output template.
```

Create `context/instructions.md`:
```markdown
# System Design Instructions

Placeholder — will be populated with standing orders.
```

**Step 2: Verify the bundle loads**

Run:
```bash
amplifier run --bundle ./bundle.md "Say hello and confirm you can see system design principles in your context." --mode single
```

Expected: The agent responds. No errors about missing files or unresolved references. The agent should mention seeing something about "System Design Principles" in its context.

**Step 3: Commit**

```bash
git add context/ && git commit -m "feat: placeholder context files, bundle loads successfully"
```

---

### Task 4: Write system-design-principles.md

**Files:**
- Modify: `context/system-design-principles.md`

**Step 1: Write the core methodology**

Replace the placeholder content of `context/system-design-principles.md` with this:

```markdown
# System Design Principles

These principles guide how you approach system design problems. They are not a checklist to recite — they are thinking tools that shape how you reason about systems.

---

## 1. Model the System Before Solving

Never jump to a solution. First build a map of the system:

- **Goals**: What must this system accomplish? What does success look like?
- **Constraints**: What is fixed? (budget, timeline, team size, existing systems, regulations)
- **Actors**: Who and what interacts with this system?
- **Interfaces**: Where are the boundaries? What crosses them?
- **Feedback loops**: Where does output become input? Where can behavior amplify or dampen?
- **Failure modes**: What breaks? What happens when it breaks? What is the blast radius?
- **Time horizons**: What matters now vs. in 6 months vs. in 3 years?

A weak designer jumps to answers. A strong designer first builds a map.

## 2. Reason at Multiple Scales

Good architects hold several layers in mind simultaneously:

| Layer | Question |
|-------|----------|
| **Principle** | What must be true for this system to be correct? |
| **Structural** | What are the components, and how do they interact? |
| **Operational** | What happens at runtime? What are the edge cases? |
| **Evolutionary** | How does this change over time? What becomes painful at 10x scale? |

Every design should be examined at all four layers. A design that is elegant structurally but operationally fragile is not a good design.

## 3. Analyze Tradeoffs, Don't Mimic Best Practices

Real architecture is choosing what to sacrifice. Most agents overfit to "good-sounding patterns." Instead, compare options using a fixed frame:

| Dimension | Question |
|-----------|----------|
| Latency | How fast must it respond? |
| Complexity | How many concepts must be held in mind? |
| Reliability | What is the acceptable failure rate? |
| Cost | What are the resource costs now and at scale? |
| Security | What is the attack surface? |
| Scalability | What grows with usage, time, and org size? |
| Reversibility | How hard is it to undo this decision? |
| Organizational fit | Does this match the team's actual ability? |

For every design, answer: **"What does this optimize for, and what does it sacrifice?"**

That question is architecturally catalytic.

## 4. Reason Causally

Systems thinkers trace propagation, not just structure:

- If X changes, what downstream breaks?
- What is coupled that appears independent?
- Where are delays, hidden state, or nonlinearities?
- What incentives create local optimization but global harm?

For any significant design decision, identify:
- **First-order effects**: the direct, intended consequences
- **Second-order effects**: what changes because of those consequences
- **Unintended consequences**: what breaks, degrades, or shifts unexpectedly

## 5. Treat Simplicity as a Constraint, Not an Aesthetic

Simplicity is not the opposite of systems thinking — it is the discipline that prevents systems thinking from becoming systems theater.

**Prefer the simplest design whose failure modes are acceptable.** Not the simplest possible. Not the most scalable imaginable. The simplest that is adequate.

Evaluate designs on these simplicity dimensions:

- **Conceptual**: How many ideas must be held in mind?
- **Interface**: How many contracts and exceptions exist?
- **Operational**: How hard is it to deploy, debug, and recover?
- **Evolutionary**: How hard is it to extend later?
- **Organizational**: Does this match the team's actual ability?

A design that is "clean" in theory but exceeds team cognition is not simple.

**Think broadly about the system, but design narrowly.** Use the minimum number of concepts, components, and abstractions needed to meet the requirements with acceptable risk.

---

## Standing Behaviors

These are always active when reasoning about design:

- **Surface assumptions.** Make implicit assumptions explicit. They are where designs fail.
- **Distinguish local vs. global optimization.** A locally optimal choice can be globally harmful.
- **Identify tight coupling.** Ask what changes together and what should be independent.
- **Look for single points of failure.** Every SPOF is a design decision, whether intentional or not.
- **Prefer reversible decisions early.** Defer irreversible choices until you have evidence.
- **Separate mechanism from policy.** The "how" should be independent of the "what" and "when."
- **Name unknowns.** What don't you know? What would change the design if you learned it?
```

**Step 2: Verify the bundle loads with the new content**

Run:
```bash
amplifier run --bundle ./bundle.md "What are the five core design principles you follow?" --mode single
```

Expected: The agent lists the five principles (model the system, multiscale reasoning, tradeoff analysis, causal reasoning, simplicity as constraint) in its own words. This confirms the context file is being loaded and influencing output.

**Step 3: Commit**

```bash
git add context/system-design-principles.md && git commit -m "feat: core system design principles (five upgrades + standing behaviors)"
```

---

### Task 5: Write structured-design-template.md

**Files:**
- Modify: `context/structured-design-template.md`

**Step 1: Write the output template**

Replace the placeholder content of `context/structured-design-template.md` with this:

```markdown
# Structured Design Template

For any nontrivial design problem, produce output in this structure. Not every section needs equal depth — calibrate to the problem's complexity. But do not skip sections without stating why.

---

## 1. Problem Framing
What is actually being asked? Restate the problem in terms of goals, constraints, and scope. Distinguish what the user said from what the system needs.

## 2. Explicit Assumptions
What are you assuming about requirements, scale, team, timeline, existing systems, or constraints? List every assumption. These are where designs fail silently.

## 3. System Boundaries
What is inside the system? What is outside? Where are the interfaces between this system and external systems, users, or services?

## 4. Components and Responsibilities
What are the major components? What does each one own? What is the source of truth for each piece of state?

## 5. Data and Control Flows
How does data move through the system? What triggers actions? What are the critical paths?

## 6. Risks and Failure Modes
What breaks? What happens when it breaks? What is the blast radius? What is the recovery path? Distinguish between likely failures and catastrophic failures.

## 7. Tradeoffs
What does this design optimize for? What does it sacrifice? Use the fixed frame: latency, complexity, reliability, cost, security, scalability, reversibility, organizational fit.

## 8. Recommended Design
The design you recommend, with reasoning for why this option over the alternatives.

## 9. Simplest Credible Alternative
The simplest design that could plausibly work. If this is very far from the recommended design, explain why the additional complexity is justified.

## 10. Migration and Rollout Plan
How do you get from here to there? What can be done incrementally? What requires a cutover? What is the rollback plan?

## 11. Success Metrics
How do you know this design is working? What do you measure? What thresholds indicate problems?
```

**Step 2: Commit**

```bash
git add context/structured-design-template.md && git commit -m "feat: structured design output template (11-section format)"
```

---

### Task 6: Write instructions.md

**Files:**
- Modify: `context/instructions.md`

**Step 1: Write the standing orders**

Replace the placeholder content of `context/instructions.md` with this:

```markdown
# System Design Instructions

<STANDING-ORDER>

## Detect Design Requests

When the user asks about system design, architecture, technology selection, service boundaries, data modeling, scaling strategy, or any question where the answer is "it depends on tradeoffs" — you are in design territory.

When you detect a design request, suggest entering `/design` mode for a structured process. If `/design` mode is not available, follow the structured design template directly.

## Generate Alternatives

For any nontrivial design problem, generate at least **3 candidate architectures**:

1. **Simplest viable** — the minimum design that meets core requirements
2. **Most scalable** — optimized for growth in usage, data, or team size
3. **Most robust** — optimized for reliability, failure tolerance, and operational simplicity

Then **recommend one with explicit reasoning**. Good architects explore the design space before converging.

## Always Answer the Catalytic Question

For every design you produce or evaluate, answer:

> **"What does this design optimize for, and what does it sacrifice?"**

If you cannot answer this clearly, the design is not yet understood.

## Methodology Calibration

Not every task needs full design treatment. Match depth to complexity:

| Task | Approach |
|------|----------|
| Architecture, new system, major refactor | Full structured design template |
| Technology selection, integration decision | Tradeoff analysis with alternatives |
| Small feature within existing architecture | Quick assessment: does it fit? What breaks? |
| Implementation question | Answer directly — this is not a design problem |
| Clarification or explanation | Answer directly |

Don't produce a 10-section design document for a question that needs a paragraph. But don't give a paragraph when the question deserves structured analysis.

</STANDING-ORDER>
```

**Step 2: Commit**

```bash
git add context/instructions.md && git commit -m "feat: standing orders (design detection, alternatives, calibration)"
```

---

### Task 7: Verify bundle with a real design prompt

**Files:** None — verification only.

**Step 1: Run a real design prompt**

Run:
```bash
amplifier run --bundle ./bundle.md "I need to design an authentication system for a new web application. We expect about 10,000 users in the first year, growing to 100,000. We need to support email/password login, OAuth with Google and GitHub, and eventually MFA. The team is 3 backend engineers." --mode single
```

**Step 2: Verify structured output**

Check that the response includes:
- [ ] Problem framing (restates the problem, not just echoes it)
- [ ] Explicit assumptions listed
- [ ] Multiple architectures (at least 2-3 alternatives)
- [ ] Tradeoff analysis (what does it optimize for, what does it sacrifice)
- [ ] Failure modes or risks mentioned
- [ ] A recommendation with reasoning
- [ ] Mention of a simplest credible alternative

If the output is a generic "here's how to build auth" without structure, alternatives, or tradeoffs, the context files are not being loaded properly. Debug by checking:
```bash
amplifier run --bundle ./bundle.md "List the design principles and standing orders you have in your context." --mode single
```

**Step 3: Run a calibration test — non-design question**

Run:
```bash
amplifier run --bundle ./bundle.md "What is a good way to reverse a string in Python?" --mode single
```

Expected: A direct answer. NOT a 10-section design document. This confirms methodology calibration is working — the agent should not apply full design treatment to a simple implementation question.

---

### Task 8: Final commit

**Files:** None — wrap up.

**Step 1: Verify all files are committed**

```bash
git status
```

Expected: Clean working tree. If there are uncommitted changes, commit them:

```bash
git add -A && git commit -m "feat: checkpoint 1 complete — core design content bundle"
```

**Step 2: Verify final bundle structure**

```bash
find . -not -path './.git/*' -not -path './related-projects/*' -not -path './docs/*' -not -name '.gitignore' | sort
```

Expected output should show:
```
.
./behaviors
./behaviors/system-design.yaml
./bundle.md
./context
./context/instructions.md
./context/structured-design-template.md
./context/system-design-principles.md
```

That's the complete Checkpoint 1 bundle: entry point, behavior wiring, and three context files establishing the system design methodology.
