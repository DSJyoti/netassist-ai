# DATA_MODELS.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.2
**Status:** Canonical Data Contract  
**Scope:** MVP  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the canonical data models used by NetAssist AI.

All application components must follow these definitions:

- Log parsing and normalization
- Deterministic detection rules
- Event correlation
- Recommendation generation
- Analysis summaries
- API request and response schemas
- Automated tests

`DATA_MODELS.md` is the single source of truth for the MVP data contract. Other documents, especially `API_SPECIFICATION.md`, `DETECTION_RULES.md`, and `TESTING_STRATEGY.md`, must align with this document.

The reference code in this document is a contract specification. It does not imply that the application models have already been implemented or tested.

## 2. MVP Design Principles

1. **Deterministic detection:** Detection rules and severity assignments are controlled by Python logic.
2. **Evidence-first results:** Every detected event must retain source evidence that supports the detection.
3. **No invented information:** Unknown devices, interfaces, timestamps, and resource identifiers must remain unknown rather than being fabricated.
4. **Structured responses:** Warnings and recommendations use structured objects, not plain strings.
5. **Cautious correlation:** Related events may be correlated, but correlation alone does not prove causality.
6. **Privacy by default:** Uploaded logs and analysis results are not permanently stored by the MVP.
7. **AI separation:** AI is disabled in the initial MVP and must not control detections, evidence, or severity.
8. **Consistent serialization:** Enum values use their documented uppercase strings.
9. **Contract consistency:** Summary counts, identifiers, references, and processing status must agree with the data returned in the same response.
10. **Fail safely:** Invalid input or failed processing must not produce a misleading successful result.

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

Severity represents the deterministic classification of observed log evidence. It does not guarantee business impact or confirm a root cause.

### 3.2 AnalysisStatus

| Value | Meaning |
|---|---|
| `COMPLETED` | Analysis completed without a known material processing limitation |
| `PARTIAL` | Analysis produced results, but a processing limitation affected coverage or completeness |

A warning does not automatically require `PARTIAL` status. The application must determine whether the limitation materially affects analysis coverage.

Requests that cannot produce a valid `AnalysisResult` must return an appropriate HTTP error response. A `FAILED` status is not part of the MVP `AnalysisStatus` enum.

### 3.3 Confidence

| Value | Meaning |
|---|---|
| `HIGH` | Strong match against the configured rule and available evidence |
| `MEDIUM` | Reasonable match, but some evidence is missing or ambiguous |
| `LOW` | Limited supporting evidence |
| `UNKNOWN` | Confidence cannot be meaningfully assigned |

Confidence describes the strength of a rule match or correlation evidence. It is not a probability that a proposed root cause is correct.

### 3.4 WarningSeverity

| Value | Meaning |
|---|---|
| `INFO` | Informational processing message |
| `WARNING` | A limitation or condition the user should review |
| `ERROR` | A processing problem affecting part of the analysis |

A warning does not automatically mean the entire analysis failed. If processing cannot produce a valid result, the application should return an appropriate API error instead of disguising a failed request as a successful analysis.

### 3.5 RecommendationActionType

| Value | Meaning |
|---|---|
| `READ_ONLY_CHECK` | Inspect configuration or status without making changes |
| `VISUAL_INSPECTION` | Visually inspect the relevant device, cable, port, or physical connection |
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

### Evidence rules

- Line numbers start at `1`.
- Evidence must originate from the uploaded input.
- A timestamp must not be invented when one cannot be extracted.
- Preserve enough original text to let the user verify the match.
- Represent evidence as a list because an event or correlation may be supported by multiple log lines.
- Evidence excerpts must not silently replace the original source with an AI-generated explanation.
- If a timestamp includes timezone information, preserve it. Do not silently reinterpret a timezone-naive timestamp as UTC.
- `source_file` is display metadata, not a filesystem path to open or execute.

Every detected `NetworkEvent` must contain at least one evidence item. Correlation evidence may be empty only when the correlation implementation has a documented basis for the relationship and does not fabricate source evidence.

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

### Recommendation rules

- Recommendations must relate to the detected event and its evidence.
- Prefer read-only checks and additional evidence collection.
- Configuration changes, restarts, resets, and other potentially disruptive actions must not be executed by the MVP.
- Any mention of a potentially disruptive future action must clearly distinguish it from an immediately safe check and require appropriate authorization.
- Recommendations are suggestions, not proof of a root cause.
- The value of `requires_authorization` must reflect the proposed action. It must not be set to `false` merely to simplify the user interface.

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

### Event rules

