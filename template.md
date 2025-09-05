---
# This YAML front-matter defines the schema for the SEDAD V1 Gold Standard template.
# agent-oz reads this configuration to parse, validate, and shard any spec
# that adheres to this standard.

name: "sedad-gold-v1"
version: "1.0.0"

# All of these top-level sections are required for a spec to be considered compliant.
# The `audit` command will check for their presence.
required_sections:
  - Intent
  - Behavior
  - Contracts
  - Evaluation Plan
  - Observability

# This map allows developers to use common synonyms for section headings in their
# specs. agent-oz will normalize them to the canonical key (e.g., "Goals" -> "Intent").
synonyms:
  Intent: ["Intent", "Objective", "Goals", "Purpose", "Problem Narrative"]
  Behavior: ["Behavior", "Behaviour", "Capabilities", "Functional Requirements"]
  Contracts: ["System Architecture & Data Contracts", "Contracts", "Architecture", "Schemas"]
  Evaluation Plan: ["Evaluation Plan", "Evals", "Quality Gates", "Testing Strategy"]
  Observability: ["Observability", "Traces", "Logging", "Monitoring"]

# This section defines the grammar and minimum requirements for a single,
# atomic capability clause within the "Behavior" section.
clause:
  # A regex to identify the start of a new capability.
  id_regex: "^## \\^cap-\\d{2,}"

  # A list of required sub-headings or content that must exist within each
  # capability clause for it to be classified as 'ready' for a Task Card.
  # The `audit` and `next` commands use this to gate quality.
  # Cardinality can be specified, e.g., Rules(3..7).
  needs:
    - "Inputs"
    - "Outputs"
    - "Rules(3..7)" # Requires 3 to 7 rules to be present.
    - "Example(GIVEN/WHEN/THEN>=1)" # Requires at least one G/W/T example.
---

# Part 1 — Intent

# Spec & Evals Driven Agent Development (SEDAD) Template

## **1.1 Objective**

Every system needs a **clear objective** for why a new module or capability exists. This section should capture that purpose in a **single sentence** that is easy to remember.

* Keep it **concise**: “The objective of this module is to ensure all downstream services consume consistent, robust data.”  
* Ensure it is **action-oriented**: start with a verb (“Provide”, “Normalize”, “Guarantee”).  
* This statement becomes the **north star**: all later design decisions are measured against whether they support the objective.

## **1.2 Problem Narrative**

A **problem narrative** explains the context in plain language:
* **What problem exists today?** Describe the pain points or inconsistencies that exist without this module. *Example:* “Different services currently implement their own data cleaning routines, leading to duplication and inconsistent results.”
* **Why is it important to solve?** Explain the consequences of leaving it unresolved (inaccuracy, unreliability, wasted developer time, user-facing errors).
* **Who benefits?** Downstream services, end-users, and the team that must maintain the system.

A good problem narrative is **1–2 paragraphs** and should be written so even a non-technical stakeholder could follow.

## **1.3 Guiding Principles**

Define a **ranked list** of non-negotiable principles that govern how the system must behave. These principles act as the “constitution” for the module.

Examples of guiding principles:
* **Single Source of Truth:** All downstream services must use the same canonical outputs.  
* **Robustness First:** The system must gracefully handle missing or corrupted inputs.  
* **Determinism:** Given the same inputs, the system must always produce the same outputs.  
* **Observability:** Every significant operation must generate structured logs or metrics.  
* **Minimal Coupling:** This module must be reusable by multiple services without deep dependencies.

Each principle should have a **short description** and a clear rationale.

## **1.4 Why SEDAD (Spec & Evals Driven Agent Development)**

This section connects the philosophy of the project to the broader methodology:
* **Spec:** Specifications are written as **lossless artifacts**. They capture the intent in unambiguous, machine-readable form.
* **Evals:** Evaluations define how “good” the system is by creating measurable, repeatable tests of quality.
* **SEDAD Loop:**
  1. Specify Intent (write the spec)  
  2. Automate Execution (build from spec)  
  3. Measure Performance (evals)  
  4. Refine & Iterate (close the loop)

