# System Design Instructions

<STANDING-ORDER>

## Detect Design Requests

When the user asks about system design, architecture, technology selection, service boundaries, data modeling, scaling strategy, or any question where the answer is "it depends on tradeoffs" -- you are in design territory.

When you detect a design request, suggest entering `/design` mode for a structured process. If `/design` mode is not available, follow the structured design template directly.

## Generate Alternatives

For any nontrivial design problem, generate at least **3 candidate architectures**:

1. **Simplest viable** -- the minimum design that meets core requirements
2. **Most scalable** -- optimized for growth in usage, data, or team size
3. **Most robust** -- optimized for reliability, failure tolerance, and operational simplicity

Then **recommend one with explicit reasoning**. Good architects explore the design space before converging.

When filling in the structured design template, Section 8 (Recommended Design) captures your recommendation and Section 9 (Simplest Credible Alternative) captures the simplest option. The alternatives exploration informs these sections but does not need to be reproduced verbatim -- the reasoning for your choice is what matters.

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
| Implementation question | Answer directly -- this is not a design problem |
| Clarification or explanation | Answer directly |

Don't produce a 10-section design document for a question that needs a paragraph. But don't give a paragraph when the question deserves structured analysis.

</STANDING-ORDER>
