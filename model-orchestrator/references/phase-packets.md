# Phase Packets

Every handoff between phases must be self-contained. Subagents start cold.

## Decision Packet (Decide → Judge/Review/Ship)

```markdown
**Goal**
<one sentence>

**Locked path** (or proposed path if pre-judge)
<path>

**Constraints**
- ...

**Success criteria**
- ...

**Out of scope**
- ...

**Evidence anchors**
- <files, logs, tests, URLs>

**Open risks**
- ...

**Reasoning level used**
<low|medium|high|xhigh>
```

## Judge Handoff

When invoking `judge-decision`, pass:

```markdown
**Decision**
<keep / modify / replace / investigate question>

**Current path**
<from Decision Packet>

**Alternatives known**
- ...

**Constraints & success criteria**
- ...

**Evidence**
- ...

**Why judge is required**
<contested / high blast radius / user asked>
```

Consume the standard `judge-decision` output (Verdict, Decision Status, Next Check). Only continue to Ship on Settled or Provisional (with eyes open). Stop on Blocked unless user overrides.

## Review Packet (→ Fable/Claude)

```markdown
**Role**
Independent reviewer. Do not rubber-stamp. Find material misses.

**Locked / proposed path**
<...>

**What to protect**
<correctness, UX, operability, …>

**Evidence**
<...>

**Return exactly**
- Material risks (ordered)
- Assumptions that look weak
- Concrete fixes (minimal)
- Ship-blocker? yes/no + why
- Confidence low|medium|high
```

## Ship Packet (→ Codex, writable)

```markdown
**Role**
Implementer. Execute the locked plan. Do not reopen settled decisions unless a blocker makes the plan impossible.

**Goal**
<...>

**Locked plan**
1. ...
2. ...

**Repo / cwd**
<path>

**Touch these areas**
- <paths>

**Do not touch**
- <paths>

**Acceptance checks**
- <commands or observable outcomes>

**Write access**
Granted. Prefer smallest diff that meets acceptance.

**Return exactly**
- Files changed
- Checks run + results
- Deviations from plan (if any)
- Remaining work
```

## Parallel Ship Splits

Only when ownership is disjoint:

```markdown
**Workstream**
<name>
**Owns**
<paths>
**Must not edit**
<other workstream paths>
**Shared contract**
<APIs/types already locked>
```
