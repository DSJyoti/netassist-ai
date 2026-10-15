# NetAssist AI — Requirements Traceability Matrix

**Project:** NetAssist AI  
**Document Type:** Requirements Traceability  
**Status:** Planning  
**Version:** 1.0

---

## 1. Purpose

This document maps product requirements to planned application components, verification methods and acceptance criteria.

It helps ensure that:

- Every important requirement has an implementation plan.
- Every critical behavior has a corresponding test.
- Security and privacy requirements are not overlooked.
- Scope changes are reviewed before implementation.
- The MVP can be evaluated against explicit completion criteria.

The statuses in this document describe planning, not completed implementation.

## 2. Requirement Status Definitions

| Status | Meaning |
|---|---|
| Planned | Requirement has been identified and documented |
| In Progress | Implementation or verification has started |
| Implemented | Code has been written |
| Verified | Relevant acceptance criteria and tests have passed |
| Deferred | Intentionally postponed beyond the current scope |

A requirement must not be marked Verified merely because its code exists.

## 3. Functional Requirements

| ID | Requirement | Planned component | Verification | Acceptance criteria |
|---|---|---|---|---|
| FR-001 | Accept supported `.log` and `.txt` files | Upload validation | API tests | Supported files are accepted |
| FR-002 | Reject unsupported file types | Upload validation | Negative tests | Unsupported files are rejected safely |
| FR-003 | Enforce a configurable file-size limit | Upload validation | Boundary tests | Oversized uploads are rejected |
| FR-004 | Handle empty and malformed files | Validation and parser | Edge-case tests | Clear error or warning is returned |
| FR-005 | Parse supported network log formats | Log parser | Parser unit tests | Recognized lines are parsed consistently |
| FR-006 | Preserve source-line references | Parser and evidence model | Evidence tests | Findings point to relevant source lines where available |
| FR-007 | Normalize detected events | Event model | Model tests | Events follow the defined schema |
| FR-008 | Detect interface state changes | Detection engine | Rule tests | Supported interface events are recognized |
| FR-009 | Detect link flapping | Detection engine | Rule tests | Repeated transitions meeting defined criteria are detected |
| FR-010 | Detect interface errors and packet drops | Detection engine | Rule tests | Supported error patterns produce findings |
| FR-011 | Detect BGP neighbor changes | Detection engine | Rule tests | Supported BGP events are recognized |
| FR-012 | Detect routing-related changes | Detection engine | Rule tests | Supported routing patterns produce findings |
| FR-013 | Detect ARP-related warnings | Detection engine | Rule tests | Supported ARP patterns produce findings |
| FR-014 | Detect VLAN-related warnings | Detection engine | Rule tests | Supported VLAN patterns produce findings |
| FR-015 | Detect MAC movement or flapping | Detection engine | Rule tests | Supported MAC events produce findings |
| FR-016 | Assign severity to findings | Detection engine | Classification tests | Severity follows documented criteria |
| FR-017 | Attach evidence to findings | Evidence model | Evidence tests | Findings include available supporting evidence |
| FR-018 | Correlate potentially related events | Correlation engine | Correlation tests | Only defined correlation conditions are applied |
| FR-019 | Distinguish correlation from causation | Correlation and report layers | Scenario tests | Possible relationships are not presented as confirmed causes |
| FR-020 | Generate safe recommendations | Recommendation engine | Recommendation tests | Recommendations are relevant, cautious and human-reviewed |
| FR-021 | Provide analysis through a REST API | FastAPI backend | Integration tests | Defined endpoints return documented responses |
| FR-022 | Display analysis results in a browser | Frontend | Manual and integration tests | Users can inspect findings and evidence |
| FR-023 | Export structured results as JSON | Report/API layer | Schema tests | Export follows the defined result structure |
| FR-024 | Operate without an AI model | Core analysis pipeline | AI-disabled tests | Core detection and reporting work with AI disabled |
| FR-025 | Return useful warnings when no events match | Analysis and report layers | Integration tests | Empty findings are returned as a valid result with appropriate messaging |

## 4. Security and Privacy Requirements

| ID | Requirement | Planned component | Verification | Acceptance criteria |
|---|---|---|---|---|
| SR-001 | Treat uploaded files as untrusted input | Validation and parser | Security tests | Invalid input is handled safely |
| SR-002 | Limit upload size | Upload validation | Boundary tests | Configured limits are enforced |
| SR-003 | Avoid executing uploaded content | Parser and application | Code review and security tests | File contents are treated as data |
| SR-004 | Avoid automatic network commands | Application architecture | Code review | No automatic command execution exists |
| SR-005 | Avoid automatic device configuration changes | Application architecture | Code review | No configuration-changing workflow exists |
| SR-006 | Avoid unnecessary permanent file storage | Processing layer | Integration tests | Uploaded files are not retained by default |
| SR-007 | Avoid exposing raw logs unnecessarily | API, logging and frontend | Privacy review | Responses and logs contain only necessary information |
| SR-008 | Keep secrets out of source control | Configuration and Git | Repository review | No credentials are committed |
| SR-009 | Handle malicious-looking log text as data | Parser, frontend and optional AI layer | Security tests | Log content cannot become executable markup or trusted instructions |
| SR-010 | Keep the MVP functional without external AI services | Core pipeline | Offline or dependency-isolation tests | Core analysis does not require a paid AI API |

