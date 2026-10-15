# DATA_MODELS.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.1  
**Status:** Canonical Data Contract  
**Scope:** MVP  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the canonical data models used by NetAssist AI.

All components must follow these definitions:

- Log parsing and normalization
- Deterministic detection rules
- Event correlation
- Recommendation generation
- Analysis summaries
- API request and response schemas
- Automated tests

Other project documents, especially `API_SPECIFICATION.md` and `DETECTION_RULES.md`, must align with this document.

## 2. MVP Design Principles

1. **Deterministic detection:** Detection rules and severity assignments are controlled by Python logic.
2. **Evidence-first results:** Every detected event must retain the source evidence that supports it.
3. **No invented information:** Unknown devices, interfaces, timestamps and resource identifiers must remain unknown rather than being fabricated.
4. **Structured responses:** Warnings and recommendations use structured objects, not plain strings.
5. **Cautious correlation:** Related events may be correlated, but correlation does not prove causality.
6. **Privacy by default:** Uploaded logs and analysis results are not permanently stored in the MVP.
7. **AI separation:** AI is disabled in the initial MVP and must not control detections, evidence or severity.
8. **Consistent serialization:** Enum values are represented using their documented uppercase strings.

## 3. Enumerations

### 3.1 Severity

Every detected event has one severity level.

| Value | Meaning |
|---|---|
| `INFO` | Informational state change or normal operational event |
| `LOW` | Minor issue or low-priority warning |
| `MEDIUM` | Potentially significant issue requiring investigation |
| `HIGH` | Significant issue that should be investigated promptly |
| `CRITICAL` | Severe event requiring immediate attention and verification |

Severity represents the rule-based classification of the observed log evidence. It does not guarantee business impact or confirm a root cause.

### 3.2 AnalysisStatus

| Value | Meaning |
|---|---|
| `COMPLETED` | The analysis completed without a known processing limitation that requires a partial status |
| `PARTIAL` | The analysis produced results, but one or more processing limitations affected coverage |

Failed requests that cannot produce a valid `AnalysisResult` should use the appropriate HTTP error response. They do not need to return an `AnalysisResult` with a `FAILED` status.

### 3.3 Confidence

| Value | Meaning |
|---|---|
| `HIGH` | Strong match against the configured rule and available evidence |
| `MEDIUM` | Reasonable match, but some evidence is missing or ambiguous |
| `LOW` | Limited supporting evidence |
| `UNKNOWN` | Confidence cannot be meaningfully assigned |

Confidence describes the strength of the rule match or correlation evidence. It is not a probability that a proposed root cause is correct.

### 3.4 WarningSeverity

| Value | Meaning |
|---|---|
| `INFO` | Informational processing message |
| `WARNING` | A limitation or condition the user should review |
| `ERROR` | A processing problem affecting part of the analysis |

A warning does not automatically mean the entire analysis failed.

### 3.5 RecommendationActionType

| Value | Meaning |
|---|---|
| `READ_ONLY_CHECK` | Inspect configuration or status without making changes |
| `VISUAL_INSPECTION` | Visually inspect the relevant device, cable, port or physical connection |
| `LOG_REVIEW` | Review related logs or additional evidence |
| `ESCALATION` | Escalate the issue to an authorized network administrator or support team |

### 3.6 RecommendationRiskLevel

| Value | Meaning |
|---|---|
| `LOW` | Low-risk investigation or observation |
| `MEDIUM` | Investigation requiring additional care or access |
| `HIGH` | Potentially sensitive action requiring expert review or authorization |

The MVP provides troubleshooting guidance only. It does not execute recommendations or change network configurations.

---

## 4. Evidence Model

**Purpose:** Preserve the original log evidence behind an event or correlation.

| Field | Type | Required | Description |
|---|---|---|---|
| `line_number` | Integer | Yes | One-based line number in the uploaded file |
| `text` | String | Yes | Original relevant log line or evidence excerpt |
| `timestamp` | Datetime or null | No | Timestamp extracted from the evidence, when available |
| `source_file` | String or null | No | Source filename or source identifier, when available |

