# Design Review Questions

A Socratic question bank for evaluating designs. These questions complement the
system design principles and adversarial perspectives -- they provide the
specific probes that surface design flaws early.

Apply these questions at every design phase and review step. They are organized
by evaluation lens, not by workflow phase -- each lens may be relevant at
multiple points.

---

## 1. Comprehending the Existing Design

Before proposing anything new, understand why the existing design was chosen.
Every design is an artifact of its designer's tradeoffs, constraints, and mental
model. Skipping this step leads to proposals that re-introduce problems the
original design already solved.

- Why was this designed this way? What tradeoffs did the original designer make?
- Is there some reason we can decipher about why this approach was chosen over
  alternatives?
- What is the mental model? Is it human-facing or system-facing?
- What does the choice of X over Y tell us about the constraints the designer
  was working under?
- What problems does the current design solve that aren't obvious at first
  glance?

**When to apply:** Step 1 of any review. Phase 1 of any design that modifies an
existing system. Before generating candidate architectures.

---

## 2. Design Integrity

Every proposal exists in the context of prior decisions. A proposal that
contradicts previous design decisions without explicitly reconciling them
creates architectural drift -- the system gradually becomes a patchwork of
inconsistent ideas.

- Is this proposal consistent with the previous design decisions we have made?
  If not, how would we reconcile them?
- Who does this belong to? Does it really belong here?
- Are we changing the responsibility of an existing component? If so, is that
  change intentional and acknowledged?
- Does this introduce a second way to do something the system already does?
  If so, what is the migration path for the first way?
- Would someone unfamiliar with this proposal's history understand why the
  system works this way?

**When to apply:** When evaluating any proposal that modifies existing systems.
During candidate architecture generation. During refinement.

---

## 3. Developer Experience (DX)

A design that developers cannot understand, use correctly, or debug is a failed
design regardless of its architectural elegance. DX is not a luxury -- it
determines whether the system will be used as intended.

- How many concepts must a developer hold in their head to use this correctly?
  Are we increasing or decreasing that number?
- Does naming clearly indicate purpose and how each named thing fits into the
  overall system? Is naming consistent?
- Is this more or less challenging for developers to understand the overall
  mental model?
- Does added complexity indicate a need for tooling, rather than that the design
  is wrong? Simple things with good tools are often better than complex things
  that are "easier" but harder to work with.
- What does the error experience look like? When a developer makes a mistake,
  does the system help them find it?
- Can a new team member understand this from the documentation, or does it
  require tribal knowledge?

**When to apply:** When evaluating any design's usability. During tradeoff
analysis (DX is a real cost dimension). During adversarial review.

---

## 4. Design Simplicity (Socratic)

Simplicity is not an aesthetic preference. It is a design constraint that
reduces failure surface, cognitive load, and maintenance cost. These questions
force the conversation back to fundamentals when complexity is creeping in.

- Isn't there a simpler way to do this? What if we just... ?
- Let's take a step back -- what are we actually trying to accomplish here?
- Are there existing computer science concepts related to this work that might
  inform our design? Are we reinventing something that already has a name?
- Does this amount of logic indicate that we should consider revising the
  architecture? If so, what concept have we uncovered that we didn't know
  before? How would this new concept affect the rest of the system?
- Are we mixing two concepts here? Aren't these actually separate concerns?
- Are these different things or the same thing? What are the unique qualities
  that make them different? What are they actually, then?

**When to apply:** When generating candidates. During refinement. When any
component exceeds expected complexity.

---

## 5. Ecosystem and Tooling Consistency

Diverging from ecosystem conventions has a real cost: developers must learn
non-standard approaches, existing tooling doesn't work, and the community
can't help. The burden of proof is on the divergence, not on convention.

- Are there existing ecosystem concepts (language conventions, standard library
  patterns, framework idioms) that could inform this design?
- Are there existing popular libraries that would replace much of this work?
  What is the cost of building vs. adopting?
- Does this diverge from standard ecosystem practices? If so, what do we gain
  that justifies the divergence?
- Will a developer familiar with the ecosystem's conventions be surprised by
  this design? If so, is the surprise justified?

**When to apply:** When evaluating technology choices. During candidate
architecture generation. When building tooling or developer-facing APIs.