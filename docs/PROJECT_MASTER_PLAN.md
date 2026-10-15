# NetAssist AI — Master Project Plan

**Version:** 1.1
**Status:** Planning / Documentation Baseline
**Product:** Evidence-Based Network Troubleshooting Assistant  
**Budget:** Free tools and locally runnable components where practical
**Initial Deployment:** Local Windows PC
**Architecture:** Modular Monolith with Deterministic Analysis and Optional Local RAG/AI

---

## 1. Project Overview

NetAssist AI is a local-first network troubleshooting assistant designed to analyze router and switch log files, detect supported network events, preserve source evidence, identify possible relationships between events, and provide structured troubleshooting recommendations.

The project aims to make network-log investigation more organized and understandable without requiring engineers to manually inspect every log entry.

The application will use deterministic Python logic as its foundation. An optional future Retrieval-Augmented Generation (RAG) and local language-model layer may explain existing findings using a curated local knowledge base.

The core application must remain useful when all AI-related components are disabled or unavailable.

**Current project stage:** Documentation and planning. Features must not be described as implemented until their code and tests exist.

---

## 2. Problem Statement

Network devices can produce large volumes of logs containing interface transitions, routing events, authentication failures, device errors, and other operational messages.

Investigating these logs manually can be time-consuming because engineers must:

- Find relevant messages among unrelated entries.
- Determine which supported events occurred.
- Trace events back to their original log lines.
- Compare event timestamps and available device information.
- Distinguish observed facts from possible explanations.
- Decide which troubleshooting checks to perform next.

NetAssist AI will organize supported log observations into a structured analysis result that helps users investigate incidents more efficiently.

It will assist investigation, not replace professional judgment or claim to establish a root cause when the evidence is insufficient.

---

## 3. Product Objectives

The project aims to:

1. Accept supported router and switch log files.
2. Validate file type, size, and input handling.
3. Parse supported log formats using Python.
4. Detect documented network events through deterministic rules.
5. Preserve original evidence and source line numbers.
6. Assign severity according to deterministic rule definitions.
7. Identify candidate relationships between events where sufficient evidence exists.
8. Produce structured summaries and safe troubleshooting recommendations.
9. Provide a local browser-based interface for submitting logs and reviewing results.
10. Expose a documented FastAPI interface for analysis.
11. Include automated tests, representative fixtures, and documented limitations.
12. Add optional local RAG and AI explanations in future phases without changing authoritative detections.
13. Keep uploaded logs and analysis processing local by default.
14. Maintain a clear distinction between implemented functionality and planned enhancements.

---

## 4. Target Users

- Network engineers
- Network operations teams
- IT support engineers
- Network troubleshooting learners

The initial application is intended for local analysis of user-supplied log files, not autonomous network administration.

---

## 5. Product Scope

### 5.1 MVP — First Working Product

The initial MVP will focus on a reliable deterministic analysis workflow.

**Input and validation**

- Accept `.log` and `.txt` files.
- Enforce the documented maximum upload size of 5 MiB.
- Validate supported file types and handle malformed inputs safely.
- Preserve original line numbers and relevant source evidence.

**Parsing and detection**

- Implement a modular parser for supported log formats.
- Implement documented deterministic detection rules.
- Start with supported interface state transitions, BGP neighbor events, device errors, and authentication failures as defined in `DETECTION_RULES.md`.
- Preserve unknown or unsupported messages without inventing classifications.
- Use stable rule identifiers and canonical data models.

**Correlation and summaries**

- Apply documented correlation rules where applicable.
- Distinguish candidate relationships from confirmed causation.
- Generate deterministic summaries and structured recommendations.
- Preserve the original event evidence and deterministic severity.

**Backend API**

The MVP will implement the endpoints defined in `API_SPECIFICATION.md`:

- `GET /health`
- `GET /ready`
- `GET /api/v1/info`
- `POST /api/v1/analyze`

The analysis endpoint will return the canonical `AnalysisResult` described in `DATA_MODELS.md`.

**Browser interface**

- Provide a local browser-based interface for uploading a supported file.
- Display analysis results and relevant evidence.
- Handle validation and processing errors clearly.
- Keep the UI consistent with the actual API response contract.

**Quality and privacy**

- Include unit tests and API tests.
- Use representative sample logs and expected results.
- Avoid permanent storage of uploaded logs and results by default.
- Keep AI and RAG disabled during the initial MVP.

Detailed filtering, search, report export, and additional UI features must be included in the approved implementation scope before being treated as MVP requirements.

### 5.2 Future Enhancements

The following are potential enhancements, not current MVP commitments:

- Additional vendor-specific log formats.
- More detection rules for VLAN, ARP, MAC learning, packet drops, and routing events.
- Improved timestamp and device-aware event correlation.
- Optional local troubleshooting knowledge base.
- Document chunking, local embeddings, and vector retrieval.
- Optional local LLM explanations with validated source citations.
- Downloadable reports and additional export formats.
- Search, filtering, and advanced dashboard features.
- Optional local analysis history and database storage.
- Authentication and multi-user access if required.
- Authorized read-only network-device integrations.
- Deployment for an approved team environment.

Any feature that changes persistence, data exposure, deployment, or security assumptions requires a separate design review before implementation.

---

## 6. Technology Strategy

The project will prioritize free, maintainable tools and local execution.

| Component | Planned technology | Scope |
|---|---|---|
| Programming language | Python | Core implementation |
| Backend API | FastAPI | Local HTTP interface |
| Data validation | Pydantic | Canonical models and request/response validation |
| Log parsing | Python standard library and regular expressions where suitable | Deterministic parsing |
| Frontend | HTML, CSS, JavaScript | Local browser UI |
| ASGI server | Uvicorn | Local application serving |
| Testing | pytest and HTTPX | Unit and API tests |
| Version control | Git and GitHub | Source and documentation history |
| Optional local LLM | Ollama or a suitable alternative | Future AI phase |
| Optional embeddings | A compatible local embedding model | Future RAG phase |
| Optional vector retrieval | FAISS or Chroma | Future RAG phase |
| Optional persistence | SQLite | Future enhancement only |

These are planned choices. Compatibility, dependency requirements, model licenses, and local hardware constraints must be checked before adopting optional AI/RAG components.

The initial MVP must not require a paid API, paid subscription, external AI service, or downloaded AI model.

Local models may require an initial download, disk space, memory, and processing resources. Once installed and configured, the optional AI/RAG components should operate locally without transmitting private log data externally.

---

## 7. Architecture Principles

The project will follow these principles:

1. **Deterministic first:** The core analysis pipeline must not depend on AI.
2. **Modular design:** Parsing, detection, correlation, orchestration, API, UI, RAG, and AI responsibilities remain separated.
3. **Evidence preservation:** Detected events must retain traceable source evidence.
4. **Canonical contracts:** Data models and API responses must follow their documented specifications.
5. **Conservative detection:** Unsupported observations must not become invented events.
6. **Explicit uncertainty:** Correlation does not automatically establish causation.
7. **Safe recommendations:** The application suggests checks but does not execute them.
8. **Local-first privacy:** Uploaded logs must not be sent to external AI services by default.
9. **Optional AI:** AI and RAG failures must not invalidate deterministic analysis.
10. **RAG separation:** Retrieved reference material can support explanations but cannot establish network observations.
11. **No implicit persistence:** Uploaded logs and analysis results are not permanently stored by default.
12. **Test before expansion:** Each feature must meet its acceptance criteria before the next phase begins.
13. **Document actual behavior:** Documentation must distinguish current implementation from future design.

---

## 8. High-Level Architecture

The planned system follows this workflow:

```text
Browser UI
    |
    v
FastAPI Application
    |
    v
Upload Validation
    |
    v
Log Parser
    |
    v
Deterministic Detection Engine
    |
    v
Correlation and Summary Logic
    |
    v
Canonical AnalysisResult
    |
    +----------------------------+
    |                            |
    v                            v
Deterministic Output       Optional RAG/AI
                                 |
                                 v
                       Local Knowledge Retrieval
                                 |
                                 v
                        Local AI Explanation
                                 |
                                 v
                         Output Validation
                                 |
                                 v
                       Validated Explanation
                                 |
                                 v
                         Browser Presentation
```

The optional AI/RAG branch will be implemented only after the deterministic workflow is stable.

The canonical analysis result remains authoritative. Optional explanations must not silently modify the events, evidence, severity, or correlations in that result.

The implementation will use a modular monolith rather than introducing microservices for the initial local application.

---

## 9. Development Roadmap

Implementation will follow the detailed phase order in `IMPLEMENTATION_PLAN.md`.

### Phase 0 — Repository and Environment

- Verify the project repository and documentation baseline.
- Confirm Python and Git availability.
- Prepare the virtual environment and dependency configuration.
- Establish the test framework and development workflow.
- Review `.gitignore` and repository hygiene.

**Exit criteria:** The development environment and repository are ready for implementation.

### Phase 1 — Canonical Data Models

- Implement the models specified in `DATA_MODELS.md`.
- Define the canonical enums and structured fields.
- Validate model invariants and serialization.
- Add unit tests for valid and invalid model data.

**Exit criteria:** The canonical data models and their tests are stable.

### Phase 2 — Upload Validation

- Implement the documented file type and size limits.
- Validate upload handling.
- Handle malformed or unsupported files safely.
- Ensure temporary resources are cleaned up.

