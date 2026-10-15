# TESTING_STRATEGY.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant
**Document Version:** 1.1
**Status:** Testing Design
**Testing Framework:** Pytest  
**API Testing:** FastAPI TestClient / HTTPX
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the testing strategy for NetAssist AI.

Testing will verify that the application safely validates uploaded network logs, parses supported input, detects documented events, preserves source evidence, generates appropriate recommendations, and returns responses consistent with the canonical API contract.

Testing is required throughout development, not only before release.

The initial MVP must be testable without paid APIs, an installed AI model, internet access during core test execution, or access to production network devices.

## 2. Testing Objectives

The test suite must verify that:

1. Supported log files are processed according to documented rules.
2. Implemented detection rules identify expected events.
3. Unrelated log lines do not produce unsupported detections.
4. Every detected event contains traceable source evidence.
5. Event types and severity values match the configured rules.
6. Correlations do not automatically imply causality.
7. Recommendations remain safe and advisory.
8. Invalid and oversized uploads are handled appropriately.
9. API responses follow the canonical schemas.
10. Core functionality works when AI is disabled.
11. Sensitive information is handled carefully.
12. Previously fixed defects remain fixed.
13. Processing limitations and unrecognized lines are reported consistently.
14. Results are deterministic for identical input and ruleset versions, except for explicitly nondeterministic metadata such as generated identifiers or processing duration.

## 3. Testing Principles

- **Repeatable:** Identical input and ruleset versions produce consistent detection decisions.
- **Isolated:** Unit tests verify specific behaviors.
- **Independent:** Core tests do not require internet access or production systems.
- **Safe:** Synthetic logs are used instead of real customer or production logs.
- **Evidence-based:** Expected detections identify the source lines that support them.
- **Fast:** Unit tests remain suitable for frequent execution.
- **Maintainable:** Tests are organized around the application's components.
- **AI-independent:** Core tests do not require a local LLM, embedding model or paid API.
- **Contract-driven:** Tests use the canonical models and API specification.
- **Privacy-aware:** Uploaded content must not be unnecessarily retained or exposed.

## 4. Testing Levels

| Test level | What it verifies | Example |
|---|---|---|
| Unit tests | Individual functions and rules | Detect an interface-down event |
| Component tests | Connected backend components | Parser output enters the detection engine |
| API tests | HTTP endpoints and response contracts | Upload a valid log file |
| Integration tests | Complete backend workflow | Validate, parse, detect and return events |
| Security tests | Unsafe input and privacy boundaries | Reject an oversized file |
| Regression tests | Previously fixed issues remain fixed | A corrected BGP rule still works |
| Acceptance tests | MVP requirements are satisfied | Analyze a supported log without AI |

## 5. Test Tools

The initial testing stack will use:

- **pytest:** Test execution, fixtures and assertions.
- **FastAPI TestClient:** API endpoint tests.
- **HTTPX:** HTTP support for the FastAPI testing stack.
- **Python standard library:** Temporary files, text fixtures and basic utilities.

No paid testing platform is required.

Dependencies must be installed in the project's virtual environment and recorded in the project dependency configuration.

## 6. Proposed Test Directory Structure

```text
tests/
  conftest.py
  unit/
    test_file_validation.py
    test_log_parser.py
    test_network_event_models.py
    test_detection_rules.py
    test_correlation_engine.py
    test_recommendation_engine.py
    test_analysis_summary.py
  api/
    test_health.py
    test_ready.py
    test_info.py
    test_analyze_endpoint.py
  integration/
    test_analysis_pipeline.py
  security/
    test_upload_security.py
    test_untrusted_log_content.py
  regression/
    test_known_issues.py

sample_logs/
  interface_down.log
  interface_up.log
  bgp_neighbor_down.log
  bgp_neighbor_up.log
  device_error.log
  authentication_failure.log
  clean_log.log
  malformed.log
```

This is the proposed structure, not a statement that these files already exist. Files will be created as their associated components are implemented.

