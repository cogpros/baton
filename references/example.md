# Canonical Baton Example — Shape and Density Reference

This is a template-shape reference. The skill's design was extracted from a single ~580-line, 14-section continuation prompt that proved cold-session-executable in practice — this doc preserves that shape so future batons can match it.

**Stats reference:**
- 580 lines / ~46 KB (a substantive multi-step session; 200-400 lines is fine for single-step work)
- 14 numbered top-level sections + a 60-second context recap above them
- Drafted, then audited (5 gaps closed), then dry-run as a cold session (4 friction-points closed) before shipping

---

## What a strong baton at this density carries

A 580-line baton of this shape includes:

- **Cold-start reads** — ordered list of 8-12 anchor docs with EXISTS/GREENFIELD status per file. Path-explicit, never "see prior session."
- **PRISM (or equivalent review) synthesis** — if a review pass produced findings, the baton points at the archive + summarizes the verdict + counts Tier-1 findings (so cold session knows what's already been ratified vs. what's open).
- **Mechanical fixes inline** — N specific edits with embedded SQL / bash / jq filters / hunk locations. Apply-able from the prompt alone OR a pointer to the archive's full reasoning.
- **Operator-decision flags** — N flags surfaced but not yet resolved, each with full trade-off context + recommended answer. Cold session can assemble the Discord post (or equivalent surface) from the flag spec alone.
- **Step-by-step execution** — per-step files, schemas, embedded commands, verification commands, commit message templates. Each step ends with "how do I know this worked" as a 1-line bash/curl/sqlite3.
- **War stories durable across sessions** — load-bearing operational lessons from THIS session that the next session must inherit. E.g., recurring defect classes, kickstart-after-edit lessons, scope-cut-hedging episodes.
- **Self-test** — a boolean checklist that defines "done" for the cold session, mirroring the war-story lessons.

---

## What "cold-session-executable" actually means

A cold session opens the baton with **zero context** from the prior session. They read it top-to-bottom and:

1. Within 60 seconds, know **where am I, what just happened, what do I do next** (the recap section earns its name).
2. Within 5 minutes, have **read the cold-start anchor docs** and run the verification ledger.
3. From there, every step's "do X" instruction includes the **exact command, file path, and verification check** — no composing required.

If a cold session ever has to ask "what does this mean?" or "where's the script?" the baton failed an audit point. The skill's Step 3 (cold-session dry-run) exists specifically to catch those gaps before shipping.

---

## Why 580 lines is the upper end, not the floor

Smaller sessions should produce smaller batons. Heuristic:

| Session shape | Baton size |
|---|---|
| Single bug fix, known root cause | 100-200 lines |
| Single feature, well-scoped | 200-400 lines |
| Multi-step execution + decision flags | 400-600 lines |
| Architectural reframe + N mechanical fixes + M operator-decision flags + multi-substep execution | 580+ lines |

The cost of a long-but-explicit baton is 5 minutes of cold-session reading. The cost of a short-but-vague baton is hours of misdirection. Bias to longer when in doubt — the dry-run will catch theater.

---

## Use as a template

When authoring your own baton:

1. Copy `references/template.md` as your starting structure (24 required sections).
2. Fill in your session-specific content per Step 1 of the SKILL.md procedure.
3. After drafting, run the self-audit (Step 2) — score against the 9-question rubric.
4. Cold-session dry-run (Step 3) — walk through as if you've never seen the work before; catch every "compose this yourself" gap.
5. Apply gap-closure fixes (Step 4).
6. Cite the baton path in your session-end wrap-up (Step 5).
7. Emit `baton_emitted` event (or your fleet's equivalent observability signal) + append a row to `references/AUTORESEARCH-SELF-ASSESSMENT.md` (Step 6).

The shape above is what passes self-test ≥7/9. Anything less is iteration territory — don't ship until the bar's met.
