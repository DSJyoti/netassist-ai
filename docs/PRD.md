# NetAssist AI — Product Requirements Document

**Document Status:** Planning  
**Version:** 1.0  
**Product:** NetAssist AI  
**Product Type:** Network Log Analysis & Troubleshooting Assistant  
**Primary Environment:** Local Windows Development  
**Budget:** Free / Open-Source Tools Only

---

## 1. Product Overview

NetAssist AI is an evidence-based network troubleshooting assistant that analyzes network device logs and helps users identify possible networking issues.

The system accepts supplied log files, parses important events, detects known networking problems using deterministic rules, extracts supporting evidence, assigns severity, and provides recommended read-only troubleshooting checks.

The system is designed to help network engineers, students, developers, and IT teams reduce the time required to manually inspect large network logs.

The initial version will work only with user-supplied log files.

NetAssist AI will **not** directly connect to production routers or switches and will **not** automatically execute configuration commands.

---

## 2. Problem Statement

Network devices can generate large amounts of logs.

Examples include:

- Interface up/down events
- Link flaps
- Packet drops
- CRC errors
- Input/output errors
- BGP neighbor changes
- ARP events
- MAC learning events
- VLAN-related events
- Routing events
- Authentication or system events

Manually searching these logs can be slow and error-prone.

A user may need to:

1. Find important events.
2. Determine their severity.
3. Identify the affected interface, IP, VLAN, or neighbor.
4. Understand when the event occurred.
5. Check whether multiple events are related.
6. Decide what should be investigated next.

NetAssist AI aims to automate the first level of this investigation while preserving the original evidence.

---

# 3. Product Goals

## 3.1 Primary Goals

NetAssist AI should:

- Accept network log files.
- Validate uploaded files.
- Parse log lines.
- Detect known networking events.
- Extract useful metadata.
- Preserve exact source evidence.
- Assign severity.
- Group related events.
- Provide troubleshooting recommendations.
- Allow users to search and filter results.
- Generate a structured analysis report.
- Provide JSON output for machine-readable results.
- Provide a browser-based interface.
- Work without an external paid AI service.

---

## 3.2 Secondary Goals

The system should eventually support:

- Multiple network vendors.
- More advanced event correlation.
- Historical analysis.
- Network topology context.
- Optional local LLM analysis.
- RAG-based troubleshooting knowledge.
- Device integrations in read-only mode.
- Automated report generation.
- Multi-user access.
- Authentication.
- Deployment to a server.

These are not required for the first MVP.

---

# 4. Non-Goals

The MVP will NOT:

- Automatically configure routers.
- Automatically configure switches.
- Execute commands on network devices.
- Change routing configuration.
- Change VLAN configuration.
- Restart interfaces.
- Modify firewall rules.
- Automatically remediate production incidents.
- Claim a root cause without evidence.
- Depend on a paid LLM API.
- Require internet access for the core analyzer.
- Store uploaded logs permanently by default.

---

# 5. Target Users

## 5.1 Network Engineer

A network engineer can upload a router or switch log and quickly identify important events.

Example:

```text
Upload router.log
        ↓
NetAssist AI
        ↓
Detected Events
        ↓
Interface Gi0/1 DOWN
BGP Neighbor DOWN
CRC Errors
        ↓
Recommended Checks
```

---

## 5.2 Network Operations / IT Support

Support engineers can use the system as a first-level investigation tool before escalating an incident.

---

## 5.3 Networking Student

Students can upload sample logs and understand what different networking events mean.

---

## 5.4 Developer / AI Engineer

Developers can use the structured analysis output as input for future AI-based explanations or automation.

---

# 6. Core User Journey

The primary user journey is:

```text
Open NetAssist AI
        ↓
Upload Log File
        ↓
File Validation
        ↓
Log Parsing
        ↓
Event Detection
        ↓
Evidence Extraction
        ↓
Severity Assignment
        ↓
Event Correlation
        ↓
Troubleshooting Recommendations
        ↓
Analysis Dashboard
        ↓
Export Report / JSON
```

---

# 7. Functional Requirements

## FR-001 — Log File Upload

