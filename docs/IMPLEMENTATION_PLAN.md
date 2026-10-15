# IMPLEMENTATION_PLAN.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.1  
**Status:** MVP Implementation Roadmap  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the implementation sequence for NetAssist AI.

The project will be built incrementally, starting with a working deterministic backend and progressing toward a browser-based interface.

Implementation must follow the contracts defined in:

- `docs/PROJECT_MASTER_PLAN.md`
- `docs/PRD.md`
- `docs/ARCHITECTURE.md`
- `docs/TECH_STACK.md`
- `docs/DATA_MODELS.md`
- `docs/API_SPECIFICATION.md`
- `docs/DETECTION_RULES.md`
- `docs/SECURITY_AND_PRIVACY.md`
- `docs/TESTING_STRATEGY.md`
- `docs/REQUIREMENTS_TRACEABILITY.md`

`DATA_MODELS.md` defines the canonical data contract. `API_SPECIFICATION.md` defines the HTTP interface, and `DETECTION_RULES.md` defines the detection behavior.

## 2. Implementation Principles

1. Build one component at a time.
2. Keep the application as a modular monolith.
3. Use Python 3.12, FastAPI and Pydantic v2.
4. Keep MVP detection deterministic.
5. Implement tests alongside each component.
6. Preserve evidence and source line numbers.
7. Never invent unavailable log information.
8. Do not add a database or permanent analysis history to the MVP.
9. Do not require paid APIs or cloud services.
10. Do not execute network commands or make device configuration changes.
11. Keep AI disabled until the deterministic pipeline is working and tested.
12. Update `PROJECT_STATUS.md` after each meaningful milestone.

## 3. MVP Scope

### Included

- Local FastAPI application
- Browser-based log upload and results display
- `.log` and `.txt` file support
- Configurable upload limit, initially 5 MiB
- Input validation and safe file handling
- Log parsing and normalization
- Deterministic network-event detection
- Evidence-backed recommendations
- Conservative event correlation
- Structured analysis summaries and warnings
- Automated unit and API tests
- Health, readiness and application-information endpoints
- Local setup and usage documentation

### Excluded

- Production router or switch access
- Automated device commands
- Network configuration changes
- Database-backed history
- Saved-analysis retrieval or export endpoints
- User accounts and multi-tenancy
- Paid AI APIs
- AI-generated detections
- Production deployment and security claims that have not been implemented and tested

---

## 4. Recommended Project Structure

The final structure should follow the existing repository and the architecture documentation. The following is the intended logical structure; do not create duplicate folders if equivalent ones already exist.

