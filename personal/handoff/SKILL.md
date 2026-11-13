---
name: handoff
description: Use when the user hands over one or more tracker issues (Linear IDs, a label, a view) to fix without them in the loop, and wants proof at the end - "go fix these", "take PRI-12 and PRI-15", "work through the iPad bugs and show me". One branch and PR per issue, questions asked on the issue, video or screenshot proof per platform.
---

# Handoff

The user won't watch. They come back to PRs, proof and a short report. When in
doubt, ask on the issue and wait; a wrong guess costs them more than a pause.

The project's agent guide supplies the specifics: how to show and record each
platform, which checks are cheap, the tracker team, concurrency limits, and the
ship flow. Read it first.

## 1. Read every issue

For each issue: the description, every comment, and every image or video.
Write down in one line what "fixed" means and on which platform. If the issue
names no platform, use the one in its screenshots; if there is none, ask.

## 2. Ask instead of guessing

If anything would change what you build (two plausible readings, a design
choice, missing repro steps), don't start that issue:

- Comment on the issue with the specific question and the options you see,
  each with what you'd do. Mention the user.
- Put the issue in a waiting state the tracker supports (a "Needs input"
  label, or the guide's convention).
- Watch for the answer. With the `agentline` tools (see the
  `agentline-linear` skill), call `issue_watch` and the reply arrives as a new
  message; don't poll. Without them, if the harness can schedule work (for
  example T3 `schedule_task` with an interval), schedule a check of the waiting
  issues' comments every 20–30 minutes, bound to this thread, and delete it when
  nothing is waiting. Otherwise say in the report which issues are waiting.
- Move on to the next issue meanwhile.

## 3. Work each issue on its own branch

- One branch per issue, named after the issue, in the project's normal
  checkout. Add a worktree only when it's actually needed: issues worked in
  parallel, or a checkout with someone else's uncommitted changes in it. Stay
  within the guide's limit on parallel heavy work (builds, browser suites,
  simulators).
- Iterate on the code without rebuilding or rerunning tests after every edit.
  Build and test only once you are confident the fix is complete. If that run
  fails, go back to fixing, not to re-running.
- Then do what the ship flow requires: regression test for real bugs, the
  project's full check, review loop, PR with "Fixes <ID>". Don't merge unless
  the user said to.

## 4. Prove it

- On the issue's platform, record proof: a short video for anything with
  interaction or motion, a screenshot for static layout. Show the before state
  too when it's cheap (e.g. the main branch in the same scenario).
- Other platforms: do the **cheap** checks yourself and attach their proof.
  For **expensive** ones (the guide says which), don't run them; ask in the
  report whether the user wants them.
- Upload the proof to the issue as a comment that says what each file shows.
  Then delete every local copy. Proof never goes in the repo and doesn't stay
  on the machine.

## 5. Clean up

Push the branch, then remove any worktree you added, stop any servers you
started and close any devices or simulators you opened. The branch and PR stay.

## 6. Report

One table, then the open questions:

| Issue | Result | PR | Proof | Checked on |
| --- | --- | --- | --- | --- |

Result is fixed, waiting for an answer, or blocked (with why). End with the
expensive platform checks you can run if the user wants them.