This is where you explain to the junior developer (or LLM agent): **your job is not to guess**, it is to follow the specification, implement faithfully, and prove correctness via evaluations.

# **Part 2 ▪ Behavioral Specification (The What)**

## **2.1 Core Capabilities**

List the **distinct capabilities** the module provides. Each capability is written as a **clause** with an identifier (e.g., `^cap-01`).

A clause has:
* **ID:** A unique label (`^cap-01`)  
* **Name:** Short action phrase (e.g., *“Frame Normalization”*)  
* **Description:** 2–3 sentences describing what it does.  
* **Inputs:** The data it consumes.  
* **Outputs:** The data it produces.  
* **Rules:** Any conditions or fallbacks.

Keep each clause atomic — one capability \= one clause.

## **2.2 Executable Examples (GIVEN / WHEN / THEN)**

Every capability must have **testable examples**. Use the **GIVEN / WHEN / THEN** format:
* **GIVEN** a specific input or starting state  
* **WHEN** an action is taken or data flows through the system  
* **THEN** the expected result is described clearly

*Example:*

* **Clause:** `^cap-01` Frame Normalization
* **Test:**
  * GIVEN: An input frame with raw coordinates `[x=120, y=400]`  
  * WHEN: The module applies normalization  
  * THEN: The output must re-center at `(0,0)` and scale to unit length

The point: a developer or agent can directly turn this into a unit test.

## **2.3 Out-of-Scope (Boundaries of V1)**

Define **what this module will NOT do**. This is critical to prevent scope creep.

Examples:
* Will not handle multi-camera calibration.  
* Will not infer missing data beyond short gaps.  
* Will not persist raw frames (only summaries).

By making these boundaries explicit, the team avoids wasted effort on “nice-to-haves” that aren’t part of the current milestone.

# **Part 3 ▪ System Architecture & Data Contracts (The How)**

## **3.1 Placement in Codebase**
Define **where this module belongs** in the repo.
* Should it live in `utils/analysis/`?  
* Should it be shared across multiple services?  
* Should it be in its own microservice?

For a junior developer: be specific about **directory path** and **file names** so they know exactly where to work.

## **3.2 Module Structure**

Define the **internal structure** of the module:
* **Pure Functions:** small, testable units (e.g., `normalizeFrame()`).  
* **Series Functions:** operate on sequences (e.g., `normalizeSeries()`).  
* **Helpers:** math utilities (e.g., vector rotation, scaling).

Show a **blueprint**:
```
/utils/analysis/
  normalization.ts
    - normalizeFrame()
    - normalizeSeries()
    - calculateAngles()
    - smoothTimeSeries()
```

## **3.3 Service Integration Path**

Describe **how other services call this module**.

* Which CPU services import it?  
* What is the API surface?  
* How are results cached or returned?

This ensures the module is not built in isolation but fits into the wider pipeline.

---

## **3.4 Data Contracts**

Define the **types and shapes** of the inputs/outputs.

*Example:*

```ts
type NormalizedFrame = {
  t: number; // timestamp
  points: Record<string, {x: number; y: number; conf: number}>;
  T: {origin: [number,number]; theta: number; scale: number};
  quality: {scaleRef: string; confOk: boolean};
};
```

A junior developer must never guess. The **contract is explicit and versioned**.

---

## **3.5 Output Payload Shape**

Show the **exact JSON structure** to be returned or stored.

* Include metadata (fps, scaleRef, quality flags).  
* Include arrays of time-indexed values.  
* Include summary statistics if required.

This guarantees downstream services know what to expect.

---

## **3.6 Optional Persistence**

Define whether the outputs are:

* **Ephemeral (cache only)** → stored in Redis/memory.  
* **Persistent (long-term)** → stored in a database with schema.

Be explicit about retention, versioning, and whether it is required in V1.

# **Part 4 ▪ Evaluation Plan (The Test)**

The **Evaluation Plan** defines *how success is measured*. A module is not considered “done” until it passes its evaluations. Evals are not just about functional correctness (“does it run?”), but also about **robustness, stability, and quality**.

