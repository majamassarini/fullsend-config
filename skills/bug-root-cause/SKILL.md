---
name: bug-root-cause
description: >-
  REQUIRED for every issue that reports misbehavior (a bug, something that
  "doesn't work", "misbehaves" or "stays on"). Read it before choosing the
  triage action: it says how to trace each symptom into the code, possibly
  in another repository, and when the result must be `prerequisites` or
  `split` instead of `sufficient`.
---

# Bug root cause

Triage of a bug is not done when the problem is clear: it is done when each
symptom is traced to the code that produces it, or you have stated what you
could not trace. A comment that only lists "areas to investigate" for a
developer is not a triage result.

## 1. Load the context skills

- `repo-map`: where each kind of behavior lives, and how to read the other
  repositories with `gh`.
- Any skill about this repository's own components (for example
  `homino-config` and `appliance` in homino). Use the procedure they give
  to find the affected component's definition and wiring.

## 2. Trace every symptom

Make a list of the distinct symptoms in the issue and its comments. For each
one:

1. Find the component in this repository (its definition, type and wiring).
2. Follow it into the code that implements the behavior, in whichever
   repository that is. Read the actual functions involved: the state
   transitions, event handlers, triggers or queries, not just the module
   list.
3. Write down: symptom → repository, file, function → why that code
   produces the symptom (quote the condition or line), or "not verified"
   and what you would need.

A cause you did not read in the code is a hypothesis: say so.

## 3. Choose the action from where the causes are

- **All causes in this repository:** `sufficient`. Put the traced causes
  in `root_cause_hypothesis` and a concrete `recommended_fix` (file and
  change), not "investigate X".
- **Cause, or part of the fix, in another repository, and this issue can't
  be fixed until it lands:** `prerequisites`, with a `create` entry for
  that repository. Its `body` explains the root cause there, with file,
  function and evidence, for a developer of that repository. Say in
  `comment` what, if anything, will still be needed here afterwards.
- **Independent parts in different repositories:** `split`, one sub-issue
  per part, each with `repo` set to where its fix lives and the traced
  evidence in its body.
- **Not enough information to trace a symptom:** `insufficient`, asking
  the single question that would let you trace it.

The coder agent can only change the repository it runs in and can't open
issues elsewhere. If triage doesn't file the issue in the right repository,
nobody will.

## 4. Comment

The `comment` must contain the per-symptom trace from step 2, short: one
line or bullet per symptom with the repository and file. Don't restate the
issue.
