# DETECTION_RULES.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.2
**Status:** MVP Detection Contract  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the deterministic detection rules for identifying supported network-related events in uploaded log files.

The detection engine converts supported log lines into normalized `NetworkEvent` objects. Every generated event must conform to the canonical data model in `DATA_MODELS.md` and the API response contract in `API_SPECIFICATION.md`.

The MVP prioritizes deterministic, transparent, evidence-backed detection over speculative diagnosis.

This document defines the intended rule behavior. A documented rule is not considered implemented or validated until its code and tests are complete.

## 2. Scope

### In scope

- Interface state transitions
- BGP neighbor or session state transitions
- Explicit device or service error messages covered by documented patterns
- Supported authentication-failure messages
- Evidence extraction and source line preservation
- Deterministic severity and confidence assignment
- Structured, safe troubleshooting recommendations
- Limited, evidence-based event correlation
- Reporting parsing limitations and unrecognized input

### Out of scope

- Automatic router or switch access
- Executing shell commands or network-device commands
- Changing network configuration
- Confirming root cause from a single log line
- Predicting future outages
- Treating unrecognized log lines as confirmed faults
- AI-generated detections in the initial MVP
- Claiming that a log pattern proves malicious intent or business impact

## 3. Detection Principles

1. **Determinism:** Identical input processed with the same parser and ruleset must produce consistent detection decisions and ordering.
2. **Evidence-backed events:** Every generated event must contain at least one valid source evidence item.
3. **Stable rule IDs:** Every event must identify the exact rule that generated it.
4. **No invented information:** Unknown timestamps, devices, interfaces and resource identifiers remain `null`.
5. **Conservative interpretation:** Detect what a supported pattern establishes, not what the system assumes.
6. **Explicit severity:** Every rule defines its default severity.
7. **Traceability:** Each event must be traceable to its original source line and rule.
8. **Safe recommendations:** Recommendations prioritize inspection, log review and authorized read-only checks.
9. **Correlation is not causation:** Related events must not be presented as confirmed root causes.
10. **Unknown input remains unknown:** Unsupported lines must not be forced into a detection category.
11. **Versioned behavior:** Changes to matching criteria, extraction, severity or correlation must be documented and tested.
12. **No unsupported claims:** Detection output must distinguish observed facts from possible explanations.

## 4. Severity Policy

| Severity | General policy | Example |
|---|---|---|
| `INFO` | Informational or normal state transition | Interface becomes operational |
| `LOW` | Documented issue with limited apparent impact | A specifically supported low-impact warning |
| `MEDIUM` | Issue that warrants investigation | A supported device error |
| `HIGH` | Significant state change or potential service disruption | Interface or BGP session goes down |
| `CRITICAL` | Explicitly documented severe condition requiring urgent attention | A separately validated rule for a severe failure |

The initial six rules use the severities defined in the rule registry below.

`CRITICAL` is part of the canonical severity enum, but no initial rule is assigned this severity by default.

Severity is a deterministic rule classification. It is not a measured probability, confirmed business impact or proof of root cause.

Severity must not be raised merely because multiple events occur close together in time. Any severity change requires a documented ruleset update and corresponding tests.

## 5. Canonical Event Output

Every successful rule match must produce a `NetworkEvent` conforming to `DATA_MODELS.md`.

| Field | Requirement |
|---|---|
| `event_id` | Unique within the current analysis |
| `rule_id` | Exact stable ID of the matched rule |
| `event_type` | Canonical machine-readable event category |
| `title` | Human-readable event title |
| `severity` | Valid canonical severity enum |
| `timestamp` | Parsed event timestamp or `null` |
| `device` | Extracted device identifier or `null` |
| `interface` | Extracted interface identifier or `null` |
| `resource_type` | Supported resource category or `null` |
| `resource_id` | Extracted resource identifier or `null` |
| `description` | Factual description of the observed evidence |
| `evidence` | One or more structured evidence objects |
| `possible_causes` | Supported hypotheses or an empty list |
| `recommendations` | Structured recommendation objects |
| `confidence` | Valid canonical confidence value |
| `tags` | Searchable event labels |

The implementation must use the canonical field names and types. It must not introduce undocumented alternatives to represent the same information.

Optional fields must remain `null` or use the canonical model's documented default when evidence is unavailable. Do not use fabricated placeholders such as `"unknown-device"` as if they were extracted identifiers.

