# Precision Task Optimizer

Method for challenges, jobs, quests, competitions, and bounty tasks: maximize the probability of the best valid result. Adapted from a playbook the user pasted on 13 Nov 2026. This playbook never overrides the user's standing gates: wallet actions still need a preview plus their explicit yes per action, submissions still need their per-version decision where the standing rules require it, and no platform rule, eligibility restriction, or security control is ever bypassed.

## Identify the task, then extract requirements

- Name the task category first (quiz, coding, logic, Web3 quest, trading competition, creative, research, freelance assignment, multi-stage) and adapt the method to it; never apply one strategy blindly to every task.
- Before solving, extract and record: main objective, exact instructions, eligibility, scoring criteria and reward structure, submission format, required evidence, deadline and timezone, restrictions and prohibited actions, minimums and maximums, and hidden or easily overlooked conditions.
- Keep an internal requirement checklist. Distinguish mandatory requirements from optional bonuses and recommendations. A small overlooked instruction can invalidate the whole submission.
- Compact form of that checklist, a task spec (salvaged 15 Nov 2026 from evaluating the pasted universal-task-intelligence playbook — the playbook itself was rejected as a standalone skill: ~90% duplication of existing skills plus authorization language weaker than the standing gates): OBJECTIVE (what must be achieved), SUCCESS CRITERIA (how success is measured), RESTRICTIONS (what is prohibited), RESOURCES (tools, APIs, files, accounts available), DEADLINE (when, with timezone), DELIVERABLE (what must be submitted or produced), RISK (what could cause failure). Do not invent missing requirements; mark them unknown.

## Competitive intelligence

- Investigate what determines success: official rules and documentation, previous winners and successful submissions, evaluation criteria and judging patterns, common participant mistakes, public competitor strategies, historical results, and rule or scoring changes.
- Prioritize official sources and cross-check important facts with independent sources. Never fabricate competitor information, previous winners, or judging preferences.

## Generate and compare strategies

- Generate at least three viable strategies when the task allows: conservative (correctness, reliability, compliance), balanced (winning potential vs time, effort, risk), and aggressive (higher potential reward within all rules and risk limits).
- Evaluate each on: probability of success, expected score or reward, time required, cost and resources, complexity, failure probability, rule compliance, and repeatability. Pick the best expected outcome for the actual objective, not the most complicated approach.
- **Proposal berbasis riwayat (OPT-4, fold-in batch 3, optuna/optuna MIT, 16 Nov 2026, konseptual).** Proposal strategi/varian berikutnya harus MEMAKAI RIWAYAT: bandingkan wilayah parameter dari trial-trial terbaik vs trial-trial sisanya, dan usulkan nilai yang lebih mirip wilayah terbaik. Larangan: proposal acak tanpa merujuk riwayat trial setelah ≥5 trial selesai. (Sumber: `optuna/samplers/_tpe/sampler.py:88-94` — model atas nilai terbaik vs sisanya, rasio `l(x)/g(x)`.)

## Simulate and test before executing

- Test the proposed solution with the method that fits: manual verification, calculations, Python simulations, backtesting, sandbox testing, coding tests, multiple hypothetical scenarios, and historical comparisons.
- Test at least three scenarios where applicable: best case, expected case, worst case. Run sensitivity analysis to find which assumptions or variables matter most.
- For technical tasks, test edge cases, invalid inputs, boundary conditions, and failure handling. For quizzes, verify the answer independently and eliminate misleading alternatives. For trading competitions, evaluate drawdown, execution costs, liquidity, and risk-adjusted performance. For creative work, evaluate originality, clarity, relevance, audience fit, and judging criteria.
- Never claim a simulation or test was performed unless it actually was.
- **Metrik + baseline sebelum optimasi (DSPY-1).** Urutan mengikat metrik tertulis → baseline terukur → varian → promosi didefinisikan penuh di `leaderboard-strategy-optimizer`, seksi "Loop satu siklus". Rencana simulasi/tes di skill ini menyatakan metriknya sebelum varian apa pun dijalankan.

