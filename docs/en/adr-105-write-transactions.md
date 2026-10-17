<p align="right"><b>English</b> · <a href="../fr/adr-105-transactions-ecriture.md">Français</a> · <a href="../../README.md">← Back</a></p>

# ADR-105 — Write transactions and policy as data (full text)

Second ADR published in full. It closes two gaps in the kernel: an approved write task used to write directly into my files (now: an **isolated worktree**, a diff, a **second approval bound to the diff's fingerprint**), and risk rules lived in code (now: a **sealed file** that can be read, simulated and tested, and that can only raise risk, never lower it).

*Translation of the French original, reproduced verbatim from the private repository's decision log (`DECISIONS.md`). File paths are those of the private repository.*

---

## ADR-105 — Kernel: write transactions (P1-2) and policy as data (P1-4)

**Status**: accepted, delivered on 2026-10-15. **Operator's choice**: “we finish the kernel, P1-2 and P1-4” (after a project review: consolidate the kernel before any extension). Audit `docs/architecture/07-agentic-runtime-audit.md` §9.4, §9.5, P1-2, P1-4.

**Context** — Two holes remained in the kernel. (1) Approval protected the ACTION, not the STATE: an approved write task wrote directly into the operator's tree, then verification observed after the fact; a failure left the files modified. The worktree only existed for the Coworking autonomous queue. (2) Policy was scattered across TypeScript; no rule could be read, simulated or tested as data.

### P1-2 — write transactions
- **WHAT**: `src/core/kernel/transaction.ts` (pure decision, opening, diff, integration, rollback, maintenance) orchestrated by `task-manager.ts`. Sequence: PRECHECK → SNAPSHOT (git worktree from HEAD, `%LOCALAPPDATA%\Hermes\worktrees\<id>`, branch `hermes/q-<id>`) → APPLY (the executor receives the worktree as `workspace.root`) → VERIFY (on the isolated result) → DIFF (Impact Preview) → APPROVE (diff fingerprint) → COMMIT (`integratePatch`: fingerprints of the operator's files, `git apply --check`, apply, no commit) or ROLLBACK (worktree discarded).
- **Eligibility** (`transactionDecision`, pure): `allowEdits` without `readOnly`, an executor with file access AND listed in `policies/transactions.json` (`hermes-agent`, `claude-code`, `cursor`, `shell`), a folder inside a git repository with at least one commit. Outside a transaction, by construction: `cv`/`salon` toolsets, Coworking steps (one tree shared by the mission, integrated by Coworking), a task with `branch` (planner), a folder outside git. A direct write says so: `rollback: { kind: "none", detail }`.
- **State machine**: two transitions added — `VERIFYING → WAITING_APPROVAL` (diff ready) and `WAITING_APPROVAL → COMPLETED` (diff integrated). COMPLETED remains unreachable without VERIFYING.
- **Second approval**: `approveTask` on an `awaiting-integration` transaction integrates instead of executing; `actionDigest` = fingerprint of the patch (what the UI transmits); a different `expectedDigest` → `ApprovalInvalidatedError`; patch altered on disk → refusal; operator's files modified since the diff → nothing is applied, the task stays pending. `high` risk → Windows Hello on the diff fingerprint. `approval.phase: "diff"` records the decision. `denyTask` discards the worktree.
- **Dry run**: `--dry-run` / `dryRun: true` (in the action envelope) = transaction stopped after the diff, patch kept in `memory/tasks/<id>/` (ignored by git), never integrated. Plain refusal at creation if the task cannot be isolated.
- **Impact Preview** (no LLM): files, lines +/−, bytes, tools actually called during the turn; bounded (`maxFiles`, `maxPatchBytes`), truncation flagged. Jarvis › Tasks: preview, fingerprint, “Integrate” / “Discard” buttons; CLI: `printTask`.
- **Maintenance**: `npm run hermes -- transactions [prune]`; a worktree whose task is finished or orphaned is removed, never a pending diff nor an autonomous-queue folder (no transaction attached). The disk is the source of truth.
- **Ledger**: `transaction.opened`, `transaction.diff`, `transaction.integrated`, `transaction.rolled_back`, `transaction.integration_refused`, `transaction.pruned`.

### P1-4 — policy as data
- **WHAT**: `src/core/kernel/policy-data.ts` + `policies/risk.json` (sealed: `policies/` is already in `security/seal-scope.txt`). Two effects only: `floor` (raises to `medium` or `high`, never `low`) and `deny` (motivated refusal through the PolicyDecisionPoint's `registerDenyRule`). Closed conditions: `executor`, `profile`, `requestedBy`, `allowEdits`, `readOnly`, `hasParent`, `toolProfile`, `workspaceInside` (aliases `@repo`, `@hermes-home`), `instructionMatches` (≤ 200 characters). Strict parser (unknown key, empty `when`, invalid regex, duplicate id = refusal).
- **Untouchable fixed rules**: `src/core/policy.ts` unchanged (git_commit and high → approval, medium → approval). Data can only add floors or refusals: “monotonic risk”.
- **Closed**: `policies/risk.json` unreadable → every task refused, clear reason; `doctor` “kernel (policy as data)” in `fail`. File absent → no data rule (the fixed base stays whole).
- **Simulation**: `explainPolicy` is THE function of enqueueTask (declared → derived → profile floor → data rules); `npm run hermes -- policy simulate "…"` calls it with the real PolicyDecisionPoint and the transaction decision, without creating a task. `policy show` lists the rules.
- **Rules shipped**: `formulation-destructive` (write + `rm -rf`, forced push, hard reset, `git clean -f`, `Remove-Item -Recurse`… → high, hence Hello); `profils-hermes-proteges` (no write under `HERMES_HOME/profiles`: they are deployed by scripts, under the seal).

- **INTERFACE**: `Task.transaction`, `Task.dryRun`, `TaskApproval.phase`; `EnqueueParams.dryRun`; HTTP `dryRun` in `POST /api/tasks` (an agent cannot request it); CLI `task --dry-run`, `policy [show|simulate]`, `transactions [prune]`; `doctor` “kernel (policy as data)”, “kernel (transactions)”; `KERNEL_INTERFACES` += `WorkspaceTransaction`, `PolicyData`.
- **SECURITY**: the operator's tree is never touched by an isolated task before their gesture; the approval covers what was written (observed), not what was announced; the turn's capability token (ADR-081) is issued with the worktree as root, so the PEP bounds the write to it. Tests: `HERMES_WORKTREES_DIR` isolated in `tests/helpers/isolated-memory.ts`.
- **TEST**: `tests/kernel-transaction.test.ts` (14: pure decision, Impact Preview, integration, refusals, rollback on failed verification, verification on the isolated result, dry run, dry-run refusal, empty diff, operator's files modified, fingerprint and patch altered, direct mode, `approveDiff: never`, maintenance); `tests/kernel-policy-data.test.ts` (11: parser, monotonicity, fixed rules, every rule of the real file, rule coverage test, floor and refusal through the real door, unreadable file closed, simulation = real); e2e `panels.spec.ts` “pending diff” (3 widths, axe, “Integrate” transmits the diff fingerprint).
- **ROLLBACK**: `"mode": "direct"` in `policies/transactions.json` (previous behaviour, tested); empty `rules` in `policies/risk.json`. Both files are sealed: re-seal after modification.

**Real proof (2026-10-15)**: `coder` task (`anthropic` brain), throwaway repository in the scratchpad. Approved instruction → real turn in `hermes/q-2c01296e` (`write_file` tool) → `file:notes/preuve.md` verification on the isolated result → 1-file diff +1/−0 pending; operator's repository verified intact at that moment → diff approved → file present, no commit, no worktree nor branch left. 19 events in the ledger, from `task.created` to `task.completed`. `npm test` **1348/1348**, two consecutive runs; `npm run ui:gate` green, 59 browser tests. Observed along the way: the transaction decision launched two git processes for EVERY write task (Coworking steps included), which slowed long suites under load (intermittent Coworking failures, passing alone); git is now only launched when the decision depends on it.

**Acknowledged gaps and next steps**:
- `git apply` follows `core.autocrlf` (true system-wide on this machine): the integrated file takes a checkout's line endings, like the Coworking integration.
- The 4242 dashboard must be restarted to serve this code (operator's gesture: never without their consent). The UI (`ui/dist`) is already rebuilt.
- Not yet: a transaction per Coworking mission (one worktree per workspace, integrated on “accept”); a saga for non-file effects (Notion, e-mail: the TIK executors already have `beginEffect`/`finishEffect`); `network`/`autonomy` policies (P2-3, P2-10); non-regression on the ledger's past decisions; a Jarvis screen for `policy simulate`.

---

> Reproduced **verbatim** (translated) from the private repository's decision log, without edits: file names, test counts and acknowledged gaps included.
