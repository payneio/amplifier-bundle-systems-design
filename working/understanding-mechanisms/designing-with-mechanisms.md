# Bundle Developer's Guide to Amplifier Mechanisms

## Introduction

Amplifier exposes [seven composable primitives](mechanisms/README.md) for building agent systems: 

- [modes](mechanisms/modes.md)
- [recipes](mechanisms/recipes.md)
- [skills](mechanisms/skills.md)
- [agents](mechanisms/agents.md)
- [hooks](mechanisms/hooks.md)
- [tools](mechanisms/tools.md)
- [content](mechanisms/content.md)

They overlap by design. The goal is not to pick one, but to assign **clear ownership** and **correct attachment points**, then compose them through **behaviors** — the YAML wiring layer that assembles mechanisms into bundles.

> **Use combinations freely. Avoid duplicated authority.**

## Attachment Points

> **Where does it bind, and how long does it live?**

| Mechanism | Attaches to | Lifetime | State location | Authored as | Token impact |
|-----------|-------------|----------|----------------|-------------|-------------|
| Mode | Session | Togglable, ephemeral re-injection | `session_state["active_mode"]` | `.md` with YAML frontmatter | Ephemeral per-turn cost while active; never stored |
| Recipe | Workflow execution | Per-invocation, resumable | `state.json` on disk | `.yaml` | Each step runs in isolated context; only text output propagates |
| Agent | Sub-session | One-shot or resumable | Kernel session (in-memory) | `.md` with YAML frontmatter | Context sink -- parent pays ~200-500 tokens for summary |
| Skill | Task pattern | On-demand (inline or forked) | None (stateless) or fork session | `SKILL.md` in a directory | L1 visibility: ~1K tokens/turn ephemeral; L2 load: compactable tool result |
| Hook | Lifecycle event | Session lifetime | Module state + `session_state` | Python module | Ephemeral injections: per-turn cost; non-ephemeral: stored permanently |
| Tool | LLM decision | Session lifetime | Module state | Python module | Results stored in message history; primary compaction target |
| Content | Context window | Per-injection | None (consumed by LLM) | `.md` files | Fixed cost every turn; immune to compaction |

---

## The Wiring Layer: Behaviors

Behaviors are YAML files that compose mechanisms into reusable capability packages. They are how mechanisms become available in a session.

```yaml
# Example: behaviors/my-feature.yaml
includes:
  - bundle: some-dependency:behaviors/base
hooks:
  - module: hooks-mode
    source: git+https://github.com/microsoft/amplifier-bundle-modes@main#subdirectory=modules/hooks-mode
    config:
      search_paths: ["@my-bundle:modes"]
tools:
  - module: tool-mode
    source: git+https://github.com/microsoft/amplifier-bundle-modes@main#subdirectory=modules/tool-mode
    config:
      gate_policy: warn
  - module: tool-skills
    config:
      skills:
        - "@my-bundle:skills"
agents:
  include:
    - my-bundle:agents/specialist
context:
  include:
    - my-bundle:context/instructions.md
```

**Key pattern:** A behavior wires hooks, tools, agents, and context into a single composable unit. Bundles compose behaviors via `includes:`. This is the layer between individual mechanisms and the final bundle.

---

## What Each Mechanism Should Own

| Concern | Primary owner | Why not elsewhere |
|---------|---------------|-------------------|
| Session-wide constraints | **Mode** | Hooks enforce deterministically; skills/recipes can't guarantee session-wide policy |
| Multi-step workflow ordering | **Recipe** | Checkpointing and approval gates require persistent state |
| Task-specific expertise | **Skill** | Portable, discoverable, loadable on demand without sub-session overhead |
| Complex sub-task reasoning | **Agent** | Needs isolated context, own tool surface, potentially different model |
| Guaranteed lifecycle behavior | **Hook** | Fires deterministically, not subject to LLM judgment |
| LLM-decided capabilities | **Tool** | Model chooses when to invoke based on schema |
| Reference knowledge | **Content** | Passive — no execution, just context |

> **Authority Rule:** one concern, one source of truth.

---

## Choosing the Right Mechanism

Ask in order:

1. Must it always apply, regardless of what the LLM decides? **Hook** (code-decided enforcement) or **Mode** (policy overlay)
2. Is it a multi-step process with checkpoints? **Recipe**
3. Is it reusable task knowledge, loadable on demand? **Skill**
4. Does it need external system access? **Tool**
5. Does it need isolated reasoning with its own context? **Agent**
6. Is it just reference information? **Content**

**Distinguishing hooks from modes:** Hooks are Python modules — use them when you need programmatic logic (parse responses, compute values, call APIs on lifecycle events). Modes are declarative `.md` files — use them when you need tool policy tiers and injected guidance without writing code.

