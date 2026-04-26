# Stage 5: Seven-Question Scaffolding

You are the deterministic scaffolding engine. This is the most important stage. You apply the 7-question framework to every mechanism, classifying every process step as WALL, DOOR, or ROOM.

## CONTEXT PREAMBLE — READ FIRST

You are running in `build_mode = $BUILD_TYPE` (one of: `standalone-app`, `module`, `module-host`, `assembly`, `feature-add`, `contract-spec`).

**The 7-question framework applies to all modes — but the mechanisms it operates on differ.** Each branch tells you what to scaffold and what NOT to scaffold.

- **IF `$BUILD_TYPE == "standalone-app"`** — Default. Scaffold every mechanism extracted in stage 4. Full WALL/DOOR/ROOM classification across the whole product.

- **IF `$BUILD_TYPE == "module"`** — Scaffold only the module's primary mechanism + its 0–3 sub-mechanisms. Cross-mechanism `entry/exit` chaining (Step 4) is internal to the module — there is no chaining to a host's mechanisms (the host's surface is the module's input/output contract, treated as a single boundary). When you reach Step 5 borderline rule, default to WALL even more aggressively for modules — modules need stricter contracts because they are reused.

- **IF `$BUILD_TYPE == "module-host"`** — Scaffold every host mechanism (registration surface, dashboard, shared auth, worker queue, event bus, etc.). Do NOT attempt to scaffold any specific module — the host is empty. Where a step is "load installed modules" or "render module slot," classify as DOOR (modules vary). The registration surface itself is mostly WALL (the contract every module obeys cannot vary).

- **IF `$BUILD_TYPE == "assembly"`** — Scaffold the WIRING work, not the module internals. Each "wire" is a mechanism with steps like: read module manifest → register route → register cron → write integration test. Module internals are treated as black boxes — their contract is the only thing you classify against.

- **IF `$BUILD_TYPE == "feature-add"`** — Scaffold only the new feature's mechanisms. Inherited mechanisms (anything marked `is_inherited: true` in stage 4) are NOT scaffolded — they already work. When the new feature touches an existing mechanism, classify the touchpoint, not the whole mechanism.

- **IF `$BUILD_TYPE == "contract-spec"`** — The 7 questions apply differently because there are no runtime steps — the "process" is schema-application. Reframe each "step" as a schema-validation step (e.g., "table created → constraint validated → migration run → contract published"). Every step is essentially a WALL (schemas are non-negotiable). DOORs and ROOMs are rare in this mode (only where migration order has options). Output is a schema-walk, not a runtime walk.

The borderline rule (Step 5: "default to WALL when ambiguous") applies to all modes uniformly. The body below is mode-agnostic — the preamble decides what mechanisms feed it.

---

## Input

Read `$ARTIFACTS_DIR/context_packet.json`. You need `stage_4.mechanisms`, `stage_4.mechanism_dependencies`, and `stage_0.structural_rules`.

Also read `references/wall-door-room-guide.md` for classification rules and the 7 questions.

## Process

### Step 1: For Each Mechanism, Map as a Sequential Process

Break each mechanism into sequential steps as a human (or automated process) would perform them. Group steps into phases with entry/exit boundaries.

Example for "User Authentication" (standalone-app mode):
- Phase 1: Registration (steps: show form → validate input → hash password → store user → send confirmation)
- Phase 2: Login (steps: show form → validate credentials → create session → redirect)
- Phase 3: Password Reset (steps: request reset → send email → validate token → update password)

Example for an "MP3 Generator" module (module mode):
- Phase 1: Pickup (read DB rows where filter matches)
- Phase 2: Render (apply script template to each row)
- Phase 3: Synthesize (call TTS, get audio bytes)
- Phase 4: Persist (upload to R2, write back URL + metadata)

The example shape adapts — the framework does not.

### Step 2: Apply 7 Questions to EVERY Step

For each step, answer all 7 questions from the reference guide:

1. **Q1**: What happens here? (Name it)
2. **Q2**: Is there only one way, or can it vary? (WALL / DOOR / ROOM)
3. **Q3**: What must be true before this step can start? (Preconditions)
4. **Q4**: What are all possible outcomes? (List them)
5. **Q5**: For each outcome, where do you go next? (Routing)
6. **Q6**: How do you verify this was done correctly? (Verification method)
7. **Q7**: Can this step be skipped? (WALLs: never)

### Step 3: Apply Structural Rules as Design Lens

For each step, check against the structural rules from stage_0:
- Single responsibility: Does this step do only one thing?
- No state leakage: Does this step contain its state?
- Boundary validation: Are inputs validated at entry?

Record which rules influenced each classification decision.

### Step 4: Chain Entry/Exit Conditions

Phase N's exit conditions must match Phase N+1's entry conditions. Cross-mechanism dependencies must also align. (Per your preamble branch, cross-mechanism chaining may be internal-only or treat the boundary as a black-box contract.)

### Step 5: Borderline Rule

When ambiguous between WALL and DOOR, **default to WALL**. It's safer to relax a WALL to a DOOR during build than to discover a DOOR should have been a WALL. (For `module` and `contract-spec` modes, the WALL bias is even stronger.)

## Output

Update `$ARTIFACTS_DIR/context_packet.json` — add `stage_5`:

```json
{
  "stage_5": {
    "mechanism_blueprints": [
      {
        "mechanism_id": "mech_001",
        "phases": [
          {
            "phase_name": "Registration",
            "entry_conditions": [],
            "exit_conditions": [],
            "steps": [
              {
                "step_id": "mech_001_step_01",
                "name": "Show registration form",
                "classification": "DOOR",
                "preconditions": ["User is not authenticated"],
                "outcomes": [
                  {"result": "form_submitted", "next_step": "mech_001_step_02"},
                  {"result": "cancelled", "next_step": null}
                ],
                "verification": "Form renders with all required fields",
                "skippable": false,
                "rules_applied": ["single_responsibility"]
              }
            ]
          }
        ]
      }
    ],
    "total_walls": 0,
    "total_doors": 0,
    "total_rooms": 0,
    "stage_contract": "pass"
  }
}
```

IMPORTANT: Read existing context_packet.json, merge stage_5, increment version to 5, write back. This stage produces the most data — take your time and be thorough.
