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
