# NetAssist AI — Technical Stack

**Document Status:** Planning  
**Version:** 1.1  
**Product:** NetAssist AI  
**Development Environment:** Windows  
**Budget:** Free / Open Source Only

---

# 1. Technology Strategy

NetAssist AI will use a simple, modular, Python-first technology stack.

The MVP will prioritize:

- Reliability and correctness
- Simplicity and maintainability
- Testability
- Free and open-source technologies
- Local execution and privacy
- Easy development in VS Code
- Minimal external dependencies
- Clear separation between detection, correlation, and recommendations
- Consistency between documentation, implementation, and tests

The project will begin as a modular monolith. It will not introduce distributed systems, external AI services, databases, or complex frontend frameworks without a justified requirement.

The deterministic analyzer must work independently of any AI component.

---

# 2. Core Technology Stack

| Area | Technology | Purpose |
|---|---|---|
| Programming Language | Python 3.12+ | Main application language |
| Backend Framework | FastAPI | REST API and application backend |
| ASGI Server | Uvicorn | Run the FastAPI application |
| Data Validation | Pydantic | Validate structured data and API models |
| Parsing | Python standard library and Regex | Parse supported network logs |
| Detection | Custom Python | Apply deterministic detection rules |
| Correlation | Custom Python | Identify potentially related events |
| Recommendations | Custom Python | Generate rule-based troubleshooting guidance |
| Frontend Structure | HTML | Dashboard structure |
| Frontend Styling | CSS | Dashboard presentation |
| Frontend Logic | JavaScript | Upload interactions and result rendering |
| Testing | Pytest | Automated testing |
| API Testing | HTTPX | Test FastAPI endpoints |
| Version Control | Git | Source control |
| IDE | VS Code | Development environment |
| Configuration | Environment variables | Runtime configuration |
| Documentation | Markdown | Project and technical documentation |
| Optional Local AI | Ollama | Future local LLM execution |
| Future Local Storage | SQLite | Optional analysis history |

The core MVP will not require Ollama, SQLite, a vector database, or paid APIs.

---

# 3. Programming Language

## Python

Python is the primary programming language for NetAssist AI.

Reasons for selecting Python:

- Strong text-processing capabilities.
- Built-in regular-expression support.
- Suitable date and time handling.
- A mature testing ecosystem.
- Integration with FastAPI and Pydantic.
- Straightforward implementation of deterministic rules.
- Compatibility with future AI and RAG components.

The parser, detection engine, correlation engine, and recommendation engine will remain Python-based.

The core analyzer must not depend on an LLM to identify events or assign severity.

---

# 4. Python Version and Environment

Target Python version:

```text
Python 3.12+
```

The installed Python version must be verified before implementation begins.

The project will use a local virtual environment:

```text
.venv/
```

All project dependencies must be installed inside this virtual environment.

The virtual environment must not be committed to Git.

The implementation should use a project-level `pyproject.toml` to declare dependencies and development configuration.

The exact dependency versions will be selected and locked during environment setup.

---

# 5. Backend Framework

## FastAPI

FastAPI will provide the application's HTTP API.

Responsibilities:

- Serve the dashboard.
- Accept supported log-file uploads.
- Validate incoming requests.
- Enforce upload restrictions.
- Invoke the analysis pipeline.
- Return structured analysis results.
- Return consistent error responses.
- Provide health and readiness endpoints.
- Expose application information.
- Generate API documentation during development.

FastAPI is suitable because it is Python-native, supports Pydantic models, and integrates with HTTPX-based tests.

FastAPI must not contain the main detection logic. Detection and correlation should remain in separate application modules.

---

# 6. ASGI Server

## Uvicorn

Uvicorn will run the FastAPI application during local development.

The expected development command is:

```bash
uvicorn backend.app.main:app --reload
```

This command assumes that the project package structure and FastAPI application object match the documented project layout.

The final command will be verified after the project skeleton has been created.

The development reload option is intended for local development, not as a production security configuration.

---

# 7. Data Validation

## Pydantic

Pydantic will define and validate structured data models used by the API and analyzer.

Core models include:

- `DetectedEvent`
- `Correlation`
- `Recommendation`
- `AnalysisSummary`
- `ProcessingMetadata`
- `AnalysisResult`

The models must remain consistent with:

```text
docs/DATA_MODELS.md
```

The API specification must remain consistent with:

```text
docs/API_SPECIFICATION.md
```

The primary analysis endpoint will return an `AnalysisResult` directly as a JSON response.

The initial MVP does not require database persistence or a separate analysis-retrieval endpoint.

Validation rules should reject invalid or inconsistent values rather than silently accepting them.

---

# 8. Log Parsing

The initial parser will use Python's standard library.

Expected tools include:

- String operations
- Regular expressions
- `datetime`
- `pathlib`
- `typing`
- Explicit parsing functions

No external parsing framework is required for the MVP.

The parser should:

- Process supported `.log` and `.txt` files.
- Preserve original line numbers.
- Retain relevant evidence for detected events.
- Handle blank lines and malformed records safely.
- Avoid treating an unrecognized line as a confirmed incident.
- Record parsing limitations where appropriate.
- Avoid unnecessary retention of raw log content.

Parser behavior must be deterministic and covered by automated tests.

---

# 9. Regular Expression Strategy

Regular-expression patterns will be organized within dedicated rule modules.

Expected structure:

```text
backend/app/rules/
    interface_rules.py
    bgp_rules.py
    error_rules.py
```

These filenames are an initial organizational proposal. The final structure should remain consistent with `docs/ARCHITECTURE.md` and the implementation plan.

Regex patterns should not be scattered throughout unrelated application files.

Each rule should have a clearly defined purpose and test cases.

Patterns should be specific enough to reduce false positives while allowing known variations in log formats.

A regex match alone must not imply that the entire incident has been understood.

---

# 10. Detection Engine

The detection engine will use custom Python logic.

No external AI system is required for deterministic detection.

Conceptual flow:

```text
Uploaded Log
     |
     v
File Validation
     |
     v
Parser
     |
     v
Parsed Records
     |
     v
Rule Registry
     |
     v
Matching Rules
     |
     v
Detected Events
```

The detection engine should support multiple rules matching the same log when the evidence supports multiple events.

The engine must:

- Apply registered detection rules.
- Produce structured events.
- Preserve evidence and source line numbers.
- Assign severity using predefined rule logic.
- Avoid inventing details not present in the evidence.
- Produce consistent results for the same input.

AI must not override deterministic detection results or severity assignments.

---

# 11. Rule Registry

The application will maintain a centralized registry of detection rules.

Conceptual example:

```python
RULES = [
    interface_down_rule,
    interface_up_rule,
    interface_flap_rule,
    bgp_neighbor_down_rule,
    crc_error_rule,
]
```

This example illustrates the intended design and is not a commitment to a particular implementation style.

The registry may use functions, classes, or another lightweight mechanism.

Each rule should define, as appropriate:

- A stable rule identifier.
- The event type.
- Matching conditions.
- Severity.
- Description.
- Required evidence.
- Recommended troubleshooting steps.
- Known limitations.

Rules should be independently testable and easy to review.

New rules should not require modifications throughout unrelated parts of the application.

---

# 12. Severity Model

Severity will use controlled values:

```text
INFO
LOW
MEDIUM
HIGH
CRITICAL
```

An enum or validated model should be used instead of unrestricted severity strings.

Conceptual example:

```python
from enum import Enum


class Severity(str, Enum):
    INFO = "INFO"
    LOW = "LOW"
    MEDIUM = "MEDIUM"
    HIGH = "HIGH"
    CRITICAL = "CRITICAL"
```

The final model must follow `docs/DATA_MODELS.md`.

Severity should be assigned through explicit, documented detection logic.

An LLM must not independently change the severity of an event in the MVP or in a future explanation-only AI mode.

Severity indicates the configured rule's assessment, not a guarantee of business impact.

---

# 13. Event Model

Each detected event should use a structured data model.

The conceptual event information includes:

```text
event_id
rule_id
event_type
severity
timestamp
resource_type
resource_id
line_number
evidence
description
recommendations
confidence
```

The exact field names, data types, optional fields, and validation rules must be defined in `docs/DATA_MODELS.md`.

The application must not maintain competing event schemas in different modules.

Event identifiers should be unique within the relevant analysis result.

Source evidence and line numbers should be retained so that users can verify why an event was reported.

A confidence value, when present, must follow a documented interpretation. It must not be presented as a statistically calibrated probability unless that calibration has actually been established.

---

# 14. Correlation Engine

The correlation engine will initially use custom Python logic.

Potential signals include:

- Timestamp relationships.
- Shared resources.
- Shared interfaces.
- Shared IP addresses.
- Related event types.
- Configurable time windows.

Conceptual example:

```text
Interface DOWN
       +
BGP Neighbor DOWN
       +
Close Timestamp Relationship
       |
       v
Possible Correlation
```

Correlation identifies potentially related events. It does not prove that one event caused another.

The correlation engine should:

- Use explicit correlation criteria.
- Handle missing timestamps safely.
- Avoid assuming that similar events necessarily share a cause.
- Preserve references to the events supporting each correlation.
- Produce consistent, testable results.

Correlation logic must not depend on an LLM in the initial MVP.

---

# 15. Recommendation Engine

The recommendation engine will initially use deterministic mappings from detected events to troubleshooting guidance.

Conceptual example:

```text
interface_down
       |
       v
Check interface state
Check physical connectivity
Check interface counters
```

Another example:

```text
crc_error
       |
       v
Check interface counters
Check cable condition
Check transceiver condition
Check the remote interface
```

Recommendations should be stored in structured rule definitions where practical.

Recommendations must:

- Be linked to relevant events or rules.
- Explain the intended troubleshooting step.
- Avoid claiming that a proposed cause is confirmed.
- Prefer read-only diagnostic checks.
- Avoid executing commands automatically.
- Avoid changing network device configurations.

The user remains responsible for deciding whether to perform a recommended action.

---

# 16. Frontend

The MVP frontend will use:

```text
HTML
CSS
JavaScript
```

A heavy frontend framework is not required initially.

The initial dashboard should support:

- Uploading `.log` and `.txt` files.
- Displaying the analysis summary.
- Viewing detected events and their evidence.
- Filtering and searching events.
- Viewing correlations.
- Viewing recommendations.
- Displaying validation errors.
- Displaying analysis status.
- Showing clear empty and error states.

The frontend will render the `AnalysisResult` returned by the API.

Saved-analysis history is outside the MVP scope.

Export functionality should only be implemented if explicitly included in the agreed MVP requirements. It must not require permanent server-side storage.

A framework such as React or Next.js may be considered later if the UI becomes sufficiently complex.

---

# 17. Frontend Communication

The frontend will communicate with FastAPI through HTTP endpoints.

The main analysis flow is:

```text
Browser Dashboard
       |
       | POST /api/v1/analyze
       v
FastAPI
       |
       v
Upload Validation
       |
       v
Parser
       |
       v
Detection Engine
       |
       v
Correlation Engine
       |
       v
Recommendation Engine
       |
       v
AnalysisResult
       |
       | JSON Response
       v
Browser Dashboard
```

The frontend must not directly access Python internals.

The frontend must treat the API response as structured data and render it safely.

Log content must not be inserted into the page as trusted HTML. User-provided text should be rendered using safe text-handling techniques.

---

# 18. API Format

The API will use JSON for structured responses.

File uploads will use multipart form data.

The initial MVP endpoints are:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Basic application health check |
| GET | `/ready` | Application readiness check |
| GET | `/api/v1/info` | Application and supported capability information |
| POST | `/api/v1/analyze` | Validate and analyze an uploaded log file |

The primary analysis endpoint is:

```text
POST /api/v1/analyze
```

It will accept a supported log file, validate the upload, process the log, and return an `AnalysisResult` directly as a JSON response.

The exact request, response, and error schemas must follow:

```text
docs/API_SPECIFICATION.md
docs/DATA_MODELS.md
```

Illustrative examples in this document must not override the agreed API specification.

The initial MVP will not provide a separate endpoint to retrieve previously saved analyses because permanent analysis storage is not required.

---

# 19. Testing Stack

## Pytest

Pytest will be the primary testing framework.

Tests should cover:

- File parsing.
- Detection rules.
- Severity assignment.
- Event validation.
- Correlation logic.
- Recommendation generation.
- Upload validation.
- File-size restrictions.
- Unsupported file types.
- Malformed and empty input.
- API responses.
- Error handling.
- Temporary-file cleanup.
- Privacy-related behavior.

Tests should include normal cases, edge cases, invalid input, and expected failure conditions.

## HTTPX

HTTPX will be used for FastAPI endpoint testing.

Endpoint tests should cover:

```text
GET /health
GET /ready
GET /api/v1/info
POST /api/v1/analyze
```

The endpoint paths and response schemas must match `docs/API_SPECIFICATION.md`.

Tests must not require access to live network devices, paid services, or external AI APIs.

---

# 20. Test Data

Synthetic test logs will be stored separately from application source code.

Proposed directory:

```text
sample_logs/
```

Potential test files include:

```text
interface_down.log
interface_flap.log
bgp_events.log
interface_errors.log
arp_events.log
vlan_events.log
mac_events.log
mixed_incident.log
malformed.log
```

These are proposed fixtures. Each file should be added only when its relevant test cases are defined.

Sample logs should be synthetic or appropriately sanitized.

Tests must not depend on live network devices or production systems.

Real network logs must not be committed because they may contain sensitive operational information such as IP addresses, hostnames, usernames, device details, or credentials.

---

# 21. Version Control

## Git

Git will be used for source control.

The repository should contain:

- Application source code.
- Automated tests.
- Documentation.
- Synthetic sample logs.
- Configuration templates.
- Dependency configuration.
- README.
- `.gitignore`.

The repository must not contain:

```text
.venv/
.env
Secrets
Passwords
Private Credentials
Temporary Uploads
Generated Caches
Real Sensitive Network Logs
```

Only necessary project artifacts should be committed.

---

# 22. Git Ignore Configuration

The project should include a `.gitignore`.

Initial entries may include:

```gitignore
# Python virtual environment
.venv/

# Python cache
__pycache__/
*.py[cod]

# Environment secrets
.env
.env.*

# Allow the safe example configuration
!.env.example

# Test artifacts
.pytest_cache/
.coverage
htmlcov/

# Local editor settings
.vscode/

# Temporary and generated files
tmp/
temp/
uploads/
*.tmp

# Local databases
*.db
*.sqlite
*.sqlite3

# Operating system files
.DS_Store
Thumbs.db
```

The final `.gitignore` must be reviewed when the actual project structure is created.

Do not add a broad `*.log` rule without checking its impact on intentional synthetic fixtures. If log files are ignored, the final configuration must explicitly allow the approved synthetic sample logs.

Secrets must never be committed, even if a local ignore rule is accidentally missing.

---

# 23. Configuration

Runtime configuration will use environment variables.

Initial settings may include:

```text
APP_ENV=development
LOG_LEVEL=INFO
MAX_UPLOAD_SIZE_MB=5
AI_ENABLED=false
```

The actual values and defaults must be defined consistently in the implementation and project documentation.

A template file will be provided:

```text
.env.example
```

The actual `.env` file must not be committed.

Configuration must be validated at application startup where appropriate.

The upload-size limit must be enforced by the backend, not only by the browser.

`AI_ENABLED=false` will remain the default for the initial MVP.

---

# 24. Logging

The backend will initially use Python's built-in logging facilities.

The application may log:

- Application startup and shutdown.
- API request failures.
- Analysis duration.
- File validation failures.
- Parsing errors.
- Unexpected exceptions.
- Operational status information.

The application should avoid logging raw network logs or unnecessary sensitive data.

Logs should not contain passwords, authentication tokens, or complete uploaded file contents.

Unexpected exceptions should be recorded in an appropriate internal log while the client receives a safe, structured error response.

---

# 25. Error Handling

The application will use consistent error responses.

Illustrative example:

```json
{
  "error": {
    "code": "UNSUPPORTED_FILE_TYPE",
    "message": "Only .log and .txt files are supported."
  }
}
```

This example illustrates the error-response concept. The final structure and error codes must be defined in `docs/API_SPECIFICATION.md`.

Potential error categories include:

- Unsupported file type.
- File exceeds the size limit.
- Empty or invalid upload.
- Invalid request.
- Parsing failure.
- Unexpected internal error.

The backend must not return internal stack traces or sensitive implementation details to the user.

Error handling should not expose uploaded file contents.

Temporary-file cleanup must be attempted when processing succeeds and when processing fails.

---

# 26. Storage Strategy

The MVP does not require a database.

Initial processing flow:

```text
Upload
   |
   v
Validate
   |
   v
Process in Memory or Temporary Storage
   |
   v
Build AnalysisResult
   |
   v
Return JSON Response
   |
   v
Clean Up Temporary Data
```

The application should process data in memory whenever practical.

Temporary storage may be used when necessary, but temporary files must be removed after processing. Cleanup should also be attempted on error paths.

The application must not permanently save uploaded logs or analysis results by default.

`AnalysisResult` is the response model returned by the API. Its existence does not imply permanent storage or future retrieval.

The initial MVP will not require:

- A database.
- User accounts.
- Saved-analysis history.
- A retrieval endpoint.
- Cloud storage.
- A database migration framework.

---

# 27. Future Database

SQLite may be considered in a future phase.

Potential use cases include:

- Analysis history.
- Saved reports.
- Rule-version records.
- User preferences.
- Incident history.

SQLite must not be implemented in the MVP unless an explicit requirement changes the agreed scope.

Any future persistence feature must define its retention policy, privacy implications, data model, and deletion behavior before implementation.

---

# 28. Optional AI Stack

AI is not required for the MVP.

The deterministic analyzer must work fully without an LLM.

If local AI is introduced later, Ollama is a candidate for local model execution.

Potential benefits include:

- Local inference.
- No mandatory paid API.
- Reduced need to send logs to an external AI service.
- Experimentation with explanation generation.

Actual performance will depend on the selected model and available hardware.

Local execution does not automatically guarantee privacy or security. The application must still control what data is sent to the model, how prompts are constructed, and whether logs or outputs are retained.

The default must remain:

```text
AI_ENABLED=false
```

AI integration will be considered only after deterministic analysis and its tests are reliable.

---

# 29. AI Model Strategy

No specific model will be hard-coded into the initial architecture.

If local AI is added later, model settings should be configurable.

Illustrative configuration:

```text
AI_PROVIDER=ollama
AI_MODEL=<configured-model>
```

The selected model should be evaluated against:

- Available RAM.
- CPU/GPU capability.
- Response latency.
- Explanation quality.
- Context-window requirements.
- Local resource consumption.
- Reliability with malformed or incomplete evidence.

AI-generated explanations must be clearly separated from deterministic findings.

The AI component may explain structured evidence, summarize events, or clarify documented troubleshooting steps.

It must not:

- Create unsupported detections.
- Change rule-assigned severity.
- Invent evidence.
- Treat correlation as proof of causality.
- Execute network commands.
- Modify network device configurations.

---

# 30. RAG Technology

Retrieval-Augmented Generation (RAG) is a future capability.

The MVP does not require:

- A vector database.
- An embedding model.
- A document-ingestion pipeline.
- A retrieval framework.
- An external knowledge service.

If a concrete RAG requirement emerges later, a suitable open-source stack can be evaluated.

A potential future flow is:

```text
Approved Documentation
        |
        v
Document Processing
        |
        v
Chunking
        |
        v
Embeddings
        |
        v
Local Vector Store
        |
        v
Relevant Evidence Retrieval
        |
        v
Optional LLM Explanation
```

Technology selection must be based on actual requirements, available hardware, evaluation results, and maintenance costs.

RAG must not be introduced solely to make the MVP appear more advanced.

---

# 31. Containerization

Docker is not required for the first local MVP.

The initial development environment is Windows with Python and VS Code.

The initial execution model is:

```text
Windows
   |
   v
Python Virtual Environment
   |
   v
Uvicorn
   |
   v
FastAPI Application
```

Docker may be considered later if reproducible deployment or a target environment requires it.

The project must remain straightforward to run without containers.

---

# 32. Deployment Strategy

Initial deployment target:

```text
Local Windows Machine
```

The first version is intended for local development and single-user use.

Potential future deployment options include:

- Linux server.
- Internal company server.
- Docker-based deployment.
- Cloud virtual machine.

Cloud deployment is not required for the MVP.

A future deployment must review authentication, network exposure, upload handling, access control, data retention, and operational monitoring before the application is exposed beyond the local machine.

The local MVP must not be assumed to be production-ready for multi-user or public network access.

---

# 33. Free-Only Technology Policy

The project will prioritize free and open-source technologies.

Expected components include:

- Python.
- FastAPI.
- Uvicorn.
- Pydantic.
- Pytest.
- HTTPX.
- HTML.
- CSS.
- JavaScript.
- Git.
- VS Code.
- SQLite if justified in a future phase.
- Ollama if appropriate for future local AI.
- Other free/open-source libraries only when justified.

The core analyzer must not require a paid API, subscription, cloud account, or external AI provider.

Before adding any dependency, verify its licensing, maintenance status, compatibility, and actual necessity.

---

# 34. Technologies Explicitly Deferred

The following technologies and capabilities are outside the initial MVP unless a documented requirement justifies a change:

```text
React
Next.js
Kubernetes
Kafka
Redis
PostgreSQL
Elasticsearch
Cloud-Managed Databases
Paid LLM APIs
Paid Observability Platforms
Microservices
Message Queues
Complex Agent Frameworks
Vector Databases
RAG Pipelines
Automatic Network Remediation
Production Device Integrations
User Authentication and Multi-Tenant Support
Persistent Analysis History
```

Deferring these technologies reduces implementation complexity and keeps the initial product focused on reliable log analysis.

A future scope change must be documented rather than introduced silently.

---

# 35. Modular Monolith Strategy

The MVP will use a modular monolith.

Conceptual architecture:

```text
                  NetAssist AI
                       |
              FastAPI Application
                       |
              Analysis Pipeline
                       |
          +------------+------------+
          |            |            |
        Parser      Detector    Correlator
                       |            |
                       +-----+------+
                             |
                   Recommendations
                             |
                     AnalysisResult
                             |
                      JSON Response
                             |
                      Browser UI
```

The exact module boundaries must follow `docs/ARCHITECTURE.md`.

A modular monolith provides separation of responsibilities without the operational overhead of distributed services.

Each core component should be testable independently.

---

# 36. Why Not Microservices?

Microservices are unnecessary for the MVP because:

- The application is initially single-user.
- Processing is local.
- The workload is expected to be small.
- Distributed scaling is not a current requirement.
- Independent network services would increase setup complexity.
- A modular monolith is easier to test and debug.
- Local deployment is simpler.

If future requirements demand independent scaling or deployment, component boundaries can be reviewed.

No migration to microservices is planned unless a concrete need emerges.

---

# 37. Dependency Management

Python dependencies will be managed through the project's packaging configuration.

The preferred configuration file is:

```text
pyproject.toml
```

Dependencies should be explicitly declared.

Development and test dependencies should be clearly identified.

The project should avoid installing application packages globally.

All project dependencies should be installed inside `.venv/`.

Dependency versions should be selected and locked during environment setup.

The dependency configuration must be sufficient for another developer to reproduce the documented local setup.

---

# 38. Initial Dependency Set

The expected initial dependency set is:

```text
fastapi
uvicorn
python-multipart
pydantic
pytest
httpx
```

These packages serve the following purposes:

| Package | Purpose |
|---|---|
| `fastapi` | API framework |
| `uvicorn` | ASGI development server |
| `python-multipart` | Multipart file-upload handling |
| `pydantic` | Data validation |
| `pytest` | Automated tests |
| `httpx` | API endpoint tests |

The exact versions will be determined during setup.

Additional packages should be introduced only when an implementation requirement justifies them.

Avoid adding duplicate or unnecessary dependencies.

---

# 39. Standard Library Preference

Before introducing a third-party package, evaluate whether Python's standard library is sufficient.

Useful modules include:

```text
re
json
pathlib
datetime
logging
enum
typing
uuid
tempfile
os
time
```

These modules may support:

- Text parsing.
- JSON serialization.
- File and path handling.
- Timestamp processing.
- Logging.
- Controlled values.
- Type annotations.
- Identifier generation.
- Temporary-file management.
- Environment configuration.
- Execution-time measurement.

The standard library should be preferred when it meets the requirement safely and clearly.

---

# 40. Architecture-to-Technology Mapping

| Component | Technology |
|---|---|
| Browser UI | HTML, CSS, JavaScript |
| API | FastAPI |
| ASGI Server | Uvicorn |
| File Upload | FastAPI and `python-multipart` |
| Parsing | Python standard library |
| Regex | Python `re` |
| Data Models | Pydantic |
| Detection | Custom Python |
| Correlation | Custom Python |
| Recommendations | Custom Python |
| Configuration | Environment variables |
| Logging | Python logging |
| Testing | Pytest |
| API Testing | HTTPX |
| Version Control | Git |
| Documentation | Markdown |
| Optional Local LLM | Ollama |
| Future Local Database | SQLite |

This mapping describes the intended technology choices for the MVP and possible future phases. It does not imply that every listed component must be installed immediately.

---

# 41. Development Workflow

Development should follow the documented implementation plan.

Expected sequence:

```text
Documentation Review
        |
        v
Environment Verification
        |
        v
Virtual Environment Setup
        |
        v
Project Skeleton
        |
        v
Data Models
        |
        v
Parser
        |
        v
Detection Rules
        |
        v
Correlation
        |
        v
Recommendations
        |
        v
API Integration
        |
        v
Frontend
        |
        v
Automated Testing
        |
        v
Security and Reliability Review
        |
        v
Optional AI Evaluation
```

The exact task order must remain consistent with `docs/IMPLEMENTATION_PLAN.md`.

Core data contracts should be established before dependent modules are implemented.

The frontend should consume the agreed API rather than depend on internal Python objects.

AI and RAG must remain deferred until the deterministic analyzer is reliable.

---

# 42. Development Quality Rules

Every implementation should follow these rules:

1. Make small, reviewable changes.
2. Implement one feature at a time.
3. Run relevant tests after each meaningful change.
4. Keep functions and modules focused.
5. Avoid unnecessary abstractions.
6. Preserve source evidence and line numbers.
7. Do not hide uncertainty.
8. Do not add dependencies without a reason.
9. Keep documentation synchronized with implementation.
10. Never commit secrets or sensitive real logs.
11. Validate uploaded files on the backend.
12. Enforce upload limits on the server.
13. Clean up temporary files on success and failure paths.
14. Return structured errors without exposing internal stack traces.
15. Keep detection and severity deterministic.
16. Treat correlations as possible relationships, not confirmed causes.
17. Do not execute network commands automatically.
18. Do not permanently retain uploaded logs by default.
19. Keep AI disabled until explicitly approved and evaluated.
20. Document any scope change before implementing it.

---

# 43. Definition of Technology Completion

The MVP technology setup will be considered complete when:

- The installed Python version has been confirmed.
- The virtual environment can be created and activated.
- Required dependencies are declared.
- The FastAPI application starts successfully.
- Uvicorn runs the application successfully.
- Pytest runs successfully.
- HTTPX can test the API.
- Git is initialized and configured.
- `.gitignore` is reviewed.
- The documented project structure exists.
- The required health endpoint works.
- The readiness endpoint works.
- The application information endpoint works.
- The analysis endpoint accepts valid supported files.
- Invalid uploads return appropriate errors.
- The response follows the `AnalysisResult` model.
- Temporary-file cleanup is tested where applicable.
- The analyzer works without an LLM.
- No database is required for the MVP.
- The relevant documentation matches the implementation.

Completion must be established through actual tests rather than assumed from configuration alone.

---

# 44. Final Technology Decision

The NetAssist AI MVP will use:

```text
Python 3.12+
      +
FastAPI
      +
Uvicorn
      +
Pydantic
      +
Python Standard Library and Regex
      +
Deterministic Detection Rules
      +
Custom Correlation Logic
      +
Rule-Based Recommendations
      +
HTML / CSS / JavaScript
      +
Pytest
      +
HTTPX
      +
Git
```

Optional future local AI:

```text
Ollama
```

Optional future storage:

```text
SQLite
```

The project will remain a **modular monolith** during the MVP.

The initial application will analyze supported network logs locally, return a structured `AnalysisResult`, and avoid permanent storage by default.

The system will prioritize reliable deterministic analysis, evidence preservation, safe file handling, and testable behavior before adding AI, RAG, databases, cloud infrastructure, or complex frontend frameworks.

All implementation decisions must remain consistent with:

- `docs/PROJECT_MASTER_PLAN.md`
- `docs/PRD.md`
- `docs/ARCHITECTURE.md`
- `docs/API_SPECIFICATION.md`
- `docs/DATA_MODELS.md`
- `docs/SECURITY_AND_PRIVACY.md`
- `docs/DETECTION_RULES.md`
- `docs/IMPLEMENTATION_PLAN.md`

If a conflict is identified, the relevant documents must be reconciled before implementation continues.