---

## **4.1 Scale Invariance Tests**

* **Goal:** Ensure outputs remain consistent regardless of input scale (e.g., zoom levels, sensor resolution).

* **Approach:**

  * GIVEN: Same input data at different scales  
  * WHEN: Normalization is applied  
  * THEN: Results must align within a set tolerance (e.g., ±3% error).  
* **Metric:** Root Mean Square Error (RMSE) between outputs.

* **Threshold:** RMSE ≤ tolerance → PASS.

## **4.2 Perspective Drift Tests**

* **Goal:** Prevent distortion when the viewpoint changes slightly.

* **Approach:**
  * GIVEN: A constant source observed from two slightly different angles  
  * WHEN: Outputs are compared  
  * THEN: Drift over time must remain below threshold.  
* **Metric:** Angular drift over N frames.

* **Threshold:** ≤ 5° drift in a 5-second window.

## **4.3 Jitter Reduction Tests**

* **Goal:** Verify smoothing effectively reduces frame-to-frame noise.

* **Approach:**
  * GIVEN: A noisy input sequence with high variance  
  * WHEN: Smoothing is applied  
  * THEN: Variance is reduced without distorting overall trend.  
* **Metric:** Standard deviation before vs. after.

* **Threshold:** ≥30% reduction in jitter std-dev.

## **4.4 Fallback Stability Tests**

* **Goal:** Ensure robustness when data is missing or unreliable.

* **Approach:**
  * GIVEN: A sequence where key inputs are missing intermittently  
  * WHEN: Fallback methods are applied  
  * THEN: Outputs remain stable and do not produce spikes.  
* **Metric:** Number of invalid frames vs. gap-filled frames.

* **Threshold:** System must recover for gaps ≤ 5 frames without large discontinuities.

## **4.5 Performance Targets**

* **Goal:** Ensure the module meets operational service level objectives (SLOs).

* **Metrics:**

  * **Latency:** End-to-end processing time per request.  
  * **Throughput:** Number of frames/records handled per second.  
* **Thresholds (example):**

  * Latency ≤ 2s for cache miss; ≤ 500ms for cache hit.  
  * Must handle 10k frames in under 1 minute in batch mode.

---

## **4.6 Regression Detection**

* **Goal:** Catch silent quality degradations.

* **Process:**

  * Run automated evaluations on every pull request.  
  * Compare against previous baseline scores.  
  * Flag if quality metrics drop by more than 2%.

---

**Definition of Done (Evals):** The module passes **all functional tests** *and* achieves evaluation scores above defined thresholds.

---

# **Part 5 ▪ Agentic Feedback Loop (The Debug)**

The **Feedback Loop** makes the module transparent and debuggable for both humans and AI agents. It defines how the system explains itself when something goes wrong.

---

## **5.1 Debug Mode Requirements**

* Must be **opt-in** (activated with a flag or config setting).  
* Must produce **structured logs**, not free-form text.  
* Must never slow down production runs when disabled.

---

## **5.2 JSONL Trace Format**

* Logs must be written in **JSON Lines (JSONL)** format: one valid JSON object per line.

* Each log entry must include:

  * `timestamp`  
  * `event_type`  
  * `frame_id` or `record_id`  
  * `inputs` (key values that drove the calculation)  
  * `intermediate_state` (transform parameters, scale reference, etc.)  
  * `outputs` (final result values)  
  * `quality_flags` (confidence, fallback usage, missing data)

---

## **5.3 Rich Payload Examples**

**Example Debug Entry (simplified):**

```json
{
  "timestamp": "2025-08-16T09:45:00Z",
  "event_type": "NORMALIZE_FRAME",
  "frame_id": 120,
  "inputs": { "raw_points": [ [320,450], [500,470] ] },
  "intermediate_state": {
    "origin": [320,450],
    "theta": -0.28,
    "scale": 0.012,
    "scale_ref": "withers_croup"
  },
  "outputs": {
    "points": { "neck": [0.0,0.0], "croup": [1.0,0.0] },
    "angles": { "neck_tilt": 15.6 }
  },
  "quality_flags": { "conf_ok": true, "fallback_used": false }
}
```

