---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)
- Manager wants it fixed NOW (systematic is faster than thrashing)

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully**
   - Don't skip past errors or warnings
   - They often contain the exact solution
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce Consistently**
   - Can you trigger it reliably?
   - What are the exact steps?
   - Does it happen every time?
   - If not reproducible → gather more data, don't guess

3. **Check Recent Changes**
   - What changed that could cause this?
   - Git diff, recent commits
   - New dependencies, config changes
   - Environmental differences

4. **Gather Evidence in Multi-Component Systems**

   **WHEN system has multiple components (CI → build → signing, API → service → database):**

   **BEFORE proposing fixes, add diagnostic instrumentation:**
   ```
   For EACH component boundary:
     - Log what data enters component
     - Log what data exits component
     - Verify environment/config propagation
     - Check state at each layer

   Run once to gather evidence showing WHERE it breaks
   THEN analyze evidence to identify failing component
   THEN investigate that specific component
   ```

   **Example (multi-layer system):**
   ```bash
   # Layer 1: Workflow
   echo "=== Secrets available in workflow: ==="
   echo "IDENTITY: $([ -n "${IDENTITY:-}" ] && echo SET || echo UNSET)"

   # Layer 2: Build script
   echo "=== Env vars in build script: ==="
   printenv IDENTITY >/dev/null 2>&1 && echo "IDENTITY in environment" || echo "IDENTITY not in environment"

   # Layer 3: Signing script
   echo "=== Keychain state: ==="
   security list-keychains
   security find-identity -v

   # Layer 4: Actual signing
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **This reveals:** Which layer fails (secrets → workflow ✓, workflow → build ✗)

5. **Trace Data Flow**

   **WHEN error is deep in call stack:**

   See `root-cause-tracing.md` in this directory for the complete backward tracing technique.

   **Quick version:**
   - Where does bad value originate?
   - What called this with bad value?
   - Keep tracing up until you find the source
   - Fix at source, not at symptom

### Phase 2: Pattern Analysis

**Find the pattern before fixing:**

1. **Find Working Examples**
   - Locate similar working code in same codebase
   - What works that's similar to what's broken?

2. **Compare Against References**
   - If implementing pattern, read reference implementation COMPLETELY
   - Don't skim - read every line
   - Understand the pattern fully before applying

3. **Identify Differences**
   - What's different between working and broken?
   - List every difference, however small
   - Don't assume "that can't matter"

4. **Understand Dependencies**
   - What other components does this need?
   - What settings, config, environment?
   - What assumptions does it make?

### Phase 3: Hypothesis and Testing

**Scientific method:**

1. **Form Single Hypothesis**
   - State clearly: "I think X is the root cause because Y"
   - Write it down
   - Be specific, not vague

2. **Test Minimally**
   - Make the SMALLEST possible change to test hypothesis
   - One variable at a time
   - Don't fix multiple things at once

3. **Verify Before Continuing**
   - Did it work? Yes → Phase 4
   - Didn't work? Form NEW hypothesis
   - DON'T add more fixes on top

4. **When You Don't Know**
   - Say "I don't understand X"
   - Don't pretend to know
   - Ask for help
   - Research more

### Diagnosis dari trace sebelum hipotesis (GEPA-4, fold-in batch 3, gepa-ai/gepa MIT, 16 Nov 2026)

Diagnosis kegagalan WAJIB membaca trace lengkap (pesan error, data
profiling, log penalaran) dan menghasilkan perbaikan TERTARGET atas
penyebab yang terdiagnosis; perbaikan yang tidak mengutip bukti trace
= tebakan dan tidak boleh dipromosikan. Saat mutasi, bawa serta (a)
fakta niche/domain yang ditemukan di trace, (b) strategi generalisasi
yang terbukti bekerja. Hipotesis ditulis SETELAH trace dibaca, bukan
sebelumnya. (Sumber: gepa `README.md`;
`src/gepa/strategies/instruction_proposal.py:155,164`.)

Rantai keputusan (ATL-4): format rantai observasi → aksi → validasi →
replan/stop yang tertaut ke ID tool call didefinisikan penuh di
`handoff-brief`, seksi "Rantai keputusan". Keputusan diagnosis di
skill ini direkam dalam format itu.

### Phase 4: Implementation

**Fix the root cause, not the symptom:**

1. **Create Failing Test Case**
   - Simplest possible reproduction
   - Automated test if possible
   - One-off test script if no framework
   - MUST have before fixing
   - Use the `superpowers:test-driven-development` skill for writing proper failing tests

2. **Implement Single Fix**
   - Address the root cause identified
   - ONE change at a time
   - No "while I'm here" improvements
   - No bundled refactoring

3. **Verify Fix**
   - Test passes now?
   - No other tests broken?
   - Issue actually resolved?
   - Use the `superpowers:verification-before-completion` skill before claiming success

4. **If Fix Doesn't Work**
   - STOP
   - Count: How many fixes have you tried?
   - If < 3: Return to Phase 1, re-analyze with new information
   - **If ≥ 3: STOP and question the architecture (step 5 below)**
   - DON'T attempt Fix #4 without architectural discussion

5. **If 3+ Fixes Failed: Question Architecture**

   **Pattern indicating architectural problem:**
   - Each fix reveals new shared state/coupling/problem in different place
   - Fixes require "massive refactoring" to implement
   - Each fix creates new symptoms elsewhere

   **STOP and question fundamentals:**
   - Is this pattern fundamentally sound?
   - Are we "sticking with it through sheer inertia"?
   - Should we refactor architecture vs. continue fixing symptoms?

   **Discuss with your human partner before attempting more fixes**

   This is NOT a failed hypothesis - this is a wrong architecture.

## Fold-in: repro gate, hypothesis log, bisect, sibling check

Source: JayRHa/agent-skills `debug-detective` (reproduce gate,
hypothesize with prediction-before-experiment, bisect fast-path,
sibling check, root-cause vs trigger distinction), MIT; via
EVALUATION-20261116.md section 4 (deep-read) and section 6 (fold-in
principles). Fit: this skill's four phases already follow the same
shape; the fold-in adds the missing gates — a hard repro gate before
leaving Phase 1, written predictions before experiments, a bisect
fast-path for regressions, and a sibling check before closing. The Iron
Law is unchanged.

### Repro gate (hardens Phase 1)

Do not leave Phase 1 until you can trigger the bug on demand, or you
have a measured failure rate written down (for example "fails 3 in 20
runs under load X"). If the bug is intermittent, quantify the rate first
and try to raise it — loop it, add load, shrink timeouts — before
proceeding. A bug you cannot reproduce is a bug you cannot confirm
fixed. Write the repro down: exact command/input, environment, and
failure rate.

### Hypothesis log: prediction before experiment (hardens Phase 3)

For each hypothesis, write the prediction BEFORE running the experiment:
what observation would confirm it, and what would refute it. Rank
candidate hypotheses by likelihood x cheapness-to-test and test the
cheapest discriminating one first. One variable per experiment. Refuted
hypotheses are progress — log them with their actual result; they shrink
the search space. Do not design experiments that can only confirm a
favorite theory.

### Bisect fast-path for regressions

When the symptom is "worked before this change", use `git bisect` with
the repro as the test command to find the offending commit. Bisect turns
a linear search into a logarithmic one. Carve-out (binding): never
bisect by rewriting or checking out inside a make.fun entry repository
or an XE canonical repository — bisect only in a disposable worktree or
copy, and never rewrite beacon-root history or canonical history.

### Sibling check (hardens Phase 4)

After the fix is verified, search the codebase for the same root-cause
pattern elsewhere before closing: the same root cause often manifests in
siblings. Distinguish root cause from contributing factors and from the
trigger — ask "why" until you reach something actionable, and confirm
you can explain the full causal chain from root cause to symptom. A gap
("and then somehow...") means the diagnosis is not done.

### Not folded (deliberate)

- The external `git_bisect_helper.sh` script: external scripts are never
  executed unread; plain `git bisect` with the repro command is enough.
- "Commit experimental changes on a scratch branch": forbidden for entry
  and canonical repositories; use a disposable worktree instead.
- A separate investigation-log template file: log hypotheses and results
  in the session/report, not in a new artifact format.

## Bounded investigation: hard stops + progress heartbeat (L-4)

Fold-in batch 2 (langgraph `TimeoutPolicy` + task timeouts, 16 Nov
2026). Before a long diagnosis, declare three bounds: a maximum number
of hypotheses/steps, a wall-clock cap, and an idle bound — the maximum
time without new evidence. Emit a progress heartbeat (newest evidence
+ current hypothesis) at intervals, or be treated as stalled. Hitting
a bound is a named terminal condition: stop, report which bound was
hit plus the last evidence and the current hypothesis, and ask. Never
silently extend a bound to keep going.

## Error taxonomy + never re-run a non-idempotent step blind (AP-1, split)

Fold-in batch 2 (apify/agent-skills `apify-integration-development`,
no LICENSE file, paraphrased CUSTOM prose, 16 Nov 2026). ONLY the
non-retry half is folded; the retry-on-429 half of the same source
line is REJECTED outright (see the fence below).

- Classify API errors into three categories: retryable, auth, or
  permanent. Show the error text the API actually returned, not a
  generic HTTP message. Hand errors to the reasoning consumer as DATA
  (JSON objects), never as raised exceptions — the consumer needs
  something to read and reason about.
- NEVER re-run a non-idempotent operation when the outcome is
  ambiguous — the duplicate is a real, side-effecting (and billable)
  operation. Verify state first (hash, nonce, status) before anything
  is sent again, per the ambiguous-send lesson (AGENTS.md: RPC SEND
  AMBIGU).
- FENCE (absolute): the upstream sentence on the same source line
  that prescribes retries with backoff on 429 and 5xx responses is
  REJECTED outright. 429 / rate-limit = hard stop, never retried.
  Retry stays limited to idempotent steps under an explicit contract
  (see precision-task-optimizer).

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Pattern says X but I'll adapt it differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when already tried 2+)**
- **Each fix reveals new problem in different place**

**ALL of these mean: STOP. Return to Phase 1.**

**If 3+ fixes failed:** Question the architecture (see Phase 4.5)

## your human partner's Signals You're Doing It Wrong

**Watch for these redirections:**
- "Is that not happening?" - You assumed without verifying
- "Will it show us...?" - You should have added evidence gathering
- "Stop guessing" - You're proposing fixes without understanding
- "Ultra-think this" - Question fundamentals, not just symptoms
- "We're stuck?" (frustrated) - Your approach isn't working

**When you see these:** STOP. Return to Phase 1.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## When Process Reveals "No Root Cause"

If systematic investigation reveals issue is truly environmental, timing-dependent, or external:

1. You've completed the process
2. Document what you investigated
3. Implement appropriate handling (retry, timeout, error message)
4. Add monitoring/logging for future investigation

**But:** 95% of "no root cause" cases are incomplete investigation.

## Supporting Techniques

These techniques are part of systematic debugging and available in this directory:

- **`root-cause-tracing.md`** - Trace bugs backward through call stack to find original trigger
- **`defense-in-depth.md`** - Add validation at multiple layers after finding root cause
- **`condition-based-waiting.md`** - Replace arbitrary timeouts with condition polling
- **RCA: escape point and labeled links (P-4).** Fold-in batch 2
  (PramodDutta/agentskills `rca-analyzer`, no LICENSE file,
  paraphrased CUSTOM prose, 16 Nov 2026). In a postmortem, label every
  link of the causal chain `evidence` or `hypothesis`, and never state
  a causal link the evidence does not back. Every RCA MUST answer the
  escape point: why testing and review missed it. Every corrective
  and preventive action carries how its landing will be verified. Use
  the five fishbone factor categories (People, Process, Tooling, Data,
  Environment) as a checklist, not a mandatory diagram.
