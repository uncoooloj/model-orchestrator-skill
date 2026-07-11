# Model Classes & Reasoning Levels

Resolve **family + level** to the best slug available on the current host. Slugs drift; families do not.

## Families

| Family | Also called | Best at | Default role |
| --- | --- | --- | --- |
| **Grok** | Cursor Grok, xAI | Judgment, framing, cutting scope, pushback | Decide |
| **Fable / Claude** | Fable, Anthropic, Claude | Careful analysis, taste, missed risks, instruction fidelity | Review |
| **Gemini** | Google Gemini | Long-context synthesis, sprawling mixed evidence | Synthesize |
| **Codex** | OpenAI Codex coding agent | Tool-loop execution, repo edits, grind-to-done | Ship |
| **Sol** | GPT generalist | Broad competent execution | Ship fallback |

## Cursor Task slug preferences (resolve in order)

Update this list when host model names change. Always pick the first available match.

### Decide — Grok class

| Level | Prefer |
| --- | --- |
| medium | `grok-4.5-fast-xhigh` (or current Grok caller) |
| high / xhigh | `grok-4.5-fast-xhigh` |

CLI fallback: `call-grok` when available.

### Review — Fable / Claude class

| Level | Prefer |
| --- | --- |
| medium | `claude-4.6-sonnet-medium-thinking`, `claude-sonnet-5-thinking-high` |
| high | `claude-fable-5-thinking-high`, `claude-opus-4-8-thinking-high` |
| xhigh | `claude-opus-4-7-thinking-xhigh`, `claude-opus-4-8-thinking-high` |

CLI fallback: `call-claude` when available.

### Synthesize — Gemini class

| Level | Prefer |
| --- | --- |
| medium / high | host Gemini slug if present; else `call-gemini` |

CLI fallback: `call-gemini` when available. Use for large/mixed evidence compression, not as default Review or Ship.

### Ship — Codex class (write by default)

| Level | Prefer |
| --- | --- |
| low | `gpt-5.1-codex-mini`, `composer-2.5-fast` |
| medium | `gpt-5.3-codex`, `gpt-5.1-codex-max-medium` |
| high | `gpt-5.1-codex-max-medium`, `gpt-5.3-codex` |

CLI fallback: `call-codex` with `workspace-write` when available. Read-only only when user asks plan-only.

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

| Host capability | Decide | Review | Synthesize | Ship |
| --- | --- | --- | --- | --- |
| Cursor Task + `model` | Grok slug or caller | Fable/Claude slug | Gemini slug if any | Codex slug, writable |
| `call-grok` | Grok CLI | — | — | — |
| `call-claude` | — | Claude CLI | — | — |
| `call-gemini` | — | — | Gemini CLI | — |
| `call-codex` | — | — | — | Codex CLI write |
| All adapters present | `call-grok` | `call-claude` | `call-gemini` | `call-codex` write |
| No adapters | caller (label gaps) | caller (label gaps) | caller (label gaps) | caller (label gaps) |

Use whatever subset is installed. Do not invent missing families.

## Anti-collusion

- Do not send your preferred verdict into a Review/Judge/Synthesize subagent as the answer to confirm.
- Send evidence + question + constraints; ask for independent judgment.
- For contested paths, `judge-decision` owns the debate protocol and consults every available cross-family external.
