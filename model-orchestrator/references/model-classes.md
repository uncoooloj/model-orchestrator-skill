# Model Classes & Reasoning Levels

Resolve **family + level** to the best slug available on the current host. Slugs drift; families do not.

## Families

| Family | Also called | Best at | Default role |
| --- | --- | --- | --- |
| **Grok** | Cursor Grok, xAI | Judgment, framing, cutting scope, pushback | Decide |
| **Fable / Claude** | Fable, Anthropic, Claude | Careful analysis, taste, missed risks, instruction fidelity | Review |
| **Codex** | OpenAI Codex coding agent | Tool-loop execution, repo edits, grind-to-done | Ship |
| **Sol** | GPT generalist | Broad competent execution | Ship fallback |

## Cursor Task slug preferences (resolve in order)

Update this list when host model names change. Always pick the first available match.

### Decide — Grok class

| Level | Prefer |
| --- | --- |
| medium | `grok-4.5-fast-xhigh` (or current Grok caller) |
| high / xhigh | `grok-4.5-fast-xhigh` |

CLI fallback: `call-grok`.

### Review — Fable / Claude class

| Level | Prefer |
| --- | --- |
| medium | `claude-4.6-sonnet-medium-thinking`, `claude-sonnet-5-thinking-high` |
| high | `claude-fable-5-thinking-high`, `claude-opus-4-8-thinking-high` |
| xhigh | `claude-opus-4-7-thinking-xhigh`, `claude-opus-4-8-thinking-high` |

CLI fallback: `call-claude`.

### Ship — Codex class (write by default)

| Level | Prefer |
| --- | --- |
| low | `gpt-5.1-codex-mini`, `composer-2.5-fast` |
| medium | `gpt-5.3-codex`, `gpt-5.1-codex-max-medium` |
| high | `gpt-5.1-codex-max-medium`, `gpt-5.3-codex` |

CLI fallback: `call-codex` with `workspace-write` (or host equivalent). Read-only only when user asks plan-only.

### Ship fallback — Sol

| Level | Prefer |
| --- | --- |
| medium | `gpt-5.6-sol-medium` |
| high | `gpt-5.5-medium`, `gpt-5.4-medium` |

Use Sol only when Codex-class is unavailable.

## Level selection heuristics

- **low**: mechanical, reversible, <~15 min of human work equivalent
- **medium**: normal feature/bugfix with a clear plan
- **high**: API/schema design, subtle bugs, product copy/UX, multi-file architecture
- **xhigh**: contested decision, migration, security-sensitive, hard-to-revert Ship

When unsure between two levels, pick the higher one for Decide/Review and the lower one for Ship once the plan is locked.

## Host capability matrix

| Host capability | Decide | Review | Ship |
| --- | --- | --- | --- |
| Cursor Task + `model` | Grok slug or caller | Fable/Claude slug | Codex slug, writable |
| `call-grok` only | Grok CLI | caller | caller |
| `call-claude` only | caller | Claude CLI | caller or `call-codex` |
| `call-codex` only | caller | caller | Codex CLI write |
| All three adapters | `call-grok` | `call-claude` | `call-codex` write |
| No adapters | caller (label gaps) | caller (label gaps) | caller (label gaps) |

## Anti-collusion

- Do not send your preferred verdict into a Review/Judge subagent as the answer to confirm.
- Send evidence + question + constraints; ask for independent judgment.
- For contested paths, `judge-decision` owns the debate protocol — this skill only decides when to invoke it.
