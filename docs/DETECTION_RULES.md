# DETECTION_RULES.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.1  
**Status:** MVP Detection Contract  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the deterministic rules used to detect network-related events from uploaded log files.

The detection engine converts supported log lines into normalized `NetworkEvent` objects. Each event must follow the canonical model in `DATA_MODELS.md` and the response contract in `API_SPECIFICATION.md`.

The MVP prioritizes transparent, evidence-backed rules over speculative diagnosis.

## 2. Scope

### In scope

- Interface state changes
- BGP neighbor state changes
- Explicit device or service error messages when supported by a documented rule
- Evidence extraction and source line preservation
- Deterministic severity assignment
- Safe troubleshooting recommendations
- Limited event correlation when sufficient evidence exists

### Out of scope

- Automatic router or switch access
- Executing shell commands
- Changing network configuration
- Confirming root cause from a single log line
- Predicting future outages
- Claiming that unrecognized logs indicate a fault
- AI-generated detections in the initial MVP

## 3. Detection Principles

1. **Deterministic:** The same input and ruleset must produce consistent detections.
2. **Evidence-backed:** Every event must include at least one source evidence item.
3. **Stable rule IDs:** Each rule has a documented, stable identifier.
4. **No invented fields:** Unknown devices, timestamps, interfaces and resource identifiers remain `null`.
5. **Conservative interpretation:** Detect what the log states, not what the system guesses happened.
6. **Explicit severity:** Each rule specifies its severity.
7. **Traceability:** Every event's `rule_id` identifies the rule that generated it.
8. **Safe recommendations:** Recommendations prioritize inspection and read-only checks.
9. **Correlation is not causation:** Related events must not be presented as confirmed root causes.
10. **Unknown input remains unknown:** Unsupported lines are not forced into a detection category.

---

## 4. Severity Policy

| Severity | General policy | Example |
|---|---|---|
| `INFO` | Informational state transition or normal event | Interface becomes operational |
| `LOW` | Minor issue with limited evidence of impact | A documented low-impact warning |
| `MEDIUM` | An issue that warrants investigation | Repeated recoverable error messages |
| `HIGH` | A significant state change or likely service disruption | Interface goes down |
| `CRITICAL` | An explicitly documented severe condition that warrants urgent attention | A rule-defined severe failure with strong evidence |

Severity is assigned by the rule configuration. It is not a measured probability, confirmed business impact or proof of root cause.

A severity must not be raised solely because multiple events appear close together in time.

## 5. Canonical Event Output

Every successful rule match must produce a `NetworkEvent` conforming to `DATA_MODELS.md`.

| Field | Rule-engine requirement |
|---|---|
| `event_id` | Unique within the current analysis |
| `rule_id` | Exact stable ID of the matched rule |
| `event_type` | Canonical machine-readable event category |
| `title` | Human-readable event title |
| `severity` | Configured enum value |
| `timestamp` | Parsed timestamp or `null` |
| `device` | Extracted device identifier or `null` |
| `interface` | Extracted interface identifier or `null` |
| `resource_type` | Resource category or `null` |
| `resource_id` | Resource identifier or `null` |
| `description` | Factual description of the matched evidence |
| `evidence` | One or more structured evidence objects |
| `possible_causes` | Supported potential explanations or an empty list |
| `recommendations` | Structured recommendation objects |
| `confidence` | Rule-match confidence |
| `tags` | Searchable event labels |

The event must not include undocumented fields as substitutes for these canonical fields.

## 6. Evidence Extraction

For each matching log line, the parser should preserve:

- Original one-based line number
- Relevant original text
- Extracted timestamp, if available
- Source filename or source identifier, if available

Example:

```json
{
  "line_number": 12,
  "text": "Interface Gi0/1 changed state to down",
  "timestamp": null,
  "source_file": "sample.log"
}
```

This example is illustrative. The actual evidence text must exactly match the uploaded source line or a clearly documented excerpt.

If a timestamp cannot be parsed, use `null`. Do not generate a timestamp from the current time and present it as the event time.

