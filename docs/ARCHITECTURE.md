# NetAssist AI — System Architecture

**Document:** System Architecture  
**Project:** NetAssist AI  
**Version:** 1.1
**Status:** Design Phase  
**Architecture Style:** Modular Layered Architecture with Dual Analysis Paths
**Backend:** Python + FastAPI  
**Frontend:** HTML + CSS + JavaScript  
**Core Principle:** Deterministic, Evidence-Based Analysis with Optional Local RAG

---

## 1. Architecture Overview

NetAssist AI is a local-first network log troubleshooting application. It analyzes uploaded network logs, identifies supported events, preserves source evidence, identifies potential relationships between events, and provides safe troubleshooting recommendations.

The architecture has two analysis paths:

- **Path A — Deterministic Rule-Based Analysis:** A reliable baseline that works without an LLM, external AI service, or internet connection.
- **Path B — RAG-Assisted Explanation:** An optional local Retrieval-Augmented Generation pipeline that retrieves relevant networking knowledge and uses a local language model to explain detected events.

Both paths share the same upload validation, parsing, normalized event data, and deterministic detection engine.

The RAG path supplements the deterministic results. It does not replace the detection engine or independently determine authoritative event severity.

The initial implementation will prioritize the rule-based path. RAG will be implemented after the core analyzer is tested.

### Architecture Principles

1. Detection is mandatory; AI is optional.
2. Evidence is required for important detections.
3. RAG explanations must be grounded in retrieved knowledge and available log evidence.
4. The application must remain useful when the local AI components are unavailable.
5. Uploaded logs and analysis results must not be permanently stored by default.
6. No automatic network-device commands or configuration changes are permitted.
7. Paid external AI APIs are not required.

## 2. High-Level Architecture

```text
                         USER
                           |
                           v
                 WEB DASHBOARD
                  HTML/CSS/JS
                           |
                       HTTP/REST
                           |
                           v
                       FASTAPI
                           |
                           v
                  UPLOAD VALIDATION
                           |
                           v
                      LOG PARSER
                           |
                           v
                NORMALIZED LOG RECORDS
                           |
                           v
              DETERMINISTIC DETECTION
                           |
                           v
                 NORMALIZED EVENTS
                           |
              +------------+------------+
              |                         |
              v                         v
       PATH A: RULE-BASED        PATH B: RAG-ASSISTED
              |                         |
       Event correlation        Evidence/context selection
              |                         |
       Recommendations          Knowledge retrieval
              |                         |
              |                   Local embeddings
              |                         |
              |                   Vector search
              |                         |
              |                   Retrieved passages
              |                         |
              |                   Local LLM explanation
              |                         |
              +------------+------------+
                           |
                           v
                  RESULT ASSEMBLY
                           |
                           v
                    ANALYSIS RESULT
                           |
                           v
                    JSON RESPONSE
                           |
                           v
                    WEB DASHBOARD
```

The rule-based path must not depend on RAG, a vector database, a local language model, or internet access.

The RAG path is optional and may be disabled through configuration.

## 3. Two Analysis Paths

### 3.1 Path A — Deterministic Rule-Based Analysis

This path is the mandatory baseline.

```text
Parsed Records
     |
     v
Rule Matching
     |
     v
Normalized Events
     |
     v
Event Correlation
     |
     v
Safe Recommendations
     |
     v
Analysis Result
```

Responsibilities:

- Detect supported network events using documented rules.
- Assign severity through deterministic rule definitions.
- Preserve source line numbers and supporting evidence.
- Identify potential relationships among detected events.
- Generate predefined, human-verifiable recommendations.
- Produce a structured analysis result.
- Continue working when AI is disabled or unavailable.

The same input and ruleset should produce the same core detections, subject to controlled configuration and implementation behavior.

### 3.2 Path B — RAG-Assisted Explanation

This path adds knowledge retrieval and optional local LLM explanations to the shared analysis pipeline.