## 6. Evidence Extraction

For each matching source line, preserve the following information when available:

- Original one-based line number
- Original source text, or a clearly documented excerpt
- Parsed timestamp, if available
- Source filename or identifier, if available

Example:

```json
{
  "line_number": 12,
  "text": "Interface Gi0/1 changed state to down",
  "timestamp": null,
  "source_file": "sample.log"
}
```

This example is illustrative. The actual evidence must come from the uploaded input.

Requirements:

- Line numbers must refer to the original uploaded file, not a filtered or renumbered list.
- The evidence text must preserve the original line accurately.
- A timestamp must not be fabricated from the current time.
- If a timestamp cannot be parsed, use `null`.
- If a device, interface or resource identifier cannot be extracted reliably, use `null`.
- Evidence must remain associated with the event generated from it.
- Multiple events generated from one source line must retain traceability to that same line.

Sensitive log content must not be copied unnecessarily into recommendations, error messages or application logs.

## 7. Rule Registry

The following six rules define the initial MVP detection candidates.

| Rule ID | Event type | Default severity | Purpose |
|---|---|---|---|
| `IFACE_DOWN_001` | `INTERFACE_DOWN` | `HIGH` | Detect an explicit interface-down transition |
| `IFACE_UP_001` | `INTERFACE_UP` | `INFO` | Detect an explicit interface-up transition |
| `BGP_NEIGHBOR_DOWN_001` | `BGP_NEIGHBOR_DOWN` | `HIGH` | Detect an explicit BGP neighbor-down transition |
| `BGP_NEIGHBOR_UP_001` | `BGP_NEIGHBOR_UP` | `INFO` | Detect an explicit BGP neighbor-up transition |
| `DEVICE_ERROR_001` | `DEVICE_ERROR` | `MEDIUM` | Detect a documented device-error pattern |
| `AUTH_FAILURE_001` | `AUTHENTICATION_FAILURE` | `MEDIUM` | Detect a supported authentication-failure pattern |

These rules become operational only after their patterns, extraction behavior, normalized output and tests have been implemented.

The rule registry must be the implementation's source of truth for stable rule IDs and their configured behavior. Rule registration order and event processing order must be deterministic.

Severity changes or changes to the meaning of a rule require a documented ruleset update.

## 8. Rule: Interface Down

**Rule ID:** `IFACE_DOWN_001`
**Event type:** `INTERFACE_DOWN`
**Default severity:** `HIGH`

### Detection intent

Detect an explicit log statement indicating that a network interface transitioned to a down state.

### Matching requirements

- The line must match an implemented and documented interface-down pattern.
- The pattern must indicate a down-state transition or an explicitly supported down-state message.
- A generic mention of an interface or the word `down` is insufficient.
- The interface identifier should be extracted when the input supports reliable extraction.

### Normalized fields

- `resource_type`: `interface` when supported by the matched pattern
- `resource_id`: extracted interface identifier or `null`
- `interface`: extracted interface identifier or `null`
- `device`: extracted device identifier or `null`
- `timestamp`: parsed timestamp or `null`
- `severity`: `HIGH`

### Possible causes

Potential explanations may include physical connectivity problems, administrative shutdown, configuration changes or a remote condition.

These are hypotheses only. They must not be presented as confirmed causes based on the matching line alone.

### Recommendations

Use structured recommendation objects conforming to `DATA_MODELS.md`.

Suitable recommendations include:

1. Review the interface's current status through an authorized read-only method.
2. Inspect related interface logs around the event time.
3. Check whether maintenance or a configuration change was expected.
4. Escalate to the responsible network administrator if the interface remains down or the impact is unclear.

The application must not execute device commands.

### False-positive controls

- Do not match a generic mention of an interface being down unless the documented pattern supports that interpretation.
- Avoid duplicate events for the same source line and rule.
- Preserve the original source evidence.
- Do not infer device or interface identifiers that are absent from the input.

## 9. Rule: Interface Up

**Rule ID:** `IFACE_UP_001`
**Event type:** `INTERFACE_UP`
**Default severity:** `INFO`

### Detection intent

Detect an explicit log statement indicating that a network interface transitioned to an up state.

### Matching requirements