This log is machine-readable **and** human-interpretable.

---

## **5.4 Activation Rules**

* Debug mode should only be enabled in:

  * Development environment  
  * Test runs  
  * Evaluation harnesses  
* Must never run by default in production.

* Must be enabled via explicit flag (`--debug`) or environment variable (`DEBUG_MODE=true`).

---

## **5.5 Feedback Loop into Evaluations**

* Debug logs provide **evidence** for why a test failed.

* Evaluation harness must:

  * Read JSONL logs  
  * Match results against expected behavior  
  * Generate structured reports (PASS/FAIL with reasons)  
* This creates a **closed loop**:

  * Spec defines intent →  
  * Implementation produces logs →  
  * Eval checks logs against spec →  
  * Failures → refine spec or implementation.

# **Part 6 ▪ Governance & Boundaries (The Contract)**

This chapter turns the playbook into **operational rules**. It defines what’s out of scope, what “done” means, and the **exact** workflow the developer/agent must follow to ship high-quality work.

---

## **6.1 Explicit Constraints (Out of Scope for V1)**

List constraints clearly to prevent scope creep. Tailor the examples to your domain.

* **Functional boundaries**

  * No multi-sensor/multi-camera fusion in V1.  
  * No long-term stateful learning; stateless processing only.  
  * No 3D calibration or depth inference (2D weak-perspective only).  
* **Data & inputs**

  * Treat missing/low-confidence inputs with fallbacks; do not guess beyond the specified window.  
  * No external data sources or internet calls during core processing.  
* **Performance**

  * Optimize for latency & determinism; no background jobs unless specified.  
* **Security & privacy**

  * No persistence of raw input unless explicitly required.  
  * No secrets in code or logs; use environment variables only.

**Rule:** When in doubt, **stop and add a boundary note** to the spec. Boundaries are features.

---

## **6.2 Definition of Done (DoD)**

You are not done until **all** boxes are checked.

**Specs & Docs**

- [ ] Foundational Spec updated (Objective, Clauses, Executable Examples).  
- [ ] Boundaries/out-of-scope documented.  
- [ ] ADR written for any key design choice (e.g., fallback chain).

**Code & Structure**

- [ ] Module implemented at the **agreed path** (e.g., `utils/analysis/…`).  
- [ ] Public API stable and typed; no leaking internals.  
- [ ] Structured JSONL debug logs behind a flag.

**Quality & Tests**

- [ ] Unit tests (≥ 90% coverage on core logic).  
- [ ] Integration tests for service call path.  
- [ ] Evaluation suite passes thresholds (Part 4).  
- [ ] Negative/edge tests for fallbacks and missing data.

**Ops**

- [ ] Health/readiness endpoints (if service).  
- [ ] Metrics exported (rate, errors, duration).  
- [ ] CI pipeline green (lint, test, eval, security scan).

**Governance**

- [ ] Aligns with guiding principles (Part 1.3).  
- [ ] PR reviewed; Conventional Commit used; links to spec clause IDs.

---

## **6.3 Governance Flow (Propose → Approve → Implement → Audit → Commit)**

**Step 0 — Read the Spec.** Internalize Objective, Clauses, Examples, Boundaries.

**Step 1 — Propose Plan (before coding).** Post a short plan for the specific clause(s) you’ll implement:

* Files to create/modify (exact paths).  
* Public API signatures & data contracts.  
* Fallback chain & error handling.  
* Test list (unit, integration, eval).  
* Observability plan (logs/metrics).  
* Risks \+ rollback plan.

**Output:** a “Plan” comment/document. **Wait for approval.**

**Step 2 — Implement (TDD / SEDAD).**

* Write failing tests from Executable Examples.  
* Implement until tests pass.  
* Keep functions pure & deterministic where possible.

**Step 3 — Audit (Automated).**

* Run **linters**, **unit/integration tests**, **eval suite**.  
* Ensure thresholds in Part 4 are met/regressed ≤ allowed deltas.

