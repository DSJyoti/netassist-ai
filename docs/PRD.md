# NetAssist AI — Product Requirements Document

**Version:** 1.1
**Status:** Planning / Documentation Baseline
**Product:** NetAssist AI  
**Product Type:** Local-First Network Log Analysis and Troubleshooting Assistant
**Primary Environment:** Local Windows PC
**Budget:** Free and Open-Source Tools Where Practical

---

## 1. Product Overview

NetAssist AI is a local-first, evidence-based network troubleshooting assistant that analyzes user-supplied network device logs and identifies supported networking events.

The application validates uploaded files, parses log lines, detects known events through deterministic Python rules, preserves source evidence, assigns severity, identifies cautious possible relationships between events, and provides read-only troubleshooting recommendations.

The core analyzer must operate without an external paid AI service. An optional local Retrieval-Augmented Generation (RAG) and Large Language Model (LLM) capability may be added in a later phase.

NetAssist AI is a decision-support tool. It does not directly connect to production routers or switches, execute network commands, or automatically change network configurations.

## 2. Problem Statement

Network devices generate logs containing interface state changes, routing events, authentication failures, and other operational information.

Manually reviewing these logs can be time-consuming. Engineers need to identify important events, determine where and when they occurred, preserve supporting evidence, and decide what to investigate next.

NetAssist AI aims to simplify the first stage of this investigation by converting raw log files into structured, searchable analysis results.

The application must distinguish facts directly supported by logs from hypotheses and possible event relationships.

## 3. Product Goals

### 3.1 Primary Goals

The MVP shall:

- Accept supported `.log` and `.txt` files.
- Validate uploaded files before analysis.
- Parse logs line by line where practical.
- Detect a limited, explicitly defined set of network events.
- Preserve original evidence and source line numbers.
- Extract timestamps and resource identifiers when available.
- Assign deterministic severity levels.
- Generate structured event summaries.
- Identify cautious, evidence-backed possible event relationships.
- Provide safe, read-only troubleshooting recommendations.
- Return results through a documented JSON API.
- Provide a browser-based interface for uploading logs and reviewing results.
- Operate without a paid LLM or external AI API.
- Avoid permanent storage of uploaded logs and analysis results by default.

### 3.2 Secondary Goals

The following are future enhancements, not MVP commitments:

- Additional network vendors and log formats.
- Advanced link-flap and interface-counter analysis.
- Expanded VLAN, ARP, MAC learning, and routing event detection.
- More advanced event correlation and incident grouping.
- Optional local RAG and local LLM explanations.
- Natural-language questions about logs.
- Downloadable reports.
- Optional local history and persistence.
- Read-only network-device integrations.
- Authentication, multi-user access, and server deployment.

## 4. Non-Goals

The MVP will not:

- Connect directly to production network devices.
- Execute commands found in uploaded logs.
- Execute troubleshooting commands on routers or switches.
- Modify router, switch, VLAN, routing, or firewall configurations.
- Automatically remediate incidents.
- Claim a verified root cause from temporal correlation alone.
- Require a paid AI API.
- Require an internet connection for deterministic analysis after dependencies are installed.
- Store uploaded logs or analysis history permanently by default.
- Require a database.
- Require authentication or multi-user functionality.

## 5. Target Users

### 5.1 Network Engineers

Upload a log file, inspect detected events, and review evidence and recommended checks.

### 5.2 IT Support and Network Operations Teams

Use structured findings as an initial investigation aid before escalating an incident.

### 5.3 Networking Students

Learn to interpret supported network events using sample logs and evidence-based explanations.

### 5.4 Developers and AI Engineers

Use structured analysis results as a foundation for future local AI explanations and RAG-based troubleshooting.

## 6. Core User Journey

1. Start NetAssist AI locally.
2. Open the browser interface.
3. Select a `.log` or `.txt` file.
4. Validate the file.
5. Parse the log and preserve source line information.
6. Detect supported events using deterministic rules.
7. Assign severity and extract available metadata.
8. Identify possible relationships where supported by evidence.
9. Generate recommendations and structured warnings.
10. Display the analysis summary and event details.
11. Inspect the structured JSON response through the API.

The analysis flow must not require an LLM.

## 7. Functional Requirements

### FR-001 — Log File Upload

The application shall accept user-supplied files with the `.log` and `.txt` extensions.

Unsupported file types shall be rejected with a clear error response.