### Rules

- Line numbers start at `1`.
- Evidence must come from the uploaded input.
- A timestamp must not be invented when one cannot be extracted.
- Preserve enough original text to allow the user to verify the match.
- Evidence should be represented as a list because a single event may be supported by multiple log lines.
- Evidence excerpts should not silently replace the original source with an AI-generated explanation.

---

## 5. Recommendation Model

**Purpose:** Provide structured, reviewable troubleshooting guidance.

| Field | Type | Required | Description |
|---|---|---|---|
| `recommendation_id` | String | Yes | Stable identifier for the recommendation type |
| `title` | String | Yes | Short recommendation heading |
| `description` | String | Yes | Explanation of the suggested investigation |
| `action_type` | RecommendationActionType | Yes | Category of recommended action |
| `risk_level` | RecommendationRiskLevel | Yes | Relative risk of the suggested action |
| `requires_authorization` | Boolean | Yes | Whether authorization is needed before carrying out the suggested action |

### Rules

- Recommendations must be related to the detected event and its evidence.
- Prefer read-only checks and additional evidence collection.
- Configuration changes, restarts, resets and other potentially disruptive actions must not be executed by the MVP.
- If a potentially disruptive action is mentioned as future guidance, it must be clearly distinguished from an immediately safe check and must require appropriate authorization.
- Recommendations are suggestions, not proof of a root cause.

---

## 6. NetworkEvent Model

**Purpose:** Represent one normalized event detected by a deterministic rule.

| Field | Type | Required | Description |
|---|---|---|---|
| `event_id` | String | Yes | Unique identifier within the current analysis |
| `rule_id` | String | Yes | Stable ID of the rule that produced the event |
| `event_type` | String | Yes | Machine-readable event category |
| `title` | String | Yes | Human-readable event title |
| `severity` | Severity | Yes | Rule-assigned severity |
| `timestamp` | Datetime or null | No | Parsed event timestamp, when available |
| `device` | String or null | No | Device identifier extracted from the logs |
| `interface` | String or null | No | Interface identifier extracted from the logs |
| `resource_type` | String or null | No | Resource category, such as `interface` or `bgp_neighbor` |
| `resource_id` | String or null | No | Identifier of the affected resource, when known |
| `description` | String | Yes | Factual description of the observed event |
| `evidence` | List of Evidence | Yes | Source evidence supporting the detection |
| `possible_causes` | List of strings | Yes | Potential explanations, when supported by the rule |
| `recommendations` | List of Recommendation | Yes | Structured troubleshooting guidance |
| `confidence` | Confidence | Yes | Strength of the rule match |
| `tags` | List of strings | Yes | Searchable labels for categorization |

### Rules

1. `event_id` must be unique within one `AnalysisResult`.
2. `rule_id` must match a rule defined in `DETECTION_RULES.md`.
3. Rule IDs must remain stable across releases unless a documented rule change requires otherwise.
4. `event_type` describes the observed event, not an unverified root cause.
5. Unknown values must be represented as `null`, not as fabricated values.
6. Use an empty list when there are no known possible causes, recommendations or tags.
7. Every detected event must include supporting evidence.
8. Event severity must come from deterministic rule logic.
9. A rule match must not be presented as proof that a particular device or component caused another event.
10. `resource_type` and `resource_id` are optional because not every log line identifies a specific resource.

### Event ID convention

Event IDs need only be unique within a single analysis. They do not imply permanent storage or a globally persistent event identity.

A practical implementation may generate identifiers such as `evt_001`, `evt_002` and so on during a request.

---

## 7. Correlation Model

**Purpose:** Group events that may be related by timing, shared resources or rule-defined relationships.

