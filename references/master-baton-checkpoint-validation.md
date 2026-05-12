# Master Baton Checkpoint Validation Pattern

Use this when a long-running arc moves from exploration/grill into artifactization or implementation. The goal is to prevent stale pre-decision language from becoming a false premise for downstream artifacts.

## When to apply

- A master baton started as exploratory, but the session has since crystallized a role/architecture stance.
- The user says to validate documents before creating downstream artifacts.
- A cold session is likely to inherit compacted context and could accidentally redo resolved work.
- The work involves multiple tracking surfaces (vision doc, master baton, session baton, CONTEXT.md, implementation plans).

## Pattern

1. **Run PRISM-lite / wiki-style doc consistency review**
   - Compare the master baton, CONTEXT.md, session baton, and nearby matrix/strategy docs.
   - Look specifically for stale sections named “open questions,” “next safe slice,” “not settled,” “current unknown,” “do not implement yet,” and “self-test.”
   - Classify each item as: resolved, drafted-but-not-final, still open, or implementation-not-live.

2. **Verify live state before downstream claims**
   - Check whether referenced profiles/workspaces/repos/events actually exist.
   - If source paths are cited, verify them against the current checkout.
   - If an event/bus/API contract is accepted design but not registered/live, label it explicitly as not live.

3. **Patch, don’t duplicate**
   - Update the authoritative master baton rather than creating a new master baton.
   - Add a dated “current checkpoint” near the top.
   - Mark older instructions as historical when they have been superseded.
   - Split stale “open questions” into “resolved / working decisions” and “still open.”

4. **Preserve implementation uncertainty precisely**
   - Good: “Role stance is drafted; physical profile is not verified.”
   - Bad: “Everything is ready.”
   - Good: “Current observed registry path is X; verify topology before editing.”
   - Bad: “Canonical registry is somewhere in Atlas.”

5. **Name and index the next artifact set**
   - After checkpoint validation, list the downstream artifacts that can now be created.
   - Add created artifacts to the master baton's authoritative file table with accurate status labels: working draft, proposed design, active, live, etc.
   - After artifact creation, update language from “not created yet” to “exists as working draft; refine, don’t recreate.”
   - Separate safe draft artifacts from implementation artifacts that require live verification.

6. **Run a focused Round 2 after patching**
   - Ask a reviewer/subagent to verify only the prior high/medium findings and newly patched text.
   - Patch residual stale items immediately, especially “still open” items that became verified during review.

## Common fixes

- Replace “discover what X should become” with “current working stance is Y; remaining unknowns are A/B/C.”
- Replace “next session should grill” with “treat grill as complete unless new contradiction found.”
- Replace broad “not settled” lists with “drafted but not final-artifactized.”
- Add “implementation status verified DATE” for profiles, workspaces, event registrations, and source paths.
- Add exact current paths for registries/configs instead of vague canonical references.

## Why it matters

Master batons are often read by cold sessions as authoritative. If they retain pre-grill uncertainty after decisions crystallize, the next session will redo work or build on false premises. The checkpoint pass turns the baton back into a map rather than a fossilized transcript of an earlier uncertainty state.
