---
name: baton
runtime: portable
description: |
  Two modes — decide before invoking.
  SESSION BATON (/baton): next-session execution prompt a cold session can run end-to-end.
  24-section template: state-inventory + cold-start reads + verification + embedded commands
  + decision tree + war stories + self-test. Triggers: "/baton", "write the baton",
  "make a baton", "carry-forward prompt", "next-session prompt", "continuation prompt",
  "/handoff-prompt".
  MASTER BATON (/master-baton): macro orientation doc for arcs spanning many sessions,
  compactions, repos, or gates. Stable across sessions; session batons cite it as their
  first context load. Triggers: "/master-baton", "master baton", "cross-session baton",
  "vision baton".
  SECTION CLOSURE (/baton-close): mark completed sections in an existing multi-section
  baton so a cold session doesn't re-execute closed work. Triggers: "/baton-close",
  "mark section closed", "update the baton", "close section X".
  NOT FOR — sessions that complete all their work (use session-end only) or open-ended
  exploratory sessions with no obvious next-session shape.
license: MIT
compatibility: |
  Designed around Atlas agent fleet conventions (data/ workspace + bus event emit + memory
  diary structure + <universal-conventions-doc>). Portable to other agent runtimes if the host fleet
  has equivalents for: a per-session scratchpad directory, a memory/diary file format, and
  a way to emit lightweight events for observability. Shell + bash are the only hard tool
  dependencies; everything else is convention.
metadata:
  author: jeremyknows
  version: "1.4.0"
  category: "Business Process Automation"
  tags: [continuation, session-handoff, meta-skill, observability, daily-feedback]
---

# baton — Next-Session Continuation Prompt Skill

The baton is the artifact a cold session reads to pick up where the prior session left off. A good baton lets a fresh session walk the work end-to-end without needing the prior session in-context. A bad baton produces hours of "what does this mean?" + accidental scope-creep + missed verification.

This skill formalizes the prompt → audit → dry-run pattern that produced a 580-line / 46.8KB cold-start-ready continuation prompt during the 2026-05-08 bus North Star session — the canonical reference example.

---

## When to invoke

### Session baton — `/baton`

**Invoke when ANY of:**
- Session shipped artifacts that next session needs to act on (specs, ADRs, partial implementations)
- Session surfaced operator-decision flags that next session must resolve
- Session shipped Tier-0/-1 work that has follow-on Tier-2/-N
- Session ran PRISM that produced mechanical-fix list or NEEDS-WORK verdict
- Session built half a feature and needs the other half

Trigger phrases: "write the baton", "/baton", "make a baton", "continuation prompt", "next-session prompt", "carry-forward prompt", "/handoff-prompt"

### Master baton — `/master-baton`

**Invoke when ANY of:**
- Operator says "/master-baton" or "master baton"
- The work spans many sessions, context compactions, repos, or gates
- Operator says the shape is still unknown and wants a durable orientation artifact first
- A North Star doc, vision doc, or sprint plan needs to stay load-bearing across sessions

Trigger phrases: "/master-baton", "master baton", "cross-session baton", "vision baton"

See `references/master-baton-pattern.md` for the master baton shape (16-point required structure, recommended location, pitfalls). When a master baton moves from exploration/grill into artifactization or implementation, also read `references/master-baton-checkpoint-validation.md` and run its checkpoint pass before creating downstream artifacts.

### Section closure — `/baton-close`

**Invoke when:**
- Session completed one or more sections of an existing multi-section baton but NOT all sections
- The baton file still shows completed sections as live work — a cold session would re-execute them
- Operator says "/baton-close", "mark section closed", "close section X", "update the baton"

Trigger phrases: "/baton-close", "mark section closed", "update the baton", "close section X", "mark X done in the baton"

### Skip when (both modes)

- Session completed all work end-to-end (use `session-end` skill only — no baton needed)
- Session was open-ended exploration with no concrete next-session shape
- Session is a 1-2 message Q&A
- Operator says "no continuation needed"

---

## The pattern (two modes)

| Mode | Trigger | Steps |
|------|---------|-------|
| **Session baton** | `/baton`, "write the baton" | 0 → 1 → 2 → 3 → 4 → 5 → 6 |
| **Master baton** | `/master-baton`, "master baton" | 0 → 1M → 2 → 3 → 6 |
| **Section closure** | `/baton-close`, "mark section closed" | 1C → 6 |

Both modes share Step 0 (state-inventory), Steps 2–3 (self-audit + dry-run), and Step 6 (emit + autoresearch row). Session mode uses Step 1 (24-section draft). Master mode uses Step 1M (macro orientation doc) and skips Steps 4–5 (dry-run gap-closing and session-end cite).

