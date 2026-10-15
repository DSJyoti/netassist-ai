# NetAssist AI — Development Workflow

**Project:** NetAssist AI  
**Document Type:** Engineering Development Standards  
**Status:** Planning  
**Applies To:** Backend, Frontend, Detection Rules, Testing and Optional AI

---

## 1. Purpose

This document defines the standard workflow for implementing, testing, reviewing and maintaining NetAssist AI.

The objective is to build the application incrementally, prevent regressions, keep the code maintainable and ensure that every reported network finding is supported by evidence.

## 2. Development Principles

All development work must follow these principles:

1. **Small changes:** Implement one logical feature at a time.
2. **Evidence first:** Every detected event must be traceable to input evidence.
3. **Test continuously:** Write tests alongside implementation.
4. **Fail safely:** Invalid input must not crash the entire application.
5. **Explicit uncertainty:** Do not present a possible correlation as a confirmed root cause.
6. **Security by design:** Treat uploaded files and log contents as untrusted input.
7. **Human control:** Never execute network commands or change device configurations automatically.
8. **Free-first tooling:** Avoid dependencies on paid services for the MVP.
9. **Document decisions:** Record important changes to requirements, interfaces and architecture.
10. **No premature complexity:** Do not introduce unnecessary frameworks, databases or infrastructure.

## 3. Standard Feature Development Cycle

Every feature must follow this workflow:

```text
Understand Requirement
        |
        v
Review Relevant Documentation
        |
        v
Define Expected Behavior
        |
        v
Write or Update Tests
        |
        v
Implement Small Change
        |
        v
Run Targeted Tests
        |
        v
Run Related Regression Tests
        |
        v
Review Security and Error Handling
        |
        v
Update Documentation
        |
        v
Mark Feature Complete
```

A feature is not complete merely because the code runs. It must satisfy its acceptance criteria and have appropriate test coverage.

## 4. Implementation Order

Follow the sequence defined in `IMPLEMENTATION_PLAN.md`.

### Stage 1 — Environment and Repository

- Verify Python and Git installations.
- Create the project directory and virtual environment.
- Initialize Git.
- Add project configuration and dependency files.
- Create the planned source and test directories.
- Verify that the application can be started locally.

### Stage 2 — Backend Foundation

- Create the FastAPI application.
- Add configuration management.
- Implement health and readiness endpoints.
- Define structured error responses.
- Add initial API tests.

### Stage 3 — Data Models

- Implement the models specified in `DATA_MODELS.md`.
- Define severity, confidence and event categories.
- Validate required fields and permitted values.
- Test valid and invalid model inputs.

### Stage 4 — File Validation and Parsing

- Accept only supported file types.
- Enforce the configured maximum upload size.
- Decode text safely.
- Handle empty, malformed and unsupported files.
- Preserve original source-line numbers.
- Avoid retaining uploaded content unnecessarily.

### Stage 5 — Detection Engine

- Implement one detection rule at a time.
- Give each rule a stable identifier.
- Include evidence and severity in each finding.
- Add positive, negative and edge-case tests.
- Ensure unrelated log lines do not trigger false findings unnecessarily.

### Stage 6 — Correlation and Recommendations

- Correlate only events that meet explicit criteria.
- Keep correlation separate from individual detections.
- Explain the evidence behind a suggested relationship.
- Generate safe, human-reviewed recommendations.
- Avoid claiming a confirmed root cause without sufficient evidence.

### Stage 7 — API and Frontend

- Connect the API to the analysis pipeline.
- Display validation errors clearly.
- Show event severity, evidence and recommendations.
- Display warnings and processing limitations.
- Add JSON export.
- Test the complete upload-to-results workflow.

### Stage 8 — Security and Release

- Test oversized and malformed files.
- Check that sensitive information is not unnecessarily logged.
- Verify that no network commands are executed.
- Run the full automated test suite.
- Review setup instructions and known limitations.
- Confirm that the MVP works with AI disabled.

## 5. Coding Standards

### Python

- Use clear, descriptive names.
- Add type hints to public functions and important interfaces.
- Keep functions focused on one responsibility.
- Use Pydantic for request and response validation where appropriate.
- Use Python's standard logging facilities.
- Handle expected errors explicitly.
- Avoid broad exception handling unless errors are re-raised or handled safely.
- Do not hardcode secrets, local file paths or environment-specific settings.
- Avoid unnecessary dependencies.

### API Design