## 7. Rule Registry

The following rules define the initial detection candidates. They become active only after the corresponding patterns are implemented and tested.

| Rule ID | Event type | Severity | Purpose |
|---|---|---|---|
| `IFACE_DOWN_001` | `INTERFACE_DOWN` | `HIGH` | Detect an explicit interface-down transition |
| `IFACE_UP_001` | `INTERFACE_UP` | `INFO` | Detect an explicit interface-up transition |
| `BGP_NEIGHBOR_DOWN_001` | `BGP_NEIGHBOR_DOWN` | `HIGH` | Detect an explicit BGP neighbor-down transition |
| `BGP_NEIGHBOR_UP_001` | `BGP_NEIGHBOR_UP` | `INFO` | Detect an explicit BGP neighbor-up transition |
| `DEVICE_ERROR_001` | `DEVICE_ERROR` | `MEDIUM` | Detect an explicitly documented device error pattern |
| `AUTH_FAILURE_001` | `AUTHENTICATION_FAILURE` | `MEDIUM` | Detect a supported authentication-failure pattern |

**Important:** The table defines the intended rule contract. It does not prove that the patterns are already implemented. Do not mark a rule as operational until matching, extraction and tests have been completed.

Severity values may be adjusted only through a documented ruleset change supported by the evidence and test cases.

---

## 8. Rule: Interface Down

**Rule ID:** `IFACE_DOWN_001`

**Event type:** `INTERFACE_DOWN`

**Default severity:** `HIGH`

### Detection intent

Detect an explicit log statement indicating that a network interface transitioned to a down state.

### Input requirements

- The line must match an implemented, documented interface-down pattern.
- The match must represent a state transition or explicit down-state message.
- The parser should extract the interface identifier when available.

### Normalized fields

- `resource_type`: `interface` when the interface is identified
- `resource_id`: extracted interface identifier, otherwise `null`
- `interface`: extracted interface identifier, otherwise `null`
- `device`: extracted device identifier, otherwise `null`
- `timestamp`: parsed timestamp, otherwise `null`
- `severity`: `HIGH`

### Possible causes

Potential causes may include physical connectivity problems, administrative shutdown, configuration changes or a remote condition. These are hypotheses only and must not be presented as confirmed causes without additional evidence.

### Recommended actions

1. Review the interface's current status using an authorized read-only method.
2. Inspect related interface logs around the event time.
3. Check whether a maintenance or configuration change was expected.
4. Escalate to the responsible network administrator if the interface remains down or the impact is unclear.

Recommendations must not cause the application to execute device commands.

### False-positive controls

- Do not match a line that merely mentions an interface being down as part of an unrelated explanation unless the pattern explicitly supports that interpretation.
- Avoid duplicate events for the same line and rule.
- Preserve the exact evidence line.

---

## 9. Rule: Interface Up

**Rule ID:** `IFACE_UP_001`

**Event type:** `INTERFACE_UP`

**Default severity:** `INFO`

### Detection intent

Detect an explicit log statement indicating that a network interface transitioned to an up state.

### Normalized fields

- `resource_type`: `interface` when known
- `resource_id`: extracted interface identifier, otherwise `null`
- `interface`: extracted interface identifier, otherwise `null`
- `severity`: `INFO`

### Recommended actions

No action is required solely because an interface became operational. If the interface recently experienced an outage, review adjacent logs to understand the sequence of events.

### False-positive controls

Do not classify a general mention of an interface as an up event unless the documented pattern confirms the state transition.

---

## 10. Rule: BGP Neighbor Down

**Rule ID:** `BGP_NEIGHBOR_DOWN_001`

**Event type:** `BGP_NEIGHBOR_DOWN`

**Default severity:** `HIGH`

### Detection intent

Detect an explicit log statement indicating that a BGP neighbor or session transitioned to a down state.

### Normalized fields

- `resource_type`: `bgp_neighbor` when supported by the parser
- `resource_id`: extracted neighbor identifier, otherwise `null`
- `device`: extracted device identifier, otherwise `null`
- `severity`: `HIGH`

