# NetAssist AI — Testing Strategy

**Document:** Testing Strategy  
**Project:** NetAssist AI  
**Version:** 1.0  
**Status:** Design Phase  
**Testing Framework:** Pytest  
**API Testing:** FastAPI TestClient / HTTPX

---

## 1. Purpose

This document defines the testing strategy for NetAssist AI.

Testing will verify that the application safely parses uploaded network logs, detects supported events, preserves evidence, generates appropriate recommendations, and returns consistent API responses.

Testing is required throughout development rather than only before release.

The initial MVP must be testable without a paid API, an installed AI model, or access to production network devices.

## 2. Testing Objectives

The test suite must verify that:

1. Supported log files are processed correctly.
2. Detection rules identify expected events.
3. Unrelated log lines do not produce unsupported detections.
4. Every detected event includes traceable evidence.
5. Severity values match the documented rules.
6. Event correlations do not automatically imply causality.
7. Recommendations remain safe and advisory.
8. Invalid uploads are rejected appropriately.
9. API responses follow the documented schemas.
10. The application works when AI is disabled.
11. Sensitive information is handled carefully.
12. Changes do not silently break previously working features.

## 3. Testing Principles

- **Repeatable:** The same input and ruleset should produce the same deterministic results.
- **Isolated:** Each test should verify a specific behavior.
- **Independent:** Tests should not require internet access or production systems.
- **Safe:** Use synthetic logs rather than real customer or production logs.
- **Evidence-based:** Expected detections must specify the source lines supporting them.
- **Fast:** Unit tests should run quickly during development.
- **Maintainable:** Detection tests should be easy to extend as rules are added.
- **AI-independent:** Core tests must not require an AI model.

## 4. Testing Levels

| Test level | What it verifies | Example |
|---|---|---|
| Unit tests | Individual functions and rules | Detect an interface-down event |
| Component tests | Connected backend components | Parser output enters the detection engine |
| API tests | HTTP endpoints and responses | Upload a valid log file |
| Integration tests | End-to-end backend workflow | Upload, parse, detect, and return events |
| Security tests | Unsafe inputs and privacy boundaries | Reject an oversized file |
| Regression tests | Previously fixed issues remain fixed | A corrected BGP rule still works |
| Acceptance tests | MVP requirements are satisfied | Complete analysis without AI |

## 5. Test Tools

The initial test stack will use:

- **pytest:** Test execution and assertions.
- **FastAPI TestClient:** API endpoint testing.
- **HTTPX:** HTTP support used by the testing stack.
- **Python standard library:** Temporary files, text fixtures, and basic utilities.

No paid testing platform is required.

Dependencies should be installed in the project's virtual environment.

## 6. Test Directory Structure

Proposed structure:

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
  crc_errors.log
  bgp_neighbor_down.log
  link_flap.log
  clean_log.log
  malformed.log
