# baton

Three-mode skill for session continuity. Author cold-session-executable prompts, orient multi-session arcs, and close out completed work — all with the same discipline.

The "baton" is the artifact a cold session reads to pick up where the prior session left off. A good baton lets a fresh session walk the work end-to-end without needing the prior session in-context. A bad baton produces hours of "what does this mean?" plus accidental scope-creep plus missed verification.

## Three modes

| Mode | Trigger | When to use |
|------|---------|-------------|
| **Session baton** (`/baton`) | "write the baton", "/baton", "make a baton" | Session has concrete follow-on work for the next session |
| **Master baton** (`/master-baton`) | "master baton", "/master-baton", "cross-session baton" | Work spans many sessions, context compactions, repos, or gates — needs a stable arc doc |
| **Section closure** (`/baton-close`) | "/baton-close \<path\>", "mark section closed", "close section X" | Session completed some sections of a multi-section baton but not all — mark closed so cold sessions don't re-execute |

## What it does

**Session baton (`/baton`):**
- Captures **state inventory** — commits + decisions + execution + war stories + scope ceiling — before drafting
- Produces a **24-section continuation prompt** with cold-start reads, verification ledger, embedded execution commands, decision tree, and self-test
- Adds a **macro-context anchor** when a master baton, North Star spec, sprint plan, or `CONTEXT.md` exists for the overarching project
- Runs a 5-step pattern: state-inventory → draft → self-audit → cold-session dry-run → close gaps → emit observability event

**Master baton (`/master-baton`):**
- Writes a **macro orientation doc** a cold session loads before the per-session baton — stable across compactions, repos, gates
- NOT an execution prompt — orients the arc, identifies the next safe slice, inventories authoritative files and phases

**Section closure (`/baton-close`):**
- Reads an existing multi-section baton and **marks completed sections** with inline `STATUS: CLOSED` markers
- Updates frontmatter with `closed_sections` / `open_sections` lists so the cold session's section selector is accurate
- 5-minute operation: no re-draft, no dry-run — just state-transition markers + bus event

All modes emit a `baton_emitted` bus event (with mode-appropriate payload) and log to a per-skill autoresearch self-assessment file so improvements compound.

## Quick start

```bash
# Session baton — pick up where you left off
"write the baton"
"/baton"

# Master baton — orient a multi-session arc
"/master-baton"
"master baton"

# Section closure — mark completed sections without re-drafting
"/baton-close ~/path/to/existing-baton.md"
"mark section A closed"
```

The session baton writes to `<workspace>/agents/<agent>/data/<YYYY-MM-DD>-next-session-<topic>-prompt.md` (gitignored by Atlas convention).
The master baton writes to a durable docs namespace (versioned, not gitignored).

## When to invoke

**Session baton:**
- Session shipped artifacts the next session needs to act on (specs, ADRs, partial implementations)
- Session surfaced operator-decision flags that next session must resolve
- Session ran PRISM that produced a mechanical-fix list or NEEDS-WORK verdict
- Session built half a feature and needs the other half

**Master baton:**
- Work spans many sessions, context compactions, repos, or gates
- A North Star doc, vision doc, or sprint plan needs to stay load-bearing across sessions
- Operator says the shape is still unknown and wants a durable orientation artifact first

**Section closure:**
- Session completed one or more sections of an existing multi-section baton but NOT all sections
- The baton file would mislead a cold session because completed sections still appear as live work

**Skip when:**
- Session completed all its work end-to-end (use `session-end` only — no baton needed)
- Session was open-ended exploration with no concrete next-session shape
- Session is a 1-2 message Q&A

## Compatibility

Designed around Atlas agent fleet conventions:

- A per-session scratchpad directory (`data/` in Atlas agent workspaces; gitignored)
- A `memory/` diary file format with YAML frontmatter and an index file
- A bus event emitter (`emit-event.sh` in Atlas)
- An agent-common conventions doc (`agent-common.md` in Atlas)

Portable to other agent runtimes if the host fleet has equivalents for the four points above. Shell + bash are the only hard tool dependencies; everything else is convention.

If your runtime lacks a bus, the `baton_emitted` step degrades gracefully — the skill writes the autoresearch row regardless, so the daily-feedback signal still compounds.

## Configuration — adapting to your fleet

The skill assumes these shapes. Map each to your fleet's equivalent:

| What the skill expects | What your fleet provides |
|---|---|
| Per-session scratchpad at a path like `<workspace>/agents/<agent>/data/<filename>.md` | A writable, gitignored directory for session-scoped scratch |
| Diary file at `<workspace>/agents/<agent>/memory/<topic>.md` with YAML frontmatter | Your fleet's session-diary format (the skill assumes `name`, `description`, `type` keys) |
| Diary index at `<workspace>/agents/<agent>/memory/MEMORY.md` | Your fleet's per-agent diary index file |
| A bus event emitter (default invocation: `<bus-emit-script>`) | Your fleet's lightweight event emitter — or `:` (no-op) if no bus exists |
| A universal-conventions doc (the skill auto-imports `<universal-conventions-doc>`) | Your fleet's equivalent doc, or skip the import line |
| A workspace-schema doc (referenced as `<workspace-layout-doc>`) | Your fleet's equivalent — purely informational |
| Skill installation root at `<skill-dir>/<skill-name>/SKILL.md` | Wherever your runtime loads skills from |
| `pending_approvals` SQL example | Substitute any DB/table relevant to your fleet |

Nothing in the algorithm requires a specific fleet — only the worked examples do. When in doubt, follow the structural pattern (24 required sections, 5-step process, 9-question self-test) and substitute paths. The skill author's machine is a single-Mac multi-agent setup; the conventions baked in reflect that.

## Self-Test (target ≥7/9)

| # | Question |
|---|----------|
| 1 | Did I do the state-inventory FIRST (Step 0) — commits + decisions + execution + war stories + scope ceiling? |
| 2 | Does the 60-second recap actually answer "where am I, what just happened, what do I do next" in one screenful? |
| 3 | Are operator quotes preserved verbatim (not paraphrased)? |
| 4 | Does each operator-decision flag have full trade-off context + recommended answer? |
| 5 | Does each step have an embedded verification command (1-line bash/curl/sqlite3) the cold session can run? |
| 6 | Did I dry-run the prompt as a cold session and apply ALL friction-point fixes? |
| 7 | Are war stories from THIS session connected to next-session steps where they apply? |
| 8 | Does the prompt include a self-test (boolean checklist defining "done") for the cold session? |
| 9 | Did I check for a macro/vision/context/master-baton doc and either (a) reference it in §2 + cold-start reads, or (b) confirm none exists? |

Below 6/9 is high probability of cold-session misdirection — iterate before shipping.

## Anti-patterns

The skill explicitly catches and refuses these failure modes:

- "Read the prior session diary" as the only context — diaries are narrative, not action plans
- "Continue from where we left off" without specifying *where*
- Lists of work without effort estimates
- Operator-decision flags without trade-offs or recommended answers
- No DO-NOT list (cold session scope-creeps without one)
- No verification commands ("make sure X works" is not actionable)
- Stale numbers inline (cite the verify-script + invocation, never bake-in)
- No self-test (cold session has no way to know it's done)

## Pairs with

- `session-end` — closing-doc ritual; baton is one optional output of session-end
- `pre-pr` — sibling Process & Workflow skill with the same Q14 / autoresearch pattern
- `verification-before-completion` — "evidence before assertions" is the principle baton operationalizes for the next session
- `boil-the-ocean` — the discriminator for "is the cold-session prompt complete or am I shipping 80%"

## File structure

```
baton/
├── SKILL.md                # Skill instructions + frontmatter
├── LICENSE.txt             # MIT
├── README.md               # This file
└── references/
    ├── template.md                         # 24-section structural template
    ├── example.md                          # Pointer to canonical 580-line example
    ├── master-baton-pattern.md             # Required structure for master batons (10 required + 4 conditional sections)
    ├── master-baton-checkpoint-validation.md  # Checkpoint pass before downstream artifacts depend on master baton
    ├── twin-anchor-runtime-campaign.md     # Pattern for paired master-baton + vision/spec doc
    ├── role-crystallization-pattern.md     # master-baton → matrix → grill sequence
    └── AUTORESEARCH-SELF-ASSESSMENT.md     # One row per baton authored — daily improvements log
```

## Limitations

1. **State-inventory at context-exhaustion.** If the session ran 4h+ and you're invoking baton at the very end with low remaining context, state-inventory quality drops. Mitigation: invoke earlier, not as a death-rattle ritual.
2. **Operator silent on decision flags.** When the session ends with N flags surfaced but no answers, the baton's flag section becomes "options + my recommendation" — that's correct, not a gap. Anti-pattern is dropping the flag.
3. **Dry-run skipped under time pressure.** The most-skipped step (Step 3 dry-run is "re-read what you just wrote" which feels redundant). Self-Test Q6 is the gate.
4. **Skill mistaken for session-end.** baton is OPTIONAL; session-end is mandatory. Don't author batons reflexively — only when there's concrete carry-forward.
5. **Frontmatter / sections drift over versions.** As host fleet conventions evolve, the 24-section template may go stale. The autoresearch log is the canary — friction-point trend up means the template needs review.

## License

MIT — see LICENSE.txt.

## Author

Jeremy Jannielli ([@jeremyknows](https://github.com/jeremyknows))

Skill extracted from the 2026-05-08 Atlas bus-North-Star session that produced the canonical 580-line / 46.8KB cold-start-ready continuation prompt.