Do not populate the `interface` field with a BGP neighbor identifier. These are different resource types.

### Possible causes

Potential explanations include connectivity loss, peer-side changes, session configuration issues or a routing-process problem. A matching log line alone does not establish which explanation is correct.

### Recommended actions

1. Review the current BGP session state using an authorized read-only method.
2. Inspect nearby routing and connectivity logs.
3. Compare timestamps with other relevant events, if timestamps are available.
4. Escalate persistent session failures to the network administrator.

### False-positive controls

- Require an explicit down-state pattern.
- Do not interpret every mention of BGP as a failed session.
- Do not infer a specific neighbor identifier if it is absent from the log.

---

## 11. Rule: BGP Neighbor Up

**Rule ID:** `BGP_NEIGHBOR_UP_001`

**Event type:** `BGP_NEIGHBOR_UP`

**Default severity:** `INFO`

### Detection intent

Detect an explicit log statement indicating that a BGP neighbor or session transitioned to an up or established state.

### Normalized fields

- `resource_type`: `bgp_neighbor` when known
- `resource_id`: extracted neighbor identifier, otherwise `null`
- `severity`: `INFO`

### Recommended actions

No action is required solely because the session became established. If a preceding down event exists, the correlation engine may report the observed sequence when its correlation criteria are met.

An up event does not prove that every routing function or downstream service is healthy.

---

## 12. Rule: Device Error

**Rule ID:** `DEVICE_ERROR_001`

**Event type:** `DEVICE_ERROR`

**Default severity:** `MEDIUM`

### Detection intent

Detect a documented error pattern emitted by a supported network device or network service.

### Requirements

- The matching error pattern must be explicitly listed in the implemented ruleset.
- The rule must distinguish actual error messages from generic uses of the word `error`.
- The event description must reflect the observed message.
- Device and resource identifiers must be extracted only when supported by the input.

### Severity policy

`MEDIUM` is the default for this initial rule. If specific error patterns have different documented severity levels, implement them as separate rules or explicitly documented rule variants rather than inferring severity from arbitrary text.

### Recommended actions

1. Review the full message and adjacent log lines.
2. Identify the affected component if the evidence provides that information.
3. Check for related events using available timestamps or identifiers.
4. Escalate repeated or persistent errors to the responsible administrator.

### False-positive controls

Do not classify every line containing `error`, `failed` or a similar word as a network-device error without a documented pattern.

---

## 13. Rule: Authentication Failure

**Rule ID:** `AUTH_FAILURE_001`

**Event type:** `AUTHENTICATION_FAILURE`

**Default severity:** `MEDIUM`

### Detection intent

Detect a supported log pattern explicitly indicating an authentication failure.

### Requirements

- The line must match an implemented authentication-failure pattern.
- A generic mention of a user, login or authentication process is insufficient.
- Usernames, addresses and other identifiers must not be fabricated.
- Sensitive identifiers should not be unnecessarily repeated in descriptions or application logs.

### Recommended actions

1. Review the relevant authentication logs.
2. Check whether the attempt was expected and whether repeated failures occurred.
3. Follow the organization's established security escalation process if suspicious activity is indicated.

The event alone does not prove malicious intent or a successful compromise.

### False-positive controls

Do not equate a failed authentication attempt with a confirmed security incident.

---

## 14. Confidence Assignment

Confidence describes how well the observed line matches a documented rule and its required evidence.

| Confidence | Guidance |
|---|---|
| `HIGH` | The line clearly matches a supported pattern and required fields are adequately supported |
| `MEDIUM` | The line matches, but optional context or some identifiers are missing |
| `LOW` | The match is ambiguous but is still accepted by a deliberately conservative rule |
| `UNKNOWN` | The implementation cannot meaningfully assign a confidence level |

The initial MVP must not represent confidence as a calibrated probability unless a separate, validated probabilistic method is introduced.

Confidence must not automatically determine severity. Severity and confidence describe different things.

---

## 15. Duplicate Detection

The detection engine should avoid creating duplicate events for the same input line and rule.