Additional fixtures may be introduced for future detection rules, but they must not be treated as MVP acceptance requirements.

## 7. Synthetic Test Data

All initial test fixtures must be synthetic.

Do not use real production logs containing customer information, credentials, private infrastructure details or other sensitive data.

Each fixture must have a documented purpose and an expected result.

### Example: `interface_down.log`

```text
2026-10-15T10:14:00Z switch-01 INFO System started
2026-10-15T10:15:00Z switch-01 WARN Interface GigabitEthernet1/0/1 changed state to down
2026-10-15T10:16:00Z switch-01 INFO Monitoring continues
```

Expected behavior:

- The parser processes the supported records.
- The detection engine identifies the interface-down event if the implemented pattern supports this format.
- The event references original source line 2.
- The event uses rule ID `IFACE_DOWN_001`.
- The event severity matches the documented rule configuration.
- The unrelated lines do not produce interface-down events.
- The system does not claim to know the physical root cause from this evidence alone.

The expected result must be validated against the implemented parser, rule patterns and canonical data models.

## 8. File Validation Tests

Test the following scenarios:

| Scenario | Expected result |
|---|---|
| Valid `.log` file | Accepted |
| Valid `.txt` file | Accepted |
| Unsupported extension | Rejected with a controlled error |
| Empty file | Handled according to documented validation behavior |
| File exceeding the configured size limit | Rejected |
| Missing upload field | Controlled validation error |
| Invalid text encoding | Handled according to documented error or partial-analysis policy |
| Filename containing path traversal characters | Must not enable path traversal |
| Very long line | Handled within documented resource limits |
| Binary content disguised as text | Rejected or safely handled according to validation policy |

Tests must verify that rejected uploads do not unnecessarily trigger parsing or detection.

The upload limit must remain aligned with `API_SPECIFICATION.md` and `TECH_STACK.md`: **5 MiB maximum** for the initial MVP.

## 9. Parser Tests

The parser must be tested against:

- Supported timestamp formats.
- Missing timestamps.
- Supported log levels.
- Device names and interface identifiers.
- Unexpected whitespace.
- Empty lines.
- Malformed records.
- Unrecognized message formats.
- Very long lines.
- Unicode characters.
- Multiple records with identical timestamps.
- Inputs containing text that resembles commands or instructions.

Expected results must be explicit for each fixture.

The parser must not invent missing timestamps, device names, interface names or resource identifiers.

The original one-based source line numbers must remain correct even when some lines are empty, malformed or unrecognized.

## 10. Detection Rule Tests

Each implemented MVP rule must have:

1. At least one positive test.
2. At least one negative test.
3. An expected rule ID and event type.
4. An expected severity.
5. Expected evidence line references.
6. A structured recommendation check.
7. Missing-field and timestamp edge cases.
8. Relevant false-positive tests.
9. Duplicate-handling tests.
10. Deterministic behavior tests.

### Initial MVP detection matrix

| Rule ID | Positive test | Negative or edge test |
|---|---|---|
| `IFACE_DOWN_001` | Explicit interface-down transition | Unrelated mention of an interface |
| `IFACE_UP_001` | Explicit interface-up transition | Interface remains down |
| `BGP_NEIGHBOR_DOWN_001` | Explicit BGP neighbor-down message | General mention of BGP without a down transition |
| `BGP_NEIGHBOR_UP_001` | Explicit BGP neighbor-up or established message | No supported state transition |
| `DEVICE_ERROR_001` | Documented device-error pattern | Generic use of the word `error` |
| `AUTH_FAILURE_001` | Supported authentication-failure message | Generic mention of login or authentication |

Only these six rules belong to the initial MVP detection contract.

Additional rules such as link flap, CRC errors, ARP failures, duplicate IP, VLAN mismatch and MAC flapping are future candidates. They require separate scope approval, documentation, implementation and tests before becoming active capabilities.

A planned rule must not be counted as passing before its implementation and tests exist.

## 11. Severity Tests

Severity is assigned through the documented deterministic rule configuration.

Tests must verify that:

- Each implemented rule produces its configured severity.
- `INFO` events are included in event and severity counts.
- Unknown conditions do not automatically become `CRITICAL`.
- Arbitrary wording changes do not alter severity unless the documented rule explicitly defines that behavior.
- AI output, when introduced, cannot change deterministic severity.
- Severity changes are accompanied by updated tests and documentation.

The canonical severity values are `INFO`, `LOW`, `MEDIUM`, `HIGH` and `CRITICAL`.

## 12. Evidence Tests

For every detected event, verify that:

- At least one valid evidence item exists.
- Source line numbers are valid and one-based.
- Evidence originates from the uploaded input.
- The evidence text preserves the source line or a documented excerpt.
- The event's rule ID matches the rule that generated it.
- Original evidence is not replaced by AI-generated text.
- Missing timestamps or identifiers remain unknown.
- Sensitive values are not unnecessarily reproduced in user-facing descriptions or application logs.

If a rule requires multiple lines, tests must verify all required evidence references.

## 13. Correlation Tests

Correlation tests must verify that:

- Correlations are created only when documented criteria are satisfied.
- At least two distinct valid events are referenced.
- Every related event ID exists in the current analysis.
- Correlation criteria are applied consistently.
- Missing timestamps do not cause fabricated event ordering.
- Missing resource identifiers do not get invented.
- Interface-down followed by BGP-down is not automatically treated as a confirmed causal relationship.
- Correlation confidence follows the documented implementation.
- Duplicate or contradictory correlations are handled consistently.
- `is_confirmed_cause` remains `false` for the initial MVP's simple temporal or resource-based correlations.

Correlation must not change the severity, rule ID or source evidence of the underlying events.

## 14. Recommendation Tests

Recommendations must be:

- Relevant to the detected event.
- Consistent with the documented rule.
- Structured according to `DATA_MODELS.md`.
- Advisory rather than automatically executable.
- Focused on inspection, log review and safe read-only checks.
- Clear about when administrator authorization may be required.
- Free from invented device states and unsupported root-cause claims.

Tests must verify that the application does not execute commands or change network configuration.

## 15. API Tests

Test the four MVP endpoints:

- `GET /health`
- `GET /ready`
- `GET /api/v1/info`
- `POST /api/v1/analyze`

Additional result retrieval, export or history endpoints are out of scope unless separately approved and implemented.

For the analyze endpoint, verify that:

- A valid upload returns the documented success status.
- The response conforms to the canonical `AnalysisResult` schema.
- Event and severity counts match the returned events.
- Evidence line numbers are preserved.
- Unsupported file types return controlled errors.
- Oversized uploads return the documented error.
- Unexpected failures do not reveal stack traces, secrets or sensitive implementation details.
- A valid file with no recognized events returns a valid analysis result with an empty event list.
- Processing warnings use the canonical `AnalysisWarning` structure.
- Analysis status follows the documented completion criteria.
- The response's metadata matches the actual processing configuration.

Tests must not assume permanent result storage or history retrieval. The initial MVP does not require a database.

## 16. Integration Tests

Integration tests must exercise the deterministic analysis workflow:

```text
Synthetic Log
     |
     v
File Validation
     |
     v
Parser
     |
     v
Detection Rules
     |
     v
Event Normalization
     |
     v
Correlation
     |
     v
Recommendations
     |
     v
AnalysisResult
     |
     v
API Response
```

At least one integration test must verify that a known fixture produces the expected final event list, evidence references and summary.

Integration tests should use real application components wherever practical rather than mocking every stage.

The integration suite must also cover a valid file that produces zero detected events and a file that triggers a documented processing limitation.

## 17. Security and Privacy Tests

Security tests must include:

- Unsupported and oversized uploads.
- Path traversal attempts in filenames.
- Malformed text and unexpected characters.
- HTML or script-like content in log lines.
- Prompt-injection-like text embedded in logs.
- Synthetic passwords or tokens.
- Excessively long lines and resource-limit cases.
- Invalid AI output when the optional AI layer is introduced.
- Verification that uploaded content is never executed.
- Verification that the application does not initiate outbound AI requests in local-only mode.
- Verification that temporary files, if used, are cleaned up after processing.
- Verification that uploaded logs and results are not permanently retained by default.