## Adversarial review, then optimize

- Act as a strict independent evaluator before accepting a solution: what could make it wrong, which requirement might be overlooked, is there a better reading of the instructions, does it violate any rule, is the evidence sufficient, could an evaluator reject it, is there a simpler and more reliable approach, what happens if the assumptions are wrong.
- Fix critical weaknesses before proceeding. Then optimize: remove unnecessary steps, fix weak reasoning, improve accuracy and clarity, strengthen evidence, and satisfy every mandatory requirement. When two solutions perform similarly, choose the one with lower execution risk and fewer dependencies.

## Execution control

- Run a final verification checklist before submitting or acting: answer correct, every mandatory requirement met, format correct, evidence valid, within deadline, all links, files, and references working, reversibility understood, and explicit user approval obtained where required.
- Never claim completion without checking the actual result. For transactions, payments, account changes, or irreversible actions, show the full action details and obtain explicit user approval first.
- **Progress ledger koordinator (AG-2).** Checklist refleksi tiap putaran delegasi — terpenuhi penuh? mengulang? progres maju? siapa berikutnya? instruksi persis apa? — didefinisikan penuh di `handoff-brief`, seksi "Progress ledger koordinator". Kontrol eksekusi di skill ini memakai ledger itu tiap putaran.

## Learn after every task

- Compare actual performance against expectations and record: selected strategy, predicted result, actual result, errors and missed requirements, time spent, successful techniques, failed approaches, and improvements for future tasks. Use memory and persistent records.
- Do not assume one successful strategy works for every future challenge.
- **Logbook per generasi (DEAP-5, fold-in batch 3, parafrasa custom, LGPL — tanpa kutipan sumber).** Tiap eksperimen ditutup dengan logbook satu baris per generasi: nomor generasi, jumlah evaluasi, statistik ringkas (terbaik/median), dan juara generasi itu. Silsilah juara final dapat dilacak mundur sampai seed lewat catatan asal-usul (DEAP-4, di `leaderboard-strategy-optimizer`). (Pointer: deap `deap/tools/support.py`, kelas logbook/statistik.)

## Quality gate and reporting

- Before final submission, report each axis separately as PASS/RISK — requirement compliance, accuracy and correctness, strategic effectiveness, evidence and verification, execution readiness, efficiency — never added, averaged, or weighted into one overall score (revision 16 Nov 2026: the old weighted 25/25/20/15/10/5 composite targeting 95/100 was pseudo-precision with no data source behind its weights, and one number hid which axis was weak). The evidence axis is graded by the evidence rung reached (see verification-before-completion, E0–E6). The risk axis BLOCKS completion when RISK: fix it, or earn BLOCKED with a named failed alternative — never average risk away against a well-written report. Internal percentages never gate anything: pseudo-precision is not evidence.
- Report concisely but completely: task analysis, competitive strategy, simulation results, best solution with justification, risk and weakness review, execution checklist, and final status (ready, needs revision, awaiting approval, submitted, or verified completed). Always distinguish verified facts, estimates, assumptions, and simulated results.
- Short status shape for quick reports, where a track does not already define its own format (same 15 Nov 2026 salvage): STATUS (completed / partial / blocked), RESULT (what was achieved), VERIFICATION (what evidence confirms it), ISSUES (what remains unresolved), NEXT ACTION (the most useful next step).

## Loop gates and chain discipline (fold-in, 16 Nov 2026)

Source: simota/agent-skills `_common/LOOP_PRECONDITIONS.md` +
`nexus/SKILL.md` (MIT), adapted. Never launch an ungated loop:

1. **Verifiable completion oracle** — a command or predicate where exit 0 ⟺ done. Subjective goals ("improve the UX") are rejected. Conversions: split oracle, frontier exhaustion, coverage set, differential parity.
2. **Hard-stop bound** — iteration cap / budget / timeout, enforced EXTERNALLY, never by agent self-assessment. Default: loop ≤ 3 cycles in-session.
3. **Maker ≠ checker** — the generator does not grade its own work; an independent evaluator (separate subagent session) decides DONE. For make.fun, the checker must not read entry files before writing its judgment (transcript authorship rules).
4. **Persistent memory** — state lives outside the conversation (files / DB / git), not in context.
5. **Drift awareness** — quality can decay across iterations even when tests keep passing. Bound the loop + read-a-sample discipline. Never assert ROI as fact.