**Step 4 — Commit Protocol.**

* Use **Conventional Commits**.  
* Reference spec clause IDs in the footer.

Example:

```shell
git commit -m "feat(normalization): add time-series smoothing with EMA" \
  -m "- Implements ^cap-03 with fallback when gaps ≤ 5 frames" \
  -m "Implements: ^cap-03"
```

**Step 5 — PR & Review.**

* PR description links to spec sections & ADR(s).  
* Include screenshots of eval reports and sample debug JSONL.  
* Reviewer checklist: spec alignment, clarity, tests, logs, metrics, security.

**Step 6 — Merge & Tag.**

* Squash & merge.  
* Tag release if the public API changed (semver).

---

## **6.4 Human as Governor / Agent as Implementer**

* **Governor** (human): owns **intent & boundaries**; approves plans; arbitrates tradeoffs.  
* **Agent** (developer/LLM): owns **faithful implementation**; raises spec gaps; never invents unspecified behavior.  
* **Trust anchor:** the **Spec** \+ **Evals**. If unclear, **pause & clarify** the spec; don’t guess.

---

# **Part 7 ▪ Operational Excellence**

This chapter ensures the module is **observable, testable, secure, and continuously deliverable**.

---

## **7.1 Logging & Observability**

**Structured Logging (JSONL)**

* One JSON object per line; no multiline.

* Required fields:
  * `timestamp`, `level`, `service`, `event_type`, `trace_id`  
  * `inputs` (summaries only; no PII/secrets)  
  * `intermediate_state` (transform params, choices)  
  * `outputs` (key results)  
  * `quality_flags` (confidence, fallback\_used)  
* Levels: `DEBUG` (dev only), `INFO`, `WARN`, `ERROR`

Example:

```json
{
  "timestamp":"2025-08-16T10:21:45Z",
  "level":"INFO",
  "service":"normalization",
  "event_type":"NORMALIZE_SERIES_DONE",
  "trace_id":"d1a2-…",
  "frames_in": 1800,
  "frames_valid": 1774,
  "fallback_used_pct": 3.1,
  "latency_ms": 426
}
```

**Metrics (Prometheus-style)**

* **Rate**: requests/frames processed per second  
* **Errors**: error count & error rate  
* **Duration**: histogram of processing time  
* **Quality**: eval scores, jitter reduction, fallback share

**Health/Readiness**

* `/health` → 200 OK with JSON { status, deps }  
* `/ready` reflects external dependency readiness

**Tracing**

* Propagate `trace_id` across function boundaries & async tasks.

---

## **7.2 Testing Strategy**

**Unit Tests**

* Pure functions, deterministic outcomes.  
* Cover happy paths and edge cases.  
* Target ≥ 90% line/branch coverage for core logic.

**Property-Based Tests** (where applicable)

* Validate invariants (e.g., idempotence, bounds).

**Integration Tests**

* Exercise service boundaries (I/O, caches).  
* Mock external systems; verify contracts & timeouts.

**Eval-Based Tests (from Part 4\)**

* Run on a curated corpus with ground truth/baselines.  
* Produce machine-readable report (JSON) for CI gates.

**Negative & Chaos Tests**

* Missing inputs, low-confidence data, timeouts, corrupt payloads.  
* Verify fallbacks & graceful degradation.

**Determinism Controls**

* Seed all randomness.  
* Freeze time in tests where timing matters.

---

## **7.3 CI/CD Integration**

**Pipeline Stages (example)**

1. **Prepare**

   * Install deps, restore caches  
2. **Static Checks**

   * Lint, type-check, formatting  
3. **Unit & Integration Tests**

   * Parallelized; coverage report uploaded  
4. **Security**

   * Dependency vulnerability scan  
5. **Build**

   * Container image with SBOM  
6. **Evals**

   * Run evaluation harness; publish JSON report  
   * **Quality gates**: fail PR if thresholds regress  
7. **Artifact & Tag**

   * Push image; tag with semver or commit SHA  
8. **Deploy to Staging**

   * Run smoke tests \+ shortened evals  
9. **Promote to Production**

   * Manual approval \+ canary (optional)

