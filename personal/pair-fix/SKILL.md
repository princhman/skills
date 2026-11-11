---
name: pair-fix
description: Use when the user is present and steering - testing on a device, sending screenshots of bugs, saying "just fix", "this is broken", "this should...". Fast fix-and-show loop with no tests, CI waits or review rounds.
---

# Pair fix

The user is in the loop and is the tester. Your job is the shortest path from
their report to them seeing the fix.

## Each report

1. **Understand it.** Look at the screenshot or video closely: which screen,
   which platform, what's wrong. If two readings are plausible, ask one short
   question; otherwise start.
2. **Record it** if the project keeps a tracker (its agent guide says where).
   One issue per bug, with the screenshot, before or while you fix.
3. **Fix it in the checkout the user is looking at.** No side worktrees.
4. **Make it visible the cheapest way** the project guide lists for that
   platform: usually a running dev server that reloads on save. Don't build,
   type-check or run tests just to show a change. When only a device build
   works, batch several fixes into one install.
5. **Reply briefly:** what you changed, where to look, anything you're unsure
   of. Then wait for the next report.

## Rules

- **No tests, CI waits or review loops** unless the project guide makes an
  exception for an area (for example, a fragile subsystem that always needs a
  regression test).
- **One platform:** fix it where the user is testing. Don't port or polish for
  other platforms until they say it's right.
- **Stay small.** If a report turns out to be a redesign or a deep bug, say so
  and offer to hand it off (`handoff` skill) instead of sprawling.
- **Delegating:** for parallel reports, delegate each fix with "fix only, don't
  test, don't wait for CI" and the platform. Read each result before telling
  the user it's done.

## When the user is done

The session's fixes are uncommitted or on a branch. Ask whether to ship them;
if yes, follow the project's ship flow (tests, review, PR, CI).