The denominator is frozen before cycle 1. Each failed precondition maps to a named anti-pattern (loopmaxxing, overbaking/runaway, nodding loop, amnesiac loop, silent erosion) — convert it with one focused question, or stop.

Chain discipline (Nexus salvage — only these three principles; the full Nexus orchestration machine is REJECTED as unproven bureaucracy):

- **Minimum viable chain.** Use the minimum chain of steps; never spawn an agent to re-check another's output — that is a sequential VERIFY step, not a sibling.
- **Two identical failures ⇒ diagnose, not retry.**
- **The bar never moves to meet the output.** `BLOCKED` is earned by a named failed alternative; hard core precedes easy polish; no artifact ships with TODO/stub residue.

Mandatory guard: retry is for IDEMPOTENT steps only (reads, checks, compiles). Retrying non-idempotent operations (signing, broadcast, submit, claim) stays FORBIDDEN by the binding bans. Nexus "L1 max 3" retry is NOT folded raw.

### Hard three-dimensional bounds + heartbeat (L-4)

Fold-in batch 2 (langgraph `TimeoutPolicy` + task timeouts, 16 Nov
2026), strengthening the hard-stop bound above. Every loop declares
THREE bounds before running: (1) a maximum step count, (2) a
wall-clock cap per attempt that progress never refreshes, (3) an idle
bound — the maximum time without observable progress. Long work that
emits no natural progress MUST emit a heartbeat (a periodic progress
log) or be treated as stalled. Hitting a bound is a NAMED terminal
condition: report which bound was hit, the last step, and the last
state. Raising a bound is an explicit, recorded decision — never a
silent tweak so the loop "finishes".

### Bounded polling + ceilings the model cannot widen (AP-4)

Fold-in batch 2 (apify/agent-skills `apify-integration-development`,
no LICENSE file, paraphrased CUSTOM prose, 16 Nov 2026). Every poll
loop carries a bound: the operation's own timeout, extended by a
grace margin, under an absolute ceiling that always applies — never
an unbounded loop. On a
non-terminal status, surface the resumable id so the next poll does
not start from zero. Cost and quota ceilings are set by the principal
(user/developer) and CANNOT be widened by the model.

### Anggaran delegasi + terminasi (fold-in batch 3, 16 Nov 2026)

**Anggaran delegasi supervisor (ODR-4, open_deep_research MIT).** (a)
Bias ke satu agen, kecuali paralelisasi jelas menguntungkan; (b)
refleksi SEBELUM delegasi (rencana) dan SESUDAH tiap hasil (cukup? apa
hilang? lanjut atau selesai?); (c) batas keras iterasi supervisor +
batas paralel per iterasi, dinyatakan di muka; (d) scaling rule: fakta
sederhana = 1 agen, perbandingan = 1 agen per elemen, subtopik
non-overlapping dengan instruksi mandiri lengkap. (Sumber:
`src/open_deep_research/prompts.py:92,105-106,109`,
`src/open_deep_research/configuration.py:73,103`.)

**Taksonomi termination condition (AG-3, microsoft/autogen MIT — pola
saja; framework-nya DIARSIPKAN dan ditolak).** Di awal loop
delegasi/koordinasi apa pun, deklarasikan minimal 3 kondisi terminasi
dari taksonomi: (a) batas iterasi/pesan, (b) batas waktu, (c) sinyal
berhenti eksplisit (stop/handoff/eksternal); tambah batas token bila
loop panjang. Tanpa deklarasi ini, loop tidak boleh jalan. (Sumber:
`python/packages/autogen-agentchat/src/autogen_agentchat/conditions/_terminations.py:24-559`,
11 kelas.)