**Distinguishing skills from agents:** Inline skills are cheaper (no sub-session overhead) and portable (Agent Skills spec). Fork skills converge with agents but require less infrastructure (a directory vs. a bundle). Use agents when you need: resumable sessions, specific tool composition, or deep context isolation.

---

## Failure Modes (Misplaced Attachment)

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **Policy leakage** (too low) | Safety rules in skills or recipe prompts — inconsistent enforcement | Move to a **mode** with tool policy tiers |
| **Workflow hidden in skills** | Step sequences encoded in skill bodies — brittle, no checkpointing | Move to a **recipe** |
| **Expertise smearing** (too high) | Heuristics duplicated across recipe step prompts | Centralize in a **skill** |
| **Skill hypertrophy** | A skill that spawns agents, manages state, enforces policy | Split: policy to mode, workflow to recipe, expertise stays in skill |
| **Capability fiction** | APIs described in context but not exposed as tools | Wire as a **tool** (or MCP resource) |
| **Cognitive overload** | Single agent prompt does everything | Decompose into **delegated agents** with focused concerns |
| **Knowledge entanglement** | Docs copy-pasted into agent prompts, skills, and recipe steps | Centralize as **content** files, reference via `@mention` |
| **Missing hook, using mode** | You need programmatic logic (compute, API calls) but used a mode | Write a **hook** module — modes are declarative only |

---

## Context Window Economics

Every mechanism has a token cost. Understanding that cost -- and which costs are fixed vs compactable -- is essential for designing bundles that stay effective across long sessions.

### The Three-Layer Model

The context window sent to the LLM on each turn has three distinct layers with different persistence and compaction behavior:

```
Context Window (e.g., 200K tokens)
+-- LAYER 1: System Prompt (never compacted)
|   Instruction body + all @mentioned files + context.include files
|   Re-read from disk every turn. Fixed cost. Immune to compaction.
|
+-- LAYER 2: Message History (compactable)
|   User/assistant messages, tool calls and results.
|   Accumulates during session. Compacted when budget pressure hits 92%.
|   8-level cascade: truncate tool results -> remove old messages -> stub.
|
+-- LAYER 3: Ephemeral Injections (per-turn, not stored)
|   Skills visibility list, active mode body, hook injections.
|   Appended AFTER compaction runs. Never enters message history.
|   Cost repeats every turn but never compounds.
|
+-- RESERVED
    4096 tokens safety margin + 50% of max_output_tokens + 800 for compaction notice
```

**Budget formula:** `context_window - (max_output_tokens x 0.5) - 4096`

Compaction triggers at 92% of budget and targets 50%. The system prompt sits outside the compactable portion entirely -- it's a permanent tax.

### How Each Mechanism Affects the Budget

| Mechanism | Layer | Compactable? | Cost model |
|-----------|-------|-------------|------------|
| **Content** (@mentions) | System prompt | Never | Fixed per turn. Every @mentioned file's full text is included in every LLM call. |
| **Content** (context.include) | System prompt | Never | Same as @mentions. Accumulates across behavior composition. |
| **Agents** (delegation) | Message history | Yes | Parent pays only ~200-500 tokens per delegation (tool call + summary result). Child absorbs all exploration tokens in its own context. |
| **Skills** (L1 visibility) | Ephemeral | Never | ~1,000-1,500 tokens per turn (all skill names + descriptions). Scales with number of composed skills. |
| **Skills** (L2 load) | Message history | Yes | Full skill content returned as tool result. Subject to normal compaction (truncated, then removed). |
| **Skills** (fork) | Isolated session | N/A | Like agents: runs in child context, parent sees only the result. |
| **Hooks** (ephemeral injection) | Ephemeral | Never | Per-turn cost while hook fires. Not stored. Disappears after each LLM call. |
| **Hooks** (persistent injection) | Message history | Yes | Stored permanently via `context.add_message()`. Subject to compaction. |
| **Modes** | Ephemeral | Never | Full mode markdown body injected every turn while active. Stops immediately on deactivation. |
| **Recipes** | Isolated per step | N/A | Each step runs in a fresh child session. Only text output propagates via `{{variables}}`. No conversation history carries between steps. |
| **Tools** (results) | Message history | Yes, first | Tool results are the primary compaction target. Truncated to 250 chars before messages are removed. Last 5 results protected. |

### Design Implications

**The fixed-cost trap.** Content files and context.include entries are the most dangerous budget item because they're invisible -- always present, never compacted, and they accumulate across behavior composition. A bundle that composes three behaviors each adding 2K tokens of context has a 6K token permanent floor before any conversation starts.

**Compaction favors tool results over conversation.** The 8-level cascade truncates tool results first (Levels 1-2), then removes old messages (Level 3+). This means heavy tool use is more sustainable than heavy context file use -- tool results eventually get compacted away, but @mentioned content never does.

**Ephemeral injections add up silently.** Skills visibility (~1K tokens) + an active mode (~500-2K tokens) + status context hooks can easily consume 3-4K tokens per turn that never appear in the message history but still eat into the available window.

