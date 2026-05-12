# Master Baton Pattern

Use this reference when the operator asks for a "master baton" or when a workstream spans many sessions, compactions, repos, or subprojects.

A master baton is not a normal next-session execution prompt. It is the macro orientation document that future session-level batons cite first.

## Trigger signals

- Operator says "master baton".
- The work will span multiple sessions/context compactions.
- There is an overarching vision doc, North Star doc, sprint plan, or CONTEXT.md that needs to stay load-bearing.
- The operator says the shape is still unknown (for example: "it's something else and we don't know exactly what that is yet") and wants a durable orientation artifact before implementation.
- Multiple repos or branches are involved and dirty state/branch residue could mislead future sessions.
- The near-term next action is only one slice of a larger gated arc.

## Recommended location

Prefer the docs directory of the owning project/workstream, not a per-session `data/` path:

- `~/projects/<repo>/docs/<workstream>/<workstream>-master-baton.md`
- `<workspace>/agents/<agent>/docs/<workstream>-master-baton.md`

If the owning docs namespace has an index/README, add the master baton there so future sessions discover it.

## Required shape

**10 Required + 4 Conditional sections (revised 2026-05-10 per PRISM `master-baton-audit-and-registry/2026-05-10-review.md` T7).** The prior "16 required" was over-prescribed — audit across 3 active master batons showed every baton skipped at least 1-3 sections without harm. Required vs Conditional now reflects what's empirically load-bearing.

### Required (10)

1. **Top metadata:** status, owner, purpose, primary vision anchor, primary gate tracker (if applicable), pairing note for session-level batons.
2. **TL;DR for cold sessions:** current state, why it matters, what is next, what is explicitly not next.
3. **Operator intent:** preserve exact quotes that triggered the arc or changed its direction.
4. **Working doctrine:** 5-10 non-negotiable principles for this arc.
5. **Authoritative files:** path, purpose, current status; use absolute paths. When the baton is updated, verify referenced absolute paths exist and explicitly flag branch-only/residue paths rather than letting stale pointers become trusted.
6. **Architecture/workstream map:** enough to orient a cold session without rereading every source. (Was §8 in v1.)
7. **Current proof/implementation state:** what passed, what failed, what remains true. (Was §9.)
8. **Next safe session shape:** one-line goal, cold-start reads, opening commands, and safety boundaries. Absorbs the prior §11 "acceptance criteria for the next slice" — empirically the acceptance criteria duplicate the next-session shape. (Was §10; §11 removed as redundant.)
9. **Carry-forwards beyond the next slice.** (Was §13.)
10. **War stories / failure modes to preserve.** (Was §14.)

### Conditional (4 — include only when the trigger applies)

A. **Repo/branch/dirty-state inventory** — INCLUDE if the arc spans multiple repos OR has dirty branch state that could mislead future sessions. Atlas-OS uses it; atlas-extended-sprint and Hermes don't. (Was §6.)
B. **Gate or phase model snapshot** — INCLUDE if the arc is gate-driven with explicit closure criteria. Atlas-OS is gate-driven (Gates 0-N); atlas-extended-sprint uses 30/60/90; Hermes is exploration (no gates). (Was §7.)
C. **Decision tree if stuck** — INCLUDE if there are live unresolved decisions the next session needs to navigate. Atlas-OS uses it; atlas-extended-sprint uses an operator-decision queue (§5) instead; Hermes uses it. (Was §12.)
D. **Hard-rule reminders** — INCLUDE if the arc has arc-specific hard rules beyond what's already in the owning agent's CLAUDE.md. atlas-extended-sprint relies on Terminal's CLAUDE.md instead; Atlas-OS + Hermes include arc-specific hard rules. (Was §15.)

### Removed (moved or absorbed)

- Old §11 (Acceptance criteria for next slice) — **absorbed into §8 next-safe-session-shape**. Empirically duplicates.
- Old §16 (Self-test for the master baton itself) — **moved to shared reference**. Self-test is a one-time-at-authoring discipline; doesn't need to live in every baton. Canonical self-test for master batons:
  - Status header is one paragraph max, not a sprawling changelog tail
  - Operator quotes are verbatim with origin context
  - All cited file paths exist on disk (run `ls`)
  - Architecture map orients a cold session without re-reading every source
  - Carry-forwards are open items, not shipped work
  - War stories are concrete (not "be careful")
  - Conditional sections (A-D above) are included only when their trigger applies

## Differences from a normal baton

- Master batons optimize for continuity across many sessions, not execution of one session.
- They should cite session-level batons, not replace them.
- They may have no Discord ack template or session-end protocol unless the next slice needs one.
- They should avoid over-specifying implementation steps for future gates; instead record gate closures and safe sequencing.
- They should make branch/dirty-state ambiguity explicit, because macro arcs often span multiple repos and abandoned/resumed branches.

## Pitfalls

- Do not let the master baton become a new spec. It is a map and context anchor.
- Do not mark gates/phases complete just because the master baton says they exist.
- Do not hide surprising live state differences. If the repo state disagrees with an earlier summary, record the live state and interpretation.
- Do not create one-off narrow skills for each master baton. Update this class-level baton skill and store session-specific examples under `references/`.
- When the operator says they are confused where a long-running arc is tracked, do not reflexively create another master baton. First search for existing master/session batons across the owning repo and agent docs, then patch the authoritative master baton with a tracking-surface map and links to any adjacent batons. If the current chat is bloated or about to restart, also create a short `data/` recovery prompt that answers "where is everything tracked?" and points to the master baton.

## Example anchors

**For the live registry of active master batons, see `<shared-wiki>/_master-batons.md` (the canonical surface — these examples below may drift; the registry is the source of truth).**

- `<workspace>/agents/terminal/docs/atlas-chief-of-staff-master-baton.md` — Atlas Chief-of-Staff Vision master baton (renamed 2026-05-10 from `atlas-extended-sprint-master-baton.md` once GBrain memory subsystem was frozen and the chief-of-staff scope became the durable framing). 4 subsystems: memory (frozen) + orchestration + observability + action.
- `<fleet-docs>/atlas-os/atlas-runtime-independence-master-baton.md` — Atlas Runtime Independence master baton (renamed 2026-05-10 from `atlas-os-master-baton.md` to disambiguate from the localhost:3000 dashboard web app). Long-term runtime/model/harness-agnostic architecture exploration.
- `<workspace>/agents/watson/docs/hermes/hermes-setup-and-sax-strategy-master-baton.md` — Hermes/Sax strategy + role-crystallization master baton.
