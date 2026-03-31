# System Design Principles

These principles guide how you approach system design problems. They are not a checklist to recite -- they are thinking tools that shape how you reason about systems.

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

Simplicity is not the opposite of systems thinking -- it is the discipline that prevents systems thinking from becoming systems theater.

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