```text
Validated Analysis Evidence
          |
          v
Relevant Context Selection
          |
          v
Query Construction
          |
          v
Embedding Generation
          |
          v
Local Vector Retrieval
          |
          v
Retrieved Knowledge Passages
          |
          v
Local LLM
          |
          v
Explanation Validation
          |
          v
Grounded Explanation
```

The RAG path may:

- Explain a detected network event in plain language.
- Retrieve relevant troubleshooting documentation.
- Explain possible causes supported by the available evidence.
- Suggest additional read-only checks.
- Cite retrieved passages or document sections.
- Clearly communicate missing information and uncertainty.

The RAG path must not:

- Invent authoritative detections.
- Change rule-generated event severity.
- Alter event IDs, rule IDs, or source evidence.
- Convert a possible correlation into a confirmed root cause without sufficient evidence.
- Execute commands or modify network devices.
- Treat uploaded log content as trusted instructions.

**Important:** RAG is not itself an API. FastAPI remains the application's local HTTP API. The local RAG pipeline can run without a paid cloud AI API when its required models and knowledge base are available locally.

## 4. RAG Availability and Fallback

The application must distinguish between the availability of the rule-based analyzer and the availability of the optional RAG components.

| Situation | Expected behavior |
|---|---|
| AI is disabled | Return rule-based results. |
| Local LLM is not installed | Return rule-based results. |
| Vector index is unavailable | Return rule-based results and a controlled warning if RAG was requested. |
| RAG retrieval fails | Preserve the deterministic results and report the optional component failure. |
| LLM generation fails | Return the deterministic results without inventing an AI explanation. |
| Internet is unavailable | Use the local knowledge base and local models if they are already available. |
| AI works successfully | Include a validated, evidence-grounded explanation alongside the deterministic results. |

The system must not silently present an AI explanation as successful when generation failed.

The API response must communicate whether AI was used and whether optional processing encountered a problem. The exact metadata and warning fields must be defined consistently in `docs/DATA_MODELS.md` and `docs/API_SPECIFICATION.md`.

## 5. Architectural Layers

The logical layers are:

1. Presentation.
2. API and request handling.
3. Upload validation.
4. Parsing.
5. Event normalization.
6. Deterministic detection.
7. Correlation.
8. Recommendation generation.
9. Optional RAG retrieval and explanation.
10. Result assembly and validation.
11. API response and frontend rendering.

Each layer should have a clearly defined responsibility and be independently testable wherever practical.

The parser and detection engine must not depend on FastAPI or frontend code. The RAG components must remain isolated from authoritative detection logic.

## 6. Presentation Layer

The frontend uses HTML, CSS, and JavaScript.

Responsibilities:

- Upload `.log` and `.txt` files.
- Display validation and analysis errors.
- Show analysis status and summary counts.
- Display events, severity, and source evidence.
- Display potential correlations and recommendations.
- Display optional AI explanations and their source references.
- Indicate when AI is disabled, unavailable, or not used.
- Search and filter displayed results.
- Render log-derived content safely.

The frontend must not implement detection rules or treat uploaded content as executable HTML or JavaScript.

The initial MVP does not require a separate report-export feature.

## 7. API Layer

The backend uses FastAPI.

### MVP endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Basic application health |
| GET | `/ready` | Application readiness |
| GET | `/api/v1/info` | Application and supported capability information |
| POST | `/api/v1/analyze` | Validate and analyze an uploaded log file |

The complete contracts are defined in `docs/API_SPECIFICATION.md`.

`POST /api/v1/analyze` returns an `AnalysisResult` directly. The result is not permanently stored by default.

The initial MVP does not include:

- An analysis-history endpoint.
- A saved-analysis retrieval endpoint.
- A separate saved-result export endpoint.
- An endpoint for executing commands.
- An endpoint for modifying network-device configuration.

