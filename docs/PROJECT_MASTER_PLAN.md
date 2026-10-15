# NetAssist AI — Master Project Plan

**Version:** 1.0  
**Status:** Planning  
**Product:** Evidence-Based Network Troubleshooting Assistant  
**Budget:** Free tools only  
**Initial deployment:** Local Windows PC

## 1. Project Overview

NetAssist AI is a network troubleshooting application that analyzes router and switch log files, detects known network events, groups related evidence where possible, and provides structured diagnostic recommendations.

The application is intended to help network engineers investigate network incidents without manually reviewing every log entry.

## 2. Problem Statement

Network devices generate large volumes of logs. Engineers must identify important errors, understand their possible implications, investigate related events, and decide which diagnostic checks to perform.

Manual analysis can be slow, particularly when multiple events occur close together.

NetAssist AI will organize this information into an evidence-based report that helps engineers investigate problems more efficiently.

## 3. Product Objectives

1. Read supported router and switch log files.
2. Detect documented network events using deterministic rules.
3. Preserve original log evidence and source line numbers.
4. Classify events using configurable severity rules.
5. Identify potential relationships between events when sufficient evidence exists.
6. Recommend safe, read-only diagnostic checks.
7. Provide an easy-to-use browser dashboard.
8. Export reports for troubleshooting and incident handoffs.
9. Add optional AI-generated explanations without making AI essential to core functionality.

## 4. Target Users

- Network engineers
- Network operations teams
- IT support engineers
- Network troubleshooting learners

## 5. Product Scope

### MVP — First working product

- Upload TXT and LOG files.
- Validate file type, size, and content.
- Parse supported log formats.
- Detect interface-up/down and BGP neighbor events.
- Add further documented rules for packet drops and other network events.
- Display severity, source evidence, and recommended checks.
- Filter and search detected events.
- Generate downloadable JSON reports.
- Provide a local browser-based interface.
- Include automated tests and sample logs.

### Future enhancements

- More vendor-specific log formats.
- Event correlation using timestamps and device identifiers.
- Optional AI explanations.
- Retrieval from trusted networking documentation.
- Analysis history and local database storage.
- Authentication and multi-user access if required.
- Optional read-only network-device integrations.
- Deployment for authorized team use.

## 6. Technology Stack

| Component | Technology |
|---|---|
| Programming language | Python |
| Backend API | FastAPI |
| Pattern detection | Python regular expressions |
| Frontend | HTML, CSS, JavaScript |
| Testing | pytest |
| Version control | Git |
| Optional local AI | Ollama with a compatible local model |
| Optional local storage | SQLite |

All tools must be checked for compatibility, licensing, and resource requirements. The MVP must not require paid APIs or subscriptions.

## 7. Architecture Principles

- Keep parsing, detection, correlation, reporting, and AI separate.
- Make core analysis work when AI is disabled.
- Attach source evidence to every detected event.
- Distinguish observed facts from possible causes.
- Validate all uploaded files.
- Never execute commands contained in uploaded logs.
- Never change network-device configurations automatically.
- Avoid storing uploaded logs unless persistence is explicitly implemented and documented.
- Test each feature before adding the next.

## 8. Development Roadmap

### Phase 0 — Documentation
Finalize product requirements, architecture, technology decisions, detection rules, security, and testing plans.

### Phase 1 — Environment
Set up the project folder, Git, Python virtual environment, dependencies, and test framework.

### Phase 2 — Core log analyzer
Implement file reading, parsing, event detection, severity classification, and structured reports.

### Phase 3 — Networking rules
Expand coverage to supported interface, VLAN, ARP, MAC-learning, packet-drop, and routing messages.

### Phase 4 — Correlation and evaluation
Identify related events using documented rules. Evaluate detection precision, recall, and false positives.

### Phase 5 — Web dashboard
Implement file uploads, alerts, filters, event details, and report downloads.

### Phase 6 — AI assistance
Add optional evidence-grounded explanations, uncertainty handling, and suggested diagnostic checks.

### Phase 7 — Product hardening
Complete regression tests, security review, clean-install verification, documentation, and release preparation.

## 9. MVP Acceptance Criteria

The MVP is acceptable for demonstration when:

- Supported log files can be analyzed successfully.
- Test fixtures produce expected detections.
- Every event links to its original evidence.
- Unsupported and malformed files are handled safely.
- Automated tests pass.
- The dashboard displays analysis results correctly.
- Reports can be downloaded and reviewed.
- Core functionality works without AI.
- Setup instructions work on a clean environment.
- Limitations and unsupported formats are documented.

## 10. Security Boundaries

The initial version analyzes supplied files only.

It will not:
- Connect to production routers or switches.
- Execute diagnostic commands on devices.
- Modify network configurations.
- Claim a confirmed root cause from insufficient evidence.
- Upload private network logs to an external AI service automatically.

Any future device integration requires explicit authorization and separate security design.

## 11. Definition of Done

A feature is complete only when its code, tests, documentation, error handling, and acceptance criteria have been reviewed.

Planned features must not be presented as implemented features.

## 12. Final Deliverables

- Source code and Git history.
- Product and architecture documentation.
- Sample logs and expected outputs.
- Automated tests and evaluation results.
- Working local web application.
- Downloadable example reports.
- Installation and troubleshooting instructions.
- Security notes and known limitations.
- Demonstration and release checklist.