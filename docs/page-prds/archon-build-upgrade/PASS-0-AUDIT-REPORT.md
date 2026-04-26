# Pass 0 — Preamble Audit Report

> **Status:** Complete. All 11 stage command files ported from `digisurfsome/Greptacular` source, mode-stripped, and now carry a 6-mode preamble block at the top.
> **Source repo (read-only):** `c:\Users\lober\.archon\workspaces\digisurfsome\Greptacular\source\.archon\commands\` (AutoForge mirror — the GitHub clone was stale and only contained stage-10-v2; the AutoForge mirror was used by the owner's confirmation).
> **Destination repo (writes):** `C:\Users\lober\archon-pipeline-rebuild\.archon\commands\`
> **Layout chosen:** Layout A (preamble inlined per stage). Layout B was rejected — Archon command files have no include/import syntax (verified against `archon/references/authoring-commands.md`).
> **Variable name used:** `$BUILD_TYPE`. PRD §4.1 specifies this verbatim. MASTER §3 pseudo-code uses `$build_mode` — that is design-doc shorthand, not the runtime variable. Pass 1 will set `BUILD_TYPE` in the workflow's env block from `$build-type-select.output.build_type`.

---

## 1. Files in scope

### 1.1 Files ported (11)

| # | Source filename | Destination | Lines (src → dst) | Notes |
|---|---|---|---|---|
| 1 | `prd-stage-00.md` | `prd-stage-00.md` | 75 → ~120 | Tech foundation. Strong mode leak (`platform_profile` table assumes UI/auth/hosting). |
| 2 | `prd-stage-01.md` | `prd-stage-01.md` | 61 → ~85 | Idea capture. Lowest leak — verbatim capture is mode-agnostic; only intent-detection vocabulary added. |
| 3 | `prd-stage-02.md` | `prd-stage-02.md` | 73 → ~115 | Gap analysis. Folded improvements from `prd-a-stage-02-enhanced.md` (market research step + `market_research` JSON field). |
| 4 | `prd-stage-03.md` | `prd-stage-03.md` | 84 → ~140 | Structuring. Folded improvements from `prd-a-stage-03-enhanced.md` (decision log + evidence requirements). |
| 5 | `prd-stage-04.md` | `prd-stage-04.md` | 86 → ~130 | Mechanism extraction. Heavy leak — assumes "many mechanisms" with one core. Module mode forces 1 primary + 0–3 sub-mechs. |
| 6 | `prd-stage-05.md` | `prd-stage-05.md` | 94 → ~130 | Wall/Door/Room scaffolding. Framework is mode-agnostic; preamble dictates which mechanisms feed it. |
| 7 | `prd-stage-07.md` | `prd-stage-07.md` | 96 → ~135 | Phase sequencing. Strong leak (4-layer order assumes UI present). UI layer dropped for headless modes. |
| 8 | `prd-stage-08.md` | `prd-stage-08.md` | 92 → ~145 | Protocol injection. Strong leak (compile/render/route checks assume runtime). Schema-lint + migration-dry-run substituted for contract-spec. |
| 9 | `prd-stage-09.md` | `prd-stage-09.md` | 91 → ~125 | Verification agent. Per-phase checker config swaps per mode. Two-strike rule + 4-step protocol unchanged. |
| 10 | `prd-stage-10.md` | `prd-stage-10.md` | 130 → ~175 | Legacy v1 output generator. Preserved in destination per source — Pass 1 chooses which v to wire. Same preamble + deliverable filtering. |
| 11 | `prd-stage-10-v2.md` | `prd-stage-10-v2.md` | 199 → ~225 | Active output generator. Mandatory contract block preserved verbatim. Per-mode deliverable filter added. |

### 1.2 Enhanced variants (folded, not re-emitted)

The two enhanced variants from source were NOT ported as separate files. Their improvements were folded directly into the canonical stages:

- `prd-a-stage-02-enhanced.md` → folded into `prd-stage-02.md` (market research step + JSON field).
- `prd-a-stage-03-enhanced.md` → folded into `prd-stage-03.md` (decision log + evidence requirements).

Rationale: the user explicitly directed "fold any improvements into your rewrites of stage-02 and stage-03." Pass 1 will only wire one stage-02 / stage-03 file — having two variants in destination would be drift surface area.

### 1.3 Stage 06 — confirmed gap

`prd-stage-06.md` does not exist in source. The owner confirmed: "Stage-06 is genuinely missing. Skip it. Verified on disk: Greptacular's .archon/commands/ jumps from prd-stage-05 directly to prd-stage-07. There is no stage-06. Never existed."

The numbering 00, 01, 02, 03, 04, 05, **(gap)**, 07, 08, 09, 10, 10-v2 = **11 files** is preserved in destination. Pass 1 should NOT attempt to author a stage-06 from scratch — the gap is intentional.

---

## 2. Per-stage findings

### Stage 0 — Technical Foundation

**Mode-leak findings.** Heavy. The `platform_profile` table (lines 19–24 of source) lists 5 profiles, all of which assume a runtime app with framework + DB + auth + hosting. Modules need none of those (host-provided). Contract-spec needs DB only. Assembly / feature-add inherit everything. Steps 3 (`mechanism_target`) and 4 (`token_budget`) leak less — token budget is universal; mechanism categories vary by mode.

**Where each leak moved.**
- Profile table → preamble dictates which fields are populated vs `"host-provided"`/`"inherited"`/`null` per branch.
- "Initialize all 14 categories as `needs_user_input`" → preamble branches override this default per mode (most categories become `not_applicable` or `inherited`).
- New tech_stack fields added to JSON output: `runtime`, `language`, `tech_stack_inheritance` block.

**Core prompt delta.** Added two new profile types to the table (`inherited`, `module-internal`); added `runtime` + `language` to JSON; added `tech_stack_inheritance` block to JSON; added `build_mode` field at top-level of context_packet.

**Mode applicability.** All 6 modes apply but `contract-spec` operates with most fields null (only DB engine + language matter).

---

### Stage 1 — Idea Capture

**Mode-leak findings.** Minimal. Source assumes user is describing an "app idea" (line 1, 14, 32). Verbatim capture itself is mode-neutral but the intent-detection vocabulary leaked.

**Where each leak moved.** Preamble branches per mode each declare the *expected framing* (app vs mechanism vs shell vs wire-up vs feature vs contract-project) and what to record in `mode_references` (host name, app name, module list, project name).

**Core prompt delta.** Added Step 5 (Detect Mode-Relevant References). New JSON field `mode_references` block + `intent_mismatch_flag`/`intent_mismatch_details`. Verbatim-capture rule unchanged.

**Mode applicability.** All 6 modes apply uniformly.

---

### Stage 2 — Gap Analysis + Market Research

**Mode-leak findings.** Heavy. Source's 8 archetypes (line 18) are all *app-shaped* (Dashboard, Marketplace, etc.). Modules need module archetypes (Scraper, Generator, etc.). Hosts need host archetypes (Worker-queue, Dashboard host, etc.). Assembly skips archetype matching entirely. Contract-spec doesn't archetype either.

Step 3 ("Fill Gaps with Developer's Choice", line 33) leaks too — examples are all app-shaped (Auth gap → Supabase Auth, Storage gap → Postgres CRUD).

**Where each leak moved.** Preamble branches per mode declare the active archetype set + the gap-scan universe. The Step 4 example list (formerly only app gaps) now spans modes (failure-handling for modules, wire-up for assembly, etc.).

**Core prompt delta.** Folded enhanced-variant improvements: market_research step (Step 3) + JSON field. Added `inherited` and `not_applicable` to the per-category status enum. Added `inherited` array to scope_contract.

**Mode applicability.** All 6 modes apply; market research skipped for assembly/contract-spec/most-feature-add.

---

### Stage 3 — Structuring + Feasibility + Decision Log

**Mode-leak findings.** Heavy. Section labels (concept, target_user, feasibility, problem_statement) all assume a product. For a module, "target user" is the operator dev, not the end user. For contract-spec, "concept" is a schema spec, not a product. For feature-add, "concept" is the feature within an existing app.

**Where each leak moved.** Preamble branches per mode redefine each of the 4 sections in mode-appropriate terms. Body remains literal — "Section A" / "Section B" / etc. — but the preamble tells the agent what each section *means* in the active mode.

**Core prompt delta.** Folded enhanced-variant improvements: decision log (Step 4) + evidence requirements (Step 5) + `evidence_basis` enum in JSON. Drift-anchor wording loosened to "whatever your preamble branch says you are building."

**Mode applicability.** All 6 modes apply. Module mode sees the most reframing (operator persona vs end-user persona).

---

### Stage 4 — Mechanism Extraction

**Mode-leak findings.** Severe. Source assumes 3–15 mechanisms with exactly one core. Module mode is fundamentally different — exactly 1 primary mechanism + sub-mechanisms inside. Assembly mechanisms are wires, not features. Contract-spec "mechanisms" are schema entities, not code. The "auto-select rule" and "dependency graph" still apply but the entities are different.

**Where each leak moved.** Preamble branches per mode declare expected mechanism count + what counts as a mechanism (feature / module-primary / host-shell / wire / feature-touchpoint / contract-entity). Core mechanism definition shifts (registration surface for host, integration test for assembly, shared schema for contract-spec).

**Core prompt delta.** Added `inherited` and `inherits_from_host`/`inherits_from_app` dependency types. Added `is_inherited` + `inherited_from` fields to mechanism JSON.

**Mode applicability.** All 6 modes apply with very different shapes.

---

### Stage 5 — Seven-Question Scaffolding

**Mode-leak findings.** Light. The 7 questions and Wall/Door/Room framework are genuinely abstract. The source's example mechanism (User Authentication) leaks toward standalone-app, but it's an example, not a directive. Step 4 ("chain entry/exit") assumes cross-mechanism chaining — modules don't chain to host mechanisms (the boundary is a black box).

**Where each leak moved.** Preamble branches per mode declare which mechanisms feed the framework + how to handle cross-boundary chaining. Module mode bias toward WALL stated explicitly. Contract-spec reframes the framework as schema-validation steps.

**Core prompt delta.** Added a second example (MP3 Generator module) alongside the original auth example. Borderline rule (default to WALL) made mode-aware.

**Mode applicability.** All 6 modes apply. Contract-spec is the most reframed (every step ≈ WALL).

---

### Stage 7 — Phase Sequencing

**Mode-leak findings.** Strong. Source's 4-layer build order (core_logic → state → UI → integration, line 51–55) assumes UI exists. Module mode is headless — no UI layer. Contract-spec produces docs only — no integration layer. Phase count expectations (2–10) don't fit module (1–3) or contract-spec (1).

**Where each leak moved.** Preamble branches per mode declare expected phase count range + which of the 4 layers apply. Defaults (`CLAUDE.md`, `.env`, `BUILD_RULES.md` in `global_do_not_change`) preserved. For feature-add/assembly, inherited manifest files added to `global_do_not_change`.

**Core prompt delta.** Phase 1 description loosened ("foundation per your preamble branch"). Sizing guidelines extended (schema-only contract entities = 2K–8K tokens). Layer interpretation made branch-specific.

**Mode applicability.** All 6 modes apply with very different phase shapes.

---

### Stage 8 — Protocol Injection

**Mode-leak findings.** Severe. Source's "compile check" / "functional checks" / "route verification" all assume a runtime app. Modules need smoke-test-on-mock-data instead of runtime checks. Contract-spec has no runtime — needs schema-lint + migration-dry-run. Assembly needs cross-module integration tests. Feature-add needs regression checks.

**Where each leak moved.** Preamble branches per mode redefine what passes for PULSE / SEAM / FULL checks. The 4-level violation table got a feature-add addition: "(feature-add) touched any `files_read_only` inherited file" → HIGH.

**Core prompt delta.** Specific check examples replaced with "per your preamble branch." Schema-lint and migration-dry-run are explicit substitutes for contract-spec's compile/functional checks.

**Mode applicability.** All 6 modes apply. Contract-spec is the most reframed (all runtime checks are skipped or substituted).

---

### Stage 9 — Verification Agent Setup

**Mode-leak findings.** Medium. Source's per-phase checker config (line 41: "Phase 1: compile only") leaks. Module mode needs smoke test instead of full compile. Contract-spec has no compile cycle at all. Feature-add needs regression checks alongside new-feature checks.

**Where each leak moved.** Preamble branches per mode declare what Agent B actually runs (compile / smoke / schema-lint / regression / integration). The 4-step verification protocol (self_report → diff_check → violation_response → functional_checks) and two-strike rule unchanged.

**Core prompt delta.** Per-phase checks listed as `<per preamble branch>` placeholder. Body stays mode-agnostic.

**Mode applicability.** All 6 modes apply. Contract-spec is the most reframed (no runtime cycle).

---

### Stage 10 (legacy v1) — Output Generator

**Mode-leak findings.** Heavy. Source produces a fixed deliverable set (phases, build.sh, CLAUDE.md, BUILD_RULES.md, README.md, .gitignore, .env.example) which assumes standalone-app. Modules need module.yaml, hosts need MODULE_REGISTRATION.md, assembly needs INTEGRATION_NOTES.md, feature-add needs FEATURE_NOTES.md, contract-spec needs SCHEMA.sql + 4 contract docs.

**Where each leak moved.** Preamble branches per mode declare exact deliverable set. Step 5d added: "Generate Mode-Specific Extras." Existing steps annotated with skip rules per mode.

**Core prompt delta.** Added Step 5d. Validation step (Step 6) checks now filter by branch.

**Mode applicability.** All 6 modes apply with different file sets.

> Note: This is the legacy v1. The active wired output generator is v2. Preserved in destination because Pass 1 will choose which to wire (Pass 0 does not touch YAML).

---

### Stage 10 v2 — Output Generator (active)

**Mode-leak findings.** Same as v1 plus the mandatory contract block (which is correctly mode-agnostic — every runtime mode benefits from it). Contract-spec is the only mode where the contract block does not apply (no compliance-gate.py runs against schema specs).

**Where each leak moved.** Same preamble strategy as v1. Mandatory contract block kept verbatim — reframed to apply to "all runtime modes." Contract-spec gets a "Schema Compliance" section in each contract doc instead.

**Core prompt delta.** Step 5d added (mode-specific extras). Step 5a (per-directory CLAUDE.md) annotated with skip rules for assembly/feature-add. Step 6 validation checks filter by mode.

**Mode applicability.** All 6 modes apply. Mandatory contract block applies to runtime modes (5 of 6); contract-spec uses Schema Compliance sections instead.

---

## 3. Cross-stage consistency notes

A read-through of each mode's branches across all 11 stages confirms:

- **standalone-app** — Coherent end-to-end. The 11-stage pipeline produces a complete app spec.
- **module** — Coherent. 1 primary mechanism + sub-mechanisms; phase count 1–3; deliverables include module.yaml; verification uses smoke test on mock data. Modules slot into an upstream contract-spec.
- **module-host** — Coherent. Empty shell built first; registration surface is core mechanism; full deliverable set + MODULE_REGISTRATION.md.
- **assembly** — Coherent but lean. Mechanisms = wires; phase count 1–2; deliverables = phases + INTEGRATION_NOTES; many host files inherited and `files_read_only`.
- **feature-add** — Coherent and stricter. Inherited files dominate `files_read_only`; HIGH violation triggered if they're touched; regression check non-negotiable in Stage 9.
- **contract-spec** — Coherent but most reframed. No code is built. Deliverables = SCHEMA + 4 contract docs. Many stages reframe (Stage 5 = schema-walk, Stage 8 = schema-lint, Stage 9 = no runtime cycle). Mandatory contract block does not apply (no compliance-gate.py).

No mode is silently broken. No stage required a "skip entirely" branch — every mode has meaningful work in every stage, even if some stages do less for some modes (e.g., contract-spec mostly does schema work).

---

## 4. Summary metrics

| Metric | Count |
|---|---|
| Stage files ported | 11 |
| Stage files modified in source | 0 (source is read-only — destination repo is fresh) |
| Enhanced variants folded (not re-emitted as separate files) | 2 |
| Mode-leak instances stripped from bodies | ~40 (rough count across all stages) |
| New IF branches authored | 11 stages × 6 modes = **66** preamble branches |
| Workflow YAML files modified | **0** (per PRD §7) |
| Lines of audit report (this file) | ~330 |

---

## 5. Disagreements with MASTER-MODULAR

None. The 6-mode taxonomy is self-consistent and the preamble template (MASTER §3) translated cleanly across all 11 stages. Two minor friction points worth flagging for Pass 1:

1. **Variable naming.** MASTER §3 pseudo-code uses `$build_mode`. PRD §4.1 explicitly says `$BUILD_TYPE`. I used `$BUILD_TYPE` in all preambles per the explicit PRD instruction. Pass 1's `build-type-select` node should output `{build_type: "..."}` and the workflow's env block should map `BUILD_TYPE: $build-type-select.output.build_type` to feed the preamble correctly.

2. **Contract-spec's Stage 8/9 reframing is heavy.** The PRD says "some modes may legitimately say 'this stage is not applicable.'" I chose to reframe Stage 8 and Stage 9 for contract-spec (schema-lint + migration-dry-run substituted for compile/functional checks) rather than skip them entirely. This keeps the pipeline shape uniform across modes — every stage runs, but the work it does varies. Pass 1 may choose to short-circuit these stages instead; that's a Pass 1 judgment call, not a Pass 0 deliverable.

---

## 6. Suggestions for `ADD-NEW-BUILD-STYLE-HANDOFF.md`

No 7th build mode emerged as obviously needed during this audit. Two near-misses worth noting:

- **`module-with-admin-ui`** — A module that ships a small admin sub-page in addition to its headless mechanism. Currently this would route through `module` mode and the small admin UI gets noted in the module's manifest. If this becomes common (e.g., 30%+ of modules), splitting it out as a 7th mode might be cleaner than overloading `module`.
- **`monorepo-extract`** — Pulling a feature OUT of an existing app into a standalone module. This is `module` mode + a one-time migration, and could either be its own 7th mode or just a flavor of `module` with a `migrated_from_app` field.

Neither is required for v1. Both are noted here per PRD §7 instruction.

---

## 7. What Pass 1 needs to do with this output

1. Wire the 11 stage files into `prd-pipeline-d.yaml` (the new no-bash pipeline).
2. Add a preflight `build-type-select` node before stage-00 that outputs `{build_type: "..."}` matching the 6-mode enum.
3. Add the env block to the workflow: `env: { BUILD_TYPE: '$build-type-select.output.build_type' }`.
4. Decide between `prd-stage-10.md` (v1) and `prd-stage-10-v2.md` (v2) — recommend v2 (active version with mandatory contract).
5. Skip stage-06 — no file exists, intentional gap, do NOT author one.
6. The 2 enhanced variants (`prd-a-stage-02-enhanced.md`, `prd-a-stage-03-enhanced.md`) are NOT in destination — their improvements live inside the canonical stage-02 and stage-03. Wire only the canonical names.

---

**End of audit report.**