---

### Step 0 — State-inventory (~5 min, P0 — DO NOT SKIP)

Before drafting, you must know:

- **What shipped this session** — commit hashes per repo, files created/modified (with status: NEW vs CHANGED), durable artifacts (specs, ADRs, scripts, configs, PRISM archives)
- **What's pending decision** — operator-decision flags surfaced but not resolved (with full trade-off context per flag)
- **What's pending execution** — work items with explicit effort estimates, dependency order, scope ceiling for next session
- **War stories from THIS session that must carry forward** — defects caught (especially recurring defect classes), kickstart-after-edit lessons, scope-cut hedging incidents, hallucinations, etc.
- **The next-session scope ceiling** — the explicit "DO NOT do more than X" boundary so the cold session doesn't accidentally grab work that's not theirs

If these aren't answered before drafting, the prompt will be vague + the dry-run will catch the same gaps over and over.

**Tip:** if your session ran a PRISM that produced findings, the synthesis archive IS most of your state-inventory. Reference it by path; don't restate.

### Step 0C — Consolidation mode for multiple old sessions / compacted chats

If consolidation is the main task rather than a substep of baton authoring, load the `continuity-consolidation` skill first. Use this section as the baton-specific adapter when the operator says multiple old sessions/chats produced overlapping batons, summaries, draft docs, or handoffs and asks to "consolidate", "close them", "ensure baton coverage", or "update the registry".

Do **not** ask each old session to independently write a baton. That creates duplicate authoritative surfaces. Instead:

1. Inventory all evidence surfaces first: session IDs, transcript exports, continuity scratchpads, memory files, draft docs, existing master/session batons, and `_master-batons.md`.
2. Classify every source into one bucket: already covered; patch existing master baton; create narrow session baton; promote to new master baton; close/archive/supersede.
3. Register all master batons, and register only the discoverability-worthy minority of session batons: expected 3+ sessions, unresolved operator decisions, cross-agent sequencing, gateway/profile/runtime transitions, or high-risk continuity a cold session must find without already knowing the filename. Do **not** turn the registry into a list of every one-shot continuation prompt.
4. Prefer patching an existing master arc over creating a new one. A new master baton is justified only when the work is durable, multi-session, and not already represented by an active registry row.
5. Separate **active refs** from **historical evidence**. Patch active docs/prompts/continuity notes that would mislead a cold session; do not rewrite transcript-review archives or PRISM evidence unless adding an explicit superseded note.
6. If a registry row points at gitignored `agents/*/data/` session batons, verify the files exist locally and state that they are local continuity artifacts, not versioned docs.
7. Use parallel subagents for independent domains (e.g. runtime architecture, browser capability, BlueBubbles, chief-of-staff/GBrain), then independently verify their claims against files/tests before editing.
8. Write one consolidation audit note in the continuity scratchpad summarizing the final map: existing master arcs, new session batons, superseded batons, stale facts not to trust, and remaining gates.
9. Update `_master-batons.md` last, after filesystem truth is settled.

Common consolidation anti-pattern: creating one new baton per old chat. The desired output is usually fewer authoritative surfaces, not more.

### Step 1M — Master baton mode (~30-60 min)

*Use for `/master-baton` invocations. Skip for `/baton` (go to Step 1 instead).*

Write a macro orientation document. Later session-level batons cite it as their first context load — it is NOT a session execution prompt itself.

**FIRST — check the registry.** Read `<shared-wiki>/_master-batons.md` (the canonical baton registry as of 2026-05-10). If a master baton for this arc already exists, **patch it rather than create a new one** — duplicate master batons are the recurring anti-pattern in this skill (see Pitfalls below). The registry's Active Master Batons table is the source of truth; `references/master-baton-pattern.md` Example anchors may drift.

Use `references/master-baton-pattern.md` as the checklist (10 Required + 4 Conditional sections per the 2026-05-10 revision). For a narrower follow-on baton after a master baton, see `references/role-crystallization-pattern.md` — it captures the master-baton → matrix → grill sequence. Key differences from a session baton:

- Store it in the owning docs namespace (`~/projects/<repo>/docs/<workstream>/...` or `<workspace>/agents/<agent>/docs/...`), not a gitignored session `data/` path.
- Preserve the operator's exact intent and why the arc matters.
- Inventory authoritative files, branch/dirty state (only if multi-repo or dirty), relevant commits, current gates/phases (only if gate-driven), next safe slice, and carry-forwards.
- Emphasize sequencing and evidence boundaries over step-by-step implementation.
- If the arc has a paired operator-facing vision/spec/narrative doc, name the pair explicitly and add a short sync invariant: baton = continuity/live state; spec = architecture/narrative; update both when evidence, gates, risk posture, terminology, or architecture claims change. See `references/twin-anchor-runtime-campaign.md` for the pattern extracted from the Atlas runtime-independence campaign.
- For risky runtime/infra migration arcs, prefer a sacrificial mock/test harness before established production agents/routes. Name this in the master baton as a safety doctrine, not just a next task.
- Add a pointer from the owning docs index/README when one exists.
- **Append a row to `<shared-wiki>/_master-batons.md`** (Active Master Batons section) once the file is authored. Underscore-prefix wiki control file convention — any agent appends.
- If the operator is confused about where an arc is tracked, first read the registry, then `find`/`grep` for `*baton*` as a fallback. Patch the authoritative master baton with a tracking-surface map and adjacent-baton links rather than creating duplicate sources of truth. If context is bloated, write a short `data/` recovery prompt that points back to the master baton.