| Field | Type | Required | Description |
|---|---|---|---|
| `correlation_id` | String | Yes | Unique identifier within the current analysis |
| `correlation_type` | String | Yes | Category of relationship between events |
| `related_event_ids` | List of strings | Yes | IDs of events included in the correlation |
| `description` | String | Yes | Explanation of the observed relationship |
| `evidence` | List of Evidence | Yes | Evidence supporting the relationship |
| `confidence` | Confidence | Yes | Strength of the correlation evidence |
| `is_confirmed_cause` | Boolean | Yes | Whether causality has been independently confirmed |

### Correlation rules

- Every `related_event_id` must refer to an event in the same `AnalysisResult`.
- A correlation must contain at least two distinct event IDs.
- Correlation may use timestamps, device identifiers, interface identifiers or explicit rule-defined relationships.
- Missing timestamps or identifiers must not be fabricated to create a correlation.
- The system must not assume that two events are related merely because they appear close together in a log.
- `is_confirmed_cause` defaults to `false`.
- The MVP must not set `is_confirmed_cause` to `true` solely because a correlation rule matched.

**Important:** Temporal association and correlation do not establish causality.

---

## 8. AnalysisSummary Model

**Purpose:** Provide a consistent overview of processing results and detected events.

| Field | Type | Required | Description |
|---|---|---|---|
| `total_lines` | Integer | Yes | Number of input lines considered by the parser |
| `parsed_lines` | Integer | Yes | Number of lines successfully parsed according to the parser's defined criteria |
| `unparsed_lines` | Integer | Yes | Number of lines that did not match supported parsing patterns |
| `total_events` | Integer | Yes | Total number of detected events |
| `info_events` | Integer | Yes | Number of INFO events |
| `low_events` | Integer | Yes | Number of LOW events |
| `medium_events` | Integer | Yes | Number of MEDIUM events |
| `high_events` | Integer | Yes | Number of HIGH events |
| `critical_events` | Integer | Yes | Number of CRITICAL events |
| `total_correlations` | Integer | Yes | Number of correlations produced |

### Summary invariants

The implementation and tests must enforce these conditions:

**Line counts**

`total_lines = parsed_lines + unparsed_lines`

**Event counts**

`total_events = info_events + low_events + medium_events + high_events + critical_events`

**Correlation count**

`total_correlations = len(correlations)`

All counts must be non-negative integers.

A parsed line does not necessarily produce a detected event. An unparsed line does not automatically indicate a network fault; it means the parser could not interpret that line according to the supported patterns.

If parsing coverage is incomplete or another processing limitation affects results, the analysis status and warnings must communicate that limitation.

---

## 9. AnalysisWarning Model

**Purpose:** Explain processing limitations and conditions that the user should know about.

| Field | Type | Required | Description |
|---|---|---|---|
| `code` | String | Yes | Stable machine-readable warning code |
| `message` | String | Yes | Human-readable explanation |
| `severity` | WarningSeverity | Yes | Importance of the warning |
| `line_number` | Integer or null | No | Related input line, when applicable |

### Rules

- Warning codes should be stable and documented.
- Warnings must use structured objects, not plain strings.
- `line_number` must be `null` when a warning is not tied to a particular source line.
- Warnings should describe limitations such as unsupported log formats, partial parsing or skipped content.
- Warnings must not claim that an event occurred unless the detection rules support that claim.

Example warning codes may include `UNSUPPORTED_LOG_FORMAT`, `PARTIAL_PARSE` and `EMPTY_INPUT`. These codes must be implemented and documented before being treated as active application behavior.

---

## 10. ProcessingMetadata Model

**Purpose:** Record technical information about the current analysis request.

| Field | Type | Required | Description |
|---|---|---|---|
| `parser_version` | String | Yes | Version of the parser used |
| `ruleset_version` | String | Yes | Version of the detection rules |
| `processing_duration_ms` | Non-negative number | Yes | Processing duration in milliseconds |
| `ai_used` | Boolean | Yes | Whether AI was actually used for this analysis |
| `input_size_bytes` | Non-negative integer | Yes | Size of the uploaded input in bytes |

