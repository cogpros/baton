# Twin-Anchor Runtime Campaign Pattern

Use this reference when authoring or updating a master baton for a long-running runtime, gateway, memory, or orchestration migration campaign.

## Origin

Extracted from the Atlas runtime-independence campaign (2026-05-11). The campaign had two durable docs:

- `atlas-runtime-independence-master-baton.md` — continuity/live state, branches, gates, handoff doctrine.
- `atlas-runtime-independence-vision.html` — operator-facing architecture/spec narrative.

The operator wanted both to remain load-bearing across sessions, with updates when the work changes materially.

## The pattern

Name a short invariant in the master baton — e.g. **Twin-Anchor Rule**:

> The master baton tracks continuity and live state. The paired spec/vision doc tracks the architecture narrative. When evidence, gate status, risk posture, accepted terminology, or architecture claims change, update both anchors together.

Use a memorable name. Avoid unwieldy labels like “document sync invariant.”

## When to apply

Apply when a campaign has:

- Multi-session / multi-repo runtime or infrastructure work.
- A master baton plus a separate vision/spec/HTML/README that humans will keep reading.
- Gate-driven or evidence-driven state that can drift between docs.
- Operator-facing narrative that must stay aligned with implementation truth.

## Operator-facing macro model

For campaigns where the operator is guiding architecture more than line-by-line implementation, the paired vision/spec doc should carry the durable macro metaphor and decision vocabulary. The master baton should only point at it and record that it is load-bearing.

Examples from the Atlas runtime-independence campaign:

- "Harness organism" — Atlas is not a model, runtime, bridge, or memory store; it is the portable harness around them.
- Anatomy map — bus/circulation, memory/brain, runtimes/hands or nervous system, crons/autonomic system, permissions+PRISM+rollback/immune system, repos+worktrees/skeleton, agents/organs.
- "Hands / daemon / shared brain" — distinguish on-demand execution, always-on background work, and shared memory/recall.
- "Ratchet principle" — observed agent failures become durable harness improvements: rule, hook, test, validator, skill, memory policy, or rollback drill.

Keep this in the operator-facing spec/HTML/README when it helps the human make better macro decisions. Add sources/provenance for external references. In the master baton, add a concise doctrine note: macro metaphors guide decisions but never close gates; contracts, tests, sources, and rollback drills decide implementation truth.

## What to put in the master baton

Add near the header or doctrine section:

```md
**Pairs with:** `<path/to/spec-or-vision-doc>` under the **Twin-Anchor Rule**: this baton tracks continuity/live state; the spec tracks the operator-facing architecture narrative. If runtime work changes gate status, current state, risk posture, accepted terminology, or validated evidence, update both anchors in the same change.
```

Add to the doctrine list:

```md
- **Twin-Anchor Rule:** keep this master baton and `<spec>` synchronized when evidence, gates, risks, terminology, or architecture claims change. Do not let the baton become true while the vision doc rots, or the spec remain beautiful while the baton carries all operational truth.
```

## Runtime-campaign safety add-on

For risky runtime/gateway/memory/control-plane migrations, include a sacrificial harness before production routes:

```md
- **Shadow-first / mock-first policy:** use a mock/test agent or synthetic harness for runtime-boundary stress tests before touching established agents. The mock exists to validate seams without risking working routes.
```

This avoids validating against established production agents before the seam is proven.

A good progression for these campaigns is:

1. Pure synthetic payload/contract test.
2. Synthetic mock-agent test using the sacrificial agent identity.
3. Bridge-side or runtime-side recording harness with fake delivery refs and an `off`-mode negative check.
4. Real non-production mock workspace/tenant.
5. Temporary supervised runtime process with shadow mode and low-risk traffic.
6. Canary design only after artifact stability, retention, and rollback are proven.

Record each step as evidence, but keep claim language conservative: synthetic/local proof advances a gate; it does not close a live runtime boundary gate.

Skill note (2026-05-11): this ladder came from Atlas runtime-independence Gate 2 work, where the reusable lesson was not a specific commit but the progression from payload tests to a bridge-side recording harness before any real tenant or live traffic.

## Branch hygiene note

Long-running campaigns often fail by committing correct work to the wrong branch. The baton should list per-repo:

- current branch,
- branch purpose,
- whether it is local-only or pushed,
- whether merge commits are allowed,
- whether sync should use rebase/cherry-pick/backup branches,
- dirty files that belong to other work.

If branch state is awkward but intentional, say so explicitly instead of smoothing it over.

## Pitfalls

- Updating only the pretty HTML/spec while the baton’s live state drifts.
- Updating only the baton while the operator-facing doc becomes stale.
- Treating a registry/stub/doc as proof before a consumer or validation harness exists.
- Using production agent routes as the first proof of a runtime seam.
- Creating a new master baton instead of patching the existing paired anchors.

## Session learnings captured

- **2026-05-11 — Shadow validation ladder:** Atlas runtime-independence Gate 2 work showed the reusable progression for runtime-boundary campaigns: payload contract test → sacrificial mock-agent identity → bridge/runtime recording harness with fake delivery refs and off-mode negative check → real non-production tenant → supervised shadow runtime → canary design. Synthetic/local proof advances evidence but does not close a live runtime gate.

## Related skills

- `update-docs` — use its operator-facing macro-framework guidance when editing the paired vision/spec/README.
- `verification-before-completion` — use before claiming any gate or proof is advanced/closed.
