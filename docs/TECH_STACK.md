# NetAssist AI — Technical Stack

**Version:** 1.2
**Status:** Planning / Documentation Baseline
**Product:** NetAssist AI  
**Development Environment:** Windows  
**Budget:** Free and Open-Source Tools Where Practical

---

## 1. Technology Strategy

NetAssist AI will use a Python-first, modular monolith architecture.

The MVP will prioritize:

- Reliable, deterministic network log analysis.
- Local execution and privacy.
- Free and open-source technologies.
- Simple development in VS Code.
- Small, independently testable modules.
- Consistency between documentation, code, and tests.

The deterministic analyzer must work without an LLM, external AI API, database, or cloud service.

Optional local AI and RAG will be evaluated only after the core analyzer is implemented and tested.

## 2. Core Technology Stack

| Area | Technology | Purpose |
|---|---|---|
| Language | Python 3.12+ | Application and analysis logic |
| Backend | FastAPI | HTTP API and request handling |
| ASGI server | Uvicorn | Run the FastAPI application |
| Data validation | Pydantic | Validate canonical data models |
| Parsing | Python standard library and regex | Parse supported log files |
| Detection | Custom Python | Apply deterministic rules |
| Correlation | Custom Python | Identify possible event relationships |
| Recommendations | Custom Python | Generate rule-based guidance |
| Frontend | HTML, CSS, JavaScript | Browser interface |
| Testing | Pytest | Automated tests |
| API testing | HTTPX | Test FastAPI endpoints |
| Version control | Git | Source control |
| Development environment | VS Code and Python virtual environment | Local development |
| Configuration | Environment variables | Runtime settings |
| Documentation | Markdown | Project specifications |
| Optional future AI | Ollama and a compatible local model | Local LLM inference |
| Optional future storage | SQLite | Local history, if approved |

The MVP does not require Ollama, SQLite, a vector database, or paid APIs.

## 3. Python and Environment

The target Python version is **Python 3.12 or later**.

The installed version must be verified before implementation.

The project will use a local virtual environment:

```text
.venv/
```

Project dependencies will be declared in `pyproject.toml` and installed inside the virtual environment.

The virtual environment, secrets, caches, and temporary files must not be committed to Git.

Dependency versions will be selected during environment setup. The project should document how to reproduce the environment.

## 4. Backend Framework and Server

### FastAPI

FastAPI will handle:

- HTTP requests and responses.
- Supported file uploads.
- Request validation.
- Invocation of the analysis pipeline.
- Structured JSON responses.
- Consistent error responses.
- Health, readiness, and application-information endpoints.

Detection, parsing, correlation, and recommendation logic must remain in separate modules rather than being embedded in API route handlers.

### Uvicorn

Uvicorn will run the FastAPI application locally.

The planned development command is:

```bash
uvicorn backend.app.main:app --reload
```

This command assumes the corresponding package structure and application object exist. It must be verified after the project skeleton is created.

The reload option is for development, not a production deployment configuration.

## 5. Data Models

Pydantic will validate structured data used by the analysis pipeline and API.

The canonical model definitions are maintained in:

```text
docs/DATA_MODELS.md
```

The API contract is maintained in:

```text
docs/API_SPECIFICATION.md
```

The primary models include:

- `Evidence`
- `Recommendation`
- `NetworkEvent`
- `Correlation`
- `AnalysisSummary`
- `AnalysisWarning`
- `ProcessingMetadata`
- `AnalysisResult`

The exact field names, types, enums, optional values, and validation rules must follow `DATA_MODELS.md`.

The application must not introduce competing schemas such as `DetectedEvent` unless the canonical data contract is formally updated.

The analysis endpoint will return an `AnalysisResult` directly as JSON. Returning a result does not imply permanent storage.

## 6. Log Parsing

The initial parser will use Python's standard library wherever practical.

Potential modules include:

```text
re
datetime
pathlib
typing
tempfile
```

The parser shall:

- Accept supported `.log` and `.txt` files.
- Preserve original source line numbers.
- Retain original evidence for detected events.
- Extract timestamps when reliably identifiable.
- Handle blank lines and malformed records safely.
- Avoid inventing missing metadata.
- Report relevant limitations through supported warnings.
- Avoid unnecessary retention of raw log content.

Parsing behavior must be deterministic and covered by automated tests.

No external parsing framework is required for the initial MVP.

## 7. Detection Engine and Rule Registry

The detection engine will use custom Python logic and a centralized rule registry.

The initial rule identifiers are defined in `docs/DETECTION_RULES.md`:

```text
IFACE_DOWN_001
IFACE_UP_001
BGP_NEIGHBOR_DOWN_001
BGP_NEIGHBOR_UP_001
DEVICE_ERROR_001
AUTH_FAILURE_001
```

Each implemented rule must define its matching conditions, event type, severity, evidence requirements, extracted fields, and applicable recommendations.

Rules should be independently testable.

The registry must contain only implemented rules during the corresponding implementation phase. Additional rules for link flaps, CRC errors, VLANs, ARP, or MAC learning must not be treated as MVP capabilities until their requirements and tests are approved.

A documented rule is not an implemented feature until working code and tests exist.

## 8. Severity Model

The canonical severity values are:

```text
INFO
LOW
MEDIUM
HIGH
CRITICAL
```

The application will use validated enum values consistent with `docs/DATA_MODELS.md`.

Severity must be assigned through explicit, deterministic rule logic.

The presence of words such as "error" or "failure" must not independently establish a high or critical severity.

Severity represents the configured rule's assessment and is not a guarantee of business impact.

An optional AI component must not silently modify deterministic severity assignments.

## 9. Event Evidence

Every detected `NetworkEvent` must preserve supporting evidence consistent with the canonical model.

Evidence may include:

- Original source text.
- One-based source line number.
- Timestamp when available.
- Source filename when applicable.

Events must reference the rule that produced them.

The application must not invent resource identifiers, timestamps, or facts absent from the source evidence.

Event identifiers must be unique within the relevant analysis result.

## 10. Correlation Engine

The correlation engine will initially use custom Python logic.

Potential signals include timestamps, shared resources, related event types, and explicitly configured time windows.

The engine must:

- Apply documented correlation criteria.
- Reference existing event IDs.
- Preserve supporting evidence.
- Handle missing timestamps safely.
- Avoid unsupported assumptions about shared causes.
- Produce deterministic, testable results.

A close timestamp relationship does not prove causality. Correlations must follow the canonical `Correlation` model, including `is_confirmed_cause` being false by default.

Topology-aware correlation is outside the initial MVP.

## 11. Recommendation Engine

The recommendation engine will map supported event types or rules to structured troubleshooting recommendations.

Examples include checking interface state, reviewing counters, inspecting relevant logs, and verifying BGP neighbor state.

Recommendations must follow the canonical `Recommendation` model, including its action type, risk level, and authorization requirement.

The application must not automatically execute recommended commands or modify network configurations.

Recommendations are advisory; the user decides whether to perform the suggested checks.

## 12. Frontend

The MVP will use:

```text
HTML
CSS
JavaScript
```

A heavy frontend framework is not required initially.

The browser interface will support:

- Selecting and uploading supported log files.
- Displaying validation and analysis errors.
- Showing the analysis summary.
- Displaying detected events and their evidence.
- Reviewing possible correlations.
- Viewing troubleshooting recommendations.
- Providing clear loading, empty, and error states.

Advanced search, multiple filters, report downloads, and other enhancements will be implemented only according to the approved implementation plan.

The frontend must render user-supplied log text safely and must not treat it as trusted HTML.

## 13. API Contract

The initial API endpoints are:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Basic health check |
| GET | `/ready` | Readiness check |
| GET | `/api/v1/info` | Application information |
| POST | `/api/v1/analyze` | Upload and analyze a log file |

The analysis endpoint will accept multipart form data and return a canonical `AnalysisResult` JSON response.

The exact request, response, and error contracts are defined in:

```text
docs/API_SPECIFICATION.md
docs/DATA_MODELS.md
```

The initial upload limit is 5 MiB. The backend must enforce the limit independently of browser-side validation.

The MVP does not require a database, analysis-history endpoint, or report-download endpoint.

