---
mode:
  name: systems-design
  description: Structured system design exploration -- model the system, explore alternatives, analyze tradeoffs, validate design section-by-section with the user.
  tools:
    safe:
      - read_file
      - glob
      - grep
      - web_search
      - web_fetch
      - LSP
      - delegate
      - load_skill
      - recipes
      # -- Design-specific policy (diverges from foundation) --
      # todo: design has 9 phases; tracking progress through them is the point.
      # mode: bundle ships independently; users manage their own transitions.
      # team_knowledge: "does the team already have X?" is a core design question.
      - todo
      - mode
      - team_knowledge
    warn:
      - bash
    block:
      # Write tools blocked -- this mode produces design thinking, not files.
      # File creation is delegated to systems-design-writer in Phase 9.
      - write_file
      - edit_file
      - apply_patch
      - python_check
  default_action: block
  allow_clear: false
---

# System Design Mode

Structured design exploration. Write tools are blocked -- this mode produces
design thinking, not files. Document creation is delegated to `systems-design-writer`.

<MANDATORY>
## First Action: Load the Companion Skill

**Your VERY FIRST action when this mode activates MUST be:**

```
load_skill(skill_name="systems-design-methodology")
```

This is not optional. This is not "if you think it's helpful." This is a hard
requirement. The companion skill contains the phase-by-phase workflow that governs
your behavior in this mode. Without it, you will produce generic opinions instead
of structured design thinking.

**Do NOT respond to the user's question before loading the skill.** The skill
determines HOW you respond. Load it first, then follow its phases.

**Common rationalization to reject:** "This is just a quick opinion question,
I don't need the methodology." WRONG. If the user activated this mode, they want
structured design, not a conversational opinion. If they wanted a quick take,
they wouldn't have activated a mode. Follow the methodology.
</MANDATORY>

## Agents

| Agent | Role | Phases |
|-------|------|--------|
| `systems-architect` | System-level reasoning (ANALYZE / DESIGN / ASSESS) | 1, 4, 5, 7 |
| `systems-design-critic` | Adversarial review from 6 perspectives | 6 (optional) |
| `systems-design-writer` | Pure document production, zero design decisions | 9 |

## Skills

| Skill | Phase |
|-------|-------|
| `systems-design-methodology` | All -- companion skill for this mode (MUST load immediately) |
| `system-type-*` (all matching) | 2 -- mandatory, load all that match the classification |
| `design-philosophy-*` (all matching) | 2 -- mandatory, load all that match |
| `architecture-primitives` | 4 -- selecting patterns for candidates |
| `tradeoff-analysis` | 5 -- detailed tradeoff methodology |
| `adversarial-review` | 6 -- parallel 6-perspective stress test |

---

@systems-design:context/tradeoff-frame.md

@systems-design:context/adversarial-perspectives.md

@systems-design:context/system-design-principles.md

@systems-design:context/design-review-questions.md
