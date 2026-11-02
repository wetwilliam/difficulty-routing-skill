---
name: difficulty-routing
description: >-
  Use when the user brings an idea or multi-step task to delegate: split it into
  small steps and route each to a subagent at the lowest reasoning effort
  (low/high) that can do it reliably, to cut cost without losing accuracy.
---
# Difficulty routing

Idea: most steps are easy. Send easy steps to cheap/low-effort workers and save high effort for the few hard ones. A bigger model at low effort usually beats a small model at max effort; never crank effort on a weak worker to compensate.

## 1. Split
- Break the idea into the smallest steps that each have one clear output and a check (file exists, number found, test passes, link opens).
- Mark dependencies. Independent steps can run in parallel; dependent ones run in order.
- Skip steps nobody asked for (ponytail: least work that fully solves it).

## 2. Grade each step
| Level | Typical steps | Executor effort |
|---|---|---|
| Easy | fetch a known page, scrape a table, format/convert data, rename, run a given command, draw a chart from given numbers, follow a fixed procedure | `low` |
| Medium | search and pick sources, summarize, compare a few options, write a small script, light debugging | `high` (default) |
| Hard | open-ended research, conflicting sources, design decisions, judging correctness, multi-step reasoning, anything high-stakes | `high`, and keep the judgment with yourself |

Rules of thumb:
- Decisions already made and the step is mechanical -> low.
- Unknown difficulty -> high.
- Fact-checking or final judgment never goes to low.

## 3. Dispatch
- Batch adjacent easy steps into one low-effort executor brief with exact inputs, outputs and success checks, so it needn't make judgment calls.
- Give each brief only the context that step needs.
- Reuse one executor per workstream rather than spawning many.

## 4. Check: free rules first, cheap verdict second, escalate last
Three tiers, cheapest first (from microsoft/TokenEconomizer's loop guard):
1. **Free rule gate.** Check each result with plain rules, no model call: the stated check passed, link returns 200, file/number present, numbers fall in a plausible range, no contradiction with other results, the same error not repeated 3+ times, the worker not past ~2.5x the usual length for this kind of step. All pass -> accept, stop here.
2. **Short verdict only when a rule trips.** Ask for a minimal judgment, not a redo: one line or tiny JSON like `{"ok": false, "why": "...", "confidence": 0.8}`. Do it yourself if quick, or with a low-effort worker for mechanical checks. Never ask for long explanations here.
3. **Escalate only on a confident failure.** If the verdict says not ok with confidence >= 0.7, redo just that step once at high effort with a one-line hint ("try a different approach: ..."). Low confidence -> treat as ok but flag it as unchecked in the delivery. Never retry low repeatedly; never escalate the whole task for one bad step.

Also:
- When a step's check already passed (e.g. tests green, data found), tell the worker to wrap up; don't let it keep polishing.
- Keep a one-line log: step, level, effort, which tier caught it, escalated or not. Move step types that keep tripping rules up to high next time; remember repo/site facts that caused repeated failures and put them in future briefs.

## 5. Deliver
Combine results, lead with the answer, and say in one line which parts were estimated or unchecked.
