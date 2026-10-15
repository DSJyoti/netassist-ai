# NetAssist AI — System Architecture

**Document:** System Architecture  
**Project:** NetAssist AI  
**Version:** 1.0  
**Status:** Design Phase  
**Architecture Style:** Modular Layered Architecture  
**Backend:** Python + FastAPI  
**Frontend:** HTML + CSS + JavaScript  
**Core Principle:** Deterministic, Evidence-Based Analysis

---

## 1. Architecture Overview

NetAssist AI is a local-first network log analysis application that helps users identify network-related events from uploaded log files.

The initial Minimum Viable Product (MVP) uses deterministic detection rules to identify supported conditions, preserve supporting evidence, correlate potentially related events, and provide safe troubleshooting recommendations.

The architecture separates the following responsibilities:

- Browser-based user interface.
- FastAPI application and API layer.
- File validation and temporary input handling.
- Log parsing.
- Deterministic event detection.
- Event normalization.
- Event correlation.
- Recommendation generation.
- Analysis result construction.
- API response generation.

Optional AI explanations, Retrieval-Augmented Generation (RAG), persistent analysis history, and device integrations are future extensions, not MVP requirements.

### High-Level Architecture

```text
                 ┌────────────────────────┐
                 │         User           │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │     Web Dashboard      │
                 │   HTML/CSS/JavaScript  │
                 └────────────┬───────────┘
                              │ HTTP / REST
                              ▼
                 ┌────────────────────────┐
                 │       FastAPI          │
                 │     API Layer          │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │    Upload Validation   │
                 │ Type / Size / Encoding │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │      Log Parser        │
                 │ Records / Timestamps   │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   Detection Engine     │
                 │     Rule Matching      │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │  Event Normalization   │
                 │   Structured Events    │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   Correlation Engine   │
                 │ Potential Relationships│
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │ Recommendation Engine  │
                 │ Safe Advisory Checks   │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   Result Construction  │
                 │    AnalysisResult      │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │     API Response       │
                 │       JSON             │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │     Web Dashboard      │
                 │ Events / Evidence      │
                 └────────────────────────┘
```

The initial architecture does not require a database, external AI service, or production network device access.

## 2. Core Architecture Principles

The most important design decisions are:

```text
AI is optional.
Detection is mandatory.
Evidence is mandatory.
Privacy is mandatory.
```

The system must continue to perform core analysis if:

- No LLM is installed.
- Internet access is unavailable.
- AI configuration is disabled.
- An optional AI service is unavailable.

The deterministic detection engine is the source of truth for detected events and their severity.

The same input and rule version should produce the same core detection results, subject to explicitly controlled configuration and processing behavior.

## 3. Architectural Layers

The MVP uses these logical layers:

```text
Layer 1  → Presentation
Layer 2  → API
Layer 3  → File Validation
Layer 4  → Parsing
Layer 5  → Detection
Layer 6  → Event Normalization
Layer 7  → Correlation
Layer 8  → Recommendations
Layer 9  → Analysis Result Construction
Layer 10 → API Response and Presentation
```

AI/RAG is a future extension and is not part of the required MVP processing path.

Each layer must have a clearly defined responsibility and be independently testable wherever practical.

## 4. Layer 1 — Presentation

The presentation layer is a browser-based dashboard built using:

- HTML
- CSS
- JavaScript

Responsibilities:

- Upload `.log` and `.txt` files.
- Display validation and analysis errors.
- Display analysis status.
- Display summary counts.
- Display detected events and severity.
- Display evidence and source line numbers.
- Display potential correlations.
- Display troubleshooting recommendations.
- Search and filter displayed results.
- Present the returned analysis result.

The frontend must not implement the core detection rules. Detection logic belongs in the backend.

Log-derived content must be rendered safely. Uploaded text must never be interpreted as executable HTML or JavaScript.

Separate export functionality is not required for the initial MVP.

## 5. Layer 2 — API

The API layer uses FastAPI.

### MVP Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Basic application health |
| GET | `/ready` | Application readiness |
| GET | `/api/v1/info` | Application and supported capability information |
| POST | `/api/v1/analyze` | Validate and analyze an uploaded log file |

The complete endpoint contract is defined in `docs/API_SPECIFICATION.md`.