The API must validate inputs, return controlled HTTP errors, and avoid exposing internal stack traces or sensitive server information.

## 8. File Validation and Temporary Processing

Uploaded files are untrusted input.

Validation must include:

- Approved extensions: `.log` and `.txt`.
- Maximum file size: initially 5 MiB.
- Empty-file rejection.
- Safe text decoding.
- Filename sanitization.
- Path traversal prevention.
- Resource-limit checks.

The application must not trust a filename, extension, or MIME type alone.

Prefer in-memory processing when practical. If temporary disk storage is necessary:

- Use a controlled temporary directory.
- Generate safe temporary filenames.
- Never use a supplied filename as a filesystem path.
- Attempt cleanup on success and failure.
- Avoid permanent copies and archives.
- Avoid logging complete uploaded files.

Uploaded logs and results must not be permanently stored by default. A database is not required for the MVP.

## 9. Log Parser

The parser converts raw log lines into structured records.

Illustrative input:

```text
2026-10-15 10:31:22 %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

Illustrative internal record:

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

This is an illustrative parser record, not the public API response schema.

Responsibilities:

- Process log lines safely.
- Preserve source line numbers.
- Extract timestamps and known identifiers where possible.
- Retain supporting source information.
- Handle malformed timestamps and unexpected characters.
- Continue after non-fatal parsing errors.
- Record relevant parsing limitations.
- Avoid executing or interpreting log content as code.

Unknown lines must not automatically cause the entire analysis to fail.

## 10. Deterministic Detection Engine

The detection engine applies documented rules to parsed records.

Initial supported rules may cover interface state changes, BGP neighbor events, device errors, authentication failures, and other explicitly implemented conditions.

Actual support must be defined in `docs/DETECTION_RULES.md`. Mentioning a category in this document does not mean its detection rule is already implemented.

Each rule must define:

- Stable rule ID.
- Event type and description.
- Match conditions.
- Required fields.
- Severity.
- Evidence requirements.
- Recommendation mapping.
- Tests and known limitations.

Illustrative identifiers:

```text
IFACE_DOWN_001
IFACE_UP_001
BGP_NEIGHBOR_DOWN_001
BGP_NEIGHBOR_UP_001
DEVICE_ERROR_001
AUTH_FAILURE_001
```

The rule engine is the source of truth for authoritative event detections and severity.

The RAG or LLM layer must not independently change these values.

## 11. Event Normalization

The normalization layer converts supported detections into the canonical `NetworkEvent` model defined in `docs/DATA_MODELS.md`.

The canonical model includes fields such as:

- `event_id`
- `rule_id`
- `event_type`
- `title`
- `severity`
- `timestamp`
- `device`
- `interface`
- `resource_type`
- `resource_id`
- `description`
- `evidence`
- `possible_causes`
- `recommendations`
- `confidence`
- `tags`

The actual required and optional fields must follow `DATA_MODELS.md`.

Normalization must not invent missing timestamps, resource identifiers, or source evidence. Event IDs must be unique within an analysis.

## 12. Correlation Engine

The correlation engine identifies potential relationships among detected events using available dimensions such as:

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

The engine may identify a potential relationship if supported by the available evidence.

A correlation must:

- Reference event IDs from the same analysis.
- Include supporting evidence.
- Use controlled confidence values.
- Communicate uncertainty honestly.

Temporal proximity does not establish causality. The initial MVP must not mark a cause as confirmed solely because two events occurred close together.

## 13. Recommendation Engine

The recommendation engine maps deterministic events to safe troubleshooting guidance.

For example, an interface CRC error may lead to recommendations to review interface counters, inspect the physical cable, check the transceiver, or compare remote interface counters.

Recommendations must be:

- Relevant to the detected event.
- Grounded in available evidence.
- Human-verifiable.
- Clear about uncertainty.
- Advisory rather than automatically executed.

The final recommendation schema must match `Recommendation` in `DATA_MODELS.md`.

RAG may retrieve supporting documentation for these recommendations, but retrieved text must not override the deterministic event or authorize a risky action.

## 14. RAG Knowledge Base

The RAG knowledge base provides reference material for troubleshooting explanations.

Potential sources include:

- Public networking documentation.
- Vendor-neutral networking references.
- Authorized troubleshooting guides.
- Relevant RFCs.
- Authorized internal runbooks.

Only appropriately licensed and authorized content may be included.

### Knowledge ingestion pipeline

```text
Approved Documents
       |
       v