### Rules

- The initial MVP must report `ai_used = false`.
- Parser and ruleset versions must correspond to the versions used for that request.
- `processing_duration_ms` must be measured by the application, not invented.
- `input_size_bytes` must reflect the validated uploaded content size.
- Metadata is part of the response and does not imply that the uploaded file or analysis is permanently stored.

---

## 11. AnalysisResult Model

**Purpose:** Define the canonical response returned by `POST /api/v1/analyze`.

| Field | Type | Required | Description |
|---|---|---|---|
| `analysis_id` | String | Yes | Identifier for the current analysis response |
| `status` | AnalysisStatus | Yes | `COMPLETED` or `PARTIAL` |
| `filename` | String | Yes | Display filename of the uploaded file |
| `created_at` | Datetime | Yes | Time the analysis result was created |
| `summary` | AnalysisSummary | Yes | Line, event and correlation counts |
| `events` | List of NetworkEvent | Yes | Detected events |
| `correlations` | List of Correlation | Yes | Correlations between detected events |
| `warnings` | List of AnalysisWarning | Yes | Structured processing warnings |
| `metadata` | ProcessingMetadata | Yes | Parser, ruleset and processing information |

### Rules

1. `analysis_id` identifies a response; it does not imply a persistent record.
2. `created_at` must be timezone-aware. The application should generate it using UTC.
3. `filename` is display metadata only. It must not be used as a server filesystem path.
4. `events`, `correlations` and `warnings` must always be represented as arrays, even when empty.
5. `summary` counts must match the events and correlations included in the response.
6. `metadata.ai_used` must be `false` in the initial MVP.
7. The response must not imply that an analysis can be retrieved later.
8. No database or analysis-history endpoint is required for the MVP.
9. Failed requests that cannot produce valid analysis results should return an appropriate API error response instead.

---

## 12. Canonical Pydantic Model Reference

The following reference implementation illustrates how the data contract can be represented using **Python 3.12 and Pydantic v2**. It is intended to align with the field definitions above.