- The line must match a documented interface-up pattern.
- A general mention of an interface is insufficient.
- The implementation must distinguish an actual up-state transition from unrelated text.

### Normalized fields

- `resource_type`: `interface` when supported
- `resource_id`: extracted interface identifier or `null`
- `interface`: extracted interface identifier or `null`
- `device`: extracted device identifier or `null`
- `timestamp`: parsed timestamp or `null`
- `severity`: `INFO`

### Recommendations

No action is required solely because an interface became operational.

If a preceding outage event exists, reviewing adjacent logs may help explain the observed sequence.

### False-positive controls

Do not classify a general mention of an interface as an up event unless the documented pattern confirms the transition.

## 10. Rule: BGP Neighbor Down

**Rule ID:** `BGP_NEIGHBOR_DOWN_001`
**Event type:** `BGP_NEIGHBOR_DOWN`
**Default severity:** `HIGH`

### Detection intent

Detect an explicit log statement indicating that a BGP neighbor or session transitioned to a down state.

### Matching requirements

- The line must match a documented BGP-down pattern.
- The pattern must represent an explicit session-state transition or supported down-state message.
- A general mention of BGP is insufficient.
- The neighbor identifier should be extracted only when the input supports reliable extraction.

### Normalized fields

- `resource_type`: `bgp_neighbor` when supported
- `resource_id`: extracted neighbor identifier or `null`
- `device`: extracted device identifier or `null`
- `timestamp`: parsed timestamp or `null`
- `severity`: `HIGH`

Do not populate the `interface` field with a BGP neighbor identifier. These are different resource types.

### Possible causes

Potential explanations include connectivity loss, peer-side changes, session configuration issues or a routing-process problem.

A matching log line alone does not establish which explanation is correct.

### Recommendations

1. Review the current BGP session state through an authorized read-only method.
2. Inspect nearby routing and connectivity logs.
3. Compare timestamps with related events when usable timestamps are available.
4. Escalate persistent session failures to the responsible network administrator.

### False-positive controls

- Require an explicit down-state pattern.
- Do not interpret every mention of BGP as a failed session.
- Do not infer a neighbor identifier when it is absent.
- Preserve the original evidence line.

## 11. Rule: BGP Neighbor Up

**Rule ID:** `BGP_NEIGHBOR_UP_001`
**Event type:** `BGP_NEIGHBOR_UP`
**Default severity:** `INFO`

### Detection intent

Detect an explicit log statement indicating that a BGP neighbor or session transitioned to an up or established state.

### Matching requirements

- The line must match a documented BGP-up pattern.
- The pattern must distinguish an established state from a generic mention of BGP.
- The neighbor identifier should be extracted when reliably available.

### Normalized fields

- `resource_type`: `bgp_neighbor` when supported
- `resource_id`: extracted neighbor identifier or `null`
- `device`: extracted device identifier or `null`
- `timestamp`: parsed timestamp or `null`
- `severity`: `INFO`

### Recommendations

No action is required solely because the session became established.

If a preceding down event exists, the correlation engine may report the observed sequence when its correlation criteria are satisfied.

An up event does not prove that all routing functions or downstream services are healthy.

## 12. Rule: Device Error

**Rule ID:** `DEVICE_ERROR_001`
**Event type:** `DEVICE_ERROR`
**Default severity:** `MEDIUM`

### Detection intent

Detect a documented error pattern emitted by a supported network device or network service.

### Matching requirements

- The matching pattern must be explicitly registered in the implemented ruleset.
- The rule must distinguish actual error messages from generic uses of the word `error`.
- The event description must reflect the observed message.
- Device and resource identifiers must be extracted only when supported by the input.

### Severity policy

`MEDIUM` is the default severity for this initial rule.

If individual error patterns require different severities, they must use separately documented rules or explicitly documented variants. Severity must not be inferred from arbitrary text.

### Recommendations

1. Review the full message and adjacent log lines.
2. Identify the affected component when the evidence provides that information.
3. Review related events using available timestamps or resource identifiers.
4. Escalate repeated or persistent errors to the responsible administrator.

### False-positive controls

Do not classify every line containing `error`, `failed` or similar wording as a device error without a documented matching pattern.

## 13. Rule: Authentication Failure

**Rule ID:** `AUTH_FAILURE_001`
**Event type:** `AUTHENTICATION_FAILURE`
**Default severity:** `MEDIUM`