**Study note + ledger trial (OPT-3).** Format study note (ruang
pencarian dinyatakan + ledger trial + definisi frontier Pareto)
didefinisikan penuh di `leaderboard-strategy-optimizer`, seksi "Aturan
eksperimen". Loop di skill ini memakai format itu untuk eksperimen
pencarian apa pun.

## Retry and idempotency discipline (fold-in, batch 2, 16 Nov 2026)

Source: microsoft/agent-framework ADR 0052 + ADR 0028 (both proposed),
restated as procedure. This states WHEN retry is allowed; it loosens
nothing — the binding bans below stay in force:

- **Exactly-once is not realistic.** Distinguish sent, accepted by the
  platform, and observed by the user. A lost connection does not mean
  no side effect happened — the server may already have performed it
  even though the result never arrived. Cancelling a stream is not an
  application rollback.
- **Retry follows an explicit contract only.** A retry is allowed only
  where the tool's explicit contract provides it: an idempotency key,
  a bounded attempt count, explicit terminal states, stable run /
  continuation / attempt identifiers, and duplicate suppression for
  persisted attempts.
- **FENCE (absolute).** 429 / rate-limit = hard stop, never retried.
  Non-idempotent operations (signing, broadcast, submit, claim) are
  never retried. Retry stays limited to idempotent steps (reads,
  checks, compiles) under an explicit contract — see the mandatory
  guard in Loop gates above. M-3 only formulates when retry MAY run;
  it never authorizes a retry this collection forbids.

## Checkpoints and resume (fold-in L-1 + M-4, batch 2, 16 Nov 2026)

ONE section, two sources combined: langgraph checkpoint README (L-1)
+ microsoft/agent-framework `_checkpoint.py` docstring / ADR 0041 /
ADR 0022 (M-4), restated as file-based procedure. No framework code is
adopted; no dependency is installed.

- **Snapshot per step.** After each step, write a checkpoint file
  (input, output, evidence) in one ID'd thread. Writes from steps that
  completed successfully are kept when a later step fails, so resume
  does not re-run steps whose outputs are already recorded.
- **Committed state only.** A checkpoint holds only committed state;
  pending, uncommitted changes are never included.
- **Lineage + plan hash.** Each checkpoint carries a checkpoint id, the
  previous checkpoint id (lineage chain), and a hash of the plan /
  definition it was built under. Ordering comes from the lineage chain
  (and timestamp) — never from a bare step counter, which can repeat
  across pauses.
- **Resume needs an exact three-part match:** recorded output + state +
  checkpoint id. Reject the checkpoint if the plan hash changed — a
  checkpoint built under a different plan is not a resume point.
- **FENCE: resume of an impactful action requires fresh independent
  verification** (receipt, on-chain status, or server state). Local
  notes can go stale after a VM restart or crash; the external source
  is the truth for side effects, never the local file.

## Flow decomposition: shared state, reducers, named routing (L-3)

Fold-in batch 2 (langgraph `state.py` + `types.py` Send, 16 Nov 2026),
restated as procedure. Before running a multi-step flow, declare the
shared state: its keys plus ONE merge rule (reducer) per key —
last-write-wins OR append-to-list. Two writers with no declared merge
rule are a design defect, not a detail to sort out later. Routing
decisions are NAMED predicates returning the next step's name (or
END); no unwritten implicit "move to the next step" transition.
Fan-out runs the same step in parallel with different inputs; fan-in
aggregates with the declared reducer. Parallel subtasks receive their
own copy of the state, never shared writable state. FENCE: parallelize
READ/CHECK paths ONLY. Signing, broadcast, submit, and claim paths
are never parallelized or fanned out — one path, one order, one
authorization.

## Operating principle

Understand, investigate, strategize, simulate, test, challenge your own solution, optimize, verify, execute, learn. Never optimize for speed alone; optimize for the highest realistic probability of success with the lowest avoidable error rate. The objective is the strongest valid result possible, not merely a finished task.