The `POST /api/v1/analyze` endpoint returns the complete `AnalysisResult` directly in the response.

The MVP does not include:

- An analysis-history endpoint.
- A saved-analysis retrieval endpoint.
- A separate saved-result export endpoint.
- An endpoint for executing commands.
- An endpoint for modifying network-device configuration.

The API must validate input, return controlled HTTP errors, and avoid exposing internal stack traces or sensitive server details.

## 6. Layer 3 — File Validation

Uploaded files must pass validation before parsing begins.

Validation includes:

- Approved file extension: `.log` or `.txt`.
- Configurable maximum file size, initially 5 MB.
- Empty-file rejection.
- Safe text decoding.
- Filename handling.
- Path traversal prevention.
- Resource-limit checks.

The supplied filename must never be trusted as a filesystem path.

The application must not trust a MIME type or extension alone to determine whether an input is safe.

### Validation Flow

```text
Upload Request
      │
      ▼
Extension Check
      │
      ▼
File Size Check
      │
      ▼
Empty File Check
      │
      ▼
Safe Text Decoding
      │
      ▼
Validation Successful?
      │
  ┌───┴────┐
  │        │
 Yes       No
  │        │
  ▼        ▼
Parser   Controlled
         API Error
```

Invalid files must be rejected before they reach the detection engine.

## 7. Temporary File Handling and Cleanup

The MVP must not permanently store uploaded logs or analysis results.

The preferred approach is to process uploaded content in memory when practical.

If temporary disk storage is needed:

- Use a controlled temporary directory.
- Generate safe temporary filenames.
- Avoid trusting user-provided paths.
- Remove temporary files after processing.
- Attempt cleanup if parsing or analysis fails.
- Avoid creating permanent copies or archives.
- Do not write full log contents to application diagnostics.

The analysis result should be held in memory only as needed to construct and return the API response.

No database is required.

Temporary-file cleanup must be designed into the request lifecycle rather than treated as an optional later enhancement.

## 8. Layer 4 — Log Parser

The parser converts raw text lines into structured log records.

Example input:

```text
2026-10-15 10:31:22 %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

Illustrative parsed record:

```json
{
  "line_number": 154,
  "timestamp": "2026-10-15T10:31:22",
  "severity_code": "3",
  "source": "LINK",
  "message": "Interface GigabitEthernet0/1, changed state to down",
  "raw_line": "2026-10-15 10:31:22 %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down"
}
```

This is an illustrative internal parser record, not the final API response schema.

### Parser Responsibilities

- Process log lines safely.
- Preserve original source line numbers.
- Extract timestamps where possible.
- Extract known vendor or system identifiers where possible.
- Retain enough source information to support detections.
- Handle malformed timestamps and unexpected characters.
- Continue after non-fatal parsing errors.
- Record parsing limitations.
- Avoid executing or interpreting log content as code.

Unknown lines should not automatically be treated as fatal errors.

The parser must follow the resource limits defined in the security requirements.

## 9. Layer 5 — Deterministic Detection Engine

The detection engine applies documented rules to parsed log records.

```text
Parsed Records
      │
      ▼
Rule Matching
      │
      ▼
Matched Rule
      │
      ▼
Detection Evidence
      │
      ▼
Structured Event
```

Initial detection categories may include:

- Interface state changes.
- BGP neighbor events.
- Packet and interface errors.
- CRC errors.
- VLAN-related events.
- ARP events.
- MAC-related events.
- Routing events.

The implemented rules must be explicitly defined in `docs/DETECTION_RULES.md`.

A category must not be represented as supported merely because it appears in this architecture document. Actual support depends on implemented and tested rules.

## 10. Rule-Based Detection

Every detection rule must have a stable identifier.

Illustrative identifiers:

```text
IFACE_DOWN_001
IFACE_UP_001
IFACE_FLAP_001
BGP_NEIGHBOR_DOWN_001
CRC_ERROR_001
ARP_FAILURE_001
```

Each rule should define:

- Rule ID.
- Rule name.
- Description.
- Match conditions.
- Required fields.
- Severity.
- Event description.
- Evidence requirements.
- Recommendation mapping.

Example:

```json
{
  "rule_id": "IFACE_DOWN_001",
  "event_type": "interface_down",
  "severity": "HIGH"
}
```

The final severity must be assigned by the rule. An AI model must not independently change it.

## 11. Layer 6 — Event Normalization

Different network vendors and systems may represent similar conditions differently.

For example:

```text
Interface Gi0/1 down