**Agents are the primary context management tool.** The context sink pattern isn't just about specialization -- it's about token economics. Delegating a 20-file exploration to an agent costs the parent ~400 tokens. Doing it directly costs ~20K tokens that persist in message history until compacted.

### Rules of Thumb for Bundle Authors

1. **Content files: budget per byte.** Every token in an @mentioned file is paid on every turn. Keep always-present content concise and high-value. Move detailed reference material to skills (loaded on demand) or agent @mentions (loaded in child context only).

2. **Skills over content for reference material.** A 3K-token methodology guide as a content file costs 3K tokens x every turn. As a skill, it costs ~100 tokens/turn (L1 visibility) plus 3K tokens once when loaded (L2), and that load is compactable.

3. **Fork skills over inline skills for heavy work.** If a skill's execution involves reading many files or producing large outputs, use `context: fork` to run it in an isolated child session.

4. **Modes should be concise.** Mode content is injected ephemerally every turn. A 1,000-word mode file costs ~750 tokens per turn for every turn it's active. Keep mode guidance focused; move detailed methodology to a companion skill loaded once on mode activation.

5. **Watch behavior composition.** Each `includes:` that adds `context.include` entries increases the permanent system prompt floor. Audit the total context footprint when composing multiple behaviors.

6. **Prefer ephemeral over persistent hook injections.** Use `ephemeral=True` for hook context injections unless the information must survive across turns. Ephemeral injections don't accumulate in message history.

---

## Practical Example: This Bundle

The `systems-design` bundle composes all seven mechanisms:

| Mechanism | Instance | Role |
|-----------|----------|------|
| **Modes** | `system-design`, `design-review` | Enforce read-only tool policy during design; block file writes until design is complete |
| **Recipes** | `architecture-review` (staged, 2 approval gates), `design-exploration` (parallel foreach), `codebase-understanding` (sequential) | Structure multi-step design workflows with checkpointing |
| **Agents** | `systems-architect` (reasoning), `systems-design-critic` (critique), `systems-design-writer` (writing) | Isolated sub-sessions with different model roles and focused concerns |
| **Skills** | `adversarial-review` (fork, 5 parallel agents), `tradeoff-analysis`, `architecture-primitives`, `system-type-web-service`, `system-type-event-driven` | Task expertise loaded on demand — the fork skill spawns its own review agents |
| **Hooks** | `hooks-mode` (from modes bundle) | Mode enforcement and context injection |
| **Tools** | `tool-mode`, `tool-skills` (via behavior YAML wiring) | LLM-accessible mode switching and skill loading |
| **Content** | `system-design-principles.md`, `structured-design-template.md`, `instructions.md` | Design philosophy, templates, and standing orders injected into root session |

All wired together through `behaviors/system-design.yaml`, which composes the modes behavior, configures the skills tool with local skill directories, declares the three agents, and includes the context files.

## Mental Model

| Mechanism | Verb | Triggered by |
|-----------|------|-------------|
| **Mode** | constrains | User or agent activates |
| **Recipe** | organizes | User or agent invokes |
| **Agent** | thinks | Parent delegates |
| **Skill** | knows how | Agent loads on demand |
| **Hook** | observes | Lifecycle event (deterministic) |
| **Tool** | acts | LLM decides to call |
| **Content** | informs | Resolution at load/injection time |

## Closing

When mechanism choice gets confusing, ask three questions:

> **1. "Where should this attach, and who triggers it?"**

- Attach at the **highest level where it remains valid** -- session-wide constraints go in modes, not skills.
- **Code-decided behavior** (must happen every time) goes in hooks. **LLM-decided behavior** (model judges when) goes in tools.
- **Recipes own ordering**, skills own expertise, agents own reasoning. Don't smear one into another.

> **2. "What does this cost the context window, and when is that cost paid?"**

- Content that's needed **every turn** belongs in the system prompt (content files) -- but keep it concise, because it's a permanent tax that compaction can never recover.
- Content that's needed **once or occasionally** belongs in skills (loaded on demand, compactable) or agent @mentions (isolated in child context).
- Content that's needed **only during a phase** belongs in modes (ephemeral, stops when deactivated) rather than content files (permanent).
- Heavy exploration belongs in **agents** (context sinks) -- the parent pays hundreds of tokens instead of tens of thousands.

> **3. "Does this scale with session length?"**

- System prompt content and ephemeral injections are **fixed per turn** -- they don't grow, but they never shrink either.
- Tool results and conversation history **accumulate** but are **compactable** -- the system manages the pressure automatically.
- Agent delegations are the **only mechanism that doesn't accumulate** -- each delegation is a bounded cost that gets compacted like any other tool result.

The first question -- attachment point plus trigger -- resolves *which* mechanism. The second and third -- context cost and scaling behavior -- resolve *how* to use it without starving the session of working memory.