### FR-002 — File Validation

The application shall validate:

- File extension.
- File size.
- Empty input.
- Text readability and supported encoding.
- Malformed or unexpected input.

The initial maximum upload size is **5 MiB (5 × 1024 × 1024 bytes)**.

The limit shall be configurable. Validation must not rely only on the client-provided filename or content type.

### FR-003 — Log Parsing

The parser shall process log lines independently where practical.

It shall preserve:

- Original source line text.
- One-based source line number.
- Timestamp when reliably identifiable.
- Parsed message or event details when available.
- Source filename when relevant to the analysis.

Missing or unrecognized timestamps must not cause the entire analysis to fail.

The parser must not invent missing metadata.

### FR-004 — Interface State Detection

The initial rule set shall detect supported interface-up and interface-down messages.

For a matched event, the application shall record the applicable rule ID, event type, severity, description, available resource information, and source evidence.

Only explicitly tested patterns shall generate structured events.

### FR-005 — BGP Neighbor State Detection

The initial rule set shall detect supported BGP neighbor-up and neighbor-down messages.

The application shall extract the neighbor address and timestamp when available and preserve the source evidence.

A BGP event must not automatically be interpreted as proof of an underlying interface failure.

### FR-006 — Device Error Detection

The initial rule set shall detect supported device-error messages.

A generic occurrence of the word "error" shall not automatically establish a specific network fault.

The event description and severity must follow the corresponding tested rule.

### FR-007 — Authentication Failure Detection

The initial rule set shall detect supported authentication-failure messages.

Evidence must be preserved without unnecessarily exposing credentials, tokens, or other sensitive values.

The application shall not attempt authentication against any network device.

### FR-008 — Explicit Rule Registry

The initial detection rules shall follow the identifiers defined in `DETECTION_RULES.md`:

- `IFACE_DOWN_001`
- `IFACE_UP_001`
- `BGP_NEIGHBOR_DOWN_001`
- `BGP_NEIGHBOR_UP_001`
- `DEVICE_ERROR_001`
- `AUTH_FAILURE_001`

These identifiers define the initial implementation scope. Additional rules shall be added only after their patterns, extracted fields, severity, evidence behavior, and tests are documented.

A rule listed in documentation is not considered implemented until working code and tests exist.

### FR-009 — Severity Assignment

Each detected event shall receive one of the canonical severity values:

- `INFO`
- `LOW`
- `MEDIUM`
- `HIGH`
- `CRITICAL`

Severity assignment shall be deterministic and rule-based for the MVP.

Severity shall not be determined solely by the number of alarming words in a log line. A severity level does not guarantee actual business impact.

### FR-010 — Evidence Preservation

Every detected event shall contain supporting evidence consistent with `DATA_MODELS.md`.

Evidence shall preserve the source line number and original text. Timestamps and source filenames shall be included when available.

The system shall not create an event without evidence supporting the corresponding rule match.

### FR-011 — Possible Event Correlation

The analyzer may identify possible relationships between supported events using explicit, deterministic criteria.

For example, an interface-down event followed shortly by a BGP-neighbor-down event may be reported as a possible relationship.

The correlation result shall:

- Reference existing event IDs.
- Include supporting evidence.
- Describe the observed relationship.
- Include a confidence value when supported by the correlation logic.
- Avoid presenting temporal order as proof of causality.
- Leave `is_confirmed_cause` false unless a separately verified process is introduced in a future version.

Advanced topology-aware correlation is outside the initial MVP.

### FR-012 — Troubleshooting Recommendations

The application shall provide structured, read-only recommendations for supported events.

Examples include checking interface status, reviewing interface counters, inspecting relevant logs, and verifying BGP neighbor state.

Each recommendation shall follow the canonical recommendation model, including its action type, risk level, and authorization requirement.

Recommendations are guidance only. The application shall never execute the suggested commands.

### FR-013 — Analysis Summary

The analysis response shall include the canonical summary fields defined in `DATA_MODELS.md`, including total lines processed and event counts by severity.

Summary counts must be derived from the actual analysis results and remain internally consistent.

### FR-014 — Browser Interface

The MVP browser interface shall allow users to:

- Select a supported log file.
- Submit the file for analysis.
- View validation or analysis errors.
- View the analysis summary.
- Inspect detected events.
- Inspect severity, resource details, and source evidence.
- Review possible correlations, warnings, and recommendations.