These tests verify specific controls. They do not establish immunity to every security threat.

## 18. Optional AI/RAG Tests — Future Phase

The deterministic MVP test suite must not require AI, embeddings, a vector database or a local model.

When the optional local AI/RAG layer is introduced, add tests for:

- AI disabled.
- Local model unavailable.
- Model timeout or failure.
- Invalid structured output.
- References to nonexistent event IDs.
- Unsupported root-cause claims.
- Prompt injection embedded in logs or retrieved documents.
- Severity changes proposed by AI.
- Excessive input length.
- Missing or invalid evidence citations.
- Retrieval returning irrelevant documents.
- Deterministic fallback after AI failure.

Evaluation should measure evidence-reference accuracy, unsupported-claim frequency, response latency and fallback reliability.

AI explanation quality must be evaluated separately from deterministic detection accuracy.

## 19. Regression Testing

Every fixed defect should have a regression test where practical.

Examples include:

- A false-positive detection.
- Incorrect severity assignment.
- Incorrect source line number.
- Inconsistent event counts.
- A malformed file crashing the parser.
- Duplicate correlations.
- Sensitive information appearing in a report.
- An unsupported file bypassing validation.
- AI output changing deterministic detection results.

The regression suite should run whenever relevant application behavior changes.

## 20. Test Commands

After the project environment, dependencies and corresponding test files are configured, use:

Run the complete test suite:

```bash
python -m pytest
```

Run a specific test file:

```bash
python -m pytest tests/unit/test_log_parser.py
```

Run tests with more detailed output:

```bash
python -m pytest -v
```

These are planned commands. They do not imply that the test files or application code already exist.

## 21. Coverage and Quality Gates

Initial quality goals:

- All six implemented MVP detection rules have positive and negative tests.
- Every MVP API endpoint has tests.
- Core models and file validation have unit tests.
- Critical security boundaries have explicit tests.
- Every fixed defect has a regression test where practical.
- The core test suite passes without a local AI model or paid service.
- Test expectations match the current canonical models and documented ruleset.
- No candidate future rule is represented as an implemented feature.

A numerical code-coverage target may be added once the initial codebase exists. Coverage percentage alone is not proof of correctness.

## 22. Definition of Done

A feature is complete only when:

- Its documented behavior is implemented.
- Relevant unit tests pass.
- Related integration or API tests pass.
- Error cases have been considered.
- Security and privacy implications have been reviewed.
- The documentation matches actual behavior.
- Regression tests are added for fixed defects where practical.
- The feature is not described as complete solely because its code was written.

## 23. Cross-Document Consistency

Testing behavior must remain aligned with:

- `DATA_MODELS.md` — canonical data structures and invariants.
- `API_SPECIFICATION.md` — endpoint contracts, errors, warnings and analysis status.
- `TECH_STACK.md` — implementation technologies and initial rule registry.
- `DETECTION_RULES.md` — rule IDs, matching behavior, severity and correlation requirements.
- `PRD.md` — MVP scope and acceptance criteria.
- `ARCHITECTURE.md` — deterministic processing stages and optional AI/RAG path.
- `AI_DESIGN.md` — future AI/RAG behavior and fallback expectations.
- `IMPLEMENTATION_PLAN.md` — implementation order and completion criteria.

If a conflict exists, resolve it in the appropriate canonical document before changing test expectations or runtime behavior.

## Final Testing Decision

NetAssist AI will use a test-first approach for file validation, canonical data models and deterministic detection rules.

Synthetic fixtures and automated tests will provide repeatable verification without production network access, external AI services or paid tools.

The MVP will prioritize reliable evidence, safe failure handling and consistent API contracts over the number of features implemented. Optional AI/RAG functionality will be tested separately and must not compromise the deterministic analysis path.