Interface GigabitEthernet0/1 changed state to down

Gi0/1 operational state = down
```

The normalization layer converts supported detections into a consistent event representation.

Illustrative normalized fields:

```json
{
  "event_type": "interface_down",
  "resource_type": "interface",
  "resource_id": "GigabitEthernet0/1",
  "severity": "HIGH"
}
```

These fields describe the concept of normalization. The actual implemented schema must follow `docs/DATA_MODELS.md`.

Normalization must not invent missing resource identifiers or timestamps.

## 12. Canonical Network Event Model

The canonical event model is `NetworkEvent`, defined in `docs/DATA_MODELS.md`.

It includes fields such as:

- `event_id`
- `rule_id`
- `title`
- `severity`
- `timestamp`
- `device`
- `interface`
- `description`
- `evidence`
- `possible_causes`
- `recommendations`
- `confidence`
- `tags`

The exact schema, required fields, optional fields, and controlled values are defined in `DATA_MODELS.md`.

The architecture document must not introduce conflicting field names or a separate incompatible event contract.

Each detected event must have supporting evidence. Event IDs must be unique within the current analysis.

## 13. Layer 7 — Correlation Engine

The correlation engine identifies potential relationships among detected events.

Potential correlation dimensions include:

- Time.
- Device or resource.
- Interface.
- IP address.
- BGP neighbor.
- VLAN.
- Event type.

Example:

```text
10:31:22 Interface Gi0/1 DOWN

10:31:24 BGP Neighbor 192.0.2.2 DOWN
```

If the available evidence supports a relationship, the engine may produce a correlation.

```text
Potential relationship detected.
Confidence: MEDIUM
```

The confidence value must follow the controlled values defined in `DATA_MODELS.md`.

A correlation must reference existing event IDs from the same analysis and provide supporting evidence.

## 14. Correlation Safety

The correlation engine must distinguish:

```text
Event A happened before Event B
```

from:

```text
Event A caused Event B
```

The first may be established from timestamps.

The second requires additional evidence.

Therefore:

```text
Temporal relationship ≠ Confirmed root cause
```

The initial MVP should not mark a cause as confirmed merely because events occur close together.

Correlation results must communicate uncertainty honestly.

## 15. Layer 8 — Recommendation Engine

The recommendation engine maps detected events to safe troubleshooting guidance.

Example:

```text
Event:
Interface CRC errors

Possible recommendations:
1. Review interface counters.
2. Inspect the physical cable.
3. Check the transceiver.
4. Review the remote interface.
```

Recommendations must be:

- Relevant to the detected event.
- Grounded in the available evidence.
- Human-verifiable.
- Clear about uncertainty.
- Advisory rather than automatically executed.

The application must not execute commands, connect to production devices, or change network configurations.

The final recommendation schema must match `Recommendation` in `DATA_MODELS.md`.

## 16. Layer 9 — Analysis Result Construction

The analyzer service orchestrates the processing pipeline and constructs an `AnalysisResult`.

Conceptually:

```text
Uploaded Content
       │
       ▼
Validation
       │
       ▼
Parsing
       │
       ▼
Detection
       │
       ▼
Event Normalization
       │
       ▼
Correlation
       │
       ▼
Recommendations
       │
       ▼
Summary and Warnings
       │
       ▼
AnalysisResult
```

The result includes:

- `analysis_id`
- `status`
- `filename`
- `created_at`
- `summary`
- `events`
- `correlations`
- `warnings`
- `metadata`

These names must match `docs/DATA_MODELS.md`.

The analysis ID identifies the current result. It does not imply persistent storage or later retrieval.

A valid analysis with no detected events should return a valid result with an empty event list and an appropriate warning where useful.

Summary counts must match the returned events.

## 17. API Response and Frontend Rendering

The API returns the complete `AnalysisResult` directly from:

```text
POST /api/v1/analyze
```

The frontend uses this response to render the analysis summary, events, evidence, correlations, recommendations, warnings, and processing metadata.

```text
Browser
   │
   ▼
POST /api/v1/analyze
   │
   ▼
FastAPI
   │
   ▼
Analysis Service
   │
   ▼
AnalysisResult
   │
   ▼
JSON Response
   │
   ▼
Browser Dashboard
```

The initial MVP does not require:

- A database.
- A permanent result repository.
- Analysis history.
- Retrieval of previous analysis results.
- Separate saved-report export endpoints.

The browser may temporarily hold the current response to render the page. The backend must not promise that the result remains available after the request finishes.

## 18. Optional AI Layer — Future Phase

AI explanations are deferred until the deterministic MVP is implemented and tested.

A future AI layer may explain structured events, summarize incidents, or answer questions about the evidence.

Possible future responsibilities:

- Explain detected events in plain language.
- Summarize a set of related events.
- Explain networking concepts.
- Answer questions about the current analysis.
- Produce human-readable explanations.

AI must remain separate from deterministic detection.

```text
Deterministic Analysis
          │
          ▼
Validated Structured Events
          │
          ▼
Optional AI Explanation
          │
          ▼
Validated Explanation
```

AI must not:

- Create authoritative detections without matching rules.
- Change event severity.
- Change rule IDs or source evidence.
- Execute commands.
- Modify network devices.
- Present an unverified root cause as established fact.

The initial MVP must work with AI disabled.

## 19. AI Input and Output Boundaries

If AI is added later, its input should use relevant structured events and sanitized evidence rather than unrestricted raw logs whenever practical.

Before sending context to a model, the application must:

- Limit the evidence provided.
- Redact recognized secrets where practical.
- Treat all log-derived content as untrusted data.
- Avoid sending data to external services unless a separate, explicit design decision permits it.

AI output must be validated before display.

References to nonexistent events must be rejected or flagged. AI-generated explanations must remain distinguishable from deterministic observations.

The initial MVP does not require an LLM, embedding model, vector database, or RAG pipeline.

## 20. RAG Architecture — Future Scope

A future RAG layer may retrieve relevant networking knowledge to support explanations.

Potential approved sources include:

- Public networking documentation.
- Troubleshooting guides.
- Vendor-neutral networking references.
- RFCs.
- Authorized internal runbooks.

Future flow:

```text
User Question
      │
      ▼
Query Processing
      │
      ▼
Knowledge Retrieval
      │
      ▼
Relevant Context
      │
      ▼
Optional LLM
      │
      ▼
Validated Explanation
```

Only approved and appropriately licensed content should be used.

RAG must not be required for deterministic log analysis.

## 21. Reporting

The initial MVP returns a structured JSON analysis result and renders it in the browser dashboard.

The result should contain:

- Analysis summary.
- Detected events.
- Severity.
- Evidence.
- Potential correlations.
- Recommendations.
- Warnings and limitations.
- Processing metadata.

The initial MVP does not require PDF generation, permanent reports, or separate export endpoints.

Future Markdown, PDF, or other export functionality may be considered separately.

All report content must follow the data models and security requirements.

## 22. End-to-End Data Flow

```text
User
 │
 ▼
Upload Log
 │
 ▼
File Validation
 │
 ▼
Temporary Input Handling
 │
 ▼
Log Parser
 │
 ▼
Parsed Records
 │
 ▼
Detection Engine
 │
 ▼
Normalized Events
 │
 ▼
Correlation Engine
 │
 ▼
Recommendation Engine
 │
 ▼
Analysis Summary and Warnings
 │
 ▼
AnalysisResult
 │
 ▼
JSON API Response
 │
 ▼
Browser Dashboard
 │
 ▼
Temporary Data Cleanup
```

Cleanup must be attempted on both successful and unsuccessful processing paths.

The result is returned directly to the client; it is not written to a permanent database.

## 23. Error Handling

Errors must be handled at the appropriate layer.

### Invalid Upload

```text
Invalid File
     │
     ▼
Validation Failure
     │
     ▼
Controlled API Error
```

### Non-Fatal Parsing Problem

```text
Malformed Line
     │
     ▼
Record Parsing Limitation
     │
     ▼
Continue Where Safe
     │
     ▼
Include Warning
```

### Unexpected Internal Error

```text
Internal Error
     │
     ▼
Attempt Temporary Cleanup
     │
     ▼