Text Extraction
       |
       v
Cleaning and Metadata
       |
       v
Chunking
       |
       v
Local Embeddings
       |
       v
Local Vector Index
```

Each indexed passage should retain useful source metadata, such as document title, section, source location, and version where available.

The initial implementation should prefer a small, controlled knowledge collection over a large, unverified document corpus.

Knowledge-base ingestion and indexing should be separate from the log-analysis request path. The system should not rebuild the entire index for every uploaded log.

The initial MVP does not require external hosted vector databases or paid embedding APIs.

## 15. RAG Retrieval and Grounded Explanations

The RAG service receives a question or a structured explanation request derived from the current analysis.

It retrieves relevant knowledge passages and supplies them, together with selected event evidence, to the local LLM.

The explanation should distinguish:

- Observed facts from the uploaded logs.
- Relevant knowledge from retrieved documents.
- Possible causes or hypotheses.
- Recommended verification steps.
- Missing information and limitations.

The generated explanation must be validated before display.

The application should preserve source references for retrieved passages so a user can inspect the supporting material.

The LLM must be instructed to treat log contents and retrieved documents as untrusted data rather than executable instructions.

If retrieved evidence is insufficient, the system should say so rather than manufacture an answer.

Retrieval relevance, answer grounding, citation correctness, and latency must be evaluated separately.

## 16. Local AI and Configuration

The project prioritizes a local, no-paid-API architecture.

Candidate components may include:

- Python for orchestration.
- FastAPI for the application API.
- A local embedding model.
- FAISS or Chroma for vector retrieval.
- Ollama with a compatible local LLM for explanation generation.

These are candidate choices, not claims that the packages or models are already installed. The final selection must be confirmed in `docs/TECH_STACK.md` before implementation.

A local model and its dependencies may require an initial download and adequate RAM, storage, and CPU/GPU resources. Offline operation can only be expected after the required components and knowledge base are available locally.

Suggested configuration concepts:

```text
AI_ENABLED=false
AI_PROVIDER=local
MODEL_NAME=<configured-local-model>
MAX_UPLOAD_SIZE=<configured-limit>
LOG_LEVEL=<configured-level>
TEMP_DIRECTORY=<configured-directory>
```

The exact variable names and defaults must be defined consistently in the configuration and technical-stack documents.

AI should be disabled by default during initial rule-based development. Enabling AI must not be required for health checks or basic deterministic analysis.

Secrets and machine-specific configuration must not be committed to Git.

## 17. Shared Analysis Result

Both paths must preserve a consistent core analysis contract.

Conceptually:

```text
Uploaded Content
       |
       v
Validation
       |
       v
Parsing
       |
       v
Deterministic Detection
       |
       v
Normalized Events
       |
       v
+---------------------------+
| Rule-Based Path           |
| Correlation and Guidance  |
+---------------------------+
       |
       +-------------------------------+
                                       |
+--------------------------------------+
| Optional RAG Path                    |
| Retrieval and Local LLM Explanation  |
+--------------------------------------+
       |
       v
Result Assembly and Validation
       |
       v
AnalysisResult
       |
       v
