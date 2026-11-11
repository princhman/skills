---
name: perf-investigate
description: Use when something is slow, laggy, janky or freezes - "typing lags", "it freezes when", "switching notes is slow", "beachball", "2 tabs in a row is laggy". Measure first in a production-like build, fix the measured cause, and show before/after numbers.
---

# Perf investigate

Perf bugs come back when fixes are guesses. No change without a measurement
before and after.

The project's agent guide says how to build, run and profile each platform.
Read that section first; this skill is the method.

## 1. Pin the scenario

- Exact steps, data size and platform. Ask the user for the note or file if
  the slowness depends on content, and use a copy of their data, never the
  original.
- Reproduce it yourself. If you can't (for example a real-device-only freeze),
  tell the user exactly what to run while they reproduce it, and wait.

## 2. Measure in a realistic build

- Dev servers and debug UI frameworks are much slower than release and will
  mislead you. Use the guide's production-like build.
- Simulators and emulators use the host CPU; their timings don't count. Use the
  real device for mobile numbers.
- Record a baseline: a number that matches what the user feels (time to open,
  frame time while typing, length of the freeze), repeated 3+ times.

## 3. Profile and find the cause

- Freeze or hang: sample the stuck process (and the web content process for
  web views) while it's stuck. The hot stack usually names the cause.
- Slow but not stuck: a time profiler over the scenario; for web code, a JS CPU
  profile.
- Headless browser tests miss OS paths (text input, accessibility, IME). If the
  profile points into the OS, reproduce in the real app.

## 4. Fix and re-measure

- Fix the measured cause, not neighbours that look slow.
- Re-run the same scenario with the same build type and data. Report before and
  after numbers side by side. If the number didn't move, say so and revert.
- Add a regression guard where the project has a benchmark harness.

## 5. Other platforms

A perf fix in shared code can change other platforms. Do the cheap re-checks
the guide lists; ask before expensive device runs.

## Report

Scenario, build type, device; before → after with the run count; the cause in
one sentence; what's still unmeasured.
