# agent-playbook

How I run AI coding agents on a long, technical side project: hardware and software, one person, since July 2026. More than 150 agent sessions so far.

I'm not an engineer. The agents write all of the code. What I've built is a way of working that lets me trust what they hand back, and this page is that way of working. The project stays private; nothing here depends on it.

## The setup

- **A planning chat (Claude).** It reads the state of the project, reasons with me, and writes the brief for the next work session. It never writes code and never commits.
- **Claude Code.** It executes one brief per session on a Linux desktop, commits, pushes, and writes a report.
- **ChatGPT Codex.** A blind second opinion. When I needed to know whether a set of reference answers held up, I gave it the same questions without our answers and compared them element by element.
- **A laptop as the cockpit, a desktop as the workshop.** Git is the only channel between them. A small script syncs the laptop, delivers new briefs to the desktop, and writes down which commit it saw and when.

## The loop

```
brief (a file)  →  one-line launch  →  agent works, commits, pushes, reports
      ↑                                                   ↓
next brief  ←  decision (mine)  ←  verification from the files, not from the report
```

Every step leaves a file behind, so every step can be checked afterwards.

## What a brief looks like

A brief is a disposable file, launched with one line: "Execute the file …". Once launched it doesn't change. If I change my mind, the change goes in a separate addendum, and the addendum only counts once it has actually reached the machine.

```
§0  Opening gate. Clean tree, expected commit, inventory of inputs.
    If anything is missing, stop and list it.
§n  One step per task. Thresholds are written BEFORE the numbers exist.
§X  What we do NOT do (files not to touch, thresholds not to move).
§Z  Close: typed commits, push, the list of files to bring back.
    "A wall is declared, not bypassed."
```

That last sentence does more work than any other line in the brief. It's what turns "I found a workaround" into "here is exactly where I got stuck", and the second one is the report I can act on.

## Verification: the report is a claim, not proof

- **Recount everything** the report says: counts, maxima, commits. Use a command, not your eyes.
- **Check append-only logs by the hash of the prefix.** The old part has to be byte-identical, and the new part has to be exactly what was declared.
- **Anything changed that the report doesn't mention is a finding**, even when it's harmless.
- **Take the "before" from git history**, not from memory, and not from asking someone.

## Failures that turned into rules

Each of these happened, and each left a rule behind.

1. **"Written", but it wasn't.** A write reported success and the bytes on disk were the old ones. → Reread and compare the hash before saying a file exists. If a rewrite fails silently, use a new name.
2. **A gate printed PASS after computing FAIL.** The verdict line was a fixed string. → The verdict has to read the value that was just computed, and every new gate ships with a test run that must fail.
3. **`$?` after a pipe reads the last command, not the one you meant.** A stop condition never stopped. → Use `PIPESTATUS`, and prove the branch fires.
4. **A long job died with the session that launched it.** Hours of compute were lost. → Detach properly (on Linux, `systemd-run --user`), pass the environment explicitly, and save partial results at set intervals.
5. **Compound shell commands triggered approval prompts** and stalled a session nobody was watching. → Absolute paths, `git -C`, no `cd` in any form.
6. **A follow-up written in a report was never picked up.** Reports get written once and never reread. → Open items go into a live tracker, each with an observable trigger ("when someone next opens file X"), not a date.
7. **Zero is the only number a wrong search returns without an error.** `find` on a tree of symlinks said "0 files". → State an absence together with the command that produced it.
8. **A threshold moved after seeing the data isn't a threshold.** → Pre-register it, and commit it before the first number exists. The commit timestamp is the proof.
9. **A stall detector watched log silence and killed a job that was working.** The step was silent by design. → Measure CPU time, not log lines.
10. **A time budget produced a verdict stronger than the facts.** A rule with two outcomes, "works" and "impossible", met a step that simply hadn't finished. → Every rule needs one outcome per possible result: succeeded, failed, didn't finish within budget.
11. **Long chats forget.** → One chat per session cycle. When it closes, it writes a handover document, and the next chat's first message is a gate that proves it read that document.
12. **A summary is not the source.** A confident answer built on a one-line summary turned out to be wrong. → Before telling someone what a decision implies, reread the decision itself.

## What I'd tell a team adopting coding agents

- **The bottleneck moves from typing to verifying.** Budget for it.
- **Write the stop conditions before the work**, and make the agent say where it stopped.
- **Make every claim checkable from files**: hashes, counts, timestamps.
- **Keep the decisions human.** The agent decides how; the person decides what, when to stop, and what it's worth.

## Templates

**Opening gate**

```
[ ] working tree clean, pull fast-forward only
[ ] HEAD == expected commit (named in the brief)
[ ] inputs: expected / present / conforming. Stop if any is missing
[ ] tool versions printed
```

**Closing**

```
[ ] typed commits, messages passed from a file
[ ] push, then local == remote
[ ] report lists: what was done, what was NOT, walls hit, measured durations
[ ] "files to bring back": exact paths, repeated at the end of the report
```
