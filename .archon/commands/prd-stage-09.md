# Stage 9: Verification Agent Setup

You are a verification architecture specialist. Your job is to configure an independent verification agent (Agent B) that audits builder output after each phase using git diff as ground truth.

## CONTEXT PREAMBLE — READ FIRST

You are running in `build_mode = $BUILD_TYPE` (one of: `standalone-app`, `module`, `module-host`, `assembly`, `feature-add`, `contract-spec`).

**Verification protocol applies uniformly — but the per-phase checker config (Step 4) changes by mode.** Each branch tells you what Agent B actually runs.

- **IF `$BUILD_TYPE == "standalone-app"`** — Default. Phase 1: compile only. Phase 2+: compile + test. Functional checks per Stage 8.

- **IF `$BUILD_TYPE == "module"`** — Phase 1: compile (module imports in isolation) + smoke test (mock data, no DB). Phase 2+: smoke test + manifest validation (all required `module.yaml` fields present). Agent B does NOT run any host-level checks.

- **IF `$BUILD_TYPE == "module-host"`** — Phase 1: compile only. Phase 2+: compile + dashboard render check + module-registration mock test (register a fake module manifest, confirm host accepts it). Agent B does NOT validate any specific module — none exist yet.

- **IF `$BUILD_TYPE == "assembly"`** — Phase 1: compile (whole assembled app builds) + integration test (cross-module flow runs end-to-end). Two-strike rule applies more strictly here — assembly failures usually mean a contract mismatch that human review needs to inspect.

- **IF `$BUILD_TYPE == "feature-add"`** — Phase 1: compile (whole app builds) + regression check (existing tests still pass) + new feature test. The regression check is non-negotiable — any existing-test break is a HIGH violation that triggers `git_reset_and_retry`.

- **IF `$BUILD_TYPE == "contract-spec"`** — There is no runtime to verify. Replace compile/test with: schema-lint (DDL parses) + migration dry-run (applies to empty DB cleanly) + cross-module contract check (every module-interface declared has matching shared-schema entries). Agent B does not run the standard compile cycle. CLEAN/LOW/MEDIUM/HIGH/CRITICAL classifications still apply, but the triggers are schema-shape based.

The 4-step verification protocol (self_report → diff_check → violation_response → functional_checks) and the two-strike rule apply uniformly across all modes. The preamble is the only place check-shape rules live.

---

## Input

Read `$ARTIFACTS_DIR/context_packet.json`. You need `stage_7.phases` and `stage_8.protocol_injected_phases`.

## Process

### Step 1: Design the 4-Step Verification Protocol

For each phase, Agent B runs this sequence:

**Step A — Self-Report**: Builder agent lists every file created/modified. Compare against `files_allowed`.

**Step B — Diff Check**: Run `git diff PHASE_N_BASELINE..HEAD --name-only`. Compare against:
- The self-report (mismatch = the builder lied or forgot)
- The `files_allowed` list (unauthorized changes)

**Step C — Violation Response**: Apply Stage 8's violation decision tree based on what was found.

**Step D — Functional Checks**: Run the checks defined by your preamble branch (compile, test, render, route, smoke test, schema-lint, migration dry-run — whichever apply).

### Step 2: Configure Agent B

Agent B is intentionally lean:
- ~10K token context (small, focused)
- Clean context — no knowledge of the builder's reasoning
- Persistent across phases (tracks cumulative state)
- Produces single classification per phase: `CLEAN`, `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`
- On `HIGH` or `CRITICAL`: `git reset --hard` + retry with fresh builder

### Step 3: Two-Strike Rule (Non-Negotiable)

- Maximum 2 retries per phase
- On second failure: STOP for human review
- Rationale: "If 2 fresh agents fail the same phase, the problem is the phase spec, not the agents."
- This rule cannot be overridden by the builder or evaluator

### Step 4: Generate Per-Phase Checker Config

For each phase, configure per your preamble branch:
- Baseline snapshot command (`git rev-parse HEAD` before phase starts)
- Applicable functional checks (per preamble branch)
- Expected outcomes for each check
- Timeout values

### Step 5: Configure Manual Fallback

For non-CLI platforms: Phase N+1 opens with Phase N validation as a preamble. Same verification, different execution method. 30-second check.

## Output

Update `$ARTIFACTS_DIR/context_packet.json` — add `stage_9`:

```json
{
  "stage_9": {
    "verification_mode": "automated_agent_b",
    "two_strike_rule": {
      "max_retries": 2,
      "on_final_failure": "stop_for_human",
      "overridable": false
    },
    "verification_protocol": ["self_report", "diff_check", "violation_response", "functional_checks"],
    "agent_b_config": {
      "context_budget": 10000,
      "persistence": "across_phases",
      "classifications": ["CLEAN", "LOW", "MEDIUM", "HIGH", "CRITICAL"],
      "on_high_critical": "git_reset_and_retry"
    },
    "per_phase_checker_config": [
      {
        "phase_number": 1,
        "baseline_command": "git rev-parse HEAD",
        "checks": ["<per preamble branch>"],
        "timeout": 60000
      }
    ],
    "manual_fallback_config": {
      "method": "preamble_merge",
      "check_duration": "30_seconds"
    },
    "stage_contract": "pass"
  }
}
```

IMPORTANT: Read existing context_packet.json, merge stage_9, increment version to 9, write back.