```text
NetAssist-AI/
├── app/
│   ├── main.py
│   ├── api/
│   │   ├── routes.py
│   │   └── exceptions.py
│   ├── core/
│   │   └── config.py
│   ├── models/
│   │   └── data_models.py
│   ├── services/
│   │   ├── upload_service.py
│   │   ├── parser_service.py
│   │   ├── detection_service.py
│   │   ├── correlation_service.py
│   │   └── analysis_service.py
│   └── rules/
│       └── detection_rules.py
├── frontend/
│   ├── index.html
│   ├── styles.css
│   └── app.js
├── tests/
│   ├── unit/
│   └── integration/
├── sample_logs/
├── docs/
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

The browser UI can use plain HTML, CSS and JavaScript in the MVP. A frontend framework is not required.

Actual module names may be adjusted to match `ARCHITECTURE.md`, provided the responsibilities remain clear and the canonical data models are defined in one place.

## 5. Phase 0 — Repository and Environment

**Goal:** Establish a reproducible local development environment.

### Tasks

1. Inspect the existing repository and current files.
2. Confirm Python 3.12 is available.
3. Confirm the project environment and dependency manager.
4. Review the existing directory structure before creating new files.
5. Confirm `.gitignore` excludes virtual environments, secrets, temporary uploads, caches and generated artifacts.
6. Confirm `.env.example` contains placeholders only, if configuration variables are required.
7. Run the existing test suite, if any.

### Deliverables

- Working local environment
- Agreed repository structure
- Dependency configuration
- Baseline test result

### Exit criteria

- The application environment can be created using documented commands.
- No real secrets are committed.
- Existing files and work are preserved.

---

## 6. Phase 1 — Canonical Data Models

**Goal:** Implement the data contract before implementing business logic.

### Tasks

1. Implement the enums defined in `DATA_MODELS.md`.
2. Implement `Evidence`.
3. Implement `Recommendation`.
4. Implement `NetworkEvent`.
5. Implement `Correlation`.
6. Implement `AnalysisSummary`.
7. Implement `AnalysisWarning`.
8. Implement `ProcessingMetadata`.
9. Implement `AnalysisResult`.
10. Add validation for field constraints and cross-field invariants.

### Required model decisions

- Severity values: `INFO`, `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`
- Analysis statuses: `COMPLETED`, `PARTIAL`
- Structured warnings and recommendations
- Timezone-aware `created_at`
- `ai_used = false` in the initial MVP
- No persistence implied by `analysis_id`

### Tests

- Valid and invalid enum values
- Missing required fields
- Invalid evidence line numbers
- Naive versus timezone-aware creation timestamps
- Duplicate event IDs within an analysis
- Invalid correlation references
- Summary count mismatches

### Deliverables

- Canonical Pydantic models
- Model validation tests

### Exit criteria

All model tests pass, and downstream modules can import the same models without redefining them.

---

## 7. Phase 2 — Upload Validation and File Safety

**Goal:** Accept supported text files safely.

### Tasks

1. Accept a file through the `file` multipart field.
2. Validate the filename and supported extension.
3. Enforce the configured upload-size limit of 5 MiB.
4. Define a consistent text-decoding policy.
5. Reject empty or invalid content according to the API contract.
6. Avoid trusting the client-provided MIME type alone.
7. Avoid using the uploaded filename as a filesystem path.
8. Use in-memory processing where practical.
9. If temporary files are required, clean them up after success and failure.
10. Ensure exceptions do not leak file contents or sensitive paths.

### Tests

- Valid `.log` file
- Valid `.txt` file
- Missing file
- Unsupported extension
- Empty content
- Oversized file
- Invalid encoding
- Filename containing path-like characters
- Cleanup after processing failures

### Deliverables

- Upload-validation component
- Upload safety tests

### Exit criteria

Invalid uploads are rejected consistently, and valid text files can be passed to the parser without permanent storage.

---

## 8. Phase 3 — Log Parsing and Normalization

**Goal:** Convert supported raw log lines into normalized internal records.

### Tasks

1. Read the uploaded text according to the documented decoding policy.
2. Split content into source lines.
3. Preserve one-based source line numbers.
4. Identify supported timestamp formats.
5. Extract device and resource identifiers when available.
6. Preserve the original line text for evidence.
7. Classify supported and unrecognized lines according to explicit parser rules.
8. Track parsed and unparsed line counts.
9. Avoid interpreting unrecognized text as a confirmed fault.

### Tests

- Known supported log patterns
- Missing timestamps
- Invalid timestamps
- Missing device identifiers
- Missing resource identifiers
- Malformed lines
- Empty lines
- Mixed supported and unsupported formats
- Source line-number preservation

### Deliverables

- Parser service
- Normalized internal records
- Parser tests

### Exit criteria

The parser produces reproducible output and accurately reports parsing coverage.

---

## 9. Phase 4 — Deterministic Detection Engine

**Goal:** Convert normalized records into canonical `NetworkEvent` objects.

### Implementation order

Follow the rule registry in `DETECTION_RULES.md`:

1. `IFACE_DOWN_001`
2. `IFACE_UP_001`
3. `BGP_NEIGHBOR_DOWN_001`
4. `BGP_NEIGHBOR_UP_001`
5. `DEVICE_ERROR_001`
6. `AUTH_FAILURE_001`

### Tasks

1. Implement each rule as documented.
2. Assign stable rule IDs.
3. Extract event fields only when supported by the log evidence.
4. Preserve source evidence.
5. Assign the configured severity.
6. Assign a documented confidence level.
7. Generate structured recommendations.
8. Prevent duplicate detections for the same line and rule.
9. Keep unsupported patterns unclassified.
10. Track the implemented status of each rule.

### Tests

For every rule:

- Positive matching example
- Negative matching example
- Missing optional fields
- Evidence preservation
- Correct rule ID
- Correct event type
- Correct severity
- Structured recommendations
- False-positive cases
- Duplicate behavior

### Deliverables

- Detection service
- Rule registry
- Rule-specific tests

### Exit criteria

Every rule marked implemented has passing tests. Candidate rules that are not implemented must not be represented as active capabilities.

---

## 10. Phase 5 — Correlation and Analysis Summary

**Goal:** Combine supported results without inventing relationships.

### Tasks

1. Correlate only events that meet explicit criteria.
2. Require at least two distinct event IDs per correlation.
3. Validate that every related ID exists in the current analysis.
4. Use timestamps and resource identifiers only when available.
5. Preserve evidence supporting the relationship.
6. Keep `is_confirmed_cause = false` for ordinary rule-based correlations.
7. Compute event counts for all five severity levels.
8. Compute line and correlation counts.
9. Validate all summary invariants.

### Summary invariants

`total_lines = parsed_lines + unparsed_lines`

`total_events = info_events + low_events + medium_events + high_events + critical_events`

`total_correlations = len(correlations)`

### Tests

- Empty event list
- INFO-only events
- Events across multiple severity levels
- Valid correlation
- Duplicate event IDs in a correlation
- Unknown correlation references
- Missing timestamps
- Inconsistent summary counts

### Deliverables

- Correlation service
- Summary generation
- Correlation and summary tests

### Exit criteria

Correlations and summaries match the actual results and do not claim unsupported causality.

---

## 11. Phase 6 — Analysis Orchestration

**Goal:** Connect the components into one analysis pipeline.

### Processing order

```text
Upload
  → Validation
  → Text decoding
  → Parsing
  → Normalization
  → Detection
  → Correlation
  → Summary generation
  → AnalysisResult validation
  → JSON response
  → Temporary-resource cleanup