A duplicate check may use:

- Source line number
- Rule ID
- Normalized resource identifier, when available

Two different lines containing similar messages may represent separate events and should not be removed merely because their text is similar.

Any deduplication logic must preserve traceability to the source evidence.

---

## 16. Event Correlation Rules

Correlation is a separate stage after event detection. It must not change the underlying event's severity or evidence.

### Correlation prerequisites

- At least two distinct detected events are required.
- Every related event ID must exist in the current analysis response.
- The relationship must be supported by explicit correlation criteria.
- Missing timestamps or resource identifiers must not be invented.

### Initial candidate correlations

| Candidate | Required evidence | Limitation |
|---|---|---|
| Interface down followed by interface up | Same known interface and usable event ordering | Indicates a state sequence, not the cause of the outage |
| BGP neighbor down followed by BGP neighbor up | Same known neighbor and usable event ordering | Indicates a session-state sequence, not why the session failed |
| Multiple errors on the same known resource | Matching resource identifier and documented relationship | Similar messages alone do not prove a shared cause |

These are candidate behaviors. They must be implemented and tested before being described as active capabilities.

### Correlation output

Every correlation must follow the `Correlation` model in `DATA_MODELS.md`:

- `correlation_id`
- `correlation_type`
- `related_event_ids`
- `description`
- `evidence`
- `confidence`
- `is_confirmed_cause`

For the initial MVP, `is_confirmed_cause` must remain `false` unless causality is independently established through an explicitly implemented and validated mechanism. Simple temporal or resource-based correlation is not sufficient.

---

## 17. Unsupported and Unrecognized Logs

A line that matches no supported parser or detection rule must not be forced into an event category.

The parser should preserve enough information to report parsing coverage, including the source line number where appropriate.

When processing limitations affect coverage:

- Reflect the limitation through structured `AnalysisWarning` objects.
- Use `PARTIAL` status when the limitation meets the documented partial-analysis criteria.
- Keep `unparsed_lines` consistent with the actual parser result.
- Do not treat an unrecognized line as evidence that the network is faulty.

A completed analysis with zero detected events is valid.

---

## 18. Rule Versioning

The ruleset must have an explicit version reported through `ProcessingMetadata.ruleset_version`.

Changes that should trigger a documented ruleset update include:

- Adding or removing a rule
- Changing a rule's matching criteria
- Changing severity
- Changing resource extraction
- Changing confidence assignment
- Changing duplicate detection or correlation behavior

Tests should identify the ruleset version against which expected behavior was validated.

Rule IDs should remain stable when a rule's identity remains the same. If a rule's meaning changes materially, document the change and assess whether a new rule ID is appropriate.

---

## 19. Testing Requirements

Each implemented rule must have tests for:

- A valid positive match
- A non-matching line
- Missing optional identifiers
- Missing or unparseable timestamps
- Preservation of original source line numbers
- Correct rule ID and event type
- Correct severity
- Structured evidence
- Structured recommendations
- Duplicate behavior
- Relevant false-positive cases

Correlation tests must verify:

- At least two distinct valid event IDs
- No references to nonexistent events
- No correlation when required identifiers or ordering are unavailable
- No unsupported causality claims

Summary tests must verify that the event counts match the final list of detected events, including `INFO` events.

---

## 20. Definition of Done

A rule may be marked implemented only when:

- [ ] Its matching criteria are documented.
- [ ] Its stable rule ID is used by the implementation.
- [ ] Its event output follows `DATA_MODELS.md`.
- [ ] Severity is deterministic.
- [ ] Evidence is preserved with source line numbers.
- [ ] Recommendations use the structured recommendation model.
- [ ] Positive and negative tests pass.
- [ ] Known false-positive cases are considered.
- [ ] Its version and implementation status are documented.

A candidate rule listed in this document is not automatically an implemented or validated rule.

**Final decision:** The detection engine produces evidence-backed `NetworkEvent` objects. The API returns these objects through `AnalysisResult`. Correlation and recommendations add investigative context but must not turn uncertain observations into confirmed causes.
