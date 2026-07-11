---
name: model-orchestrator
description: >-
  Auto-route work across model classes without switching apps: Decide (Grok/judgment),
  Review (Fable/Claude careful analysis), Synthesize (Gemini long-context), Ship (Codex
  execution with write access). Spins up sub-agents with the right model family and
  reasoning level, runs judge-decision when contested, and uses call-claude / call-codex /
  call-grok / call-gemini when available. Use when the user wants orchestration,
  multi-model routing, decide-then-ship, review-then-implement, cross-model subagents,
  Fable/Claude review, Gemini synthesis, Codex execution, Grok judgment, or to stay in
  one chat while using the right model for each phase.
---

# Model Orchestrator

## Overview

Stay in one session. Route each phase of work to the model class that is best at it, at the reasoning level the stakes require. Do not make the user hop apps.

This skill is the orchestrator. `judge-decision` is the contested-decision engine. Optional adapters `call-claude`, `call-codex`, `call-grok`, and `call-gemini` are transports when the host cannot spawn cross-model subagents directly. Use whichever adapters are available; never simulate a missing family.

For model-class defaults and reasoning ladders, read `references/model-classes.md`. For handoff packet shapes, read `references/phase-packets.md`.

## Core Rule

Classify → route → execute → hand off a clean packet. Never simulate a different model family. If the preferred class is unavailable, say so, use the best real fallback, and keep going.

## Auto-Route Triggers

Apply this skill when the ask implies multi-phase work or a clear class mismatch with the current model, including:

- Ambiguous product/architecture that must become a locked plan, then code
- "Decide, then implement" / "review this, then ship"
- Contested path that needs `judge-decision` before execution
- Careful second-order review (taste, risk, missed edges) before writing
- Long implementation grind after the plan is clear
- Explicit `$model-orchestrator` / "orchestrate this"

Skip orchestration for tiny single-step asks the current model can finish in one pass.

## Roles

| Role | Class | Job |
| --- | --- | --- |
| **Decide** | Grok / judgment-first caller | Frame, cut scope, pick path, state assumptions |
| **Judge** | `judge-decision` workflow | Contested keep/modify/replace/investigate; consults every available cross-family external |
| **Review** | Fable / Claude | Careful risks, taste, second-order misses, instruction fidelity |
| **Synthesize** | Gemini | Long-context / sprawling-evidence compression when the bundle is large or mixed |
| **Ship** | Codex | Implement against a locked plan with write access |

Sol (GPT generalist) is a Ship fallback when Codex is unavailable, not the default executor. Gemini is not a Ship replacement when Codex is available.

## Reasoning Levels

Pick the lowest level that matches stakes. Escalate when wrong answers are expensive.

| Level | When | Prefer |
| --- | --- | --- |
| **low** | Trivial edits, renames, obvious follow-ups | Fast / mini / composer |
| **medium** | Normal implementation, clear plan | Codex medium / Sol medium / Sonnet medium |
| **high** | Architecture, careful review, product judgment | Fable high / Opus high / Grok high |
| **xhigh** | Contested, irreversible, or high blast-radius | Opus xhigh / Grok xhigh + `judge-decision` |

Exact host slugs live in `references/model-classes.md`. Prefer **family + level**, then resolve to the best available slug on this host.

## Workflow

1. **Classify the ask.**
   - Name phases needed: `Decide`, `Judge`, `Review`, `Synthesize`, `Ship` (any subset, in order).
   - Name reasoning level per phase.
   - Emit a one-screen **Routing Card** before spending tokens on the wrong class.

2. **Decide (if needed).**
   - Current agent (or a Grok-class subagent / `call-grok` when available) frames constraints, success criteria, and the proposed path.
   - If the path is contested, ambiguous with high cost of error, or the user asks to challenge/compare/decide → run **`judge-decision`** and treat its verdict as the locked decision packet. `judge-decision` will consult every available cross-family external (`call-claude` / `call-codex` / `call-grok` / `call-gemini`).
   - Do not Ship on a Blocked or unsettled contested decision unless the user explicitly overrides.

3. **Review (if needed).**
   - Spawn a Fable/Claude-class subagent (or `call-claude` when available) with the decision packet.
   - Ask for missed risks, weaker assumptions, and concrete fixes — not a rewrite of the plan unless the plan is wrong.
   - Fold accepted findings into the locked plan before Ship.

