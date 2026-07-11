# Model Orchestrator

Stay in one chat. Route each phase to the model class that is best at it.

- **Decide** — Grok / judgment: frame, cut scope, pick the path
- **Judge** — `judge-decision` when contested (consults every available cross-family external)
- **Review** — Fable / Claude: careful risks, taste, second-order misses
- **Synthesize** — Gemini: long-context / sprawling evidence compression
- **Ship** — Codex with **write access by default**

Falls back to CLI adapters when available: `call-grok`, `call-claude`, `call-gemini`, `call-codex`. Never simulates a missing family.

## Companion adapters (optional)

- `judge-decision`
- `call-grok`
- `call-claude`
- `call-gemini`
- `call-codex`

## Install

```bash
git clone https://github.com/uncoooloj/model-orchestrator-skill.git
ln -s "$(pwd)/model-orchestrator-skill/model-orchestrator" ~/skills/shared/model-orchestrator
~/skills/link-skills.sh
```

Invoke:

```text
Use $model-orchestrator to auto-route this: judge if contested, review/synthesize if needed, then ship with Codex.
```