## 5. Reliability and Quality Requirements

| ID | Requirement | Planned component | Verification | Acceptance criteria |
|---|---|---|---|---|
| QR-001 | Use consistent data models | Models | Unit tests | Valid results conform to the documented schema |
| QR-002 | Return predictable API errors | API layer | API tests | Invalid requests receive documented error responses |
| QR-003 | Keep detection rules independently testable | Detection engine | Unit tests | Each rule can be tested in isolation |
| QR-004 | Avoid false claims when evidence is insufficient | Detection and report layers | Negative and scenario tests | Unsupported conclusions are not asserted as facts |
| QR-005 | Maintain regression coverage | Test suite | Regression tests | Previously fixed issues remain covered |
| QR-006 | Document known limitations | Documentation and report layer | Review checklist | Relevant limitations are discoverable |
| QR-007 | Keep the code modular | Application architecture | Code review | Parsing, detection, correlation and reporting remain separate responsibilities |
| QR-008 | Provide understandable user feedback | Frontend and API | Manual tests | Success, warnings and errors are clearly communicated |

## 6. Optional AI Requirements — Future Scope

These requirements are deferred until the deterministic MVP has been implemented and validated.

| ID | Requirement | Planned component | Verification | Acceptance criteria |
|---|---|---|---|---|
| AI-001 | Support an optional local AI adapter | AI integration layer | Adapter tests | AI can be enabled without redesigning the core pipeline |
| AI-002 | Ground explanations in structured findings | AI explanation layer | Evidence-grounding tests | Explanations reference supplied evidence |
| AI-003 | Prevent AI from replacing detection results | AI integration layer | Contract tests | AI output cannot alter authoritative findings |
| AI-004 | Handle AI unavailability gracefully | AI adapter and API | Failure tests | Core analysis continues without AI |
| AI-005 | Treat log text as untrusted content | AI adapter | Adversarial-input tests | Log instructions are not treated as trusted system instructions |
| AI-006 | Avoid mandatory paid model services | Configuration and AI adapter | Configuration tests | A paid API is not required for the MVP |

## 7. Requirement-to-Document Mapping

| Concern | Source document |
|---|---|
| Project objectives and boundaries | `PROJECT_MASTER_PLAN.md` |
| User needs and product behavior | `PRD.md` |
| Component boundaries and data flow | `ARCHITECTURE.md` |
| Libraries and tools | `TECH_STACK.md` |
| Detection logic | `DETECTION_RULES.md` |
| Future AI behavior | `AI_DESIGN.md` |
| Security and privacy | `SECURITY_AND_PRIVACY.md` |
| API contracts | `API_SPECIFICATION.md` |
| Request, event and result models | `DATA_MODELS.md` |
| Test coverage and test strategy | `TESTING_STRATEGY.md` |
| Implementation sequence | `IMPLEMENTATION_PLAN.md` |
| Development practices | `DEVELOPMENT_WORKFLOW.md` |
| Overall progress | `PROJECT_STATUS.md` |

## 8. MVP Acceptance Gate

The MVP can be considered ready for release only when all applicable items below are verified.

- [ ] Supported log files can be uploaded and validated.
- [ ] Invalid and oversized files are rejected safely.
- [ ] Supported log patterns are parsed correctly.
- [ ] Implemented detection rules have positive and negative tests.
- [ ] Findings include available source evidence.
- [ ] Severity is assigned according to documented criteria.
- [ ] Correlations are described cautiously.
- [ ] Recommendations are safe and require human judgment.
- [ ] The API follows its documented contracts.
- [ ] The browser interface displays findings and warnings.
- [ ] JSON export works.
- [ ] Security and privacy checks pass.
- [ ] The full applicable test suite passes.
- [ ] The application works with AI disabled.
- [ ] Known limitations are documented.
- [ ] The README and setup instructions have been verified.

## 9. Change Control

When a requirement changes:

1. Assign or retain a stable requirement ID.
2. Update the source document.
3. Identify affected components and tests.
4. Update this traceability matrix.
5. Implement the change.
6. Run relevant tests and regression checks.
7. Mark the requirement Verified only after the acceptance criteria pass.

Requirements may be deferred only when the decision and its impact are documented.

---

**Core rule:** Every critical requirement must have a defined verification method, and every Verified status must be supported by actual evidence.