1. Every `event_id` must be unique within one `AnalysisResult`.
2. Every `rule_id` must match an implemented rule documented in `DETECTION_RULES.md`.
3. Rule IDs must remain stable across releases unless a documented rule change requires otherwise.
4. `event_type` describes the observed event, not an unverified root cause.
5. Unknown values must be represented as `null`, not fabricated values.
6. Use empty lists when there are no known possible causes, recommendations, or tags.
7. Every detected event must include supporting evidence.
8. Event severity must come from deterministic rule logic.
9. A rule match must not be presented as proof that a particular device or component caused another event.
10. `resource_type` and `resource_id` are optional because not every log line identifies a specific resource.
11. Event identifiers must be unique within the response, not necessarily across different requests.

### Event ID convention

A practical implementation may generate identifiers such as `evt_001`, `evt_002`, and so on during a request.

These IDs identify events within the current analysis. They do not imply permanent storage or globally persistent identity.

---

## 7. Correlation Model

**Purpose:** Group events that may be related by timing, shared resources, or explicitly defined rule relationships.

| Field | Type | Required | Description |
|---|---|---|---|
| `correlation_id` | String | Yes | Unique identifier within the current analysis |
| `correlation_type` | String | Yes | Category of relationship between events |
| `related_event_ids` | List of strings | Yes | IDs of events included in the correlation |
| `description` | String | Yes | Explanation of the observed relationship |
| `evidence` | List of Evidence | Yes | Evidence supporting the relationship |
| `confidence` | Confidence | Yes | Strength of the correlation evidence |
| `is_confirmed_cause` | Boolean | Yes | Whether causality has been independently established |

### Correlation rules

- Every `related_event_id` must refer to an event in the same `AnalysisResult`.
- A correlation must contain at least two distinct event IDs.
- Correlation IDs must be unique within one analysis.
- Correlation may use timestamps, device identifiers, interface identifiers, or explicit rule-defined relationships.
- Missing timestamps or identifiers must not be fabricated to create a correlation.
- The system must not assume that two events are related merely because they appear close together in a log.
- `is_confirmed_cause` defaults to `false`.
- In the MVP, `is_confirmed_cause` must remain `false`. A matching correlation rule is not independent confirmation of causality.

**Important:** Temporal association and correlation do not establish causality.

---

## 8. AnalysisSummary Model

**Purpose:** Provide a consistent overview of processing results and detected events.

| Field | Type | Required | Description |
|---|---|---|---|
| `total_lines` | Integer | Yes | Number of input lines considered by the parser |
| `parsed_lines` | Integer | Yes | Number of lines successfully parsed according to defined parser criteria |
| `unparsed_lines` | Integer | Yes | Number of lines that did not match supported parsing patterns |
| `total_events` | Integer | Yes | Total number of detected events |
| `info_events` | Integer | Yes | Number of INFO events |
| `low_events` | Integer | Yes | Number of LOW events |
| `medium_events` | Integer | Yes | Number of MEDIUM events |
| `high_events` | Integer | Yes | Number of HIGH events |
| `critical_events` | Integer | Yes | Number of CRITICAL events |
| `total_correlations` | Integer | Yes | Number of correlations produced |

### Summary invariants

The application and tests must enforce the following conditions.

**Line counts**

`total_lines = parsed_lines + unparsed_lines`

**Event counts**

`total_events = info_events + low_events + medium_events + high_events + critical_events`

**Event severity counts**

Each severity count must equal the number of returned events with that severity.

**Correlation count**

`total_correlations = len(correlations)`

All counts must be non-negative integers.

A parsed line does not necessarily produce a detected event. An unparsed line does not automatically indicate a network fault; it means the parser could not interpret that line according to supported patterns.

A line is counted as parsed only according to documented parser criteria. The implementation must use one consistent definition throughout the parser, summary, API response, and tests.

If parsing coverage is incomplete or another processing limitation affects results, the status and warnings must communicate that limitation.

---

## 9. AnalysisWarning Model

**Purpose:** Explain processing limitations and conditions the user should know about.

| Field | Type | Required | Description |
|---|---|---|---|
| `code` | String | Yes | Stable machine-readable warning code |
| `message` | String | Yes | Human-readable explanation |
| `severity` | WarningSeverity | Yes | Importance of the warning |
| `line_number` | Integer or null | No | Related input line, when applicable |

### Warning rules

- Warning codes should be stable and documented.
- Warnings must use structured objects, not plain strings.
- `line_number` must be `null` when a warning is not tied to a particular source line.
- A non-null `line_number` must be a valid one-based line number for the input being analyzed.
- Warnings should describe limitations such as unsupported log formats, partial parsing, or skipped content.
- Warnings must not claim that an event occurred unless the detection rules support that claim.
- A warning with severity `ERROR` must explain the affected processing stage or limitation.
- A warning does not, by itself, determine the analysis status; status must reflect the actual effect on result completeness.

