---
name: delegate
description: >
  Turns a recurring real-world job into a written, self-verifying playbook
  (SKILL.md) the assistant can run without re-explaining it. Invoke with
  /delegate, or when the user describes a job they do repeatedly and wants it
  captured — phrases like "make this repeatable", "write a runbook/SOP/
  checklist for X", "I do this every week/month", "turn what we just did into
  a playbook". Interviews the user, writes the playbook, runs the job once,
  saves a reusable toolbox, and adds an evidence-based Definition of Done —
  all without stopping for approval between steps, except two named
  exceptions. NOT for authoring general-purpose Claude Code skills, prompts,
  or agent tooling — use skill-creator for that.
---

# /delegate — turn a repeated job into a runnable, self-verifying playbook

> **Invoke with `/delegate [optional: one-line job description]`**

## Step 0 — resume or start fresh

Before anything else, check whether a matching `[job-name]/` folder already exists (the current project's `.claude/skills/`, then `~/.claude/skills/`). If found, read its `SKILL.md` and `runs.md` and continue from there instead of restarting:

- No `SKILL.md` → start at step 1.
- `SKILL.md` exists but is missing a required section → step 2.
- No `files/INDEX.md` → step 4.
- Definition of Done has fewer than 5 checks → step 5.
- `runs.md` shows a check (or, for a batch job, any single item's row for a check) with a first-pass failure and no final result yet recorded → finish that check's (or that item's) retry cycle (step 5) before doing anything else — don't treat the run as complete and don't start a new one, even if the rest of the batch's rows are done.
- Everything present and no check mid-retry → run the job under the existing Definition of Done; check `runs.md` for declined delegation proposals so they aren't re-proposed too soon.

## What this skill does

Runs the full five-step pipeline end to end:

1. **Interview** — ask at least 8 questions, one at a time, waiting for each answer, before writing any file. Cover:
   - what triggers the job, and how often
   - the inputs, and exactly where each one lives — for anything secret (password, API key, signed URL, token) record only *where to find it* (env var name, keychain item, "ask the user"), never the value
   - whether the job is one thing end-to-end, or a batch of similar independent items (per-client, per-region, per-file) — batches can run in parallel, see "Running in parallel" below
   - the steps in the order the user really does them, and which are irreversible (sends, posts, publishes, deletes, overwrites)
   - every decision the user makes, and the rule behind it
   - what the user checks before calling it finished, and what a bad version looks like — both feed the Definition of Done in step 5, don't let them go to waste
   - edge cases that have gone wrong before
   - the tone or standard the finished thing has to hit

   Batch closely related questions into one turn where natural. End with one closing question — "Anything else before I write it?" — a "no"/"done" ends the interview. If the user says "do as you see fit" / "zrób jak uważasz" at any point, stop asking immediately, list each uncovered topic with the assumption you'll use, and proceed. Do not write any files before the interview ends one of these two ways.

2. **Write the playbook** — create a folder and write `SKILL.md` from this skeleton:

   ```
   ---
   name: <slug>
   description: <one line, including the user's own trigger phrases from "what triggers the job">
   ---
   Purpose (one line)
   When to use this
   Inputs
   Steps (numbered, plain language, no jargon)
   Decisions (if X then Y)
   Definition of done — placeholder: "filled in step 5 after the first run"
   Edge cases
   ```

   Default location: `<project>/.claude/skills/<slug>/` if the job is tied to one project, otherwise `~/.claude/skills/<slug>/`, so the playbook is invocable as `/<slug>`. Only ask if neither default obviously fits — this is one of the two standing-rule exceptions below.

3. **Run it once** — execute the job using the fresh SKILL.md, show the result. If a step is irreversible (flagged in step 1), stop right before that action, show exactly what would happen, and get a one-word go before it fires — everything up to that point runs straight through. If the job is a batch of independent items, use parallel agents (see "Running in parallel"). Write outputs under `[job-name]/runs/<date>/`; for anything destructive, work on a copy rather than the only copy of real input data.

4. **Toolbox** — save reusable artifacts by default whenever the run produced any file, template, or reusable structure; skip only for genuinely one-shot jobs (ask only if that's unclear — the other standing-rule exception). If saving: put artifacts in `[job-name]/files/` with descriptive names (no dates/versions in the filename), turn specifics into `[placeholders]`, and keep one *scrubbed or synthetic* filled-in example — real secrets, credentials, or client PII never become the reference example unless the user explicitly approves it, logged in INDEX.md. Update SKILL.md so each step that needs a file points at it by name, and log it in `files/INDEX.md` (filename, purpose, date, last-reused date). Never save one-off outputs, secrets/API keys, or unapproved drafts — this applies to every file the skill writes, not just `files/`. Re-run the job (same irreversible-action gate as step 3) and report, per saved file, the exact path read and how it was used — not just "reused" or "rebuilt".

5. **Definition of Done** — replace the step-2 placeholder with 5–10 checks specific to this job. At least half must come directly from what the user said they check before calling it finished and what a bad version looks like (step 1) — don't invent checks that only confirm the output looks like what was already produced. Every check must be verifiable with evidence outside the assistant's own opinion (a source, a link that loads, a screenshot, a test run, a count, a value checked against a reference file). No vague quality words ("clear," "professional," "high quality"). Show the finished check list once for a one-line confirmation.

   From then on, before showing any output: run every check and record the **first-pass** result (before any fixing) to `[job-name]/runs.md` — one row per run per check: date, run number, first-pass result, final result, evidence, delegation split in force, orchestration mode (serial / pipeline / parallel). For a batch job (see "Running in parallel"), add an **item** column and log one row per run per check *per item* instead of one row per check — a check that fails on some items and passes on others must stay visible as a partial failure, not collapse into a single verdict for the run. Fix what fails, re-run each failed check at most twice, then report one line per check (or per item, for batches) as `first pass: fail (reason) → fixed → pass`, or `unverified` if still unconfirmed — never report a check as passing on "should be fine." If more than two checks fail on the first pass, stop and explain which part of the process caused it rather than just patching the output.

## The standing rule: run straight through

Do not pause between steps 2–5 asking "should I continue?" — proceed automatically once the interview is done. There are exactly two exceptions to *that* rule, both named above: confirming folder placement in step 2 when neither default fits, and confirming the toolbox save in step 4 when reusability is unclear. Anything else that's genuinely ambiguous gets folded into one clarifying question, asked once — not a fresh pause per step.

This is separate from the irreversible-action gate (step 3, and its batched form in "Running in parallel"): that gate isn't a pipeline-continuation pause, it's a standing safety check that applies every time a send/publish/delete/overwrite is about to fire, run or not. It never counts against or adds to the "two exceptions" above.

## Running in parallel

If the interview establishes the job processes a batch of similar, independent items (one email per client, one report per region, one file per record), run it with parallel agents once the batch has **4 or more items**; below that, serial is simpler and the overhead isn't worth it. Use the Workflow tool: `pipeline(items, item => agent(...))` for independent per-item work, or `parallel(items.map(...))` only when a later step genuinely needs all items' results together (e.g. deduping across the batch before a shared summary). Record which mode ran (serial / pipeline / parallel) in `runs.md`'s orchestration-mode column, so a later change in failure patterns can be traced to a mode switch and not mistaken for a content problem.

If any item hits an irreversible step (flagged in step 1), collect every pending irreversible action across the whole batch into one summary and get a single go/no-go before any of them fire — never one confirmation prompt per item.

Write this into the generated SKILL.md's Steps so re-runs use it too. Keep it proportionate — a batch under 4 items doesn't need this.

## The diligence-reshapes-delegation loop

Track failures per Definition-of-Done check across runs using `runs.md` (written in step 5) — not from memory, since a later session won't have this conversation's context. A "repeat" counts even if the surface symptom differs, as long as the same check is the one failing (e.g. a formatting check that failed once on wrong filename and once on wrong date is still the same check failing twice — don't reset the counter just because the mistake looked different). A check counts as failed for this loop if it failed on the **first pass** of a run, even if step 5's retry budget then fixed it before the report was shown. For a batch job, a check counts as failed for the run if *any* item's row shows a first-pass failure that run — a check failing on 3 of 10 items is one failure for this loop's counter, not zero; the per-item breakdown in `runs.md` is what you cite when proposing the reshape, to show whether the failure is isolated to certain items or systemic.

If the same check fails on its first pass in two consecutive logged runs, stop before patching it a third time. Tell the user which part of the *process* likely caused it, and propose revising the task split in SKILL.md (e.g. move a step from "assistant does this" to "human does this," or "assistant drafts, human verifies this specific part") — don't just keep fixing the output. Present the proposed change before applying it. A repeated failure is a signal the delegation was wrong, not just that the execution was wrong.

- **If the user accepts the change:** apply it, append a row to `runs.md` marking that check's counter reset (a reset is a logged row, not a deletion), and log it (old split → new split, date, reason) in a `## Delegation history` section of SKILL.md.
- **If the user declines the change:** log the decline in `runs.md` (date, run number, reason if given) and ask how they want to proceed for this run (patch and continue, skip the check, or stop). Do not re-propose the same restructuring again until `runs.md` shows the check failing on first pass twice more under the *current* split.
- **If a check fails on first pass twice again after a delegation change was already logged for it:** the reshaped split was also wrong. Say so explicitly, show the failure history for that check from `runs.md` (original split, what changed, still failing), and this time propose stopping automation on that check entirely and handing it fully to the human, rather than proposing a third variant of the split.

## Extending to a family of related jobs (a "series")

If the user is building several related playbooks (e.g. one per region, one per client, one per product line), check whether a shared master-data file or reference file already exists for the family before drafting a new one — don't invent decisions (names, symbols, categories, tone) that a shared source file has already settled. If found, add it as a required input in each job's SKILL.md and treat cross-checking against it as a standing Definition-of-Done step, not a one-off.

## Notes

- This mirrors the three-prompt process the user originally set up manually (job → playbook, toolbox save, definition of done), now split and hardened into the five steps above plus the standing resume check in step 0.