```

This is the target structure. Files will be created as the corresponding components are implemented.

## 7. Synthetic Test Data

All initial test fixtures must be synthetic.

Do not use real production logs containing customer data, credentials, or private infrastructure details.

Each fixture should have a documented purpose and expected result.

### Example: `interface_down.log`

```text
2026-10-15T10:14:00Z switch-01 INFO System started
2026-10-15T10:15:00Z switch-01 WARN Interface GigabitEthernet1/0/1 changed state to down
2026-10-15T10:16:00Z switch-01 INFO Monitoring continues
```

Expected behavior:

- The parser processes the lines it supports.
- The detection engine identifies the interface-down event.
- The event references source line 2.
- The event severity matches the documented rule.
- The unrelated lines do not produce an interface-down event.
- The system does not claim to know the physical root cause from this evidence alone.

The exact expected severity and parser behavior must match the finalized rule specification and implementation.

## 8. File Validation Tests

Test the following cases:

| Scenario | Expected result |
|---|---|
| Valid `.log` file | Accepted |
| Valid `.txt` file | Accepted |
| Unsupported extension | Rejected |
| Empty file | Controlled error |
| File larger than configured limit | Rejected |
| Missing upload field | Validation error |
| Invalid text encoding | Controlled error or documented partial result |
| Filename contains path traversal characters | No path traversal |
| Very long line | Handled within resource limits |
| Binary content disguised as text | Rejected or safely handled |

Tests must verify that rejected uploads do not trigger the detection pipeline unnecessarily.

## 9. Parser Tests

The parser must be tested against:

- Supported timestamp formats.
- Missing timestamps.
- Different log levels.
- Device names and interface identifiers.
- Unexpected whitespace.
- Empty lines.
- Malformed records.
- Unrecognized message formats.
- Very long lines.
- Unicode characters.
- Multiple records with the same timestamp.

Expected results must be explicit for each fixture.

The parser must not invent missing timestamps, device names, or interface names.

## 10. Detection Rule Tests

Each implemented rule must have:

1. At least one positive test.
2. At least one negative test.
3. An expected severity.
4. An expected rule ID.
5. Expected evidence line references.
6. A safe recommendation check.
7. Edge cases appropriate to the rule.

### Initial detection matrix

| Rule | Positive test | Negative or edge test |
|---|---|---|
| `INTERFACE_DOWN` | Interface changes to down | Similar text unrelated to interface state |
| `INTERFACE_UP` | Interface changes to up | Interface remains down |
| `LINK_FLAP` | Repeated state changes within a configured window | A single state change |
| `CRC_ERRORS` | CRC error counter increases | No CRC-related evidence |
| `BGP_NEIGHBOR_DOWN` | Neighbor-down message | Neighbor is established |
| `BGP_NEIGHBOR_UP` | Neighbor-established message | No state transition |
| `ARP_FAILURE` | Recognized ARP failure message | Ordinary ARP activity |
| `DUPLICATE_IP` | Explicit duplicate-IP evidence | A single IP address mention |
| `VLAN_MISMATCH` | Explicit mismatch evidence | VLAN mentioned without mismatch |
| `MAC_FLAPPING` | Recognized MAC movement pattern | Normal MAC learning |

Rules must only be tested as implemented. A planned rule must not be counted as passing before it exists.

## 11. Severity Tests

Severity is determined by the documented deterministic rules.

Tests must verify that:

- Each rule produces its configured severity.
- AI output cannot change severity.
- Unknown conditions do not automatically become critical.
- Severity does not depend on arbitrary changes in message wording unless the rule explicitly defines that behavior.
- Any severity change is accompanied by updated tests and documentation.

## 12. Evidence Tests

For every detected event, verify that:

- At least one evidence item exists.
- The line number is valid.
- The evidence originates from the input.
- The event's rule ID matches the rule that detected it.
- The original event evidence is not replaced by AI-generated text.
- Sensitive values are not unnecessarily reproduced in user-facing output.

If a rule requires multiple lines, the test should verify all required evidence references.

## 13. Correlation Tests

Correlation tests must verify:

- Events are linked only when documented criteria are satisfied.
- Related event IDs exist in the same analysis.
- Temporal windows are applied consistently.
- Missing timestamps do not cause fabricated ordering.
- An interface-down event followed by a BGP-down event is not automatically labeled a confirmed causal relationship.
- Correlation confidence is assigned according to documented logic.
- Repeated events do not create duplicate or contradictory correlations without justification.

## 14. Recommendation Tests

Recommendations must be:

- Relevant to the detected event.
- Consistent with the documented rule.
- Advisory rather than automatically executable.
- Focused on read-only checks and safe inspection in the MVP.
- Clear about when administrator authorization may be required.
- Free from invented device states or unsupported root-cause claims.

Tests must confirm that the application does not execute recommendations.

## 15. API Tests

Test the following endpoints and behavior:

- `GET /health`
- `GET /ready`
- `GET /api/v1/info`
- `POST /api/v1/analyze`

Additional result retrieval and export endpoints will be tested only if implemented.

For the analyze endpoint, verify:

- A valid upload returns the expected success status.
- The response follows the `AnalysisResult` schema.
- Event counts match the returned event list.
- Severity counts add up to the total event count.
- Evidence line numbers are preserved.
- Unsupported file types return controlled errors.
- Oversized uploads return the documented error.
- Unexpected failures do not reveal stack traces or secrets.
- A valid file with no recognized events returns a valid empty analysis result.

## 16. Integration Tests

Integration tests must exercise the complete deterministic workflow:

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

At least one test must verify that a known fixture produces the expected final event list and summary.

Integration tests should use the real application components rather than mocking every stage.

## 17. Security Tests

Security tests must include:

- Unsupported and oversized uploads.
- Path traversal attempts in filenames.
- Malformed text and unexpected characters.
- HTML or script-like content in log lines.
- Prompt-injection-like text in log content.
- Recognizable synthetic passwords or tokens.
- Excessively long lines and resource-limit cases.
- Invalid AI output when the AI layer is implemented.
- Verification that uploaded content is not executed.
- Verification that analysis does not initiate outbound AI requests in local-only mode.

These tests verify specific controls; they do not establish that the application is immune to every security threat.

## 18. AI Tests — Future Phase

The deterministic MVP test suite must not require AI.

When the optional AI layer is introduced, add tests for:

- AI disabled.
- Model unavailable.
- Model timeout.
- Invalid structured output.
- References to nonexistent event IDs.
- Unsupported root-cause claims.
- Prompt injection embedded in logs.
- Severity changes proposed by AI.
- Excessive input length.
- Deterministic fallback after AI failure.

Evaluation should measure evidence-reference accuracy, unsupported-claim frequency, latency, and fallback reliability.

AI quality must be evaluated separately from deterministic detection accuracy.

## 19. Regression Testing

Every fixed bug should include a regression test.

Examples:

- A false-positive detection.
- Incorrect severity assignment.
- Wrong source line number.
- Inconsistent event counts.
- A malformed file crashing the parser.
- Duplicate correlations.
- Sensitive information appearing in a report.

The regression suite should run whenever code affecting the relevant behavior changes.

## 20. Test Commands

After the project environment and dependencies are configured, the expected command will be:

```bash
python -m pytest
```

Run a specific test file with:

```bash
python -m pytest tests/unit/test_log_parser.py
```

Run tests with more detailed output:

```bash
python -m pytest -v
```

These commands should be used only after the corresponding files and test dependencies exist.

## 21. Coverage and Quality Gates

Initial quality goals:

- All implemented detection rules have positive and negative tests.
- All API endpoints in the MVP have tests.
- Core models and validation have unit tests.
- Critical security boundaries have explicit tests.
- Every fixed defect has a regression test where practical.
- The core test suite passes without an AI model or paid service.

A numerical code-coverage target may be added once the initial codebase exists. Coverage percentage alone is not proof of correctness.

## 22. Definition of Done

A feature is complete only when:

- Its documented behavior is implemented.
- Relevant unit tests pass.
- Related integration or API tests pass.
- Error cases have been considered.
- Security implications have been reviewed.
- The documentation matches actual behavior.
- No feature is described as complete solely because its code was written.

## 23. Final Testing Decision

NetAssist AI will use a test-first approach for file validation, data models, and deterministic detection rules.

Synthetic fixtures and automated tests will provide repeatable verification without requiring production network access, external AI services, or paid tools.

The project will prioritize reliable evidence and safe failure behavior over the number of features implemented.