**Exit criteria:** Supported files pass validation and invalid files are rejected safely.

### Phase 3 — Log Parsing

- Implement a modular parser for the initial supported formats.
- Preserve line numbers and relevant timestamps.
- Handle blank, malformed, and unknown lines conservatively.
- Add parser fixtures and unit tests.

**Exit criteria:** Parsing behavior is predictable and testable.

### Phase 4 — Deterministic Detection

Implement the documented detection rules in `DETECTION_RULES.md`, beginning with:

- `IFACE_DOWN_001`
- `IFACE_UP_001`
- `BGP_NEIGHBOR_DOWN_001`
- `BGP_NEIGHBOR_UP_001`
- `DEVICE_ERROR_001`
- `AUTH_FAILURE_001`

Additional rules must be documented and tested before being added.

**Exit criteria:** Known test cases produce expected events, severity, and evidence without requiring AI.

### Phase 5 — Correlation and Summary

- Implement documented candidate correlation logic.
- Preserve related event IDs and evidence.
- Avoid asserting unsupported causal relationships.
- Generate severity counts and deterministic summaries.
- Generate structured recommendations.

**Exit criteria:** Correlations and summaries follow the canonical models and pass unit tests.

### Phase 6 — Analysis Orchestration

- Connect validation, parsing, detection, correlation, and summary logic.
- Produce the canonical `AnalysisResult`.
- Handle partial or failed processing according to documented behavior.
- Add orchestration tests.

**Exit criteria:** The end-to-end internal analysis pipeline produces consistent results.

### Phase 7 — FastAPI

- Implement the four documented MVP endpoints.
- Validate request and response behavior.
- Return the documented analysis result and error envelopes.
- Verify upload limits and privacy behavior.
- Add API tests using HTTPX.

**Exit criteria:** All documented API behavior is implemented and tested.

### Phase 8 — Browser UI

- Implement local file submission.
- Display analysis results and evidence.
- Display errors without losing the original context unnecessarily.
- Connect the UI to the documented API.
- Verify behavior with representative sample files.

**Exit criteria:** A user can submit a supported log file and review the returned analysis through the browser.

### Phase 9 — End-to-End Validation

- Run the complete test suite.
- Verify deterministic behavior across representative fixtures.
- Review privacy and temporary-file cleanup.
- Validate installation instructions in a clean environment.
- Document limitations and unsupported log formats.

**Exit criteria:** The deterministic MVP meets its acceptance criteria.

### Phase 10 — Optional Local RAG and AI

This phase begins only after the deterministic MVP is stable.

- Select approved local reference documents.
- Implement document ingestion, chunking, and metadata preservation.
- Evaluate local embeddings and vector retrieval.
- Implement source and citation validation.
- Evaluate a compatible local LLM.
- Implement the optional AI adapter and output validation.
- Add timeouts, size limits, and deterministic fallback behavior.
- Update canonical models and the API specification before exposing new response fields.
- Verify that local-only mode makes no external AI requests.

**Exit criteria:** Optional explanations and references can be provided without changing authoritative deterministic analysis.

### Phase 11 — Optional Product Enhancements

Only after the core system is stable, consider:

- Additional detection rules and log formats.
- Enhanced search and filtering.
- Downloadable reports.
- Optional local history or database storage.
- Authentication and multi-user deployment.
- Authorized read-only device integrations.

Each enhancement requires its own scope, security review, tests, and acceptance criteria.

---

## 10. MVP Acceptance Criteria

The deterministic MVP is ready for demonstration when:

- Supported `.log` and `.txt` files can be submitted within the documented size limit.
- Unsupported and malformed inputs are handled safely.
- Supported test fixtures produce expected deterministic detections.
- Each detected event retains its original evidence and line references.
- Severity is assigned according to deterministic rules.
- Correlations are not presented as confirmed causation without sufficient evidence.
- Structured summaries and recommendations conform to the canonical models.
- The four documented API endpoints behave as specified.
- The browser UI can display analysis results and errors.
- Automated unit and API tests pass.
- Temporary processing resources are cleaned up appropriately.
- The application does not require an AI model, embedding model, vector index, or paid API.
- No external service receives uploaded logs by default.
- Setup instructions work in a clean local environment.
- Unsupported formats and known limitations are documented.
- The implementation status accurately reflects completed work.

---

## 11. Security and Privacy Boundaries

The initial version analyzes user-supplied files locally.

It will not:

- Connect to production routers or switches.
- Execute commands contained in log files.
- Automatically run diagnostic commands on devices.
- Modify network-device configurations.
- Treat a correlation candidate as proof of causation.
- Send private network logs to an external AI service automatically.
- Permanently store uploaded logs or analysis results by default.

Additional controls:

- Enforce upload validation and documented size limits.
- Treat log content and retrieved documents as untrusted data.
- Keep secrets out of source control.
- Avoid unnecessary logging of uploaded contents.
- Prevent optional AI components from executing commands or controlling devices.
- Require a separate design review before adding persistence, external services, authentication, or device integrations.

See `SECURITY_AND_PRIVACY.md` for the detailed security and privacy requirements.

---

## 12. Testing and Evaluation Strategy

Testing will cover individual modules and end-to-end behavior.

**Unit testing**

- Canonical data models and validation.
- File validation.
- Parser behavior.
- Detection rule matching and non-matching cases.
- Evidence preservation.
- Correlation logic.
- Summary counts and recommendation structures.

**API testing**

- Health and readiness responses.
- Application information.
- Valid and invalid uploads.
- Size-limit enforcement.
- Canonical response structure.
- Error handling and cleanup.

**End-to-end testing**

- Representative sample logs.
- Expected detections and evidence.
- Browser-to-API interaction.
- Consistent results across repeated deterministic runs.

**Future AI/RAG testing**

- Retrieval relevance.
- Citation validity.
- Unsupported-claim frequency.
- Prompt-injection resistance.
- Timeout and fallback behavior.
- No change to deterministic events or severity.
- No external data transmission in local-only mode.

Evaluation metrics must be based on reviewed test cases. Precision, recall, false-positive rates, retrieval quality, and latency should be reported only after the relevant tests and measurements have been performed.

---

## 13. Project Documentation

The following documents define the project's design and implementation contracts:

| Document | Responsibility |
|---|---|
| `README.md` | Project introduction, setup, and current status |
| `docs/PROJECT_MASTER_PLAN.md` | Product scope, objectives, roadmap, and acceptance criteria |
| `docs/PRD.md` | Product requirements |
| `docs/ARCHITECTURE.md` | System components, boundaries, and data flow |
| `docs/TECH_STACK.md` | Technology selection and dependencies |
| `docs/DETECTION_RULES.md` | Supported rules and deterministic detection behavior |
| `docs/AI_DESIGN.md` | Optional AI, RAG, citations, validation, and fallback |
| `docs/SECURITY_AND_PRIVACY.md` | Security boundaries and privacy requirements |
| `docs/API_SPECIFICATION.md` | Public API endpoints and response contracts |
| `docs/DATA_MODELS.md` | Canonical models and enums |
| `docs/TESTING_STRATEGY.md` | Unit, API, integration, and future AI/RAG testing |
| `docs/IMPLEMENTATION_PLAN.md` | Ordered implementation tasks and exit criteria |
| `docs/PROJECT_STATUS.md` | Actual implementation status and known limitations |
| `docs/DEVELOPMENT_WORKFLOW.md` | Development and Git workflow |
| `docs/REQUIREMENTS_TRACEABILITY.md` | Requirements mapped to implementation and tests |

If a design change affects more than one document, update all affected documents before implementing the change.

In particular, changes to response fields must be reflected in both `DATA_MODELS.md` and `API_SPECIFICATION.md`.

---

## 14. Definition of Done

A feature is complete only when:

- Its implementation exists.
- Its expected behavior is documented.
- Relevant tests cover success and failure cases.
- Input validation and error handling are included where required.
- Security and privacy implications have been reviewed.
- Its acceptance criteria pass.
- Related documentation is consistent.
- Its actual status is reflected in `PROJECT_STATUS.md`.

A planned feature, design example, proposed configuration, or illustrative code snippet must not be described as implemented functionality.

---

## 15. Final Deliverables

The intended deliverables are:

- Source code and Git history.
- Product, architecture, and technical documentation.
- Canonical data models and documented API.
- Sample logs and expected outputs.
- Automated tests and recorded test results.
- Working local browser application.
- Installation and troubleshooting instructions.
- Security notes and known limitations.
- Demonstration and release checklist.
- Optional RAG knowledge base and local AI explanation capability in a later phase, if implemented and validated.

The final deliverable set must distinguish completed features from future enhancements.

---

## 16. Final Project Decision

NetAssist AI will be built as a **local-first, deterministic network log analysis application with optional future RAG and local AI assistance**.

The first priority is a working, testable system that validates files, parses supported logs, detects documented events, preserves evidence, produces structured results, and exposes the documented API through a simple browser interface.

The optional RAG layer will later retrieve relevant material from a curated local knowledge base. The optional local AI layer may explain findings using that reference material and the original analysis evidence.

Neither RAG nor AI will replace deterministic detection or become a requirement for core functionality.

**The deterministic analysis result remains the source of truth. Local RAG provides supporting reference material. AI explanations remain optional, advisory, and subject to validation.**

All development will proceed incrementally, with free/local-first tools, documented acceptance criteria, automated tests, privacy by default, and no claims of functionality that has not yet been implemented.