Possible warning codes include `UNSUPPORTED_LOG_FORMAT`, `PARTIAL_PARSE`, and `EMPTY_INPUT`. These are examples only; they must be implemented and documented before being treated as active application behavior.

Invalid uploads and requests that cannot produce a valid `AnalysisResult` should use the API error contract instead of relying on an analysis warning.

---

## 10. ProcessingMetadata Model

**Purpose:** Record technical information about the current analysis request.

| Field | Type | Required | Description |
|---|---|---|---|
| `parser_version` | String | Yes | Version of the parser used |
| `ruleset_version` | String | Yes | Version of the detection rules |
| `processing_duration_ms` | Non-negative number | Yes | Measured processing duration in milliseconds |
| `ai_used` | Boolean | Yes | Whether AI was actually used for this analysis |
| `input_size_bytes` | Non-negative integer | Yes | Size of the validated uploaded content in bytes |

### Metadata rules

- The initial MVP must report `ai_used = false`.
- Parser and ruleset versions must correspond to the versions used for that request.
- `processing_duration_ms` must be measured by the application, not invented.
- `input_size_bytes` must reflect the validated content size, not the filename or an unverified client-supplied value.
- `processing_duration_ms` must be finite and non-negative.
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
| `summary` | AnalysisSummary | Yes | Line, event, and correlation counts |
| `events` | List of NetworkEvent | Yes | Detected events |
| `correlations` | List of Correlation | Yes | Correlations between detected events |
| `warnings` | List of AnalysisWarning | Yes | Structured processing warnings |
| `metadata` | ProcessingMetadata | Yes | Parser, ruleset, and processing information |

### AnalysisResult rules

1. `analysis_id` identifies a response; it does not imply a persistent record.
2. `created_at` must be timezone-aware. The application should generate it using UTC.
3. `filename` is display metadata only. It must not be used as a server filesystem path.
4. `events`, `correlations`, and `warnings` must always be arrays, including when empty.
5. Summary counts must match the events and correlations included in the response.
6. `metadata.ai_used` must be `false` in the initial MVP.
7. The response must not imply that an analysis can be retrieved later.
8. No database or analysis-history endpoint is required for the MVP.
9. Failed requests that cannot produce a valid result must return an appropriate API error response.
10. Event IDs and correlation IDs must be unique within the response.
11. Every correlation reference must resolve to an event in the same response.
12. In the MVP, every correlation must have `is_confirmed_cause = false`.
13. The status must reflect material processing limitations and must not be inferred from a warning code alone.

### Status and warning consistency

Use `COMPLETED` when the application has completed its defined analysis process and no known material limitation requires partial status.

Use `PARTIAL` when the application has returned valid results but a documented limitation materially affects analysis coverage or completeness.

A warning can accompany either status if its meaning is consistent with the actual processing result. A fatal error that prevents a valid `AnalysisResult` must use the API error contract.

---

## 12. Canonical Pydantic Model Reference

The following reference implementation illustrates the data contract using Python 3.12 and Pydantic v2.

It validates field constraints and key cross-field invariants. It is a reference for implementation and must be covered by automated tests before the application treats the contract as implemented.

```python
from datetime import datetime
from enum import Enum
from math import isfinite
from typing import Optional

from pydantic import (
    BaseModel,
    ConfigDict,
    Field,
    field_validator,
    model_validator,
)


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

    @field_validator("is_confirmed_cause")
    @classmethod
    def causality_not_confirmed_by_mvp(
        cls, value: bool
    ) -> bool:
        if value:
            raise ValueError(
                "The MVP cannot independently confirm causality"
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

    @model_validator(mode="after")
    def validate_line_counts(self):
        if self.total_lines != self.parsed_lines + self.unparsed_lines:
            raise ValueError(
                "total_lines must equal parsed_lines + unparsed_lines"
            )

        return self


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

    @field_validator("processing_duration_ms")
    @classmethod
    def require_finite_duration(cls, value: float) -> float:
        if not isfinite(value):
            raise ValueError(
                "processing_duration_ms must be finite"
            )
        return value


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

    @model_validator(mode="after")
    def validate_response_consistency(self):
        # Event identifiers must be unique within this response.
        event_ids = [event.event_id for event in self.events]

        if len(event_ids) != len(set(event_ids)):
            raise ValueError(
                "event_id values must be unique within an analysis"
            )

        # Correlation identifiers must be unique within this response.
        correlation_ids = [
            correlation.correlation_id
            for correlation in self.correlations
        ]

        if len(correlation_ids) != len(set(correlation_ids)):
            raise ValueError(
                "correlation_id values must be unique within an analysis"
            )

        # Every referenced event must exist in this response.
        known_event_ids = set(event_ids)

        for correlation in self.correlations:
            missing_ids = (
                set(correlation.related_event_ids) - known_event_ids
            )

            if missing_ids:
                raise ValueError(
                    "Every related_event_id must refer to an event "
                    "in the same AnalysisResult"
                )

            if correlation.is_confirmed_cause:
                raise ValueError(
                    "The MVP cannot independently confirm causality"
                )

        # Summary totals must match the actual returned data.
        severity_counts = {
            severity: sum(
                event.severity == severity for event in self.events
            )
            for severity in Severity
        }

        expected_counts = {
            "total_events": len(self.events),
            "info_events": severity_counts[Severity.INFO],
            "low_events": severity_counts[Severity.LOW],
            "medium_events": severity_counts[Severity.MEDIUM],
            "high_events": severity_counts[Severity.HIGH],
            "critical_events": severity_counts[Severity.CRITICAL],
            "total_correlations": len(self.correlations),
        }

        for field_name, expected_value in expected_counts.items():
            actual_value = getattr(self.summary, field_name)

            if actual_value != expected_value:
                raise ValueError(
                    f"summary.{field_name} must equal "
                    f"{expected_value} based on returned data"
                )

        if self.metadata.ai_used:
            raise ValueError(
                "AI must remain disabled in the initial MVP"
            )

        return self
```