JSON Response
```

The canonical result model is defined in `docs/DATA_MODELS.md`.

It includes the existing core fields:

- `analysis_id`
- `status`
- `filename`
- `created_at`
- `summary`
- `events`
- `correlations`
- `warnings`
- `metadata`

Any additional AI explanation, source citation, or AI-processing status fields must be added through an explicit update to the canonical data model and API specification before implementation. The architecture must not silently introduce a conflicting response schema.

The result is returned directly to the client. An analysis ID does not imply persistent storage or later retrieval.

Summary counts must match the returned events, including the INFO severity count defined in the canonical data model.

## 18. API and Frontend Integration

The browser submits an upload to `POST /api/v1/analyze`.

The backend performs deterministic analysis first. If the optional RAG path is enabled and available, it attempts to generate a grounded explanation without changing the deterministic detections.

The frontend displays:

- Analysis summary.
- Detected events and severity.
- Evidence and source line numbers.
- Potential correlations.
- Recommendations.
- Warnings and limitations.
- Optional AI explanations and their source references.
- Processing metadata and AI status where defined by the response contract.

The frontend must distinguish deterministic observations from generated explanations.

A failed optional AI request must not erase valid rule-based results.

## 19. Error Handling

Errors must be handled at the appropriate layer.

### Invalid upload

```text
Upload
  |
  v
Validation Failure
  |
  v
Controlled API Error
```

### Non-fatal parsing problem

```text
Malformed Line
  |
  v
Record Limitation
  |
  v
Continue Where Safe
  |
  v
Analysis Warning
```

### RAG or LLM failure

```text
Optional AI Failure
       |
       v
Preserve Deterministic Results
       |
       v
Return Controlled Warning
```

### Unexpected internal error

```text
Internal Error
     |
     v
Attempt Cleanup
     |
     v
Controlled API Error
```

Errors must not expose raw stack traces, secrets, or unnecessary filesystem details to the browser. Error responses must follow `API_SPECIFICATION.md`.

## 20. Privacy and Security Boundaries

Uploaded files are untrusted.

The MVP must:

- Enforce upload limits and validation.
- Avoid permanent storage by default.
- Prefer in-memory processing where practical.
- Attempt temporary-file cleanup on success and failure.
- Avoid logging full uploaded files.
- Render uploaded text safely.
- Treat log-derived text as untrusted evidence.
- Keep credentials and secrets out of source control.

The local-first design means that network logs should not be sent to external AI services by default.

Any future external AI integration would require a separate, explicit design decision, appropriate user awareness, and updated privacy documentation.

No uploaded content may be treated as executable code or trusted instructions.

## 21. No Production Device Access in the MVP

The application analyzes uploaded logs only.

It does not include:

- Production router or switch access.
- SSH connections to devices.
- Automatic command execution.
- Network configuration changes.
- Automatic remediation.

Any future read-only device integration must be separately designed with explicit authorization, read-only permissions, credential protection, connection timeouts, audit requirements, and security testing.

It must not be added implicitly through the RAG or recommendation engine.

## 22. Observability

The MVP should provide minimal operational diagnostics:

- Request outcome.
- Analysis identifier.
- Processing duration.
- Parser warnings.
- API error category.
- Optional AI processing outcome where available.

Diagnostics must not log complete uploaded files, credentials, or unnecessary sensitive network information.

Future metrics may measure retrieval latency, retrieval relevance, LLM generation latency, and explanation-validation failures without retaining sensitive log contents.

## 23. Testing Architecture

Each major component must be independently testable.

```text
Parser Tests
     |
     v
Detection Tests
     |
     v
Normalization Tests
     |
     v
Correlation Tests
     |
     v
Recommendation Tests
     |
     v
Data Model Tests
     |
     v
API Tests
     |
     v
RAG Retrieval Tests
     |
     v
AI Grounding and Citation Tests
     |
     v