Controlled API Error
```

Unexpected errors must not expose raw stack traces, secrets, or full filesystem paths to the browser.

Error responses must follow `docs/API_SPECIFICATION.md`.

## 24. Project Structure

The proposed MVP project structure is:

```text
netassist-ai/
│
├── docs/
│   ├── PROJECT_MASTER_PLAN.md
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── TECH_STACK.md
│   ├── DETECTION_RULES.md
│   ├── AI_DESIGN.md
│   ├── SECURITY_AND_PRIVACY.md
│   ├── API_SPECIFICATION.md
│   ├── DATA_MODELS.md
│   ├── TESTING_STRATEGY.md
│   ├── IMPLEMENTATION_PLAN.md
│   ├── PROJECT_STATUS.md
│   ├── DEVELOPMENT_WORKFLOW.md
│   └── REQUIREMENTS_TRACEABILITY.md
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   │   └── routes/
│   │   ├── core/
│   │   │   └── config.py
│   │   ├── models/
│   │   ├── services/
│   │   │   ├── analyzer.py
│   │   │   ├── parser.py
│   │   │   ├── detector.py
│   │   │   ├── correlator.py
│   │   │   └── recommender.py
│   │   └── rules/
│   └── tests/
│
├── frontend/
│   ├── index.html
│   ├── css/
│   └── js/
│
├── sample_logs/
│
├── tests/
│
├── README.md
├── pyproject.toml
└── .gitignore
```

This is a proposed layout, not a requirement to create every file immediately.

Optional AI modules, device integrations, database modules, and report export modules should be added only when their respective future phases are approved.

The exact file layout can be simplified during implementation without changing the architectural responsibilities.

## 25. Dependency Direction

The architecture must maintain clear dependency direction.

```text
API Layer
    │
    ▼
Analyzer Service
    │
    ├── Parser
    ├── Detection Engine
    ├── Correlation Engine
    ├── Recommendation Engine
    └── Result Construction
             │
             ▼
          Data Models
```

The parser must not depend on FastAPI.

The detection engine must not depend on the browser UI.

Rules must not depend on frontend code.

The analyzer service orchestrates components, while shared models define consistent data contracts.

This structure enables unit testing without starting the web application.

## 26. Analyzer Service

The analyzer service is the main orchestrator for one analysis request.

Conceptually:

```text
analyze(upload)
       │
       ├── Validate input
       ├── Parse records
       ├── Detect events
       ├── Normalize events
       ├── Correlate events
       ├── Generate recommendations
       ├── Build summary and warnings
       ├── Build AnalysisResult
       └── Return result
```

The service must not require a database or AI model.

Temporary file cleanup must be coordinated with the request lifecycle. Where feasible, in-memory processing should be preferred.

## 27. Analysis Result Contract

The final result schema is defined in `docs/DATA_MODELS.md`.

Illustrative response structure:

```json
{
  "analysis_id": "analysis-example-001",
  "status": "COMPLETED",
  "filename": "router.log",
  "created_at": "2026-10-15T10:31:22Z",
  "summary": {
    "total_lines": 500,
    "parsed_lines": 480,
    "unparsed_lines": 20,
    "total_events": 8,
    "critical_events": 0,
    "high_events": 2,
    "medium_events": 4,
    "low_events": 2
  },
  "events": [],
  "correlations": [],
  "warnings": [],
  "metadata": {
    "parser_version": "1.0.0",
    "ruleset_version": "1.0.0",
    "processing_duration_ms": 120,
    "ai_used": false,
    "input_size_bytes": 1024
  }
}
```

This is a schema illustration, not a claim that these counts or timings have been measured.

The final serialized field names and value constraints must match `DATA_MODELS.md`.

The result is returned directly from the analysis endpoint. It is not persisted by default.

## 28. No Production Device Access in the MVP

The initial architecture stops at uploaded-log analysis.

```text
Uploaded Log
     │
     ▼
NetAssist AI
     │
     ▼
Analysis Result
```

It does not include:

```text
NetAssist AI
     │
     ▼
SSH / Network Device API
     │
     ▼