## 14. Configuration

Runtime settings will use environment variables.

Initial proposed settings include:

```text
APP_ENV=development
LOG_LEVEL=INFO
MAX_UPLOAD_SIZE_MB=5
AI_ENABLED=false
```

The actual settings, defaults, validation, and behavior must be implemented consistently with the relevant documentation.

An `.env.example` file may contain safe configuration examples. The actual `.env` file must never be committed.

The default AI setting will remain disabled for the initial MVP.

## 15. Testing Stack

### Pytest

Pytest will be the primary testing framework.

Tests will cover:

- File validation.
- Parsing and line-number preservation.
- Timestamp extraction.
- Each implemented detection rule.
- Severity assignment.
- Model validation.
- Correlation logic.
- Recommendation generation.
- Summary consistency.
- Error handling.
- Temporary-file cleanup.
- Privacy and safety requirements.

### HTTPX

HTTPX will be used for FastAPI endpoint tests.

Tests will cover all four documented endpoints and verify response schemas, upload validation, and error behavior.

Tests must not require production devices, paid services, or external AI APIs.

## 16. Test Data

Synthetic test logs will be maintained separately from application source code, for example under:

```text
sample_logs/
```

Test fixtures should be introduced as their corresponding rules are implemented.

Potential fixture categories include interface state changes, BGP events, authentication failures, malformed input, and mixed events.

Real operational logs must not be committed unless appropriately sanitized and explicitly approved. Synthetic fixtures are preferred because network logs may contain sensitive addresses, hostnames, credentials, and infrastructure details.

## 17. Dependencies

The expected initial dependency set is:

```text
fastapi
uvicorn
python-multipart
pydantic
pytest
httpx
```

These packages support the API, ASGI server, multipart uploads, validation, and automated testing.

Exact versions will be selected during environment setup and declared in `pyproject.toml`.

Before adding another dependency, verify that it is necessary, maintained, compatible, and appropriately licensed.

The core analyzer should prefer Python's standard library where it meets the requirement safely.

## 18. Logging and Error Handling

The backend will use Python's built-in logging facilities initially.

Operational logs may record startup, shutdown, analysis duration, validation failures, and unexpected exceptions.

Raw uploaded logs, passwords, tokens, and unnecessary sensitive information must not be written to application logs.

The API shall return structured, safe error responses consistent with `API_SPECIFICATION.md`.

Internal stack traces and sensitive implementation details must not be exposed to clients.

Temporary-file cleanup must be attempted on both successful and failed processing paths.

## 19. Storage and Privacy

The MVP does not require a database.

The application should process uploaded data in memory where practical. If temporary files are necessary, they must be cleaned up after processing, including error paths.

Uploaded logs and analysis results must not be permanently stored by default.

The existence of an `AnalysisResult` response model does not imply persistence or historical retrieval.

The MVP does not require user accounts, saved-analysis history, cloud storage, or database migrations.

## 20. Git and Repository Hygiene

The repository should contain source code, tests, documentation, dependency configuration, and approved synthetic fixtures.

It must exclude:

```text
.venv/
.env
Passwords and credentials
Temporary uploads
Generated caches
Sensitive real network logs
Local database files
```

The `.gitignore` must be reviewed against the actual project structure. If log files are ignored, approved synthetic fixtures must remain trackable.

Secrets must never be committed, even if an ignore rule is missing.

## 21. Optional Local AI

Local AI is a future enhancement, not an MVP dependency.

Ollama is a candidate for running a compatible local LLM. Model choice will depend on hardware, memory, latency, quality, and licensing.

Before implementation, verify the selected model's requirements and whether the local machine can run it acceptably.

The AI component may explain structured findings, summarize evidence, or provide grounded troubleshooting guidance.

It must not:

- Invent detections or evidence.
- Change rule-assigned severity.
- Convert correlation into confirmed causality.
- Execute network commands.
- Modify network device configurations.

Local inference may require an initial model download and sufficient disk space and memory. Once required models and dependencies are installed, the deterministic analyzer must continue working without internet access.

## 22. Optional RAG

RAG will be evaluated only after the deterministic analyzer is reliable.

A potential future flow is:

```text
Approved Local Documents
        |
        v
Document Processing
        |
        v
Chunking and Embeddings
        |
        v
Local Retrieval
        |
        v
Relevant Evidence
        |
        v
Optional LLM Explanation
```

A local embedding model and vector store may be evaluated when actual requirements are known.

FAISS or Chroma may be considered, but neither is an MVP requirement. Their inclusion must follow a documented technology decision and tests.

RAG must not be added solely to increase architectural complexity.

## 23. Deferred Technologies

The following are outside the initial MVP unless the approved scope changes:

- React and Next.js.
- Docker and Kubernetes.
- Microservices and message queues.
- Redis, PostgreSQL, and Elasticsearch.
- Paid LLM APIs.
- Cloud-managed databases.
- Vector databases and RAG pipelines.
- Automatic network remediation.
- Production device integrations.
- Multi-user authentication.
- Persistent analysis history.

Deferred technologies must not be installed or implemented without a concrete requirement.

## 24. Modular Monolith

The application will remain a modular monolith for the MVP.

Its conceptual components are:

```text
Browser UI
    |
    v
FastAPI Routes
    |
    v
Upload Validation
    |
    v
Analysis Orchestration
    |
    +--> Parser
    |
    +--> Detection Engine
    |
    +--> Correlation Engine
    |
    +--> Recommendation Engine
    |
    v
AnalysisResult
    |
    v
JSON Response
```

The actual module boundaries must follow `docs/ARCHITECTURE.md`.

Each core component should be testable independently. Distributed services are unnecessary for the initial local, single-user workload.

## 25. Development Sequence

Implementation shall follow `docs/IMPLEMENTATION_PLAN.md`.

The expected high-level sequence is:

1. Verify the development environment.
2. Create the project skeleton and dependency configuration.
3. Implement canonical data models.
4. Implement upload validation.
5. Implement the parser.
6. Implement the initial detection rules.
7. Implement correlation and summary generation.
8. Implement analysis orchestration.
9. Add the FastAPI endpoints.
10. Build the browser interface.
11. Run integration, negative, and safety tests.
12. Evaluate optional AI only after the core analyzer is reliable.

This is a planning sequence, not evidence that any phase has already been implemented.

## 26. Definition of Completion

The technology setup is complete when:

- The installed Python version is verified.
- The virtual environment is reproducible.
- Required dependencies are declared.
- The application starts with the documented command.
- The four documented API endpoints behave as specified.
- The core models validate correctly.
- Upload restrictions are enforced.
- Implemented rules have automated tests.
- API tests pass.
- The response conforms to `AnalysisResult`.
- Temporary-file cleanup is tested where applicable.
- The deterministic analyzer works with AI disabled.
- No database or paid service is required.
- Documentation matches the implemented behavior.

Completion must be demonstrated through code and tests rather than inferred from documentation.

## 27. Final Technology Decision

The MVP will use Python 3.12+, FastAPI, Uvicorn, Pydantic, the Python standard library, deterministic detection rules, custom correlation and recommendation logic, HTML/CSS/JavaScript, Pytest, HTTPX, and Git.

Ollama and a local RAG stack remain optional future enhancements. SQLite remains a possible future storage choice.

The governing principles are:

1. Deterministic analysis first.
2. Preserve evidence.
3. Keep the API and data models consistent.
4. Validate all uploads on the backend.
5. Keep processing local and avoid permanent storage by default.
6. Treat correlations as possible relationships.
7. Keep recommendations advisory and read-only.
8. Add dependencies only when justified.
9. Test each implemented feature.
10. Keep optional AI separate from the core analyzer.

Relevant canonical documents:

- `docs/PROJECT_MASTER_PLAN.md`
- `docs/PRD.md`
- `docs/ARCHITECTURE.md`
- `docs/API_SPECIFICATION.md`
- `docs/DATA_MODELS.md`
- `docs/DETECTION_RULES.md`
- `docs/AI_DESIGN.md`
- `docs/SECURITY_AND_PRIVACY.md`
- `docs/IMPLEMENTATION_PLAN.md`
- `docs/TESTING_STRATEGY.md`
