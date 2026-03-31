# Structured Design Template

For any nontrivial design problem, produce output in this structure. Not every section needs equal depth -- calibrate to the problem's complexity. But do not skip sections without stating why.

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
