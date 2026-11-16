---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Evidence ladder (E0–E6) + negative-result discipline

Fold-in from simota/agent-skills `_common/EVIDENCE_LADDER.md` (MIT),
16 Nov 2026, mapped to our oracles. A claim carries the rung of its
WEAKEST premise, never its strongest inference.

| Rung | Evidence | Notes |
| --- | --- | --- |
| E0 | Model claim | Unverified assertion, however confident. |
| E1 | Static | Read code/docs; no execution. |
| E2 | Local execution | Ran it once, output seen. |
| E3 | Automated tests | ONLY if the oracle is independent — a test written from the same context as the code is E0 wearing a green check. |
| E4 | Independent tests | Property / mutation / fuzz / differential runs. |
| E5 | Integration | End-to-end in a realistic environment. |
| E6 | Production | For web3 research: on-chain proof (Transfer event, ownerOf, receipt). For entries: server-verified result. |

Rules:

- **Finding nothing is a claim.** A negative result names the search and
  its scope, or it is E0. Three parts, none optional: what was searched,
  what a hit would have looked like, what was NOT covered. "No issues
  found" with unstated scope is silence, not absence.
- **"Ask a different model" is not independence.** Independence comes
  from a different evidence MECHANISM (compiler, property, differential
  run, on-chain read), not a different vendor.
- **Status binds to a revision.** If HEAD moved after the run, the
  status is stale, not passing. Status never upgrades without a new
  observation.
- **Anti-patterns that void a claim:** laundering (derived-from-assumed
  reported at the derived rung); test-shaped assertion ("added tests"
  never run — E0 in the shape of E3); a check nobody has watched fail
  (indistinguishable from one returning zero unconditionally, and worse
  than absent because being reported as enforcement stops anyone looking
  for the missing enforcement).
- Every claim report carries a `not_verified` field: what was NOT
  verified. Record what the tests prove AND what they do not.
- **Triple-match identifiers (fold-in batch 2, KalarisLabs
  `citation-verification`, 16 Nov 2026).** A resolving identifier
  alone proves nothing. A reference is verified only when its
  identifier resolves to a canonical record whose key fields all
  match (paper: title + first author + year; contract: address +
  chain + name/symbol; transaction: hash + chain + parties/value) —
  and the cited content is opened to confirm it supports the claim.
  Unmatched references are labeled UNVERIFIED, never quietly kept.
- **Synthesis citations (fold-in batch 2, bigrzz/agent-memory
  `reflection-evidence.md`, 16 Nov 2026).** A synthesized insight
  cites at least two distinct source records, re-read before the
  synthesis is recorded. A citation that merely names a supplied
  record does not prove the cited text supports the claim — check
  support in the text. An uncited cluster member is not evidence for
  the result. Abstaining is a valid result.
- **Survei fakta 4 ember (AG-1).** Format pemisahan fakta-terberi/terverifikasi vs fakta-yang-harus-dicari vs fakta-yang-harus-diturunkan vs tebakan beralasan didefinisikan penuh di `handoff-brief`, seksi "Survei fakta 4 ember + brief sebagai kontrak". Terapkan survei itu sebelum klaim yang premisnya mencampur fakta dan tebakan.
- **Disiplin kompresi + penomoran sitasi (ODR-5).** Aturan menggabung temuan (ulangi info kunci verbatim, format output tetap, satu nomor sitasi per URL unik tanpa gap) didefinisikan penuh di `deep-research`, Phase 4. Verifikasi laporan yang menggabung temuan memakai disiplin itu.

## Grader two jobs + burden of proof (A-F2)

Fold-in batch 2 (anthropics/skills `grader.md`, 16 Nov 2026). The
checker has two jobs: grade the output, and critique the check itself.
A passing grade on a weak assertion is worse than useless — it creates
false confidence. The checker must flag assertions that are trivially
satisfied and important outcomes no assertion checks. When uncertain,
the burden of proof to pass is on the expectation: uncertain = fail,
not pass. Extract implicit claims and verify them separately from the
stated expectations. A good assertion is discriminating: it passes
when the work genuinely succeeds and fails when it does not.

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