The system shall allow the user to upload a supported network log file.

Initial supported formats:

- `.log`
- `.txt`

The system should reject unsupported file types.

---

## FR-002 — File Validation

The system shall validate:

- File extension
- File size
- File readability
- Text encoding
- Empty files
- Malformed input

The initial maximum file size should be configurable.

Initial target:

```text
5 MB
```

The limit may be changed later based on testing.

---

## FR-003 — Log Parsing

The system shall read uploaded log files line by line.

Each log line should preserve:

- Original text
- Line number
- Timestamp if available
- Parsed message
- Source information where available

Example:

```text
Line: 154
Timestamp: 2026-10-15 10:31:22
Message: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

---

## FR-004 — Interface Event Detection

The system shall detect interface state events.

Examples:

```text
Interface changed state to down
Interface changed state to up
Interface administratively down
```

Example detection:

```text
Interface:
GigabitEthernet0/1

State:
DOWN

Evidence:
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

---

## FR-005 — Link Flap Detection

The system shall identify repeated interface state changes when sufficient timestamp information is available.

Example:

```text
10:01:01 Gi0/1 DOWN
10:01:08 Gi0/1 UP
10:01:15 Gi0/1 DOWN
10:01:23 Gi0/1 UP
```

The system may classify this as:

```text
Potential Link Flap
```

The system must preserve the underlying events as evidence.

---

## FR-006 — BGP Event Detection

The system shall detect important BGP neighbor events.

Examples:

```text
BGP neighbor DOWN
BGP neighbor UP
BGP adjacency change
BGP session reset
```

The system should extract:

- Neighbor IP
- State
- Timestamp
- Original evidence

Example:

```text
Neighbor:
192.0.2.2

State:
DOWN

Severity:
HIGH
```

---

## FR-007 — Error Detection

The system should detect supported network error indicators.

Initial examples:

- CRC errors
- Input errors
- Output errors
- Packet drops
- Interface errors

Detection must depend on explicit supported patterns.

The system must not treat an arbitrary occurrence of the word "error" as a confirmed network fault.

---

## FR-008 — VLAN Event Detection

The system should detect supported VLAN-related events.

Examples may include:

- VLAN unavailable
- VLAN mismatch
- VLAN-related interface errors
- Trunk-related events

Only explicitly supported patterns should produce structured alerts.

---

## FR-009 — ARP Event Detection

The system should detect supported ARP-related events.

Examples:

- ARP resolution failure
- Duplicate IP indicators
- ARP-related errors

The exact rule set will be defined in `DETECTION_RULES.md`.

---

## FR-010 — MAC Learning Event Detection

The system should detect supported MAC-related events.

Examples:

- MAC address learned
- MAC address removed
- MAC movement
- MAC flapping indicators

Detection rules must clearly distinguish confirmed events from hypotheses.

---

# 8. Severity Model

Each detected event should receive a severity level.

Initial severity levels:

```text
INFO
LOW
MEDIUM
HIGH
CRITICAL
```

### INFO

Normal informational events.

Example:

```text
Interface changed to UP
```

### LOW

Minor event requiring awareness.

Example:

```text
Single non-critical warning
```

### MEDIUM

Potential operational issue.

Example:

```text
Repeated interface errors
```

### HIGH

Significant network problem.

Example:

```text
BGP neighbor down
```

### CRITICAL

Potentially severe network impact.

Critical classification must require strong evidence and should not be assigned merely because a log contains words such as "error" or "failure".

---

# 9. Evidence Requirements

Every detected event should contain evidence.

Minimum structure:

```text
Rule ID
Event Type
Severity
Timestamp
Affected Resource
Source Line
Evidence
Explanation
Recommended Checks
```

Example:

```json
{
  "rule_id": "IFACE_DOWN_001",
  "event_type": "interface_down",
  "severity": "HIGH",
  "resource": "GigabitEthernet0/1",
  "line_number": 154,
  "evidence": "%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down"
}
```

---

# 10. Fact vs Hypothesis

This is a core product principle.

NetAssist AI must clearly distinguish between:

### Confirmed Fact