Advanced dashboard features such as multiple filters, full-text search, historical comparisons, and downloadable reports may be added after the core workflow is validated.

### FR-015 — JSON API

The application shall return a structured JSON analysis result from:

`POST /api/v1/analyze`

The response shall conform to the canonical `AnalysisResult` model defined in `DATA_MODELS.md` and the contract documented in `API_SPECIFICATION.md`.

The API response shall contain the analysis summary, events, correlations, warnings, and processing metadata as specified by the canonical model.

The MVP does not require a separate report-download endpoint or permanent analysis-history endpoint.

### FR-016 — Health and Readiness

The application shall provide the following endpoints:

- `GET /health`
- `GET /ready`
- `GET /api/v1/info`

Their behavior and response contracts shall follow `API_SPECIFICATION.md`.

### FR-017 — Optional AI Extension

The deterministic analyzer shall operate when AI is disabled.

Future local AI functionality may generate natural-language explanations, summarize evidence, or retrieve relevant troubleshooting guidance from a local knowledge base.

AI output shall remain advisory and must not silently alter deterministic event detection, severity, source evidence, or correlation results.

The AI extension must follow `AI_DESIGN.md` and any approved changes to the canonical data models and API contract.

## 8. Canonical Data Contracts

The following documents define the authoritative implementation contracts:

- `DATA_MODELS.md` — canonical models, field names, enums, and invariants.
- `API_SPECIFICATION.md` — endpoint behavior, validation, and response formats.
- `DETECTION_RULES.md` — rule identifiers, matching behavior, and severity.
- `ARCHITECTURE.md` — component boundaries and processing flow.
- `AI_DESIGN.md` — optional local RAG and AI behavior.
- `SECURITY_AND_PRIVACY.md` — file handling, safety, and privacy controls.
- `TESTING_STRATEGY.md` — validation and test requirements.
- `IMPLEMENTATION_PLAN.md` — implementation order and phase exit criteria.

These documents must remain consistent. A new field or endpoint must not be introduced in code without updating the relevant canonical contract.

## 9. Fact, Hypothesis, and Root Cause

NetAssist AI shall distinguish between three concepts.

### 9.1 Observed Fact

A fact is directly supported by source evidence.

Example: a log line reports that an interface changed state to down.

### 9.2 Possible Relationship

A possible relationship is a hypothesis supported by a defined correlation rule.

Example: a BGP neighbor-down event occurred shortly after an interface-down event.

### 9.3 Verified Root Cause

The MVP shall not claim a verified root cause solely because events occurred near one another in time.

Descriptions and recommendations shall make uncertainty clear and identify further checks where appropriate.

## 10. Privacy and Security Requirements

The application shall:

- Process uploaded logs locally.
- Avoid sending logs to external AI services.
- Avoid permanent storage of uploaded files and results by default.
- Clean up temporary files and processing artifacts.
- Validate file size and type.
- Avoid unsafe path handling.
- Sanitize filenames before using them in display or file operations.
- Handle malformed input safely.
- Never execute content found in uploaded logs.
- Keep secrets outside source code.
- Exclude credentials, virtual environments, caches, and local secrets from Git.
- Avoid unnecessarily displaying sensitive information.

Optional local AI components must not bypass the application's privacy and safety controls.

## 11. Performance and Reliability

The initial performance target is to analyze supported log files up to 5 MiB on the local development machine.

Performance shall be measured during testing rather than assumed.

The analyzer shall:

- Process valid lines when another line is malformed, where safe and practical.
- Return structured errors for invalid uploads.
- Report relevant parsing limitations through supported warnings.
- Preserve source evidence for detected events.
- Produce deterministic results for the same input and rule-set version, excluding intentionally variable metadata such as generated IDs, timestamps, and processing duration.
- Avoid requiring external network access for the deterministic analysis path after local dependencies are installed.

## 12. Testing Requirements

### 12.1 Unit Tests

Tests shall cover:

- File validation.
- Parsing and source line numbering.
- Timestamp extraction.
- Each implemented detection rule.
- Resource extraction.
- Severity assignment.
- Summary generation.
- Correlation logic.
- Recommendation generation.
- Canonical model validation.

### 12.2 API Integration Tests

Tests shall cover:

