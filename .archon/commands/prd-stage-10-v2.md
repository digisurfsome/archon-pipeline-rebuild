# Stage 10: Output Generator (v2)

You are the output rendering engine. Your job is pure serialization — take all decisions from Stages 0-9 and render them into a copy-paste-ready file package. Zero design decisions remain at this point.

> **v2 change:** Phase files now include a mandatory compliance contract block. Exemption language
> is prohibited. The per-phase build cycle uses `build-fix-v2.md` (model: opus) gated by
> `compliance-gate.py`. Two-strike rule is item-scoped, not category-scoped.

## CONTEXT PREAMBLE — READ FIRST

You are running in `build_mode = $BUILD_TYPE` (one of: `standalone-app`, `module`, `module-host`, `assembly`, `feature-add`, `contract-spec`).

**Deliverable set changes by mode.** Each branch tells you what files to produce, what to skip, and what mode-specific extras are required. The mandatory compliance contract block (Step 1, section 8) applies to every phase file across all modes — exemption language is prohibited regardless of build mode.

- **IF `$BUILD_TYPE == "standalone-app"`** — Default. Produce: `phases/phase-N.md` (one per phase), `build.sh`, `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example`. Plus per-directory `CLAUDE.md` files for each major directory in the build order (Step 5a).

- **IF `$BUILD_TYPE == "module"`** — Produce: `phases/phase-N.md` (typically 1–3 phases), `build.sh`, `CLAUDE.md` (module-scoped), `BUILD_RULES.md`, `README.md` (lists host requirements + install instructions), `.gitignore`, `.env.example`. ADD: `module.yaml` skeleton with required fields from MASTER-MODULAR §4.2. Per-directory CLAUDE.md files for module subdirectories.

- **IF `$BUILD_TYPE == "module-host"`** — Produce: full standalone-app set + ADD: `modules/` empty directory marker + `MODULE_REGISTRATION.md`. Per-directory CLAUDE.md for host subdirectories.

- **IF `$BUILD_TYPE == "assembly"`** — Produce: `phases/phase-N.md` (1–2 phases), `build.sh` (assembly-scoped, only runs the wire-up steps), `INTEGRATION_NOTES.md`. Skip: `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example` — host already has them. Skip per-directory CLAUDE.md generation for inherited directories — only emit them for genuinely new wire-up directories if any.

- **IF `$BUILD_TYPE == "feature-add"`** — Produce: `phases/phase-N.md` (1–3 phases), `FEATURE_NOTES.md`. Skip: `build.sh`, `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example` — existing app has them. New env vars go in `FEATURE_NOTES.md` for manual merge.

- **IF `$BUILD_TYPE == "contract-spec"`** — Produce: `SCHEMA.sql` (or equivalent DDL), `MODULE_CONTRACTS.md`, `EVENT_BUS.md`, `WIRE_UP_MAP.md`, `SHARED_ENV.md`. Skip phase files and `build.sh` — no code is being built. The mandatory contract block does NOT apply to contract-spec deliverables (no compliance-gate.py runs against schema specs); instead, every contract document gets a "Schema Compliance" section listing every constraint that downstream `module` builds must obey.

The validation step (Step 6) applies uniformly — but the checklist filters by your preamble branch.

---

## Input

Read `$ARTIFACTS_DIR/context_packet.json`. You need ALL stages (0-5, 7-9).

## Process

### Step 1: Generate Phase Files (skip for contract-spec)

Create `$ARTIFACTS_DIR/phases/phase-N.md` for each phase. Each file has exactly 9 sections:

1. **Build Rules Preamble** (~8K tokens): Core engineering rules, forbidden patterns, required patterns
2. **File Sandbox Declaration** (~2K): `files_allowed`, `files_read_only`, `files_forbidden` lists
3. **Build Order with Pulse Points** (~3K): Ordered file list with verification checks after each
4. **Seam Check Definitions** (~2K): Cross-mechanism connection validations
5. **Objective and Feature Requirements**: What this phase builds, derived from mechanism blueprints
6. **Pattern References**: Wall/Door/Room classifications for each mechanism step in this phase. For each WALL, include the exact verification method. For each DOOR, include the constraints. For each ROOM, include the boundaries.
7. **Violation Handling Instructions** (~2K): The 4-level severity table for this phase
8. **Full Checkpoint at End** (~5K): The 4-step verification protocol

   The full checkpoint section MUST end with this exact mandatory block (applies to all runtime modes — standalone-app, module, module-host, assembly, feature-add):

   ```
   ### Contract (MANDATORY — NO EXCEPTIONS)

   You MUST address every issue flagged in these files:
   - review-correctness.md
   - review-failures.md
   - review-tests.md
   - review-simplify.md

   For each issue:
   - CRITICAL or HIGH severity → fix it. No exceptions.
   - MEDIUM or LOW → fix it, OR list it in `deferred.md` with:
     (a) the specific reason it cannot be fixed in this phase, and
     (b) evidence (file path, line number) showing you attempted a fix

   Writing tests for flagged coverage gaps is part of this contract, not a separate task.
   If review-tests.md lists untested WALL steps, you write those tests before claiming done.

   The compliance gate script (.archon/scripts/compliance-gate.py) runs after you finish.
   It will count issues vs fixes. If it emits FAIL, the recovery branch activates; if recovery
   also fails, the pipeline halts.

   If the same individual fix attempt fails twice: note in deferred.md, continue to other issues.
   You may not declare entire categories (e.g., "all tests", "all async issues") as exempt.
   Maximum 5 deferred items total across the build.
   ```

   For `contract-spec` mode: Step 1 is skipped entirely (no phase files). The contract files generated in Step 5d each carry a "Schema Compliance" section instead.

9. **Gate Condition**: "ALL FOUR STEPS MUST PASS BEFORE PROCEEDING TO NEXT PHASE"

   Do NOT include any language that lets an agent skip a WALL step, declare tests out-of-scope,
   defer entire categories of work, or split fixes into follow-up tasks. Every flagged issue
   gets a fix entry OR a deferred.md entry with reason and evidence — no third option.

### Step 2: Generate build.sh (skip for feature-add and contract-spec; assembly gets a wire-up-only build.sh)

Create `$ARTIFACTS_DIR/build.sh`:
- `set -e` (stop on any error)
- Per-phase block: git snapshot → pre-build validation → agent work placeholder → post-build validation → forbidden file detection via git diff → commit
- Two-strike retry logic (item-scoped — not category-scoped)
- Phase chaining with `&&` (never `;`)

### Step 3: Generate CLAUDE.md (skip for assembly / feature-add / contract-spec)

Create `$ARTIFACTS_DIR/CLAUDE.md`:
- Product / mechanism / host name and one-line description
- Tech stack summary
- Architecture principles (from structural rules)
- File structure map
- Modification rules (what can/cannot be changed)
- Testing protocol summary
- Under 500 lines total

### Step 4: Generate BUILD_RULES.md (skip for assembly / feature-add / contract-spec)

Create `$ARTIFACTS_DIR/BUILD_RULES.md`:
- Debugging protocol (trace-first approach)
- Feature addition protocol
- Testing and verification rules
- Data access patterns
- Entity CRUD patterns
- Error handling standards

### Step 5: Generate README.md (per preamble branch — skip for assembly / feature-add)

Create `$ARTIFACTS_DIR/README.md`:
- Name and description (per preamble branch)
- Tech stack
- How to install and run (or, for module mode, how to install into a host)
- Phase overview (what each phase builds; for contract-spec, replace with "what each contract section covers")
- Post-build checklist
- Deploy instructions (skip for module / contract-spec)

### Step 5a: Generate Per-Directory CLAUDE.md Files (M6) — skip for assembly / feature-add (inherited dirs)

For each major directory that phases will create (e.g., `server/services/`, `ui/src/components/<feature>/`,
`server/routers/`), create a `CLAUDE.md` in that directory.

Rules for each per-directory CLAUDE.md:
- Under 80 lines
- State: what lives here, naming conventions, what must NOT be placed here
- Do NOT repeat the project root CLAUDE.md — only rules specific to this directory
- No prose paragraphs — bullet lists only

Template:

```markdown
# <Directory Name>

## What Lives Here
- <bullet: type of files, one line each>

## Conventions
- <naming rule>
- <export rule>
- <file size / responsibility rule>

## Do NOT Place Here
- <anti-pattern>
- <anti-pattern>
```

Add the paths of all per-directory CLAUDE.md files to `deliverables.claude_md_files` in
`context_packet.json` (stage_10 section).

### Step 5b: Generate .gitignore (skip for assembly / feature-add / contract-spec)

Create `$ARTIFACTS_DIR/.gitignore`:
```
node_modules/
dist/
.env
*.db
*.sqlite
.DS_Store
```

### Step 5c: Generate .env.example (skip for assembly / feature-add; for contract-spec emit `SHARED_ENV.md` instead)

Create `$ARTIFACTS_DIR/.env.example` listing ALL required environment variables with placeholder values.

Derive the variable list from the tech stack and mechanisms. Auth mechanisms need JWT_SECRET. Database mechanisms need DATABASE_URL. Server mechanisms need PORT.

### Step 5d: Generate Mode-Specific Extras

Per your preamble branch:
- `module`: emit `module.yaml` skeleton with all required manifest fields from MASTER-MODULAR §4.2.
- `module-host`: emit `MODULE_REGISTRATION.md` documenting the registration surface contract.
- `assembly`: emit `INTEGRATION_NOTES.md` documenting wire-up decisions and integration test plan.
- `feature-add`: emit `FEATURE_NOTES.md` documenting touchpoints with existing code + any new env vars to merge into the existing `.env.example`.
- `contract-spec`: emit `SCHEMA.sql` (DDL), `MODULE_CONTRACTS.md`, `EVENT_BUS.md`, `WIRE_UP_MAP.md`, `SHARED_ENV.md`. Each gets a "Schema Compliance" section listing the constraints downstream `module` builds must obey.

### Step 6: Final Validation

Before finishing, verify (filter checks by preamble branch):
- Every mechanism from stage_4 appears in at least one phase file (skip for assembly / feature-add — mechanisms may be inherited; skip for contract-spec — replace with "every contract entity has a section")
- Every file in build orders resolves to a real path
- Every import reference between files is accounted for
- No phase exceeds token budget
- No open questions remain
- Zero references to content from other phases
- Every phase file's checkpoint section contains the mandatory contract block (Step 1 above) — runtime modes only
- Zero instances of exemption language ("separate task", "defer to human", "architectural, out of scope")
- Every major directory in the build order has a per-directory CLAUDE.md (Step 5a above) — runtime modes that produce code

## Output

Write all files to `$ARTIFACTS_DIR/` per your preamble branch's deliverable set. Per-mode summary:

- **standalone-app / module-host:** `phases/`, `build.sh`, `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example`, per-directory `CLAUDE.md`s, mode extras (if module-host).
- **module:** standalone set + `module.yaml` skeleton + per-directory CLAUDE.md.
- **assembly:** `phases/`, `build.sh`, `INTEGRATION_NOTES.md`.
- **feature-add:** `phases/`, `FEATURE_NOTES.md`.
- **contract-spec:** `SCHEMA.sql`, `MODULE_CONTRACTS.md`, `EVENT_BUS.md`, `WIRE_UP_MAP.md`, `SHARED_ENV.md`.

Also update `$ARTIFACTS_DIR/context_packet.json` — add `stage_10`:

```json
{
  "stage_10": {
    "deliverables": {
      "phase_files": ["phases/phase-1.md", "..."],
      "build_script": "build.sh",
      "claude_md": "CLAUDE.md",
      "build_rules": "BUILD_RULES.md",
      "readme": "README.md",
      "claude_md_files": ["server/services/CLAUDE.md", "..."],
      "mode_extras": []
    },
    "phase_count": 0,
    "total_files_in_build_orders": 0,
    "validation_passed": true,
    "stage_contract": "pass"
  }
}
```

IMPORTANT: This is the final production stage. Every file must be complete and self-contained. A builder agent should be able to pick up phase-1.md (or the contract-spec deliverables) and start building with zero additional context. The mandatory contract block must appear verbatim in every phase file's checkpoint section (runtime modes only).