4. **Synthesize (if needed).**
   - Use when the evidence bundle is large, mixed, or needs compression across many artifacts.
   - Spawn Gemini (or `call-gemini` when available). Do not use Gemini as the default careful Review when Claude/Fable is available.

5. **Ship (default write access).**
   - Spawn a Codex-class subagent (or `call-codex` with `workspace-write` when available) against the locked plan.
   - Default sandbox/write mode: **writable**. Use read-only only when the user asks for plan-only / review-only execution.
   - Pass a self-contained Ship packet: goal, constraints, files, acceptance checks, out-of-scope.

6. **Close the loop.**
   - Verify acceptance checks when cheap.
   - Report what ran on which class/level, what changed, and what remains.

## Routing Card

Emit this before phase work (keep it short):

```markdown
**Routing**
Phases: <Decide | Judge | Review | Synthesize | Ship>
Level: <low | medium | high | xhigh> (per phase if split)
Decide: <caller | Grok-class | skip>
Judge: <judge-decision | skip> — reason
Review: <Fable/Claude | skip> — reason
Synthesize: <Gemini | skip> — reason
Ship: <Codex write | Sol write | skip> — reason
Adapters available: <call-grok / call-claude / call-codex / call-gemini / none>
```

## Subagent Spawning

### Prefer host subagents with explicit model class

When the host can spawn subagents with a model override (e.g. Cursor `Task`):

- Set `subagent_type` to the best fit (`generalPurpose`, `explore`, `shell`, …).
- Set `model` to the resolved slug for the role + reasoning level.
- For Ship: do **not** force `readonly: true`. Writes are the default.
- For Review/Judge/Synthesize evidence gathering: `readonly: true` is preferred.
- Give each subagent a **self-contained prompt** (no reliance on parent chat memory).
- Run independent Review/Decide/Synthesize probes in parallel when they do not depend on each other.
- Ship only after Decide/Judge/Review/Synthesize outputs are folded into one locked plan.

### Fallback adapters

If model-overridden subagents are unavailable, use whichever adapters exist:

- Decide → `call-grok` when installed (and caller is not already Grok-family, or isolation is required).
- Review → `call-claude` when installed.
- Synthesize → `call-gemini` when installed.
- Ship → `call-codex` with write (`workspace-write` / equivalent) when installed.
- Judge external passes → follow `judge-decision` (consult every available cross-family adapter); do not invent cross-model output.

If no adapter works, complete the phase with the caller model and label the missing class as unavailable.

## When To Invoke `judge-decision`

Use it inside orchestration when any of these hold:

- Two or more plausible paths with real tradeoffs
- User asks to challenge, compare, sanity-check, or appoint a judge
- Ship would be expensive to reverse and confidence is not high
- Review found a material objection that reopens the decision

Pass `judge-decision` a compact evidence bundle, not the whole chat. After it returns Settled or Provisional, continue orchestration from that verdict. If Blocked, stop Ship and report the missing check.

## Phase Order Rules

- Never Ship before a required Decide/Judge completes.
- Review may run before Decide only for reconnaissance; it does not lock the plan.
- Multiple Ship subagents are allowed for parallelizable workstreams with clear file ownership; otherwise one Ship owner.
- Do not re-litigate a Settled judge verdict unless new evidence appears.

## Output Shape

```markdown
**Orchestration Result**
Routing: <phases + levels + classes used>
Decision: <locked path or "n/a">
Judge: <verdict/status or "not used">
Review: <key accepted findings or "skipped">
Ship: <what landed + verification or "skipped">
Remaining: <next action or "none">
```

## Failure Modes

- Do not hop the user to another app when a subagent or adapter can do the phase in-session.
- Do not pretend Fable/Claude/Codex/Grok/Gemini ran if they did not.
- Do not give Ship a vague vibe prompt — require a locked plan packet.
- Do not default Ship to read-only.
- Do not skip `judge-decision` on contested high-stakes paths to "save time."
- Do not let Review rewrite scope silently; surface material plan changes before Ship.
- Do not treat Sol as interchangeable with Codex when Codex is available.
- Do not treat Gemini as interchangeable with Claude Review or Codex Ship when those are available.
- Do not hard-depend on companion skills; degrade gracefully and name the gap. Prefer whichever of `call-grok` / `call-claude` / `call-codex` / `call-gemini` are present.
