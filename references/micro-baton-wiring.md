# Micro-baton wiring: one template source, N callers

Companion to `micro-baton-pattern.md`. That doc specifies the card body. This one specifies how multiple close protocols share it without drifting.

Generalized from a production fleet running three separate close protocols since 2026-07: an interactive session close, an autonomous variant for when the operator is unavailable, and a fleet-wide close that sweeps many agents at once. All three park unresolved work as kanban cards. All three emit the same six-line body. None of them carries a copy of the template.

## The drift problem

Every close protocol that can create a card is a place the card format can fork. Give each protocol its own template and within weeks you have three dialects: one drops TRIED, one renames DONE-WHEN to "acceptance", one inlines twenty lines because nobody enforced the ceiling. Pickup sessions now need to know which dialect they're reading. The whole point of a fixed six-line body, that a cold session can parse it on sight, is gone.

The fix is structural: **the template lives in exactly one file, and every caller points at it by path.** Discipline alone drifts; a pointer can't. Callers quote the six field names inline (so a reader of the caller skill knows the shape) but never restate field semantics, guardrails, or the self-test. Those live only in the reference.

## The shared clarify step

Each caller runs the same decision sequence on every unresolved item it finds. This is the step that owns card creation, so it is the single enforcement point:

1. **Finishes now?** Do it. No card.
2. **Can you write DONE-WHEN?** If not, the item is not understood well enough to park. Resolve or clarify it in this session. This is guardrail 1 acting as a router, not just a lint.
3. **Body over ~15 lines?** Write a full session baton and have the card body cite it. Guardrail 2.
4. **Otherwise:** author the six lines from the reference template and create the card with the body attached and an idempotency key derived from the item.

The creation call shape, in whatever kanban CLI or API the host runs:

```
kanban create "<item title>" \
  --body "<six-line micro baton>" \
  --idempotency-key <prefix>-<kebab-of-item> \
  --created-by <caller-id>
```

Two conventions that earn their keep:

- **Key prefixes per caller.** The interactive close prefixes its keys differently than the autonomous close (e.g. `ct-` vs `auto-`). Re-parks of the same item from the same protocol converge on one card (guardrail 3), and the prefix tells you at a glance which protocol keeps re-parking it.
- **`--created-by` names the caller.** When a card is a nag instead of a baton, you know which protocol's clarify step let it through.

## Caller profiles

The same clarify step, three postures. The difference is who supplies the six lines:

| Caller | Who fills the body | Honesty marker |
|--------|-------------------|----------------|
| Interactive close | Agent drafts, operator confirms at the clarify step | none needed |
| Autonomous close | Agent fills all six lines from session evidence | body values the agent estimated are tagged `agent-estimated` |
| Fleet-wide close | Closing agent fills from per-agent work logs; cross-agent items name the owning agent in STATE | `created-by` carries the closing agent, STATE carries the owner |

The autonomous profile is the one worth copying carefully. When no human is present to answer "what did you try?", the agent reconstructs TRIED from the transcript and FIRST COMMAND from the last known-good invocation. That works, but any field the agent inferred rather than observed gets tagged, so the dataset of cards stays honest about provenance. A confident-sounding TRIED line that was actually a guess is worse than "nothing yet".

## Drift check

Because callers only point at the template, drift is greppable. Two invariants to assert periodically (a cron, a pre-publish check, or by hand):

1. Every caller that creates cards contains a path reference to the one template file.
2. No caller contains its own definition of the field semantics (search for a second definition of DONE-WHEN or the self-test questions outside the reference).

If invariant 2 fails, a caller has grown a dialect. Delete the copy, restore the pointer.

## What stays host-specific

The pattern deliberately does not specify: the kanban tool, board taxonomy, who is allowed to seal a session, or how the close protocols are triggered. Those belong to the host fleet. The portable part is exactly three things: one template file, one clarify-step shape, key-plus-creator conventions on the creation call.

## Failure mode observed in production

The gap this wiring closed for us: before the shared step, one protocol created cards at a different phase than the others and skipped the DONE-WHEN gate entirely. Its cards passed zero of the three self-test questions. They were titles with timestamps. The fix was deleting that protocol's own card-creation code and routing it through the same clarify step as the rest. If a protocol can create a card without passing the gate, it eventually will.