- Follow the routes and schemas in `API_SPECIFICATION.md`.
- Use consistent response structures.
- Return appropriate HTTP status codes.
- Validate inputs at the API boundary.
- Avoid exposing internal tracebacks to users.
- Do not return unnecessary raw log contents.

### Frontend

- Keep presentation separate from backend detection logic.
- Show useful loading, success, empty and error states.
- Display evidence in a readable format.
- Never rely on the browser alone for file validation.
- Do not insert uploaded log text into HTML as trusted markup.

### Detection Rules

- Use stable rule identifiers.
- Define the log patterns and conditions each rule supports.
- Include evidence references wherever possible.
- Test both matching and non-matching inputs.
- Document format limitations.
- Do not silently convert uncertain interpretations into facts.

## 6. Testing Workflow

Run focused tests while implementing a feature, then run the broader suite before completing the work.

Planned commands after the project environment and test configuration are established:

```bash
python -m pytest tests/ -v
```

To run tests for an individual test module:

```bash
python -m pytest tests/test_example.py -v
```

`test_example.py` is a placeholder; use the actual test filename once it exists.

Testing must include:

- Unit tests for parsing and detection.
- API integration tests.
- Invalid-input and boundary tests.
- Security-focused tests.
- Regression tests for previously fixed defects.
- End-to-end tests using synthetic sample logs.

Tests must not depend on production routers, confidential customer logs or paid services.

## 7. Error Handling

Errors should be understandable to users and useful to developers without exposing sensitive information.

Examples include:

| Situation | Expected behavior |
|---|---|
| Unsupported file extension | Reject the upload with a clear message |
| File exceeds size limit | Reject before full processing |
| Empty file | Return a validation error or clear warning |
| Invalid text encoding | Handle safely and explain the problem |
| No recognized events | Return a valid result with an appropriate warning |
| Internal processing failure | Return a generic error and record safe diagnostic details |
| Optional AI unavailable | Continue core analysis without AI |

The exact response formats must follow `API_SPECIFICATION.md`.

## 8. Git Workflow

Use Git to track changes and provide recoverable checkpoints.

Recommended workflow:

1. Inspect the current repository status.
2. Make one logical change.
3. Run relevant tests.
4. Review the changed files.
5. Commit the change with a descriptive message.
6. Continue with the next task.

Example commit messages:

```text
docs: finalize development workflow
feat: add health endpoint
feat: validate uploaded log files
feat: implement interface state detection
test: add BGP detection regression tests
fix: handle empty log files safely
```

Do not commit:

- API keys or passwords.
- Virtual environment directories.
- Python cache files.
- Unnecessary generated reports.
- Confidential network logs.
- Local environment files containing secrets.

Follow `.gitignore` before the first commit.

## 9. AI-Assisted Development Rules

AI coding tools may help draft code, tests and documentation, but their output must be reviewed.

- Do not accept generated code without understanding its behavior.
- Verify dependency names and versions before adding them.
- Run tests after accepting code changes.
- Check that generated code matches the architecture and security documents.
- Do not allow an AI assistant to introduce automatic network access or configuration changes.
- Do not paste confidential logs, credentials or secrets into external AI services.
- Keep the core application independent of the optional AI layer.

## 10. Definition of Done

A task may be marked complete only when:

- [ ] Requirements and acceptance criteria are satisfied.
- [ ] Code follows the project's architecture.
- [ ] Relevant tests have been added or updated.
- [ ] Relevant tests pass.
- [ ] Error handling is appropriate.
- [ ] Security implications have been reviewed.
- [ ] Documentation is updated where necessary.
- [ ] No secrets or confidential files have been introduced.
- [ ] The feature behaves correctly with AI disabled, where applicable.
- [ ] Remaining limitations are documented.

## 11. Change Management

If a requirement or architecture decision changes:

1. Explain why the change is needed.
2. Identify affected documents, modules and tests.
3. Update the relevant documentation.
4. Implement the change in a small, testable step.
5. Run regression tests.
6. Update the project status tracker.

Do not expand the MVP scope without a clear reason.

## 12. Development Checkpoint

At the end of each implementation stage, record:

- Completed functionality.
- Files added or changed.
- Tests executed and their results.
- Known limitations.
- Outstanding defects.
- Next recommended task.

Update `PROJECT_STATUS.md` after the checkpoint has been verified.

---

**Engineering rule:** A feature is complete only when its behavior is tested, its risks are reviewed and its limitations are understood.
