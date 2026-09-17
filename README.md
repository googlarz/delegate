# /delegate

**Teach Claude a job once. It remembers exactly how you do it — and checks its own work every time after.**

## The problem this solves

You've got tasks you do the same way over and over — closing out the monthly expenses, writing a client update, cleaning a spreadsheet, running the same checklist before you send something out. Every time, you either do it yourself or re-explain the whole thing to Claude from scratch.

`/delegate` fixes that. Describe the job once, answer a few questions about how you actually do it, and Claude turns it into its own command — say `/monthly-expenses` — that you can run any time, without ever re-explaining it.

## What makes it different from just asking Claude to "remember this"

- **It asks before it assumes.** Claude interviews you about the real steps, the judgment calls you make, and what a *bad* result looks like — not just what a good one looks like. Vague answers get a sharper follow-up question instead of a guess.
- **It checks its own work before showing it to you.** Not a vague "this looks good" — real evidence: a number that matches your records, a file that exists where it should, a link that actually opens.
- **It's honest when something isn't going well.** If the same thing keeps going wrong run after run, it doesn't just keep quietly trying again — it tells you plainly: "this part keeps failing, maybe it should stay your call instead of mine," and lets you decide.
- **It handles big or messy jobs on its own.** If a step turns out to be more complicated than expected, it breaks it into smaller pieces by itself and works through them — doing independent parts at the same time to save time — without you having to spell any of that out.
- **It builds up a reusable toolbox.** Templates, checklists, anything worth using again gets saved for next time — with your passwords, client names, and other personal details kept out of it.

## Example

> "I close out our monthly expense report the same way every month — pull the numbers, check them against the bank statement, flag anything over €500, write a summary email."

Say that once. Claude asks a handful of follow-up questions (where the numbers come from, what "flag" means exactly, who the email goes to), then hands you back a ready `/monthly-expenses` command you can run every month from then on — one that gets more reliable over time, not less.

## Install

Needs [Claude Code](https://claude.com/claude-code). Copy the skill into your skills folder:

```bash
mkdir -p ~/.claude/skills/delegate
curl -o ~/.claude/skills/delegate/SKILL.md \
  https://raw.githubusercontent.com/googlarz/delegate/main/SKILL.md
```

## Use

```
/delegate write me a runbook for closing out the monthly expense report
```

Or just describe a job you do repeatedly and want captured — the skill will pick it up on its own.