**Mandatory distinctions before concluding (K-8).** Fold-in batch 2
(KalarisLabs `scientific-critical-thinking` + `peer-review`, 16 Nov
2026). Before any "proven"/"failed" claim, distinguish: data (what was
observed) from interpretation (what it means); correlation from
causation; statistical significance from practical importance;
exploratory from confirmatory findings; what is known from what is
uncertain; evidence against a claim from evidence for the null. Apply
consistent standards: same criteria across all cases, never stricter
standards for findings you dislike, judgments based on methodology,
not results. Reporting completeness is not design quality — never
convert missing checklist items into an automatic score.

**Pemeriksaan batas atas generasi tanpa side effect (DSPY-6, fold-in
batch 3, stanfordnlp/dspy MIT, 16 Nov 2026 — KONSEP BERPAGAR; API
Assert/Suggest sendiri DEPRECATED di sumbernya).** Untuk langkah
GENERASI MURNI tanpa side effect (draf teks, draf kode yang belum
dieksekusi/dikirim): periksa output terhadap constraint tertulis; bila
gagal, beri umpan balik spesifik dan regenerasi, maksimal N percobaan
yang dinyatakan di muka; bila tetap gagal, BERHENTI dan laporkan (pola
Assert) atau catat kegagalan dan lanjut best-effort (pola Suggest),
sesuai yang dideklarasikan. PAGAR MUTLAK: pola ini TIDAK PERNAH
berlaku untuk langkah side-effectful — wallet, submit, kirim, klaim,
bid — yang tetap memakai gerbang preview + ya eksplisit dan larangan
retry non-idempotent. (Sumber: dspy
`docs/docs/learn/programming/7-assertions.md:3,22,37,39`.)

**Snapshot evaluasi privacy-reduced (ATL-6, fold-in batch 3,
AgentTraceLab MIT, 16 Nov 2026).** Snapshot/artefak evaluasi TIDAK
BOLEH memuat argumen tool mentah, hasil tool mentah, atau state
checkpoint — yang disimpan adalah fingerprint SHA-256 (untuk deteksi
duplikat) + ringkasan bukti berbatas. Sumber data evaluasi dibuka
READ-ONLY. (Sumber: AgentTraceLab
`docs/deepresearch-integration.md`.)

## Double verification of state changes (PW-2)

Fold-in batch 2 (lackeyjb/playwright-skill `SKILL.md`, 16 Nov 2026),
generalized from login to every state change. A state change (login,
navigation, form submit, UI mint) is proven by TWO independent
observations — for example the navigation/URL AND a visible
post-action element — never by one signal. Do not claim success
without checking the resulting page or state. A first async render is
not proof: wait for the final state before concluding (see the Element
lesson in AGENTS.md).

## Approval-gated actions (fold-in batch 2, M-1)

Fold-in from microsoft/agent-framework ADR 0006 (accepted) + ADR 0049
(proposed), 16 Nov 2026, restated as procedure. Applies wherever an
action needs approval before it runs (wallet actions: see
`agent_gated_freemint` for the full protocol):

- **ID-paired request/response.** An approval request carries an id,
  the exact action, the exact arguments, and an explanation. The run
  STOPS at the request; continuation is a new invocation carrying the
  response with the SAME id. Every decision is logged (audit trail).
- **A missing, late, or expired answer never authorizes execution.**
  An answer arriving after the request was rejected or expired cannot
  authorize the old call — a new request must be made.
- **Execute the saved request.** Execution uses the exact call from
  the SAVED request/preview, never a caller-supplied replacement.
- **Approval points are policy, not agent discretion.** WHEN approval
  is required is decided by standing rules in advance. The upstream
  sentence letting the agent decide when approval is required (ADR
  0006:89) is REJECTED outright — it never enters this collection.

## Gate vocabulary: pause + named response (L-2)

Fold-in batch 2 (langgraph `types.py` interrupt + prebuilt interrupt
config, 16 Nov 2026). A gate is a pause + a surfaced value (the full
preview) + an expected response schema. Every gate NAMES the actions
allowed for that gate: accept / reject / edit-then-accept / skip —
there is no generic confirm. Resume is an explicit command carrying
the response value AND the gate id; without both, execution does not
continue. Because resume re-executes the node's logic, a gate for a
NON-IDEMPOTENT step (sign, broadcast, submit, claim) MUST sit BEFORE
the side effect, never inside logic that will be re-executed. An
approval never authorizes re-execution of an effect that already
happened.

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness
