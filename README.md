# NetAssist AI

**An Evidence-Based Network Troubleshooting Assistant**

NetAssist AI is a local-first network log analysis application designed to help users identify supported network events, examine their supporting evidence, and follow safe troubleshooting recommendations.

The initial Minimum Viable Product (MVP) will use deterministic detection rules and structured analysis rather than depending on an AI model. An optional local AI explanation layer may be evaluated after the core application is implemented and tested.

**Project status:** Documentation and planning  
**Implementation status:** Not started

---

## 1. Problem Statement

Network logs can contain large amounts of technical information. Identifying important events and understanding their possible relationships can take time, particularly during initial troubleshooting.

NetAssist AI aims to simplify this process by transforming supported log files into structured events, source evidence, severity classifications, and actionable troubleshooting recommendations.

The application is intended to support human troubleshooting decisions. It does not replace professional network diagnostics or establish a confirmed root cause when the available evidence is insufficient.

## 2. Planned MVP Features

The following features describe the planned MVP. They are not claims that the application has already been implemented.

- Upload supported `.log` and `.txt` files.
- Validate file types and enforce an upload size limit.
- Parse supported network log formats.
- Detect known network events using deterministic rules.
- Assign severity according to defined detection rules.
- Preserve supporting evidence and source line numbers where available.
- Generate safe, human-reviewed troubleshooting recommendations.
- Identify possible relationships between events without assuming causation.
- Display analysis results through a browser interface.
- Return structured analysis results through the API.
- Run locally without requiring a paid AI service.
- Keep core analysis functional when AI is disabled or unavailable.

**MVP limitation:** JSON report export, permanent analysis history, database storage, direct network-device access, and automatic remediation are outside the initial MVP scope.

## 3. Planned Architecture

```text
User
 |
 v
Browser Interface
 |
 v
FastAPI Backend
 |
 v
File Validation
 |
 v
Log Parser
 |
 v
Normalized Events
 |
 v
Deterministic Detection Engine
 |
 v
Event Correlation
 |
 v
Recommendations
 |
 v
AnalysisResult
 |
 v
Browser Results View
```

### Optional future AI component

A local AI explanation layer may consume validated, structured analysis results and explain findings in plain language.

It must not create authoritative detections, modify severity, alter source evidence, or execute network commands. The deterministic analysis result remains the source of truth.

## 4. Technology Stack

| Component | Planned technology |
|---|---|
| Programming language | Python |
| Backend API | FastAPI |
| Data validation | Pydantic |
| Initial log parsing | Python standard library and regular expressions |
| Frontend | HTML, CSS, and JavaScript |
| Automated testing | Pytest and HTTPX |
| Local development server | Uvicorn |
| Version control | Git |
| Optional future AI | A locally hosted model, subject to evaluation |

The MVP is designed around free tools and does not require a paid API. Optional local AI may require additional hardware resources and will be evaluated separately.

## 5. Planned Project Structure

```text
netassist-ai/
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
│   └── PROJECT_STATUS.md
├── backend/
├── frontend/
├── tests/
├── sample_logs/
├── scripts/
├── README.md
├── pyproject.toml
├── .env.example
└── .gitignore
```

This is the planned structure. Directories and configuration files will be created during implementation. The structure may be refined when needed for maintainability, testing, or security.

## 6. Getting Started

**Development status:** Setup instructions will be finalized after the project configuration has been created and tested.

The planned Windows development workflow is:

1. Open the project folder in Visual Studio Code.
2. Verify the installed Python version.
3. Create and activate a Python virtual environment.
4. Create the project configuration and dependency files.
5. Install the declared dependencies.
6. Start the FastAPI development server.
7. Open the browser interface.
8. Analyze a synthetic sample log.
9. Run the automated test suite.

Exact commands will be documented after the development environment and configuration files are finalized.

## 7. Planned API Endpoints