### Detection intent

Detect a supported log pattern explicitly indicating an authentication failure.

### Matching requirements

- The line must match an implemented authentication-failure pattern.
- A generic mention of a user, login or authentication process is insufficient.
- Usernames, addresses and other identifiers must not be fabricated.
- Sensitive identifiers must not be unnecessarily repeated in descriptions, recommendations or application logs.

### Recommendations

1. Review the relevant authentication logs.
2. Check whether the attempt was expected and whether repeated failures occurred.
3. Follow the organization's established security escalation process if suspicious activity is indicated.

The event alone does not prove malicious intent, successful compromise or unauthorized access.

### False-positive controls

Do not equate a failed authentication attempt with a confirmed security incident.

## 14. Confidence Assignment

Confidence describes how strongly the observed evidence satisfies a documented rule's matching criteria.

| Confidence | Guidance |
|---|---|
| `HIGH` | The source line clearly satisfies the supported pattern and its required evidence criteria |
| `MEDIUM` | The rule matches, but optional context or some identifiers are missing |
| `LOW` | A deliberately documented rule accepts an ambiguous match under explicit, tested conditions |
| `UNKNOWN` | The implementation cannot meaningfully assign a confidence level |

The initial MVP must not represent confidence as a calibrated probability unless a separate, validated probabilistic method is introduced.

Confidence must not automatically determine severity. They describe different properties.

Confidence assignment must be deterministic and consistent with the canonical enum in `DATA_MODELS.md`. An ambiguous line must not be accepted merely because the implementation can assign it a low confidence value.

## 15. Duplicate Detection and Event IDs

The detection engine must avoid creating duplicate events for the same source line and rule.

A duplicate check may use:

- Source line number
- Rule ID
- Normalized resource identifier, when available

The same line may legitimately match different rules if each match independently satisfies the documented criteria.

Different source lines containing similar messages may represent separate events. They must not be removed solely because their text is similar.

Requirements:

- Duplicate detection must be deterministic.
- Source evidence must remain traceable after deduplication.
- Event IDs must be unique within the current analysis.
- Event IDs must not be used as evidence of causal relationships.
- Event ordering must be deterministic for identical input and ruleset versions.

The event-ID generation method must conform to `DATA_MODELS.md`. If IDs are generated randomly or uniquely per request, deterministic detection refers to the resulting detection decisions and ordering, not necessarily identical IDs across separate analyses.

## 16. Event Correlation Rules

Correlation is a separate stage after event detection. It must not modify the underlying events' severity, rule IDs or source evidence.

### Correlation prerequisites

- At least two distinct detected events are required.
- Every referenced event ID must exist in the current analysis.
- The relationship must satisfy explicitly documented correlation criteria.
- Missing timestamps or resource identifiers must not be invented.
- Event ordering must not be inferred from file position as if it were event time unless the correlation rule explicitly supports that limitation.
- Correlation must not be treated as proof of root cause.

### Initial candidate correlations

| Candidate | Required evidence | Limitation |
|---|---|---|
| Interface down followed by interface up | Same known interface and usable event ordering | Indicates an observed state sequence, not the cause of the outage |
| BGP neighbor down followed by BGP neighbor up | Same known neighbor and usable event ordering | Indicates a session-state sequence, not why the session failed |
| Multiple errors on the same known resource | Matching resource identifier and a documented relationship | Similar messages alone do not prove a shared cause |

These are candidate behaviors. They must be implemented and tested before being described as active capabilities.

### Correlation output

Every correlation must follow the `Correlation` model in `DATA_MODELS.md` and include its canonical fields:

- `correlation_id`
- `correlation_type`
- `related_event_ids`
- `description`
- `evidence`
- `confidence`
- `is_confirmed_cause`

For the initial MVP, `is_confirmed_cause` must remain `false`. Temporal proximity, matching resource identifiers or sequential state transitions are not sufficient to establish causality.

Correlation descriptions must state the observed relationship without overstating what the evidence proves.

## 17. Unsupported and Unrecognized Logs

A line that matches no supported parser or detection rule must not be forced into an event category.

The parser should preserve enough information to report parsing coverage, including original source line numbers where appropriate.

When processing limitations affect coverage:

- Report relevant limitations through structured `AnalysisWarning` objects.
- Use `PARTIAL` status only when the criteria in `DATA_MODELS.md` and `API_SPECIFICATION.md` are met.
- Keep `unparsed_lines` consistent with the parser's actual output and the canonical summary definition.
- Do not treat an unrecognized line as evidence that the network is faulty.
- Do not report an analysis as fully successful if a material processing failure prevented the documented analysis from completing.

A completed analysis with zero detected events is valid when processing completed according to the documented criteria.

The implementation must distinguish between an unrecognized line, a supported line with no detected event and a processing error.

## 18. Rule Versioning

The ruleset must have an explicit version reported through `ProcessingMetadata.ruleset_version`.

Changes that require a documented ruleset update include:

- Adding or removing a rule
- Changing matching criteria
- Changing default severity
- Changing resource extraction
- Changing confidence assignment
- Changing duplicate detection
- Changing event ordering
- Changing correlation behavior

Tests must validate expected behavior against the ruleset being implemented.

Rule IDs should remain stable when a rule's identity and meaning remain the same. If a rule's meaning changes materially, document the change and assess whether a new rule ID is appropriate.

The ruleset version must identify the actual rule configuration used for an analysis. It must not imply that a candidate rule has been implemented or tested when it has not.

## 19. Testing Requirements

Each implemented rule must have tests covering:

- A valid positive match
- A non-matching line
- Missing optional identifiers
- Missing or unparseable timestamps
- Preservation of original source line numbers and text
- Correct rule ID and event type
- Correct severity
- Valid structured evidence
- Valid structured recommendations
- Deterministic confidence assignment
- Duplicate behavior
- Relevant false-positive cases
- Stable event ordering for identical inputs

Correlation tests must verify:

- At least two distinct valid event IDs
- No references to nonexistent events
- No duplicate event IDs within the correlation's related-event list
- No correlation when required identifiers or ordering are unavailable
- Correct behavior when timestamps are missing
- No unsupported causality claims
- No unintended changes to the underlying events

Summary and analysis tests must verify:

- Severity counts match the final event list, including `INFO` events.
- The total event count is consistent with the canonical summary definition.
- `unparsed_lines` reflects actual parser results.
- Analysis status follows the documented completion criteria.
- Warnings use the canonical `AnalysisWarning` structure.
- The reported ruleset version matches the ruleset used.

Tests must not rely solely on mocked success responses. Each rule must have focused unit tests, followed by integration tests through the analysis pipeline.

## 20. Definition of Done

A rule may be marked implemented only when:

- [ ] Its matching criteria are documented.
- [ ] Its stable rule ID is registered and used by the implementation.
- [ ] Its event output follows `DATA_MODELS.md`.
- [ ] Severity and confidence assignment are deterministic.
- [ ] Evidence is preserved with original source line numbers.
- [ ] Recommendations use the structured recommendation model.
- [ ] Positive and negative tests pass.
- [ ] Missing-field and timestamp cases are tested.
- [ ] Known false-positive cases are considered.
- [ ] Duplicate behavior is tested.
- [ ] The rule's version and implementation status are documented.

A candidate rule listed in this document is not automatically an implemented or validated rule.

## 21. Cross-Document Consistency

The implementation must remain aligned with these documents:

- `DATA_MODELS.md` — canonical event, evidence, recommendation, correlation, warning and summary structures
- `API_SPECIFICATION.md` — API response contract, processing status and warning behavior
- `TECH_STACK.md` — implementation technologies and initial rule registry
- `PRD.md` — MVP scope and supported capabilities
- `ARCHITECTURE.md` — processing stages and deterministic-first design
- `AI_DESIGN.md` — optional future AI/RAG behavior, which must not override deterministic detections
- `TESTING_STRATEGY.md` — unit, integration and end-to-end validation

If a required field or behavior conflicts across documents, resolve the conflict in the canonical document first rather than silently changing the runtime contract.

## Final Decision

The initial detection engine produces evidence-backed `NetworkEvent` objects using documented deterministic rules. The API returns these events through the canonical `AnalysisResult` structure.

Correlation and recommendations provide additional investigative context; they must not turn uncertain observations into confirmed causes.

The six initial rules remain candidates until their implementation and tests are complete. Unsupported input remains unclassified, processing limitations are reported explicitly, and the deterministic analysis path must remain independent of optional AI/RAG functionality.