Production Router
```

It also does not include automatic network configuration changes.

This boundary reduces operational risk and keeps the MVP suitable for safe testing with synthetic or authorized logs.

## 29. Future Read-Only Device Integration

Device integration is future scope and must be designed as a separate component.

Any future integration must define:

- Authentication.
- Authorization.
- Explicit read-only permissions.
- Credential protection.
- Connection timeouts.
- Audit requirements.
- Failure handling.
- Security testing.

It must not be added implicitly through the recommendation engine or optional AI layer.

## 30. Observability

The MVP should provide minimal operational diagnostics, such as:

- Request outcome.
- Analysis identifier.
- Processing duration.
- Parser warnings.
- API error category.

Observability must not log full uploaded files, credentials, or unnecessary sensitive network information.

Future production-oriented versions may introduce additional metrics, subject to privacy requirements.

## 31. Configuration

Configuration should be externalized where appropriate.

Examples:

```text
MAX_UPLOAD_SIZE
LOG_LEVEL
TEMP_DIRECTORY
```

Configuration values must have safe defaults and be validated at application startup.

Future AI configuration may include:

```text
AI_ENABLED
AI_PROVIDER
MODEL_NAME
```

These settings must not make AI a requirement for the MVP.

Sensitive configuration values must not be committed to source control or exposed through API responses.

## 32. Testing Architecture

Each major layer must be independently testable.

```text
Parser Unit Tests
        │
        ▼
Detection Unit Tests
        │
        ▼
Correlation Unit Tests
        │
        ▼
Recommendation Unit Tests
        │
        ▼
Data Model Tests
        │
        ▼
API Tests
        │
        ▼
End-to-End Tests
```

Testing must verify:

- Correct parsing and evidence line numbers.
- Deterministic rule matching.
- Severity and event schema consistency.
- Correlations referencing valid event IDs.
- Accurate summary counts.
- Safe recommendations.
- Controlled API errors.
- Upload limits and temporary-file cleanup.
- Correct response serialization.
- Successful operation without AI or a database.

## 33. Security Boundary

Uploaded files are untrusted.

```text
UNTRUSTED INPUT
       │
       ▼
Uploaded File
       │
       ▼
Validation
       │
       ▼
Safe Parsing
       │
       ▼
Structured Records
       │
       ▼
Deterministic Analysis
       │
       ▼
Validated Result
```

No uploaded content may be treated as executable instructions.

The frontend must safely render log-derived text, and any future AI integration must treat log content as untrusted evidence.

## 34. Architecture Principles

### Principle 1 — Evidence First

Every important detection must be traceable to source evidence.

### Principle 2 — Deterministic Core

Core event detection must not depend on AI availability.

### Principle 3 — AI as an Enhancement

AI may explain results in a future phase but cannot replace the detection engine.

### Principle 4 — Safe by Default

No automatic network changes or command execution.

### Principle 5 — Modular Design

Every major component has a clearly defined responsibility.

### Principle 6 — Testability

Core components can be tested independently.

### Principle 7 — Privacy

Uploaded network data must not leave the local environment by default.

### Principle 8 — Honest Uncertainty

The application must distinguish observed facts, potential relationships, and hypotheses.

### Principle 9 — Stateless MVP

The initial application returns results directly without requiring permanent storage or analysis history.

### Principle 10 — Consistent Contracts

The API, data models, and frontend must use the same field names and response structures.

## 35. Final Architecture Definition

NetAssist AI is a modular, local-first, evidence-driven network log analysis application.

Its initial processing pipeline is:

```text
RAW NETWORK LOG
       │
       ▼
FILE VALIDATION
       │
       ▼
LOG PARSER
       │
       ▼
DETERMINISTIC DETECTION
       │
       ▼
NORMALIZED EVENTS
       │
       ▼
CORRELATION ENGINE
       │
       ▼
RECOMMENDATION ENGINE
       │
       ▼
ANALYSIS RESULT
       │
       ▼
FASTAPI JSON RESPONSE
       │
       ▼
WEB DASHBOARD
```

The MVP uses Python, FastAPI, HTML, CSS, JavaScript, deterministic rules, and validated data models.

It does not require AI, a database, paid external services, or production device access.

The architecture leaves room for future AI explanations, RAG, persistent analysis history, report exports, and read-only device integrations without making those features dependencies of the initial system.

**Final principle:** Build a small, testable, stateless, evidence-based analyzer first. Extend it only after the core pipeline is reliable and its security and privacy requirements are verified.