```python
from datetime import datetime
from enum import Enum
from typing import Optional

from pydantic import BaseModel, ConfigDict, Field, field_validator


class ContractModel(BaseModel):
    model_config = ConfigDict(extra="forbid")


class Severity(str, Enum):
    INFO = "INFO"
    LOW = "LOW"
    MEDIUM = "MEDIUM"
    HIGH = "HIGH"
    CRITICAL = "CRITICAL"


class AnalysisStatus(str, Enum):
    COMPLETED = "COMPLETED"
    PARTIAL = "PARTIAL"


class Confidence(str, Enum):
    HIGH = "HIGH"
    MEDIUM = "MEDIUM"
    LOW = "LOW"
    UNKNOWN = "UNKNOWN"


class WarningSeverity(str, Enum):
    INFO = "INFO"
    WARNING = "WARNING"
    ERROR = "ERROR"


class RecommendationActionType(str, Enum):
    READ_ONLY_CHECK = "READ_ONLY_CHECK"
    VISUAL_INSPECTION = "VISUAL_INSPECTION"
    LOG_REVIEW = "LOG_REVIEW"
    ESCALATION = "ESCALATION"


class RecommendationRiskLevel(str, Enum):
    LOW = "LOW"
    MEDIUM = "MEDIUM"
    HIGH = "HIGH"


class Evidence(ContractModel):
    line_number: int = Field(ge=1)
    text: str = Field(min_length=1)
    timestamp: Optional[datetime] = None
    source_file: Optional[str] = None


class Recommendation(ContractModel):
    recommendation_id: str = Field(min_length=1)
    title: str = Field(min_length=1)
    description: str = Field(min_length=1)
    action_type: RecommendationActionType
    risk_level: RecommendationRiskLevel
    requires_authorization: bool


class NetworkEvent(ContractModel):
    event_id: str = Field(min_length=1)
    rule_id: str = Field(min_length=1)
    event_type: str = Field(min_length=1)
    title: str = Field(min_length=1)
    severity: Severity
    timestamp: Optional[datetime] = None
    device: Optional[str] = None
    interface: Optional[str] = None
    resource_type: Optional[str] = None
    resource_id: Optional[str] = None
    description: str = Field(min_length=1)
    evidence: list[Evidence] = Field(min_length=1)
    possible_causes: list[str] = Field(default_factory=list)
    recommendations: list[Recommendation] = Field(default_factory=list)
    confidence: Confidence
    tags: list[str] = Field(default_factory=list)


class Correlation(ContractModel):
    correlation_id: str = Field(min_length=1)
    correlation_type: str = Field(min_length=1)
    related_event_ids: list[str] = Field(min_length=2)
    description: str = Field(min_length=1)
    evidence: list[Evidence] = Field(default_factory=list)
    confidence: Confidence
    is_confirmed_cause: bool = False

    @field_validator("related_event_ids")
    @classmethod
    def require_distinct_event_ids(
        cls, value: list[str]
    ) -> list[str]:
        if len(value) != len(set(value)):
            raise ValueError(
                "related_event_ids must contain distinct IDs"
            )
        return value


class AnalysisSummary(ContractModel):
    total_lines: int = Field(ge=0)
    parsed_lines: int = Field(ge=0)
    unparsed_lines: int = Field(ge=0)
    total_events: int = Field(ge=0)
    info_events: int = Field(ge=0)
    low_events: int = Field(ge=0)
    medium_events: int = Field(ge=0)
    high_events: int = Field(ge=0)
    critical_events: int = Field(ge=0)
    total_correlations: int = Field(ge=0)


class AnalysisWarning(ContractModel):
    code: str = Field(min_length=1)
    message: str = Field(min_length=1)
    severity: WarningSeverity
    line_number: Optional[int] = Field(default=None, ge=1)


class ProcessingMetadata(ContractModel):
    parser_version: str = Field(min_length=1)
    ruleset_version: str = Field(min_length=1)
    processing_duration_ms: float = Field(ge=0)
    ai_used: bool
    input_size_bytes: int = Field(ge=0)


class AnalysisResult(ContractModel):
    analysis_id: str = Field(min_length=1)
    status: AnalysisStatus
    filename: str = Field(min_length=1)
    created_at: datetime
    summary: AnalysisSummary
    events: list[NetworkEvent] = Field(default_factory=list)
    correlations: list[Correlation] = Field(default_factory=list)
    warnings: list[AnalysisWarning] = Field(default_factory=list)
    metadata: ProcessingMetadata

    @field_validator("created_at")
    @classmethod
    def require_timezone_aware_created_at(
        cls, value: datetime
    ) -> datetime:
        if value.tzinfo is None or value.utcoffset() is None:
            raise ValueError(
                "created_at must be timezone-aware"
            )
        return value
```

**Implementation note:** This reference enforces important field constraints, but the application must also validate cross-field invariants, including summary counts, event ID uniqueness, valid correlation references and consistency between the status and processing warnings. Those checks cannot all be guaranteed by individual field types alone.

The reference models should be placed in the project's agreed data-model module during implementation. Do not create a second competing set of model definitions in the API or detection-rule modules.

---

## 13. Example Analysis Response

The following is an illustrative response showing the required structure. Its values are examples, not claims about an actual network analysis.

