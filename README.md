# Model Orchestrator

Stay in one chat. Route each phase to the model class that is best at it.

- **Decide** — Grok / judgment: frame, cut scope, pick the path
- **Judge** — `judge-decision` when the path is contested
- **Review** — Fable / Claude: careful risks, taste, second-order misses
- **Ship** — Codex with **write access by default**: implement the locked plan

The skill auto-routes from the ask, spins sub-agents at the right reasoning level, and only falls back to CLI adapters (`call-claude`, `call-codex`) when the host cannot spawn cross-model subagents.

## Why It Exists

Switching apps to get Grok judgment, Claude review, and Codex execution breaks flow. This skill keeps orchestration in-session: classify the work, emit a routing card, run the needed phases, hand clean packets between them.

## What It Does

1. Classifies the ask into `Decide` / `Judge` / `Review` / `Ship`.
2. Picks a reasoning level (`low` → `xhigh`) per phase.
3. Runs `judge-decision` when the decision is contested or high blast-radius.
4. Spawns Fable/Claude-class review subagents when careful analysis is needed.
5. Spawns Codex-class ship subagents with write access against a locked plan.
6. Reports what ran on which class, what landed, and what remains.

Companion skills are optional adapters, not hard dependencies:

- `judge-decision` — contested decision engine (invoked by this skill when needed)
- `call-claude` — Review transport when in-host Claude/Fable subagents are unavailable
- `call-codex` — Ship transport when in-host Codex subagents are unavailable

## When To Use It

- "Decide this architecture, then implement it."
- "Review for missed risks, then ship."
- "Orchestrate this across the right models."
- Any multi-phase ask where one model class should not own the whole loop.

Skip it for tiny single-pass edits.

## Install

Clone, symlink into your skills home (or runtime dirs), and re-link:

```bash
git clone https://github.com/uncoooloj/model-orchestrator-skill.git
ln -s "$(pwd)/model-orchestrator-skill/model-orchestrator" ~/skills/shared/model-orchestrator
~/skills/link-skills.sh
```

Or link straight into the runtimes:

```bash
ln -s "$(pwd)/model-orchestrator-skill/model-orchestrator" ~/.claude/skills/model-orchestrator
ln -s "$(pwd)/model-orchestrator-skill/model-orchestrator" ~/.codex/skills/model-orchestrator
```

The repo uses the public `-skill` suffix; the installed skill name stays `model-orchestrator`.

Invoke with:

```text
Use $model-orchestrator to auto-route this: judge if contested, review if needed, then ship with Codex.
```

Or just describe multi-phase work — the skill is meant to auto-route from the ask.

## Layout

```text
model-orchestrator-skill/
  README.md
  model-orchestrator/
    SKILL.md
    agents/openai.yaml
    references/
      model-classes.md
      phase-packets.md
```

## Notes

- Ship defaults to **writable**. Plan-only / read-only only when requested.
- Do not simulate cross-model output. If a class is missing, say so and degrade.
- Exact Cursor model slugs live in `references/model-classes.md` and will need occasional updates as host names change.