When retiring a master baton: move its row from Active to Archived in `_master-batons.md`, add a retirement banner to the top of the baton file (don't delete or move the file), and ensure a closeout memory diary exists at the cited path.

Then continue with self-audit, dry-run, autoresearch row, and `baton_emitted` just like a normal baton.

### Step 1C — Section closure mode (~5 min)

*Use for `/baton-close` invocations. Skip for `/baton` and `/master-baton`.*

Mark completed sections in an existing multi-section baton so a cold session doesn't re-execute closed work. Do NOT re-draft, re-audit, or re-dry-run the full baton — this step touches only closure markers and frontmatter.

**1. Read the target baton** (path provided, or the most recent session baton if omitted).

**2. Identify which sections closed this session.** Cross-reference LAST_SESSION.md, today's memory diary, or operator confirmation. A section is "closed" when its acceptance criteria passed and its artifacts are committed/verified — not merely attempted.

**3. Add a closure marker** immediately under each completed section header:

```
### §A First-fire post-mortem
**STATUS: CLOSED 2026-05-11** — [one-line outcome, e.g. "post-mortem complete, hybrid relay confirmed correct"]
```

**4. Update frontmatter:**

- If ALL sections are now closed: set `status: RETIRED YYYY-MM-DD` and add `retired_by: <diary path>`
- If sections are PARTIALLY closed: keep `status: active`, add:
  ```yaml
  closed_sections: [A, B]
  open_sections: [C]
  ```

**5. Proceed directly to Step 6.** Emit `baton_emitted` with `partial_closure: true`:

```bash
bash "$EMIT" <agent> baton_emitted \
  "Baton updated (partial closure): $(basename "$BATON_PATH") — §[X,Y] closed, §[Z] open" \
  "{\"baton_path\":\"$BATON_PATH\",\"partial_closure\":true,\"closed_sections\":\"[X,Y]\",\"open_sections\":\"[Z]\"}" \
  completion
```

---

### Step 1 — Draft the session baton prompt (~20-40 min)

*Use for `/baton` invocations. Skip for `/master-baton` (use Step 1M instead).*

Write the prompt to `<workspace>/agents/<agent>/data/<YYYY-MM-DD>-next-session-<topic>-prompt.md` (or your agent's equivalent path; data/ is FS-only / gitignored by Atlas convention).

**Required sections** (see `references/template.md` for full structure):

1. **One-line goal + spec/anchor file paths** at the very top
2. **Macro context anchor** (conditional) — *if a master baton, vision doc, North Star spec, or `CONTEXT.md` exists for this work's overarching project / sprint / direction, cite it in the anchor docs block AND add a "Macro context: where this session fits" section before the 60-second recap.* Cover: which subsystem/workstream this session lives in; which closure criterion or success metric it contributes to; how its effort fits the sprint total; pointer to any operator-decision flags from the master doc that this session might surface against; skim-vs-full reading guidance for the macro doc (so the cold session doesn't drown). Skip the section if no such doc exists — don't fabricate one to satisfy the template.
3. **60-second context recap** — orients a cold session in 1 minute (where we are, operator-stated direction, war-story defect class, this-session-shape)
4. **DO NOT list** — explicit don't-dos including scope-cut authorization, hard-rule reminders specific to this work
5. **Cold-start reads** — ordered list with EXISTS/GREENFIELD status per file (master/vision/context doc goes near the top of this list when §2 applies)
6. **Discord ack template** — exact text the cold session posts after reads
7. **System state expected at session start** — verifiable items (table format)
8. **Tools required** — `which X Y Z` line
9. **Verification ledger run** — explicit command to refresh facts (`bash <verify-script>`)
10. **PRISM synthesis status** — if applicable, point at the archive + summarize verdict + count Tier-1 findings
11. **Mechanical fixes inline list** — apply-able from prompt alone OR point at archive's full reasoning
12. **Operator-decision flags** — full trade-off context per flag + recommended answer
13. **Step-by-step execution** — per-step files, schemas, embedded commands (SQL/bash/jq), verification commands, commit message templates
14. **Acceptance criteria** — boolean checklist per step
15. **Session-end protocol** — diary path, LAST_SESSION update, MEMORY row, session_end emit, Discord wrap
16. **Carry-forwards** — work for next-next-session
17. **Discord status update cadence** — when to post during the session
18. **File path summary** — table with paths + purpose + status (committed/greenfield/etc)
19. **Session-completion signal** — boolean checklist that defines "done"
20. **Decision tree if stuck** — the "stuck? ask: which kind?" pattern
21. **War stories durable across sessions** — load-bearing operational lessons from THIS session that the next session needs to inherit
22. **Hard-rule reminders** — agent-specific hard rules + universal hard rules
23. **Bonus context** — operator quotes (verbatim), commit hashes, total session metrics
24. **Self-test before declaring complete** — boolean checklist that mirrors the war-story lessons

**Length guideline:** longer is better than shorter for continuation prompts. The canonical 2026-05-08 example is 580 lines / 46.8KB. A cold session paying the cost of reading 5 minutes of prompt is cheap; the cost of a vague prompt is hours of misdirection.

### Step 2 — Self-audit (~5-10 min)

After drafting, audit for thoroughness. Ask:

- **Is the 60-second recap actually 60 seconds of read?** Test: does it answer "where am I, what just happened, what do I do next" in one screenful?
- **Is there a macro/vision/context doc for this work, and does §2 reference it?** If a master baton, North Star spec, sprint plan, or `CONTEXT.md` exists for the overarching project, the baton MUST cite it in the anchor docs + the Macro Context section + the cold-start reads list. If you skipped §2, sanity-check: is that because no such doc exists (legitimate skip) OR because you didn't think to look (defect)? Search: `ls <workspace>/agents/<agent>/docs/*master-baton* <workspace>/agents/<agent>/docs/*north-star* <fleet-docs>/*north-star* 2>/dev/null` and any spec the work cites.
- **Are operator quotes preserved verbatim?** If the operator pushed back against scope-cuts or stated a North Star, paste the exact words. Paraphrasing loses signal.
- **Is the DO NOT list complete?** Specifically check: scope-cut authorization, hard-rule reminders, agent-workspace-tracking-policy if applicable, external-world-action authorization if applicable.
- **Are commit hashes listed?** Both this-session's commits AND prior-session commits the next session might reference.
- **Are operator-decision flags self-contained?** Each flag should have: what each option means, the trade-off, my recommended answer. Cold session should be able to assemble the Discord post from the flag spec alone.
- **Are durable artifacts cited by absolute path?** Not relative; not "see prior session" — explicit `~/path/to/file.md` for each.
- **Are war stories from this session captured?** Specifically: defect classes that recurred (the operator's "verify before authoring" rule is a war-story-anchor pattern), scope-cut-hedging episodes, kickstart-after-edit lessons, etc.

If you find a gap in the self-audit, fix it. Don't proceed to dry-run with known gaps.

### Step 3 — Dry-run as cold session (~10-20 min)

Walk through the prompt as if you were a fresh CC session opening it for the first time. For each "do X" instruction, ask:

- **Could a cold session execute this with what's in the prompt + cited artifacts?** OR would they have to compose it from scratch?
- **Are commands explicit?** "Apply mechanical fix CV-1" is NOT explicit. "Run `sqlite3 path 'UPDATE pending_approvals SET agent='watson' WHERE agent IS NULL;'`" IS explicit.
- **Are file:line cites where edits happen?** "Edit §X of spec v2" is OK if the section is named; "edit the part about XYZ" is not.
- **Are verification commands embedded?** Each step's "how do I know this worked?" should have a 1-line bash/curl/sqlite3 to run.
- **Are the cold-start reads actually ENOUGH?** Read each cited doc title + summary; would a cold session understand the architecture from those reads?
- **Are war-story patterns from this session connected to next-session work?** E.g., "kickstart-after-edit" lesson should be tied to any step that modifies a long-running daemon.

Common dry-run gaps to catch:
- Missing SQL examples for "schema fix" instructions
- Missing bash for "extend script with probes" instructions
- Missing daemon-kickstart notes for "edit running daemon" steps
- Wrong field-location guidance (top-level vs nested in JSON)
- Tier-3 nits lumped together that should be split for clarity

### Step 4 — Close dry-run gaps (~10-20 min)

Apply fixes for each gap found in dry-run. Add concrete commands, SQL, jq filters, plist labels, etc. Re-verify line count + structure.

### Step 5 — Cite the prompt path in session-end Discord wrap (~1 min)

After session-end protocol completes, the Discord wrap-up should include:

> Next-session prompt at `<workspace>/agents/<agent>/data/<filename>.md`. Cold-start ready: [N]-line, [M] sections + 60-second recap + dry-run-validated.

This signals to the operator that the prompt has been audited — they can authorize the next session to start without worrying about hidden gaps.

### Step 6 — Run the Self-Test + emit `baton_emitted` bus event (~2 min, P0 — DO NOT SKIP)

Score the Self-Test (next section). Then emit a bus event so daily improvements get logged programmatically — without this, feedback evaporates into operator-only memory.

```bash
EMIT=$(ls <bus-emit-dir>/<bus-emit-script> <bus-emit-dir>/<bus-emit-script> 2>/dev/null | head -1)
LINE_COUNT=$(wc -l < "$BATON_PATH")
SECTIONS_COUNT=$(grep -c "^## \|^### " "$BATON_PATH")
bash "$EMIT" <agent> baton_emitted \
  "Baton authored: $(basename "$BATON_PATH") — ${LINE_COUNT}-line / ${SECTIONS_COUNT} sections" \
  "{\"baton_path\":\"$BATON_PATH\",\"line_count\":$LINE_COUNT,\"sections_count\":$SECTIONS_COUNT,\"self_test_score\":\"<X/9>\",\"dry_run_friction_count\":<N>,\"target_session\":\"<topic-slug>\"}" \
  completion
```

Then append a row to `<skill-dir>/baton/references/AUTORESEARCH-SELF-ASSESSMENT.md` (the daily improvements log — see ## Autoresearch). One row per baton authored. This is THE feedback surface — without it, every baton is a one-off and lessons don't compound.

---

## Self-Test (score after every baton authoring, target ≥7/9)

| # | Question | Y/N |
|---|----------|-----|
| 1 | Did I do the state-inventory FIRST (Step 0) — commits + decisions + execution + war stories + scope ceiling? | |
| 2 | Does the 60-second recap actually answer "where am I, what just happened, what do I do next" in one screenful? | |
| 3 | Are operator quotes preserved verbatim (not paraphrased)? | |
| 4 | Does each operator-decision flag have full trade-off context + recommended answer (cold session can assemble Discord post from flag spec alone)? | |
| 5 | Does each step have an embedded verification command (1-line bash/curl/sqlite3) the cold session can run? | |
| 6 | Did I dry-run the prompt as a cold session and apply ALL friction-point fixes? | |
| 7 | Are war stories from THIS session connected to next-session steps where they apply? | |
| 8 | Does the prompt include a self-test (boolean checklist defining "done") for the cold session? | |
| 9 | Did I check for a macro/vision/context/master-baton doc and either (a) reference it in §2 + cold-start reads, or (b) confirm none exists? "I didn't look" is N. | |

**Below 6/9 = high probability of cold-session misdirection.** Don't ship at <6; iterate.

Score goes into the `baton_emitted` bus event payload (Step 6) AND the autoresearch row (next section) so daily improvements compound.

---

## Autoresearch

**Baseline:** 7/12 → 11+/12 after this audit's Tier-2 fixes (skill-doctor 2026-05-08 second-audit-of-the-day, Round 1)
**Q13 (empirical):** TBD — accumulating over next 2 weeks. Goal: 5+ batons authored with self-test scores logged in `references/AUTORESEARCH-SELF-ASSESSMENT.md`. The 2026-05-08 canonical example (580-line / 46.8KB / 14-section bus-North-Star execute prompt) is run #1, score TBD on retrospective.
**Q14 (observability):** YES — `baton_emitted` bus event emitted at Step 6 with payload including line_count + sections_count + self_test_score + dry_run_friction_count + target_session. Operator can `bus-tail.sh --type baton_emitted` to see the cadence + score-distribution.

**Daily improvements log:** `references/AUTORESEARCH-SELF-ASSESSMENT.md` — one row per baton authored (date, target session topic, line_count, sections, self_test_score, dry_run_friction_count, what landed cleanly, what cold session got tripped up by, mutation candidates surfaced). The log is read by `skill-doctor` at next audit when the row count grows enough to warrant a wiki article on baton patterns.

**Mutation candidates (top 3 surfaced THIS session — to consider for v1.1+):**
1. **`scripts/baton-state-inventory.sh` helper** — auto-collect this-session commits (`git log --since` per repo), files-touched (from $TRANSCRIPT or session-end skill output), pending memory diary path. Removes ~5 min per baton; closes Q6 PARTIAL → YES.
2. **Embedded baton-quality reflexion at next-session start** — first action of the cold session is to score the baton on a 5-question rubric (cold-start orientation time, first-friction-point, missing-info count, surprise count, self-test alignment) and emit `baton_consumed` bus event. Closes the feedback loop; gives operator a reverse-empirical signal (was the baton actually useful, or did it look good but cold-session got stuck?).
3. **Per-section friction heatmap** — as cold sessions log friction points back via baton_consumed, identify which of the 23 template sections produce the most friction (e.g., "60-second recap" too long? "Operator-decision flags" missing trade-offs?). Drives focused mutations on the template, not the whole skill.

**Self-assessment frequency:** every baton invocation appends a row. skill-doctor re-audits every 5 batons OR every 2 weeks (whichever first).

Full daily-improvements log: `references/AUTORESEARCH-SELF-ASSESSMENT.md` (created by this skill's first audit, populated organically).

---

## Anti-patterns (signs you're doing it wrong)

- **"Read the prior session diary"** as the only context — diary is a NARRATIVE, not an action plan. The baton has to direct ACTION.
- **"Continue from where we left off"** without specifying WHERE — cold session has no left-off-from.
- **Lists of work without effort estimates** — "do X, Y, Z" is uncountable; "do X (~30 min) → Y (~2h) → Z (~1h)" is bounded.
- **Operator-decision flags without trade-offs** — "should we do X or Y?" surfaces a decision but doesn't help operator decide. Each flag needs the case for X + the case for Y + my recommended answer.
- **No DO-NOT list** — cold session will scope-creep into things you didn't mean. Always have an explicit DO-NOT.
- **No verification commands** — "make sure X works" is not actionable. "Run `<command>`; expect output `<pattern>`" is.
- **Stale numbers inline** — never inline `0.34%` or `127 lines` if the number drifts. Cite the verify-script + invocation.
- **No self-test** — cold session needs a way to know "am I done?" — boolean checklist with pass/fail items mirrors the war-story-lessons.
- **Master baton treated like a normal session prompt** — master batons should orient an arc across sessions and point to the next safe slice; they should not pretend to be a single-session execution plan. Use `references/master-baton-pattern.md`.
- **Live repo state overwritten by prior summary** — if the current branch/dirty state disagrees with a compacted summary or previous wrap-up, record the live state and interpretation explicitly. Master batons are where this ambiguity must be killed.
- **Registry-first guessing** — do not update `_master-batons.md` before validating the underlying files. The registry indexes truth; it is not where you invent it.
- **Historical transcript rewriting** — patch active continuation prompts/docs that would mislead a cold session, but leave transcript-review archives and PRISM evidence intact unless adding an explicit superseded note. Evidence should remain evidence.
- **One-chat-one-baton sprawl** — when consolidating multiple old chats, collapse overlapping work into existing master arcs plus narrow session batons only where follow-up execution is real.
- **Paired-anchor drift** — if a master baton has an adjacent vision/spec/HTML/narrative doc, do not update only the baton when campaign state changes. Add a named invariant (e.g. “Twin-Anchor Rule”) and patch both anchors when evidence, gates, risks, terminology, or architecture claims move.
- **Production-agent validation as first proof** — for runtime/gateway/memory/control-plane campaigns, do not use a live trusted agent as the first validation target. Create a sacrificial mock/test agent or synthetic harness first; the baton should make that safety boundary explicit.
- **Correct work on the wrong branch** — long-running campaigns often span repos and sessions; master batons must identify the active branch per repo and name branch-hygiene constraints (backup before sync, no casual merge commits, no committing doc-state changes to an unrelated feature branch without calling it out).

---

## Canonical example

The reference baton produced by this skill on 2026-05-08:

`<workspace>/agents/terminal/data/2026-05-09-next-session-spec-v2-execute-prompt.md`

580 lines / 46.8KB / 14 sections + 60-second recap. Walks through cold-start reads → verification → Round 3 PRISM synthesis (already archived) → 14 mechanical fixes inline + 7 OD flags with full trade-off context → 4-substep Step 0 execution with embedded SQL + bash + cron commands → war stories → self-test.

Authored 2026-05-08T06:30Z; audited; dry-run produced 4 friction-point fixes (SQL example for `pending_approvals.agent` null-fill; bash for <verify-script> probe extensions; corrected loader-update guidance for top-level lineage fields; daemon-kickstart note for reaction-gate-listener.js after edit). Final ship 2026-05-08T07:00Z.

This was the prompt that THIS skill was extracted from. Read it as a template for shape + density.

---

## Pairing with `session-end`

`session-end` is the closing-doc ritual (memory diary + LAST_SESSION + MEMORY row + session_end emit + Discord wrap). `baton` is one OPTIONAL output of session-end — produced when there's continuation work for the next session.

Order:
1. Run baton skill FIRST if a continuation prompt is needed (state-inventory benefits from being done while session memory is fresh)
2. Run session-end skill SECOND (writes the diary + closing docs; references the baton if produced)

Or interleave: baton's state-inventory step (Step 0) IS most of what session-end's diary needs to capture. You can do them concurrently if confident.

---

## See also

- `<skill-dir>/session-end/SKILL.md` — the closing-doc ritual baton pairs with
- `<skill-dir>/pre-pr/SKILL.md` — sibling Process & Workflow skill with the same Q14 / autoresearch pattern
- `<shared-docs>/<universal-conventions-doc>` — universal Atlas agent conventions (memory writes, Discord rules, hard rules)
- `<fleet-docs>/<workspace-layout-doc>` — canonical agent workspace structure (where data/ lives)

---

## Known Failure Modes (using THIS skill, not batons it produces)

1. **State-inventory at context-exhaustion.** If the session ran 4h+ and you're invoking baton at the very end with low remaining context, the state-inventory's quality drops — you may miss commit hashes, conflate operator quotes, or skip a war story. Mitigation: invoke baton EARLIER (e.g., at the moment you realize the work won't finish this session), not as a death-rattle ritual. The ~20-40 min draft step needs real attention, not last-token panic.
2. **Operator silent on decision flags.** If the session ended with N operator-decision flags surfaced but no answers, the baton's flag section becomes "options + my recommendation" — that's correct. Anti-pattern: assuming silence = "do whichever" and dropping the flag from the baton. Cold session loses agency. ALWAYS preserve unresolved flags with full trade-off + your recommended answer.
3. **Dry-run skipped under time pressure.** Step 3 is the most-skipped step (Step 0 state-inventory is "do something" so it gets done; Step 3 dry-run is "re-read what you just wrote" which feels redundant). It is NOT redundant — every dry-run on the canonical example surfaced 3-4 friction points the original draft missed. Mitigation: Self-Test Q6 is the gate; if you can't tick it, ship is premature.
4. **Skill mistaken for session-end.** baton is OPTIONAL; session-end is mandatory. Some sessions don't need a baton (work complete, exploratory). Don't author batons reflexively — only when there's concrete carry-forward action. The "Skip when:" list in `## When to invoke` is the discriminator.
5. **Frontmatter / sections drift over versions.** As Atlas conventions evolve, the 23-section template in `references/template.md` may go stale. The autoresearch log is the canary — if friction-point counts trend up, the template needs review, not the skill.

---

## Dependencies

- `<skill-dir>/session-end/SKILL.md` — pairs with baton; baton runs FIRST (state-inventory benefits from fresh memory) or interleaves
- `<shared-docs>/<universal-conventions-doc>` — declares memory diary writing format (consumed by Step 0 state-inventory)
- `<fleet-docs>/<workspace-layout-doc>` — canonical agent workspace schema (declares where `data/` lives, gitignore rules)
- `<bus-emit-dir>/<bus-emit-script>` — bus event emission (Step 6 `baton_emitted`)
- `references/template.md` — 23-section structural template (referenced from §1 Step 1)
- `continuity-consolidation` skill — first-class workflow for consolidating old sessions/chats, shadow branches, overlapping docs, and baton sprawl into a source-of-truth map before baton-specific registry updates
- `references/master-baton-pattern.md` — macro-baton checklist for multi-session / multi-repo / compaction-heavy arcs; use when the operator says "master baton"
- `references/twin-anchor-runtime-campaign.md` — paired master-baton + vision/spec doc pattern for long-running runtime/infrastructure campaigns; includes mock-first safety and branch-hygiene notes
- `references/master-baton-checkpoint-validation.md` — checkpoint pass for updating exploratory master batons after decisions crystallize, before downstream artifacts or implementation depend on them
- `references/example.md` — canonical example (referenced from §Canonical example)
- `references/AUTORESEARCH-SELF-ASSESSMENT.md` — daily improvements log (created by this audit; populated per invocation)
- `bus event registration` — `baton_emitted` (and future `baton_consumed`) need to land in `<bus-emit-dir>/<bus-emit-script>` allowlist if they're not already auto-allowed. Check: `grep "baton_emitted" <bus-emit-dir>/<bus-emit-script>` — if absent, add to the info-severity allowlist via small follow-up commit.

---

## Changelog

- **v1.4.0 — 2026-05-11 (section closure mode)** — Added `/baton-close` as a third mode for marking completed sections in an existing multi-section baton. Gap surfaced 2026-05-11: Terminal's cos-pulse soak interim baton closed §A + §B but the baton file wasn't updated, leaving phantom live work visible to the cold session. Mode path: 1C → 6 (no re-draft, no dry-run). Frontmatter gains `closed_sections` / `open_sections` lists; `baton_emitted` bus event gains `partial_closure: true` payload flag. Trigger: `/baton-close`, "mark section closed", "update the baton". Does not affect `/baton` or `/master-baton` flows.
- **v1.3.1 — 2026-05-10 (checkpoint validation before artifactization)** — Added `references/master-baton-checkpoint-validation.md` and linked it from the master-baton workflow/dependencies. Use it when an exploratory master baton has moved through matrix/grill into artifact creation or implementation planning. It instructs agents to run PRISM-lite consistency review, mark superseded pre-grill language as historical, split resolved decisions from still-open questions, verify live profile/workspace/event/source state, and name the next downstream artifact set before building on the baton.
- **v1.3.0 — 2026-05-10 (mode fork promoted to first-class)** — `/master-baton` added as an explicit trigger. "When to invoke" split into two subsections (session baton vs master baton). "The 5-step pattern" replaced with "The pattern (two modes)" — added mode fork table showing session path (0→1→2→3→4→5→6) vs master path (0→1M→2→3→6). Step 1A renamed to Step 1M with a clear skip directive. Step 1 header clarified as session-baton-only. Frontmatter description rewritten to lead with the two modes. No logic changes — purely surfacing the fork that existed in v1.2.0 but was buried.
- **v1.2.0 — 2026-05-09 (master-baton variant promoted)** — Added explicit master-baton trigger and Step 1A. Created `references/master-baton-pattern.md` with class-level guidance for macro batons spanning many sessions, compactions, repos, gates, or workstreams. Added anti-patterns for treating a master baton like a normal session prompt and for trusting prior summaries over live repo state. **Origin:** Atlas OS master-baton session surfaced N>=2 real macro-baton cases (`atlas-extended-sprint-master-baton.md` and `atlas-os-master-baton.md`), enough to promote the deferred v1.1 mutation into the skill body.
- **v1.1.0 — 2026-05-09 (master-baton-context addition)** — Added §2 Macro context anchor as a new conditional Required Section between One-line goal and 60-second context recap. When a master baton, North Star spec, sprint plan, or `CONTEXT.md` exists for the overarching project, the session baton must reference it in the anchor docs + add a "Macro context: where this session fits" section + add the doc to cold-start reads. Skip the section if no such doc exists. Self-Test extended 8 → 9 questions; target raised 6/8 → 7/9. Self-audit list extended with macro-doc check. Bus event `self_test_score` payload format updated to `<X/9>`. **Origin:** 2026-05-09 master baton authoring revealed the gap — operator (Jeremy) surfaced *"Worth adding to the baton skill? Something about referencing vision or context.md docs of the overarching project / sprint / direction?"* after consuming the master-baton-aware PR-A+B baton. Lightweight scope (option 🅐) elected; heavier "master-baton-creation as skill extension" deferred until N≥2 master batons exist (skill-doctor pattern: don't generalize from N=1).
- **v1.0.0 — 2026-05-08 PM (skill-doctor Round 1)** — Frontmatter completed (version, taxonomy_category, tags, author, license). Added `## Self-Test` (8-item rubric, target ≥6/8). Added `## Autoresearch` with baseline + Q13/Q14 status + 3 mutation candidates + per-invocation row in references/AUTORESEARCH-SELF-ASSESSMENT.md. Added `## Known Failure Modes` (5 items — state-inventory at context-exhaustion, silent operator flags, dry-run skipped under time pressure, baton-vs-session-end confusion, template drift). Added `## Dependencies`. Added Step 6 — `baton_emitted` bus event emission for daily-feedback observability. Closes operator-stated ask "we're going to get a lot of feedback on this daily and we should ensure we're logging improvements." Score 7/12 → 11+/12.
- **v1 — 2026-05-08 AM** — Skill extracted from the bus North Star session that produced the canonical example. Operator (Jeremy) named it `baton` for the passing-the-baton metaphor; short, evocative, single-word.