**Quality Gates**

* Lint/test must pass.  
* Coverage ≥ 85% overall; ≥ 90% core modules.  
* Eval metrics ≥ thresholds; ≤ 2% regression on protected KPIs.

**Secrets & Config**

* Never commit secrets.  
* Use environment variables or secret manager.  
* Split config by environment (dev/stage/prod).

---

## **7.4 Temporal Metadata in Specifications**

All spec files must include YAML front matter:

```
---
title: "Normalization Module Specification"
version: 1.2.0
status: active # draft | active | deprecated | archived
lastUpdated: "2025-08-16T10:30:00Z"
dependencies:
  - "./evals/normalization-evals.md"
---
```

**Rules**

* Bump `version` on any behavioral change.  
* Update `lastUpdated` on every edit.  
* Move superseded docs to `deprecated`/`archived`.  
* Keep a `CHANGELOG.md` (high-level decisions & metric shifts).

---

## **7.5 Documentation Hygiene**

**Repository Layout (docs)**

```
docs/
  spec/            # Foundational specs
  evals/           # Eval definitions, thresholds, harness usage
  adr/             # Architecture Decision Records
  runbooks/        # Ops playbooks, on-call notes
  examples/        # Sample payloads, debug JSONL, eval reports
```

**Requirements**

* Specs are **single source of truth**; code must reference them.  
* Keep **executable examples** in spec and mirrored as tests.  
* Update docs in the same PR as code changes.  
* Include ASCII architecture diagram in READMEs.

**ADR Template (minimal)**

```
# ADR-XXX: <Decision Title>
Status: Proposed | Accepted | Deprecated
Date: YYYY-MM-DD

## Context
Why was a decision needed? Constraints and options.

## Decision
What was decided; concise and testable.

## Consequences
Tradeoffs, risks, follow-ups.
```

# Part 8 ▪ Success Metrics

Success is not subjective. This part defines the **quantitative and qualitative criteria** that prove the module is production-ready.

---

### **8.1 Audit Thresholds**

The module must achieve strong scores in three audit dimensions:

* **Specification Alignment (Faithfulness)**

  * Definition: Does the implementation match the written spec?  
  * Target: ≥ **90%** alignment score.  
* **Specification Clarity (Relevance)**

  * Definition: Was the spec sufficiently detailed to guide the implementation without inventing new complexity?  
  * Target: ≥ **90%** clarity score.  
* **Human Alignment (Answer Quality)**

  * Definition: Does the code embody architectural principles (e.g., separation of concerns, observability, determinism)?  
  * Target: ≥ **90%** alignment score.

## **8.2 Deployment Readiness Criteria**

The system is ready to ship when:

* **Correctness:** All unit, integration, and eval tests pass.
* **Stability:** Passes 24h soak test with zero unhandled exceptions.
* **Performance:** Meets latency & throughput thresholds (Part 4.5).

* **Observability:**
  * `/health` returns `200 OK`  
  * `/metrics` exposes rate, error, duration histograms  
  * Debug JSONL trace activated with flag and produces valid output  
* **Docs:**
  * Spec updated with version bump  
  * ADR(s) recorded for design choices  
  * README includes ASCII architecture diagram  
* **Governance:**
  * PR reviewed & merged  
  * Commit includes Conventional Commit message \+ spec clause IDs

## **8.3 Long-Term Stability Metrics**

After deployment, measure ongoing health:
* **Regression Score:** Evals must remain ≥ thresholds in CI/CD.  
* **Error Rate:** \< 0.1% processing failures per 10k frames.  
* **Fallback Use:** \< 20% reliance on fallback mechanisms under normal conditions.  
* **Drift Rate:** Quality scores should not degrade \>2% across 3 sprints.  
* **Documentation Currency:** All spec files updated within last 90 days.

# **Appendices**

Supporting artifacts for specs, evaluations, rules, and operations.


## **Appendix A — Implementation Directives**

* Step-by-step build plan before coding any clause.

* Propose-Approve workflow (developer/agent must post plan before implementing).

