# Baton Template — 24 Required Sections

Copy this template when authoring a new baton. Fill in `<placeholders>` with session-specific content. Aim for 200-400 lines for single-step work; up to 600+ for multi-step / multi-decision sessions.

---

```markdown
# Next-Session Prompt — <One-line topic>

**One-line goal:** <what this session should do>. **Do NOT execute <out-of-scope work> in this session.**

**Spec/anchor file paths:** `<paths>`.

**Architectural anchor:** `<paths>`. Read this before the spec.

**Macro context anchor (if applicable):** `<path to master baton / North Star spec / sprint plan / CONTEXT.md>`. The macro doc that orients this session in the larger arc. Skip this line if no such doc exists for this work.

---

## Macro context: where this session fits (CONDITIONAL — skip if no macro doc exists)

> Include this section ONLY if a master baton, North Star spec, sprint plan, or CONTEXT.md exists for the overarching project / sprint / direction. Skip if not — don't fabricate one.

**Read the macro doc FIRST** (`<path>`, ~N min read). It is the macro-context anchor for this multi-session arc and exists so cold sessions know **why** they're doing the work, not just what.

**Why this matters for THIS session:**
- **Subsystem placement:** This session lives in <subsystem/workstream> per `<macro-doc-section>`.
- **Closure criterion / success metric:** Contributes to <which metric> per `<macro-doc-section>`. Does NOT contribute to <other metrics>.
- **Effort fit:** This session's <N-Mh> is one slice of the larger <total-Mh> sprint estimate per `<macro-doc-section>`.
- **Operator-decision pointer:** <Open Q from macro doc that this session might surface against, with cite>.
- **War-story durability:** <which war stories from the macro doc apply here vs. don't>.

**Skim-vs-full reading guidance:**
- Read in full: <§sections that map to this session>
- Skim: <§sections that don't apply>
- Skip unless surprised: <pointer-only sections>

---

## 60-second session-context recap (read FIRST)

**Where we are:** <1-paragraph orientation: prior session, what shipped, where we are in the arc>

**Operator-stated direction (load-bearing — preserve in any future arc):**
- *"<verbatim quote>"* — context
- *"<verbatim quote>"* — context

**War-story durability:** <3-5 sentences naming the recurring defect classes from this arc that the next session must NOT repeat>

**This session's N-step shape:**
1. <step>
2. <step>
3. <step>

If you complete (N) and operator authorizes, you ship <work> this session. **Do not do more than this.**

---

## ⛔ DO NOT (read this first)

1. **DO NOT execute <out-of-scope> in this session.** <reason>
2. **DO NOT skip <required step>.** <reason>
3. **DO NOT modify <other agent's workspace> without explicit operator approval BEYOND what's in spec §X.**
4. **DO NOT scope-cut.** Operator has been clear (<timestamp>: *"<quote>"*). <list of decisions to honor>
5. **DO NOT skip the §1 verification ledger re-check.** <war-story citation>
6. <other DO-NOTs specific to this work>

---

## 0. Session opening (cold start)

You are <agent name>. Prior session ended <timestamp> with <brief>.

### 0.1 Read first (in order)

1. `<path>` — <one-line description>
2. `<path>` — <description>
... (typically 8-12 docs)

### 0.2 Send a Discord ack to operator

After reads, ack to `<channel>` (chat_id `<id>`):

> <exact ack text>

### 0.3 System state expected at session start

| Component | Expected state |
|---|---|
| <component> | <verifiable state> |
| ... | ... |

If any of these don't match, STOP and ask operator before proceeding.

### 0.4 Tools required (verify available)

```bash
which <tool1> <tool2> <tool3>
```

---

## 1. Run §1 verification ledger (~5 min)

```bash
bash <verification-script-path> 2>&1 | tee /tmp/<session-tag>-verification-$(date -u +%Y%m%dT%H%M%SZ).txt
```

Compare against the spec's §1 baseline. **Action on drift:** edit spec inline, bump changelog, commit, then proceed.

---

## 2. <PRISM synthesis status / other gating step>

<If applicable: PRISM archive path + verdict + Tier-1 count + OD flag count>

---

## 3. <Apply mechanical fixes / operator-decision gate / etc>

### 3.1 Apply mechanical fixes inline (~X min)

Per <archive> §<section>, N mechanical fixes:

1. **<finding-id>:** <what to fix, where>. Concrete steps:
   ```bash
   # explicit commands here, not "compose this yourself"
   ```

2. **<finding-id>:** <what to fix>. <embedded SQL / jq / bash>
   ...

### 3.2 Operator-decision gate (post N flags to Discord, BATCH)

Post to Discord after fixes applied:

> <exact Discord post template with each OD flag + trade-off context + recommended answer>

Wait for operator response. Apply per their direction → bump spec → commit.

### 3.3 Step 0 execution gate

After v2.x ready, post:

> <exact authorization request>

Wait for explicit authorization. Do NOT execute Step 0 without it.

---

## 4. Step 0 execution (~Xh focused — N substeps)

### 4.1 Step 0.1 — <substep title>

<files to create/modify, schema, embedded commands, verification, commit message template>

### 4.2 Step 0.2 — <substep title>

<same shape>

...

### 4.N Step 0 acceptance criteria (full set)

- [ ] <boolean check>
- [ ] <boolean check>
... (typically 8-15 items)

---

## 5. Session-end protocol (~15-20 min)

1. Memory diary at `<path>` — narrative prose with confidence tags
2. Update LAST_SESSION.md
3. Update memory/MEMORY.md with new top row
4. Emit `session_end` bus event:
   ```bash
   bash <emit-script> <agent> session_end "..." '{"...":...}' completion
   ```
5. Discord wrap-up summary in <channel> with commit hashes + carry-forwards

---

## 6. Carry-forwards (next-next-session)

After this session ships:
- **<work item>** (~Xh) — <description>
- **<work item>** — <description>

**Plus prior-arc carry-forwards (NOT in scope for next session):**
- <item>

---

## 7. Discord status update cadence

Post to `<channel>` (chat_id `<id>`) at these milestones:
- After §0 reads complete (cold-start ack)
- After §1 verification ledger run
- ... (per-session-shape)
- Final session-end summary

---

## 8. File path summary

| Path | Purpose | Status |
|---|---|---|
| `<path>` | <purpose> | Committed `<hash>` / GREENFIELD / EXISTS |
| ... | ... | ... |

---

## 9. Session-completion signal

You are DONE with this session when ALL of the following are true:

- [ ] <gating completion criterion>
- [ ] <gating completion criterion>
- [ ] memory diary written
- [ ] LAST_SESSION updated
- [ ] MEMORY row added at top
- [ ] session_end bus event emitted
- [ ] Discord wrap-up posted

If <X conditional>, you're done after <subset>.

---

## 10. Decision tree if you get stuck

```
Stuck? Ask: which kind?
├── Spec is unclear → re-read spec section, then ask operator
├── PRISM reviewer disagreement → cross-validate, weigh, pick majority OR escalate
├── Implementation choice not in spec → check verification ledger, follow precedent
├── Operator-decision question → push to Discord, wait
├── State surprise → STOP, ask operator
├── Hard rule conflict → re-read CLAUDE.md hard rules, default to NOT acting
└── Time pressure → there shouldn't be any; document and stop if confused
```

---

## 11. War stories durable across sessions (preserve in any future arc)

These are the load-bearing operational lessons from this arc:

1. **<defect-class war story>** — <2-3 sentences naming the defect, the closure, the lesson>
2. **<defect-class war story>** — <description>
... (typically 5-8 stories per arc)

---

## 12. Hard-rule reminders

- <agent-specific hard rules>
- <universal hard rules from CLAUDE.md / <fleet-hard-rules>>

---

## 13. Bonus context

**This is a continuation arc.** Prior session covered:
- <list of major milestones>

**Operator's stated direction (verbatim quotes):**
- *"<quote>"* — context
- *"<quote>"* — context

**Total prior-session commits to be aware of:**

| Repo | Commits |
|---|---|
| <repo> | `<hash>`, `<hash>` |

---

## 14. Self-test before declaring complete

You're DONE with this session when ALL of the following return TRUE:

- [ ] `<verification command>` runs cleanly
- [ ] `<git log check>` shows fresh commits
- [ ] <other boolean check>
... (typically 8-12 items)

The self-test exists because <war-story citation>. Self-test mirrors that lesson.
```
