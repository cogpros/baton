# baton — Autoresearch Self-Assessment Log

This is the daily-improvements log the skill produces over time. Every baton authored appends ONE row here. `skill-doctor` (or your fleet's equivalent skill auditor) re-audits every 5 batons OR every 2 weeks (whichever first), reading this log to identify mutation candidates + drift signals.

**On a fresh install,** the per-invocation table below is empty. The single example row demonstrates the column shape. Replace it with your own first authored baton's entry.

---

## Per-invocation log (newest first)

| Date | Agent | Target session topic | Baton path | Lines | Sections | Self-test (X/9) | Dry-run friction | Mutation surfaced |
|------|-------|---------------------|------------|-------|----------|-----------------|------------------|-------------------|
| YYYY-MM-DD | `<agent>` | `<one-line topic>` | `<workspace>/path/to/baton.md` | N | M | X/9 | N (`<friction-1>`; `<friction-2>`; ...) | `<mutation-pattern that could be promoted to skill body if it recurs>` |

(Example row only — delete on first real entry.)

---

## Mutation candidates (queue)

Top-of-queue items rotate into v1.X+ skill body when 2+ batons surface them independently. Seed list:

- **Auto-collected state-inventory** — a helper script that programmatically gathers this-session commit hashes per repo, files-modified list, pending memory diary path. Removes 5+ min per baton; closes Step 0 "did you actually do state-inventory?" check.
- **`baton_consumed` reverse signal** — the cold session's first action when reading a baton is to score it on a 5-question rubric (cold-start orientation time / first-friction-point / missing-info count / surprise count / self-test alignment) and emit a `baton_consumed` event. Closes the feedback loop. Operator gets reverse-empirical: did the baton actually work, or did it look good but cold-session got stuck?
- **Per-section friction heatmap** — as `baton_consumed` events accrue, identify which of the 24 template sections produce most friction. Drives focused mutations on `references/template.md` rather than full SKILL.md edits.
- **"Operator-late-add directive" structural slot** — when the operator surfaces a directive at session-end (greenlit but late), the baton needs a designated section for "verbatim operator-greenlit-but-late directive" rather than retrofitting it into an existing step.

---

## Drift canaries

If any of these trends, the skill (or template) needs review:

- Average self-test score drops below 6/9 over 5 consecutive batons
- Average dry-run friction count rises above 4 over 5 consecutive batons
- Cold-session-logged `baton_consumed` self-test alignment <5/5 over 3 batons
- Same friction-point class (e.g., "missing daemon kickstart note") recurs 3+ times

`skill-doctor` at next audit reads this section first — drift canary triggered = mandatory review round.

---

## How to read this log

Direct read:

```bash
cat <skill-dir>/baton/references/AUTORESEARCH-SELF-ASSESSMENT.md
```

Programmatic queries (if your fleet has a bus + `bus_emit` analog):

```bash
# Score histogram of recent batons
<bus-trace> --type baton_emitted | jq -r '.payload.self_test_score' | sort | uniq -c

# Average dry-run friction over last 30 days
<bus-trace> --type baton_emitted --since 30d | jq -s 'map(.payload.dry_run_friction_count) | add / length'
```

If your fleet has no bus, the log is still useful — `skill-doctor` reads this file directly at audit time, and trends emerge from manual inspection.