* Integration checklist:
  * File path  
  * API signature  
  * Data contract  
  * Fallback chain  
  * Tests & evals  
  * Observability

## **Appendix B — Foundational Specification Template**

`_TEMPLATE_Foundational_Spec.md`

```
---
title: "[Module/Feature Name] Specification"
version: 1.0.0
status: draft
lastUpdated: "YYYY-MM-DD"
dependencies: []
---

# Part 1 ▪ Core Intent (Why)
- Objective:
- Problem Narrative:
- Guiding Principles:

# Part 2 ▪ Behavioral Specification (What)
- Clauses (^cap-01, ^cap-02…)
- Executable Examples (GIVEN / WHEN / THEN)
- Out-of-scope items

# Part 3 ▪ System Architecture (How)
- File placement
- Module structure
- Data contracts
- Payload shape
- Persistence rules

# Part 4 ▪ Evaluation Plan (Test)
- Metrics & thresholds
- Corpus/datasets
- Pass/fail criteria

# Part 5 ▪ Debugging (Feedback Loop)
- JSONL schema
- Activation rules
- Example logs

# Part 6 ▪ Boundaries (Contract)
- Explicit constraints
- Definition of Done
- Governance workflow

# Part 7 ▪ Ops Excellence
- Logging rules
- Testing strategy
- CI/CD stages
- Metadata hygiene
- Documentation rules

# Part 8 ▪ Success Metrics
- Audit thresholds
- Deployment readiness
- Long-term metrics
```


## **Appendix C — Executable Example Tests**

Each clause must have at least 1 positive & 1 negative example:

```
### Clause ^cap-01: Normalize Frame
- Positive Test:
  - GIVEN raw coords [320,450], scale ref = Withers–Croup = 200px
  - WHEN normalization applied
  - THEN origin = (0,0), scale = 1.0, output stable

- Negative Test:
  - GIVEN missing Withers point
  - WHEN normalization applied
  - THEN fallback = bbox height; flag = fallback_used=true
```

## **Appendix D — Evaluation Harness**

* **Format:** YAML/JSON definition of metrics, thresholds, test corpus.  
* **Execution:** Automated in CI/CD.  
* **Output:** Machine-readable JSON summary.

Example:

```json
{
  "eval_id": "norm_eval_2025_08",
  "tests": 25,
  "metrics": {
    "scale_invariance_rmse": 2.4,
    "drift_deg_per_5s": 3.1,
    "jitter_reduction_pct": 38.2,
    "fallback_recovery_pct": 95.5
  },
  "status": "PASS"
}
```

## **Appendix E — Debug Log Schema**

```json
{
  "timestamp": "ISO8601",
  "event_type": "string",
  "trace_id": "uuid",
  "inputs": {...},
  "intermediate_state": {...},
  "outputs": {...},
  "quality_flags": { "conf_ok": true, "fallback_used": false }
}
```

## **Appendix F — Agent Rules (.mdc)**

Rules for coding agents, e.g.:
* `00-core-principles.mdc` (persona & principles)  
* `60-spec-alignment-score.mdc`  
* `61-spec-clarity-score.mdc`  
* `62-spec-human-alignment.mdc`  
* `63-score-now-workflow.mdc`  
* `64-spec-evolution-score.mdc`  
* `70-security-hardening.mdc`  
* `80-temporal-metadata-management.mdc`  
* `81-project-structure-hygiene.mdc`


## **Appendix G — ADR Template**

```
# ADR-XXX: [Decision Title]
- **Status:** Proposed | Accepted | Deprecated | Superseded by ADR-YYY
- **Date:** YYYY-MM-DD

## Context
Why was this decision needed?

## Decision
What was decided?

## Consequences
What are the positive and negative tradeoffs?
```

## **Appendix H — Future Extensions**

Ideas reserved for later versions:
* Multi-sensor/multi-camera calibration.  
* Domain-specific tuning (different breeds, body sizes, hardware).  
* Real-time adaptive normalization.  
* Edge deployments (low-power hardware).


---
Author: Gudjon Mar Gudjonsson, CEO, OZ