- Health and readiness endpoints.
- Application information endpoint.
- Successful log upload and analysis.
- Unsupported extensions.
- Empty and oversized uploads.
- Invalid or unsupported text encoding.
- Structured success and error responses.
- Temporary-file cleanup where applicable.

### 12.3 UI Tests

The initial UI validation shall cover:

- File selection and submission.
- Display of analysis results.
- Display of source evidence.
- Handling of API and validation errors.

Advanced UI automation may be added as the interface develops.

### 12.4 Negative and Safety Tests

Tests shall verify that:

- Unknown log patterns do not generate invented events.
- Malformed lines do not crash the entire analysis unnecessarily.
- Missing timestamps are handled safely.
- Uploaded text is treated as data, not as instructions.
- No network-device commands are executed.
- No paid AI service is required for core tests.
- AI-disabled operation remains functional.

## 13. MVP Acceptance Criteria

The MVP is complete when:

1. The application can be started locally using documented instructions.
2. The browser interface can submit supported log files.
3. The upload validation enforces the configured 5 MiB limit.
4. The parser preserves original evidence and line numbers.
5. The six initial rule identifiers have implemented, tested behavior.
6. Severity and summaries follow the canonical models.
7. Supported correlations are cautious and evidence-backed.
8. Recommendations follow the structured recommendation model.
9. The four documented API endpoints work according to their specifications.
10. The analysis endpoint returns a valid canonical `AnalysisResult`.
11. The browser displays summaries, events, evidence, and relevant recommendations.
12. Automated tests cover the core analysis and validation paths.
13. The core analyzer works with AI disabled and without a paid AI service.
14. Uploaded logs are not permanently stored by default.
15. No production device access or automatic configuration is performed.

Features not included in these criteria shall not block MVP completion unless they are formally added to the approved scope.

## 14. MVP Success Metrics

Success shall be evaluated using a controlled test corpus.

### Detection Accuracy

The proportion of expected supported events correctly detected.

### False Positive Rate

The proportion of generated events that are not supported by the expected test evidence.

### Evidence Accuracy

The proportion of detected events whose source line and evidence match the original input.

### Parser Reliability

The proportion of supported test files processed without an unexpected application failure.

### Reproducibility

Consistency of deterministic analysis results for identical input and rule-set versions, accounting for intentionally variable metadata.

### Safety

Successful completion of tests demonstrating that uploaded content is not executed and network configurations are not modified.

Metrics shall be measured and reported; they must not be represented as achieved results before testing.

## 15. Future Product Versions

### Version 1.1 — Detection Improvements

Potential additions:

- Additional vendor-specific patterns.
- More interface and routing rules.
- Improved timestamp handling.
- Explicitly tested link-flap detection.
- Additional error, VLAN, ARP, and MAC learning rules.

### Version 1.2 — Analysis Improvements

Potential additions:

- Advanced correlation rules.
- Incident grouping.
- Improved confidence scoring.
- Optional local history and SQLite persistence.

### Version 2.0 — Optional Local AI and RAG

Potential additions:

- Local embedding models.
- Local vector search and knowledge indexing.
- Optional local LLM explanations.
- Citation-backed troubleshooting answers.
- Natural-language investigation.
- AI-generated incident summaries.

This phase remains optional and must not make the deterministic analyzer dependent on AI.

### Version 3.0 — Extended Integrations

Potential additions:

- Explicitly authorized read-only device integrations.
- Network topology context.
- Authentication and multi-user access.
- Centralized deployment.
- Monitoring integrations.

Any future device integration or deployment must undergo separate security and privacy review.

## 16. Product Safety Principle

NetAssist AI is an analysis and decision-support system, not an autonomous network administrator.

It helps users answer:

- What happened?
- Where and when did it happen?
- What source evidence supports the event?
- Which events may be related?
- What should be investigated next?

It does not automatically decide or execute network configuration changes.

## 17. Product Definition

**NetAssist AI is an evidence-based network troubleshooting assistant that converts raw network logs into structured events, severity-aware findings, source-backed explanations, cautious possible correlations, and safe troubleshooting recommendations.**

The product follows these principles:

1. Deterministic analysis comes first.
2. Evidence must be preserved.
3. Unknown patterns must not produce invented facts.
4. Correlation does not prove causation.
5. Recommendations remain advisory and read-only.
6. Local privacy is the default.
7. AI and RAG are optional enhancements, not requirements for core analysis.
8. A documented feature is not complete until it is implemented and tested.
