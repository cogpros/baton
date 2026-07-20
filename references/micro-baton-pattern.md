# Micro baton — card-grade continuity for kanban bodies

The grain below section. A **micro baton** (`/baton micro`) is a card-sized baton emitted as a kanban ticket's body: six required lines that let a cold pickup session restart the item without re-gathering what the closing session already knew.

Origin: proposed by [@cogpros](https://github.com/jeremyknows/baton/issues/1) from production use since May 2026 — session batons, a master-baton registry, and section closure held up, but a gap opened *below section scale*. A close protocol that triages unfinished work into kanban cards was writing "title + one line," so picking a card up weeks later meant re-gathering everything the closing session already had in context. **The restart cost was being paid at read time, when context is cold and expensive. It should be paid at write time, when context is hot and nearly free.** The micro baton moves that cost.

This completes the ladder — batons turn out to be fractal. Every unit of parkable work gets one at its own scale:

| Scale | Mode | Grain |
|-------|------|-------|
| Arc | `/master-baton` | many sessions, repos, gates |
| Session | `/baton` | one session's continuation |
| Section | `/baton-close` | a completed section within a baton |
| **Card** | **`/baton micro`** | **one kanban item** |

## The card body — six required lines

```
STATE: where this item stands right now, one or two lines
TRIED: what was attempted, what failed and why. "nothing yet" is valid
FILES: absolute paths touched or to-touch
FIRST COMMAND: one copy-paste-runnable shell line the pickup session executes first
DONE-WHEN: boolean acceptance check. observable, not vibes
SESSION: date plus session reference
```

Each line earns its place:

- **STATE** — orients in one read: where the work actually sits *now*, not its history.
- **TRIED** — the anti-re-walk field. Names the dead paths so the pickup session doesn't spend an hour rediscovering that approach X fails for reason Y. "nothing yet" is a valid and honest value for a freshly-parked idea.
- **FILES** — absolute paths, touched or to-touch. A cold session should not have to grep to find the surface.
- **FIRST COMMAND** — one shell line, copy-paste-runnable verbatim. This is the cold-start ignition: the pickup session runs it first and is immediately oriented in the live state (branch, test, log tail, whatever re-establishes ground truth).
- **DONE-WHEN** — a boolean acceptance check that's observable, not vibes. "the flow works" is not a DONE-WHEN; "`curl localhost:3000/health` returns 200 and the row count in `foo` is nonzero" is.
- **SESSION** — date + a session reference, so the card's provenance is traceable back to the closing session's fuller record.

## The rule it runs under

**No kanban ticket exists without a baton to feed its restart.** Card creation and micro-baton authoring are the same act. A card without a micro baton is a future cold-start tax with no receipt.

## Three guardrails (what makes it work, not a nag)

1. **DONE-WHEN is the gate on parkability.** If you can't write DONE-WHEN, the item isn't understood well enough to park — resolve it or clarify it now, while context is hot, instead of carding a question you'll have to re-derive later. An un-parkable item is a signal, not a failure.

2. **~15-line ceiling.** If the body needs more than ~15 lines, it's a session baton wearing a card costume. Write `/baton` (a full session baton) and have the card's body cite it (`SEE: <path-to-session-baton>`) instead of inlining. Cards stay card-grade; anything larger graduates a scale.

3. **Idempotency key from the title.** Cards carry a key derived from their title. Re-parking the same item *comments the existing card* instead of cloning it — so chronic re-parking surfaces as one loud card with a visible comment history, not five quiet duplicates. The signal you want is "this keeps coming back," and it's only visible if re-parks converge on one card.

## The per-card self-test (three questions)

Before a card is a baton and not a nag, all three must be yes:

1. **Could a cold session run FIRST COMMAND verbatim** and be oriented — no edits, no missing env, no "obviously you'd cd first"?
2. **Is DONE-WHEN checkable by observation** — could someone who isn't you confirm it's done by looking, without judgment calls?
3. **Do STATE + TRIED prevent re-walking a dead path** — would the pickup session avoid the approach that already failed?

Any **no** means the card is a nag, not a baton. Fix the field or don't park the item.

## Wiring it (one template source, many callers)

The pattern that keeps this from drifting: closing-time skills that triage unresolved work into cards all route through **one** clarify step, and that step requires the micro-baton body on card creation. One template source (this reference), N callers. No copied templates to drift out of sync.

If your setup has multiple close protocols (session-end, a retro skill, a triage skill), point all of them at this one body spec rather than letting each grow its own card format.

## The self-referential war story

cogpros's own filing carried the proof: *"this post is two weeks old. i drafted it, parked it for myself to send, and it never got its own card. no baton, no restart line. it sat invisible until i half-remembered it and my agent had to dig it out of session transcripts. we paid the restart cost at read time on the exact post that says don't do that."*

The doctrine applies to itself or it doesn't hold. A parked idea with no micro baton is exactly the invisible-until-half-remembered failure the micro baton exists to prevent — including when the parked thing is a Markdown draft, not code.

## When NOT to use `/baton micro`

- **The item finishes this session.** No card, no baton — just do it.
- **The body wants more than ~15 lines.** Graduate to `/baton` (guardrail 2).
- **You can't write DONE-WHEN.** The item isn't ready to park (guardrail 1) — resolve or clarify.
- **The work spans sessions/repos/gates.** That's a `/master-baton`, not a card.
