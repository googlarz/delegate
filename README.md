# /delegate

A Claude Code skill that turns a recurring real-world job into a written, self-verifying playbook (`SKILL.md`) the assistant can run without re-explaining it.

Interviews you about the job, writes the playbook, runs it once, saves a reusable toolbox, and adds an evidence-based Definition of Done — including a delegation-reshaping loop that catches when a repeated failure means the human/assistant task split is wrong, not just the output.

## What it does

1. **Interview** — asks at least 8 questions about the job (trigger, inputs, steps, decisions, what "done" looks like, edge cases) before writing anything.
2. **Write the playbook** — generates a new `SKILL.md` for the job, invocable as its own slash command.
3. **Run it once** — executes the job immediately, with a safety gate before any irreversible action (send, publish, delete, overwrite).
4. **Toolbox** — saves reusable templates and artifacts, with secrets and PII kept out.
5. **Definition of Done** — 5–10 evidence-based checks (no vague "looks good"), logged per run so failures are tracked over time.

If the same check keeps failing across runs, the skill stops patching the output and proposes changing *who does what* instead — the diligence-reshapes-delegation loop.

For jobs that process a batch of similar independent items (one email per client, one report per region), it can fan the batch out across parallel agents via the Workflow tool.

## Install

Copy `SKILL.md` into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/delegate
curl -o ~/.claude/skills/delegate/SKILL.md \
  https://raw.githubusercontent.com/googlarz/delegate/main/SKILL.md
```

## Use

```
/delegate write me a runbook for closing out the monthly expense report
```

Or just describe a job you do repeatedly and want captured — the skill will pick it up.