### Reference implementation notes

- Pydantic v2's `model_validator` is used for cross-field checks.
- The reference enforces line-count arithmetic, event and correlation ID uniqueness, correlation references, severity counts, and the MVP causality/AI restrictions.
- The parser and orchestration layer must calculate summary fields from the actual processing results.
- If the parser has not yet been implemented, the reference model alone cannot establish what qualifies as a parsed line.
- Warning-to-status consistency requires a documented processing policy and corresponding tests; it should not be guessed from warning severity alone.
- The reference model does not replace upload validation, API error handling, or temporary-file cleanup.
- Keep these model definitions in the agreed shared data-model module. Do not create competing model definitions in the API or detection-rule modules.

---

## 13. Example Analysis Response

The following response is illustrative. It is not a claim about an actual network analysis.

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
- The application must not claim that uploaded files are deleted from every location unless its implementation can verify that behavior.

Any future persistence feature must be separately designed and documented, including retention, access control, and deletion behavior.

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
- Uploaded log content must be treated as untrusted data, not as instructions for the AI system.
- AI must not execute network commands or change device configuration.
- Any new response fields for AI explanations or citations must be documented and versioned before being added to the API contract.

AI support must not change the canonical response structure without a documented versioned change.

---

## 16. Cross-Document Contract

| Document | Required alignment |
|---|---|
| `API_SPECIFICATION.md` | Use `AnalysisResult` with structured events, correlations, warnings, and metadata |
| `DETECTION_RULES.md` | Map every implemented rule output to `NetworkEvent`, including stable `rule_id`, severity, evidence, and confidence |
| `ARCHITECTURE.md` | Keep parsing, detection, correlation, and response construction as separate responsibilities |
| `SECURITY_AND_PRIVACY.md` | Maintain the no-permanent-storage MVP policy and temporary-file cleanup |
| `TESTING_STRATEGY.md` | Test field validation, summary invariants, severity counts, evidence, correlation references, and response structure |
| `IMPLEMENTATION_PLAN.md` | Implement and test canonical models before integrating detection rules and API response handling |
| `PROJECT_MASTER_PLAN.md` | Keep the deterministic MVP separate from future optional AI/RAG functionality |

---

## 17. Definition of Done

This document is ready to serve as the canonical contract when:

- [ ] All severity levels, including `INFO`, are supported consistently.
- [ ] Warnings and recommendations use structured models.
- [ ] Detection-rule output maps to `NetworkEvent`.
- [ ] Event and correlation IDs are unique within a response.
- [ ] All correlation references point to events in the same response.
- [ ] The MVP does not claim confirmed causality from correlation alone.
- [ ] Line counts and severity counts match their respective invariants.
- [ ] `total_correlations` matches the returned correlation list.
- [ ] `created_at` is timezone-aware.
- [ ] Processing metadata reflects actual request processing.
- [ ] The MVP reports `ai_used = false`.
- [ ] The API example follows the canonical field definitions.
- [ ] Automated tests cover field validation and cross-field invariants.
- [ ] No documentation implies permanent analysis storage or history retrieval.
- [ ] Related documents use the same contract and endpoint definitions.

**Final decision:** This document is the single source of truth for NetAssist AI's MVP data contract. Other documents must align with it rather than redefining the same models independently.