The initial API contract defines the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Check whether the application process is responding |
| GET | `/ready` | Check whether the application is ready to accept analysis requests |
| GET | `/api/v1/info` | Return basic application information |
| POST | `/api/v1/analyze` | Validate and analyze an uploaded log file |

The endpoint behavior and response structures will follow `docs/API_SPECIFICATION.md` and `docs/DATA_MODELS.md`.

The analysis endpoint is planned to return an `AnalysisResult` response directly. Permanent storage or retrieval of previous analysis results is not required for the initial MVP.

## 8. Safety and Privacy

NetAssist AI is designed as a troubleshooting aid, not an autonomous network administrator.

- The application will analyze uploaded files rather than directly access production routers or switches.
- It will not automatically execute network commands.
- It will not automatically change device configurations.
- File type and size will be validated before processing.
- Uploaded log content will be treated as untrusted input.
- Sensitive information should be minimized or redacted before upload where practical.
- Uploaded logs will not need to be stored permanently by default.
- Temporary-file cleanup will be attempted after processing, including error paths.
- Recommendations will require human review before being applied.
- Observed events and inferred relationships will be clearly distinguished.
- Local-only AI operation, if introduced, must not silently transmit logs to external services.

See `docs/SECURITY_AND_PRIVACY.md` for the planned security and privacy requirements.

## 9. Planned Detection Scope

The initial detection catalogue is intended to cover supported log patterns involving:

- Interface state changes and link flapping.
- CRC errors, input/output errors, and packet drops.
- BGP neighbor state changes and resets.
- Routing updates.
- ARP failures and duplicate IP warnings.
- VLAN-related errors.
- MAC address movement or flapping.
- Selected relationships between related events.

Detection accuracy will depend on the supported log format and the evidence available in the uploaded file.

Unsupported patterns must not be guessed. They may remain unclassified or be reported as unrecognized, depending on the final parser and data-model design.

Correlation can identify a possible relationship between observations, but it does not independently prove causation.

## 10. Testing Strategy

Testing will cover:

- File validation and invalid inputs.
- Empty, malformed, and oversized files.
- Parsing and source-line preservation.
- Individual deterministic detection rules.
- Severity classification.
- Evidence and recommendation generation.
- Event correlation and uncertainty handling.
- API responses and error handling.
- Security and privacy safeguards.
- Temporary-file cleanup.
- End-to-end analysis using synthetic logs.
- Regression testing after changes.

The core test suite must not require a paid API or a downloaded AI model.

See `docs/TESTING_STRATEGY.md` for the planned testing approach.

## 11. Development Roadmap

1. Finalize and review project documentation.
2. Set up the local development environment.
3. Build the backend foundation and data models.
4. Implement file validation and log parsing.
5. Implement and test deterministic detection rules.
6. Add event correlation and recommendations.
7. Integrate the API and browser interface.
8. Validate security, reliability, and end-to-end behavior.
9. Complete MVP acceptance testing.
10. Evaluate optional local AI enhancements after the deterministic MVP is stable.

Each phase should be verified before proceeding to the next.

## 12. Project Status

The project is currently in the documentation and planning phase. Application implementation has not started.

Planned features described in this README must not be interpreted as implemented or tested functionality.

- [Project Status Tracker](docs/PROJECT_STATUS.md)
- [Implementation Plan](docs/IMPLEMENTATION_PLAN.md)
- [Architecture](docs/ARCHITECTURE.md)
- [API Specification](docs/API_SPECIFICATION.md)
- [Data Models](docs/DATA_MODELS.md)
- [Detection Rules](docs/DETECTION_RULES.md)
- [AI Design](docs/AI_DESIGN.md)
- [Security and Privacy](docs/SECURITY_AND_PRIVACY.md)

## 13. Guiding Principle

**Evidence first. Safe recommendations. Human control.**

NetAssist AI should make network troubleshooting easier without presenting assumptions as facts, changing evidence-based findings through AI, or performing risky actions automatically.