```json
{
  "analysis_id": "analysis_example_001",
  "status": "COMPLETED",
  "filename": "sample.log",
  "created_at": "2026-10-15T10:00:00Z",
  "summary": {
    "total_lines": 1,
    "parsed_lines": 1,
    "unparsed_lines": 0,
    "total_events": 1,
    "info_events": 0,
    "low_events": 0,
    "medium_events": 0,
    "high_events": 1,
    "critical_events": 0,
    "total_correlations": 0
  },
  "events": [
    {
      "event_id": "evt_001",
      "rule_id": "IFACE_DOWN_001",
      "event_type": "INTERFACE_DOWN",
      "title": "Interface down",
      "severity": "HIGH",
      "timestamp": null,
      "device": null,
      "interface": "Gi0/1",
      "resource_type": "interface",
      "resource_id": "Gi0/1",
      "description": "The log reports an interface down event.",
      "evidence": [
        {
          "line_number": 1,
          "text": "Interface Gi0/1 changed state to down",
          "timestamp": null,
          "source_file": "sample.log"
        }
      ],
      "possible_causes": [],
      "recommendations": [
        {
          "recommendation_id": "CHECK_INTERFACE_STATUS",
          "title": "Verify interface status",
          "description": "Review the current interface status and related logs.",
          "action_type": "READ_ONLY_CHECK",
          "risk_level": "LOW",
          "requires_authorization": false
        }
      ],
      "confidence": "HIGH",
      "tags": [
        "interface",
        "connectivity"
      ]
    }
  ],
  "correlations": [],
  "warnings": [],
  "metadata": {
    "parser_version": "0.1.0",
    "ruleset_version": "0.1.0",
    "processing_duration_ms": 12.5,
    "ai_used": false,
    "input_size_bytes": 46
  }
}
```

Before using this example as a test fixture, ensure that the sample line exactly matches the implemented parser and that the rule's configured severity is consistent with `DETECTION_RULES.md`.

---

## 14. Privacy and Storage

For the initial MVP:

- Uploaded logs are processed for the current request.
- The API returns an `AnalysisResult` directly.
- The application does not create a permanent analysis-history database.
- No retrieval endpoint for previous analyses is provided.
- No saved-analysis export endpoint is provided.
- Temporary upload files must be cleaned up after processing, with cleanup attempted on failure paths.
- Logs and errors should avoid unnecessarily recording sensitive uploaded content.
- The analysis ID is not a substitute for persistent storage.

Any future persistence feature must be separately designed and documented, including retention, access control and deletion behavior.

---

## 15. AI Compatibility

The initial MVP uses deterministic Python parsing and detection rules.

If local AI support is introduced in a later phase:

- AI may explain existing structured events and evidence.
- AI must not invent evidence or source line numbers.
- AI must not change deterministic event severity.
- AI must not silently add detections that did not match the approved detection pipeline.
- AI-generated explanations must be distinguishable from verified log observations.
- `metadata.ai_used` must reflect whether AI was actually used in the request.

AI support must not change the canonical response structure without a documented versioned change.

---

## 16. Cross-Document Contract

| Document | Required alignment |
|---|---|
| `API_SPECIFICATION.md` | Use `AnalysisResult` with structured events, correlations, warnings and metadata |
| `DETECTION_RULES.md` | Map every rule output to `NetworkEvent`, including stable `rule_id`, severity, evidence and confidence |
| `ARCHITECTURE.md` | Keep parsing, detection, correlation and response construction as separate responsibilities |
| `SECURITY_AND_PRIVACY.md` | Maintain the no-permanent-storage MVP policy and temporary-file cleanup |
| `TESTING_STRATEGY.md` | Test field validation, severity counts, evidence references, correlation references and response structure |
| `IMPLEMENTATION_PLAN.md` | Implement the canonical models before integrating detection rules and API response handling |

---

## 17. Definition of Done

This document is ready to serve as the canonical contract when:

- [ ] All severity levels, including `INFO`, are supported consistently.
- [ ] Warnings and recommendations use structured models.
- [ ] Detection-rule output maps to `NetworkEvent`.
- [ ] Event and correlation references remain valid within one response.
- [ ] Summary counts match the returned data.
- [ ] `created_at` is timezone-aware.
- [ ] The MVP reports `ai_used = false`.
- [ ] No documentation implies permanent analysis storage or history retrieval.
- [ ] API examples and implementation tests follow these definitions.

**Final decision:** This document is the single source of truth for NetAssist AI's MVP data contract. Other documents must align with it rather than redefining the same models independently.