End-to-End Tests
```

Testing must verify:

- Correct parsing and source line numbers.
- Deterministic rule matching.
- Correct severity and schema consistency.
- Correlations referencing valid event IDs.
- Accurate summary counts.
- Safe recommendations.
- Controlled API errors.
- Upload limits and temporary-file cleanup.
- Correct response serialization.
- Successful operation with AI disabled.
- RAG retrieval of relevant passages.
- Source references matching retrieved content.
- Clear handling of insufficient evidence.
- No AI override of deterministic events or severity.
- Fallback to rule-based results after optional AI failures.
- No permanent log storage by default.

Core tests must not require a downloaded LLM or internet access. RAG and local-model integration tests should be separable from the rule-based test suite.

## 24. Proposed Project Structure

The following is the proposed structure, not a claim that these files already exist.

```text
netassist-ai/
├── docs/
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
│   │   │   ├── recommender.py
│   │   │   └── result_builder.py
│   │   └── rules/
│   └── tests/
├── rag/
│   ├── ingestion/
│   ├── retrieval/
│   ├── generation/
│   └── evaluation/
├── knowledge_base/
│   └── README.md
├── frontend/
│   ├── index.html
│   ├── css/
│   └── js/
├── sample_logs/
├── tests/
├── README.md
├── pyproject.toml
└── .gitignore
```

The structure may be simplified during implementation without changing the architectural responsibilities.

Knowledge-base documents and generated vector indexes must be handled according to licensing, privacy, size, and Git tracking rules. Private or sensitive documents must not be committed publicly.

## 25. Dependency Direction

The intended dependency direction is:

```text
API Layer
    |
    v
Analyzer Service
    |
    +---- Parser
    |
    +---- Detection Engine
    |
    +---- Correlation Engine
    |
    +---- Recommendation Engine
    |
    +---- Optional RAG Service
    |
    +---- Result Builder
              |
              v
          Data Models
```

The parser must not depend on FastAPI. The detection engine must not depend on the frontend, RAG service, or LLM. The RAG service may consume structured analysis evidence but must not mutate authoritative detections.

The analyzer service orchestrates components, and shared data models define consistent contracts.

## 26. Implementation Order

Development will proceed in controlled phases:

1. Align the documentation and canonical data contracts.
2. Establish the Python environment and project structure.
3. Implement canonical Pydantic data models.
4. Implement upload validation.
5. Implement the log parser.
6. Implement deterministic detection rules.
7. Implement normalization, correlation, and recommendations.
8. Implement result assembly and summary validation.
9. Expose the analyzer through FastAPI.
10. Build the browser dashboard.
11. Implement knowledge ingestion and local vector retrieval.
12. Add optional local LLM explanations and output validation.
13. Integrate both paths with safe fallback behavior.
14. Test, document, and prepare the GitHub portfolio.

RAG implementation must not delay testing or break the deterministic baseline.

## 27. Final Architecture Definition

NetAssist AI is a modular, local-first network log troubleshooting application with a deterministic analysis core and an optional local RAG explanation layer.

Its shared pipeline is:

```text
RAW NETWORK LOG
       |
       v
FILE VALIDATION
       |
       v
LOG PARSER
       |
       v
DETERMINISTIC DETECTION
       |
       v
NORMALIZED EVENTS
       |
       +-----------------------------+
       |                             |
       v                             v
RULE-BASED ANALYSIS            OPTIONAL RAG
       |                       KNOWLEDGE RETRIEVAL
       |                             |
       |                       LOCAL LLM EXPLANATION
       |                             |
       +--------------+--------------+
                      |
                      v
             VALIDATED RESULT
                      |
                      v
              FASTAPI JSON API
                      |
                      v
               WEB DASHBOARD
```

The project prioritizes Python, FastAPI, deterministic rules, validated data models, local knowledge retrieval, and optional local LLM explanations.

It does not require paid external AI APIs, a database, permanent log storage, or production device access.

**Final principle:** Build a reliable evidence-based analyzer first, add RAG as a separately testable enhancement, and ensure that the system remains useful and safe when AI is unavailable.