Directly supported by the log.

Example:

```text
FACT:
GigabitEthernet0/1 changed state to DOWN at 10:31:22.
```

### Possible Relationship

A relationship suggested by multiple events.

Example:

```text
POSSIBLE RELATIONSHIP:
BGP neighbor 192.0.2.2 went DOWN approximately 2 seconds
after GigabitEthernet0/1 changed state to DOWN.
```

### Root Cause

The system should not claim:

```text
Root cause = interface failure
```

unless sufficient evidence exists.

Instead it should say:

```text
Possible cause:
Interface instability may have contributed to the BGP session loss.
Further verification is required.
```

---

# 11. Troubleshooting Recommendations

Recommendations must be read-only.

Examples:

```text
Check interface status.
Check interface counters.
Check CRC/input errors.
Check physical connectivity.
Check transceiver status.
Check BGP neighbor state.
Check routing table.
Check ARP table.
Check VLAN configuration.
```

The system must NOT automatically execute these commands.

---

# 12. Event Correlation

NetAssist AI should eventually correlate events.

Example:

```text
10:30:01 Interface Gi0/1 DOWN
10:30:03 BGP Neighbor 192.0.2.2 DOWN
```

Possible result:

```text
Correlation:
Possible relationship detected.

Evidence:
Gi0/1 went DOWN 2 seconds before the BGP neighbor went DOWN.

Confidence:
Medium

Verification:
Check whether the BGP neighbor uses Gi0/1 as its transport path.
```

Correlation must never automatically become a confirmed root cause.

---

# 13. Dashboard Requirements

The web interface should provide:

## Summary

Display:

- Total log lines
- Total detected events
- High severity events
- Critical events
- Interfaces affected
- BGP neighbors affected

---

## Event List

Each event should display:

```text
Severity
Event Type
Timestamp
Resource
Short Description
```

---

## Event Details

When a user selects an event, display:

```text
Event
Severity
Timestamp
Affected Resource
Evidence
Source Line
Explanation
Recommended Checks
Rule ID
```

---

## Filtering

Users should be able to filter by:

- Severity
- Event type
- Interface
- IP address
- Timestamp
- Rule ID

---

## Search

The dashboard should provide text search across detected events and evidence.

---

# 14. Report Generation

The system should provide a structured analysis report.

Example:

```text
NetAssist AI Analysis Report

File:
router.log

Analysis Summary:
- 1 interface down event
- 1 BGP neighbor down event
- 4 interface error events

High Severity:
- BGP neighbor 192.0.2.2 DOWN

Potential Correlation:
- Interface Gi0/1 DOWN occurred 2 seconds before
  BGP neighbor DOWN.

Recommended Checks:
1. Verify Gi0/1 operational state.
2. Check interface counters.
3. Verify physical connectivity.
4. Check BGP neighbor state.
```

---

# 15. JSON Output

The analyzer should support machine-readable JSON output.

Example structure:

```json
{
  "analysis_id": "example-id",
  "file_name": "router.log",
  "summary": {
    "total_lines": 500,
    "events_detected": 4
  },
  "events": [],
  "correlations": [],
  "recommendations": []
}
```

The exact API schema will be defined in `API_SPEC.md`.

---

# 16. AI / LLM Requirements

AI functionality is optional in the MVP.

The deterministic analyzer must work without an LLM.

Later, an AI layer may provide:

- Natural-language explanations
- Incident summaries
- Troubleshooting guidance
- Log question answering
- Multi-event reasoning
- RAG-based networking knowledge

Preferred approach:

```text
Deterministic Analyzer
        ↓
Structured Evidence
        ↓
Optional AI Layer
        ↓
Human-readable Explanation
```

The AI layer must not replace the evidence-based detection engine.

---

# 17. Privacy Requirements

Uploaded logs may contain sensitive information.

Therefore:

- Logs should not be sent to external AI services by default.
- Uploaded files should not be permanently stored by default.
- Temporary processing data should be cleaned up.
- Sensitive values should be considered during future redaction design.
- No credentials should be extracted and displayed unnecessarily.
- `.env` files and secrets must never be committed to Git.