```

### Tasks

1. Create the analysis service.
2. Call each component in the documented order.
3. Generate an analysis ID for the current request.
4. Generate a timezone-aware UTC creation timestamp.
5. Build structured warnings for relevant processing limitations.
6. Determine `COMPLETED` or `PARTIAL` according to documented criteria.
7. Populate processing metadata.
8. Validate the final response.
9. Ensure cleanup is attempted on failure paths.
10. Keep the initial MVP AI-free.

### Tests

- Successful analysis with detected events
- Successful analysis with zero events
- Partial analysis with unrecognized lines
- Multiple detected events
- Invalid upload
- Parser failure
- Detection failure
- Summary validation failure
- Temporary-resource cleanup

### Deliverables

- Analysis orchestration service
- Integration tests

### Exit criteria

The pipeline produces a valid `AnalysisResult` for supported requests and handles failures without exposing internal details.

---

## 12. Phase 7 — FastAPI Endpoints

**Goal:** Expose the analysis pipeline through the documented HTTP interface.

### Required endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Process health |
| `GET` | `/ready` | Application readiness |
| `GET` | `/api/v1/info` | Capabilities and upload configuration |
| `POST` | `/api/v1/analyze` | Analyze an uploaded log file |

### Tasks

1. Create the FastAPI application.
2. Implement health and readiness endpoints.
3. Implement the information endpoint.
4. Implement the analysis upload endpoint.
5. Connect the endpoint to the analysis service.
6. Return the canonical `AnalysisResult`.
7. Define consistent HTTP error handling.
8. Enforce upload limits.
9. Avoid exposing tracebacks and sensitive content.
10. Verify FastAPI's generated API documentation.

### Tests

- Correct HTTP status codes
- Response schemas
- Missing upload
- Unsupported file
- Oversized upload
- Invalid content
- Internal failure handling
- Consistency between `/api/v1/info` and actual configuration

### Deliverables

- Working API
- API integration tests
- Updated API documentation

### Exit criteria

All four endpoints work as documented, and their responses match `API_SPECIFICATION.md`.

---

## 13. Phase 8 — Browser UI

**Goal:** Provide a simple interface for uploading logs and reviewing results.

### Required UI capabilities

1. Display the NetAssist AI application title.
2. Allow selection of `.log` and `.txt` files.
3. Display the upload-size limit.
4. Submit the selected file to `/api/v1/analyze`.
5. Show loading and processing states.
6. Display validation errors in understandable language.
7. Display the analysis status and summary counts.
8. Show detected events with severity, rule ID and description.
9. Allow users to inspect source evidence.
10. Display structured recommendations.
11. Display correlations separately from confirmed facts.
12. Show processing warnings and partial-analysis limitations.
13. Provide a clear empty-results state.

### UI safety requirements

- Escape or safely render uploaded log text.
- Never interpret log content as HTML or executable JavaScript.
- Do not automatically execute suggested commands.
- Do not claim the system has accessed a live router.
- Do not display unsupported root-cause claims.

### Tests

- Valid file submission
- API error display
- Loading state
- Empty event list
- Partial analysis
- Long evidence text
- Special characters in uploaded log content
- Responsive layout

### Deliverables

- Browser UI
- UI integration with the API
- UI validation checklist

### Exit criteria

A user can upload a sample log and review the resulting structured analysis in the browser.

---

## 14. Phase 9 — End-to-End Validation

**Goal:** Validate the full local application before declaring the MVP complete.

### Tasks

1. Run all unit tests.
2. Run API integration tests.
3. Run browser-to-API tests where practical.
4. Test each implemented detection rule.
5. Test empty, malformed and oversized uploads.
6. Verify summary counts.
7. Verify evidence line numbers.
8. Verify that no unsupported root-cause claims appear.
9. Verify temporary-file cleanup.
10. Review privacy and error handling.
11. Run a complete manual demonstration.
12. Update `PROJECT_STATUS.md` and `README.md`.

### Deliverables

- Test results
- Known limitations
- MVP completion checklist
- Demonstration procedure

### Exit criteria

All required MVP checks pass, known limitations are documented, and no critical unresolved issue blocks the demonstration.

---

## 15. Phase 10 — Optional Local AI (Deferred)

**Status:** Not part of the initial MVP.

Local AI may be considered only after the deterministic pipeline is implemented and tested.

### Potential capabilities

- Explain an existing event in simpler language.
- Summarize already-detected evidence.
- Organize existing recommendations into a readable narrative.

### Restrictions

- No AI-generated detections in the MVP.
- No AI-controlled severity changes.
- No fabricated evidence or timestamps.
- No autonomous network commands.
- No paid API requirement.
- No requirement to install or run a local model for the initial MVP.

Any future local model integration must be optional, explicitly configured and tested. The application must continue to function when AI is disabled.

---

## 16. Implementation Priority

The implementation priority is:

1. Canonical data models
2. Upload validation and file safety
3. Parsing and normalization
4. Deterministic detection rules
5. Correlation and summary generation
6. Analysis orchestration
7. FastAPI endpoints
8. Browser UI
9. End-to-end validation
10. Optional local AI evaluation

Do not skip the model and test foundations to build UI features prematurely.

## 17. Definition of Done

The initial MVP is complete when:

- [ ] Supported `.log` and `.txt` files can be uploaded safely.
- [ ] The upload-size limit is enforced.
- [ ] Supported lines are parsed with source line numbers.
- [ ] Implemented detection rules produce canonical `NetworkEvent` objects.
- [ ] Events include source evidence.
- [ ] Recommendations and warnings use structured models.
- [ ] Correlations follow documented rules.
- [ ] Summary counts satisfy all invariants.
- [ ] The API exposes all four required endpoints.
- [ ] The browser UI can submit a file and display the response.
- [ ] Error handling avoids exposing sensitive content.
- [ ] Temporary files are cleaned up when used.
- [ ] No permanent log or analysis storage is introduced.
- [ ] AI remains disabled in the initial MVP.
- [ ] Automated tests pass.
- [ ] Documentation reflects actual implementation status.
- [ ] Known limitations are documented honestly.

## 18. Working Method

Implementation must proceed one milestone at a time.

For each milestone:

1. Inspect the existing project files.
2. Identify the exact files to create or update.
3. Implement only the current milestone's scope.
4. Run the relevant tests.
5. Fix failures before proceeding.
6. Record the outcome in `PROJECT_STATUS.md`.
7. Move to the next milestone only when the current exit criteria are satisfied.

Do not recreate the repository, overwrite working code without review or introduce unrelated features during a milestone.

**Final decision:** Build a deterministic, testable local log analyzer first. Add the browser interface after the backend pipeline is stable. Evaluate optional local AI only after the initial MVP passes its acceptance tests.
