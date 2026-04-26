# Stage 10: Output Generator

You are the output rendering engine. Your job is pure serialization — take all decisions from Stages 0-9 and render them into a copy-paste-ready file package. Zero design decisions remain at this point.

> **Note:** This is the legacy v1 output generator. The active wired generator is `prd-stage-10-v2.md` (which adds the mandatory contract block + per-directory CLAUDE.md files). This file is preserved for compatibility and gets the same preamble treatment as v2.

## CONTEXT PREAMBLE — READ FIRST

You are running in `build_mode = $BUILD_TYPE` (one of: `standalone-app`, `module`, `module-host`, `assembly`, `feature-add`, `contract-spec`).

**Deliverable set changes by mode.** Each branch tells you what files to produce and what to skip.

- **IF `$BUILD_TYPE == "standalone-app"`** — Default. Produce: `phases/phase-N.md` (one per phase), `build.sh`, `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example`.

- **IF `$BUILD_TYPE == "module"`** — Produce: `phases/phase-N.md` (typically 1–3 phases), `build.sh`, `CLAUDE.md` (module-scoped, references parent host), `BUILD_RULES.md`, `README.md` (lists host requirements + how to install into a host), `.gitignore`, `.env.example`. ADD: `module.yaml` template (manifest skeleton with required fields from MASTER-MODULAR §4.2). The README must declare `depends_on_modules` and `shared_resources`.

- **IF `$BUILD_TYPE == "module-host"`** — Produce: full standalone-app file set + ADD: `modules/` empty directory marker + `MODULE_REGISTRATION.md` (how new modules register with the host). The README must document the registration surface and shared-resource policies.

- **IF `$BUILD_TYPE == "assembly"`** — Produce: `phases/phase-N.md` (typically 1–2 phases), `build.sh`, `INTEGRATION_NOTES.md`. Skip: `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example` — these already exist in the host. Do NOT overwrite them.

- **IF `$BUILD_TYPE == "feature-add"`** — Produce: `phases/phase-N.md` (typically 1–3 phases), `FEATURE_NOTES.md`. Skip: `build.sh` (existing app already has its build), `CLAUDE.md`, `BUILD_RULES.md`, `README.md`, `.gitignore`, `.env.example`. If the feature requires new env vars, list them in `FEATURE_NOTES.md` for manual merge into the existing `.env.example`.

- **IF `$BUILD_TYPE == "contract-spec"`** — Produce: `SCHEMA.sql` (or equivalent DDL), `MODULE_CONTRACTS.md` (per-module input/output contracts), `EVENT_BUS.md` (event types + payload schemas), `WIRE_UP_MAP.md`. Skip everything else (no code is being built). Phase files become contract sections, not build orders.

The validation step (Step 6) applies uniformly — but its checklist filters by your preamble branch (e.g., `assembly` mode skips the "every mechanism in a phase" check because mechanisms are inherited).

---

## Input

Read `$ARTIFACTS_DIR/context_packet.json`. You need ALL stages (0-5, 7-9).

## Process

### Step 1: Generate Phase Files (or Contract Files for contract-spec mode)

For runtime modes (standalone-app, module, module-host, assembly, feature-add):
Create `$ARTIFACTS_DIR/phases/phase-N.md` for each phase. Each file has exactly 9 sections:

1. **Build Rules Preamble** (~8K tokens): Core engineering rules, forbidden patterns, required patterns
2. **File Sandbox Declaration** (~2K): `files_allowed`, `files_read_only`, `files_forbidden` lists
3. **Build Order with Pulse Points** (~3K): Ordered file list with verification checks after each
4. **Seam Check Definitions** (~2K): Cross-mechanism connection validations
5. **Objective and Feature Requirements**: What this phase builds, derived from mechanism blueprints
6. **Pattern References**: Wall/Door/Room classifications for each mechanism step in this phase. For each WALL, include the exact verification method. For each DOOR, include the constraints. For each ROOM, include the boundaries.
7. **Violation Handling Instructions** (~2K): The 4-level severity table for this phase
8. **Full Checkpoint at End** (~5K): The 4-step verification protocol
9. **Gate Condition**: "ALL FOUR STEPS MUST PASS BEFORE PROCEEDING TO NEXT PHASE"

For `contract-spec` mode: replace phase files with `SCHEMA.sql`, `MODULE_CONTRACTS.md`, `EVENT_BUS.md`, `WIRE_UP_MAP.md` (per the preamble branch above).

### Step 2: Generate build.sh (skip for assembly / feature-add / contract-spec)

Create `$ARTIFACTS_DIR/build.sh`:
- `set -e` (stop on any error)
- Per-phase block: git snapshot → pre-build validation → agent work placeholder → post-build validation → forbidden file detection via git diff → commit
- Two-strike retry logic
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

### Step 5: Generate README.md (per preamble branch)

Create `$ARTIFACTS_DIR/README.md`:
- Product / mechanism / host name and description
- Tech stack
- How to install and run (or, for module mode, how to install into a host)
- Phase overview (what each phase builds; for contract-spec, replace with "what each contract section covers")
- Post-build checklist
- Deploy instructions (skip for module / contract-spec — those don't deploy on their own)

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

### Step 5c: Generate .env.example (skip for assembly / feature-add — those inherit; for contract-spec produce a `SHARED_ENV.md` listing shared env vars)

Create `$ARTIFACTS_DIR/.env.example` listing ALL required environment variables with placeholder values. Example:
```
DATABASE_URL=file:./dev.db
JWT_SECRET=change-me-to-a-random-string
PORT=3001
CORS_ORIGIN=http://localhost:5173
```

Derive the variable list from the tech stack and mechanisms. Auth mechanisms need JWT_SECRET. Database mechanisms need DATABASE_URL. Server mechanisms need PORT.

### Step 5d: Generate Mode-Specific Extras

Per your preamble branch:
- `module`: emit `module.yaml` skeleton with all required manifest fields from MASTER-MODULAR §4.2.
- `module-host`: emit `MODULE_REGISTRATION.md`.
- `assembly`: emit `INTEGRATION_NOTES.md`.
- `feature-add`: emit `FEATURE_NOTES.md`.
- `contract-spec`: emit `SCHEMA.sql`, `MODULE_CONTRACTS.md`, `EVENT_BUS.md`, `WIRE_UP_MAP.md`.

### Step 6: Final Validation

Before finishing, verify (filter checks per preamble branch):
- Every mechanism from stage_4 appears in at least one phase file (skip for assembly / feature-add — mechanisms may be inherited)
- Every file in build orders resolves to a real path
- Every import reference between files is accounted for
- No phase exceeds token budget
- No open questions remain
- Zero references to content from other phases

## Output

Write all files to `$ARTIFACTS_DIR/` per your preamble branch's deliverable set. Also update `$ARTIFACTS_DIR/context_packet.json` — add `stage_10`:

```json
{
  "stage_10": {
    "deliverables": {
      "phase_files": ["phases/phase-1.md", "..."],
      "build_script": "build.sh",
      "claude_md": "CLAUDE.md",
      "build_rules": "BUILD_RULES.md",
      "readme": "README.md",
      "mode_extras": []
    },
    "phase_count": 0,
    "total_files_in_build_orders": 0,
    "validation_passed": true,
    "stage_contract": "pass"
  }
}
```

IMPORTANT: This is the final production stage. Every file must be complete and self-contained. A builder agent should be able to pick up phase-1.md (or the contract-spec deliverables) and start building with zero additional context.