---

# 18. Security Requirements

The application must:

- Validate uploaded files.
- Restrict file size.
- Avoid arbitrary command execution.
- Never execute commands contained inside uploaded logs.
- Never execute network-device configuration commands.
- Avoid path traversal.
- Sanitize filenames.
- Handle malformed input safely.
- Keep secrets outside source code.
- Avoid unnecessary persistence of logs.

---

# 19. Performance Requirements

For the MVP, the system should provide responsive analysis for normal-sized log files.

Initial target:

```text
Up to 5 MB log file
```

The system should process logs line-by-line where practical rather than loading unnecessarily large data structures into memory.

Performance will be measured during testing.

---

# 20. Reliability Requirements

The analyzer should:

- Never crash because of one malformed log line.
- Continue processing valid lines when possible.
- Report parsing limitations.
- Preserve evidence for detected events.
- Produce deterministic results for the same input and rule version.

---

# 21. Testing Requirements

Testing must cover:

### Unit Tests

- Parser
- Regex rules
- Event extraction
- Severity assignment
- Correlation logic
- Validation

### Integration Tests

- File upload
- Analysis API
- Report generation
- JSON response

### UI Tests

- Upload workflow
- Event display
- Filtering
- Search

### Negative Tests

- Empty file
- Unsupported file
- Oversized file
- Invalid encoding
- Malformed log
- Missing timestamp
- Unknown event

---

# 22. Acceptance Criteria for MVP

The MVP will be considered complete when:

- A user can start the application locally.
- A user can open the browser UI.
- A user can upload a `.log` or `.txt` file.
- The file is validated.
- The log is parsed.
- Supported network events are detected.
- Evidence is displayed.
- Severity is assigned.
- Recommended checks are shown.
- Users can filter detected events.
- Users can search results.
- JSON output is available.
- Tests cover the core analyzer.
- The application works without a paid LLM.
- No production network device access is required.
- No automatic network configuration is performed.

---

# 23. MVP Success Metrics

Initial success will be measured using:

### Detection Accuracy

Percentage of known test events correctly detected.

### False Positive Rate

Percentage of alerts that are incorrectly generated.

### Evidence Accuracy

Percentage of detected events that contain correct source evidence.

### Parser Reliability

Percentage of valid test logs processed without application failure.

### Reproducibility

Same input + same rule version should produce the same deterministic result.

---

# 24. Future Product Versions

## Version 1.1

Potential additions:

- More Cisco-style patterns
- Additional vendor formats
- Better timestamp parsing
- More interface error rules
- More routing rules

---

## Version 1.2

Potential additions:

- Event correlation engine
- Incident grouping
- Confidence scoring
- Historical analysis
- SQLite-based local history

---

## Version 2.0

Potential additions:

- Optional local LLM
- RAG knowledge base
- Natural-language investigation
- AI-generated incident summaries
- Interactive troubleshooting assistant

---

## Version 3.0

Potential additions:

- Read-only device integrations
- Network topology awareness
- Multi-user access
- Authentication
- Centralized deployment
- Monitoring integrations

---

# 25. Product Safety Principle

NetAssist AI is an **analysis and decision-support system**, not an autonomous network administrator.

The system should help answer:

```text
What happened?
Where did it happen?
When did it happen?
What evidence supports it?
What might be related?
What should I check next?
```

It should not automatically decide:

```text
What configuration should I change?
```

or execute:

```text
configure terminal
interface ...
shutdown
no shutdown
router ...
```

without an explicitly designed and separately approved future workflow.

---

# 26. Product Definition

The final product can be summarized as:

> **NetAssist AI is an evidence-based network troubleshooting assistant that converts raw network logs into structured events, severity-aware alerts, evidence-backed explanations, possible correlations, and safe troubleshooting recommendations.**

The core philosophy is:

```text
Logs
  ↓
Parse
  ↓
Detect
  ↓
Preserve Evidence
  ↓
Classify
  ↓
Correlate Carefully
  ↓
Recommend
  ↓
Explain
```

The deterministic analyzer is the foundation.

AI is an optional enhancement, not the foundation.