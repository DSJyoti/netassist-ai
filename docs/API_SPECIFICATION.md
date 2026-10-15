# API_SPECIFICATION.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.1  
**Status:** MVP API Contract  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the HTTP API contract for NetAssist AI, a local-first network log troubleshooting assistant.

The API accepts supported log files, validates the input, processes logs using deterministic Python rules, and returns structured troubleshooting results.

This document must remain consistent with:

- `DATA_MODELS.md`
- `DETECTION_RULES.md`
- `ARCHITECTURE.md`
- `SECURITY_AND_PRIVACY.md`
- `TESTING_STRATEGY.md`

`DATA_MODELS.md` is the canonical source for event, correlation, warning, recommendation, summary and analysis-result structures.

## 2. API Design Principles

1. Use REST-style HTTP endpoints.
2. Validate uploads before processing.
3. Use deterministic detection rules in the initial MVP.
4. Return consistent JSON responses.
5. Preserve source evidence and line numbers.
6. Distinguish validation failures from processing limitations.
7. Do not execute network commands or change device configuration.
8. Do not persist uploaded files or analysis results by default.
9. Do not require paid APIs or cloud AI services.
10. Keep the API modular and suitable for a future browser UI.

## 3. Technology

| Component | Technology |
|---|---|
| Backend framework | FastAPI |
| Data validation | Pydantic v2 |
| Runtime | Python 3.12 |
| ASGI server | Uvicorn |
| Request format for analysis | Multipart form upload |
| Response format | JSON |
| Testing | Pytest and HTTPX |

The application should expose interactive API documentation through FastAPI during development.

## 4. Base URL and Versioning

The local development base URL is:

`http://127.0.0.1:8000`

The API prefix for analysis operations is:

`/api/v1`

The initial MVP provides the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Check whether the application process is responding |
| `GET` | `/ready` | Check whether the application is ready to handle requests |
| `GET` | `/api/v1/info` | Return supported formats and API capabilities |
| `POST` | `/api/v1/analyze` | Upload and analyze a network log file |

Only these four endpoints are in the MVP scope.

## 5. Common Response Rules

- Successful JSON responses use the appropriate `2xx` status code.
- Request and validation errors use appropriate `4xx` responses.
- Unexpected server-side failures use `5xx` responses.
- Enum values use uppercase strings, as defined in `DATA_MODELS.md`.
- Timestamps in API responses must be timezone-aware and serialized in ISO 8601 format.
- JSON field names must match the canonical Pydantic model field names.
- Missing optional values are represented as `null` where the model defines nullable fields.
- Collection fields such as `events`, `correlations` and `warnings` are arrays, including when empty.

The API must not return fabricated event data merely to produce a non-empty response.

## 6. Endpoint: Health Check

**Method:** `GET`

**Path:** `/health`

**Purpose:** Confirm that the application process is responding.

### Successful response

HTTP status: `200 OK`

```json
{
  "status": "ok"
}
```

### Notes

- This endpoint is a basic process-health check.
- It does not prove that every parser or detection rule has been tested successfully.
- It must not expose environment variables, secrets or uploaded log content.

## 7. Endpoint: Readiness Check

**Method:** `GET`

**Path:** `/ready`

**Purpose:** Report whether the application is ready to handle analysis requests.

### Successful response

HTTP status: `200 OK`

```json
{
  "status": "ready"
}
```

If a required application component has failed initialization, the endpoint should return an appropriate non-success status, such as `503 Service Unavailable`.

### Notes

- The initial MVP does not depend on a database, cloud service or external AI provider.
- Readiness checks should reflect actual application initialization.
- Do not report a dependency as healthy if the application has not checked it.

## 8. Endpoint: Application Information

**Method:** `GET`

**Path:** `/api/v1/info`

**Purpose:** Provide information required by the browser UI before uploading a file.

### Successful response

HTTP status: `200 OK`

```json
{
  "application": "NetAssist AI",
  "api_version": "v1",
  "supported_extensions": [
    ".log",
    ".txt"
  ],
  "max_upload_size_bytes": 5242880,
  "ai_enabled": false,
  "persistent_analysis_storage": false
}
```

### Field definitions

| Field | Type | Description |
|---|---|---|
| `application` | String | Application name |
| `api_version` | String | API version identifier |
| `supported_extensions` | Array of strings | File extensions accepted by the MVP |
| `max_upload_size_bytes` | Integer | Configured maximum upload size |
| `ai_enabled` | Boolean | Whether AI processing is enabled |
| `persistent_analysis_storage` | Boolean | Whether analysis results are permanently stored |

The maximum upload size shown above is 5 MiB, or 5,242,880 bytes.

These values must reflect the application's actual configuration. If the configured upload limit changes, this endpoint must report the updated value.

## 9. Endpoint: Analyze Log File

**Method:** `POST`

**Path:** `/api/v1/analyze`

**Purpose:** Validate, parse and analyze an uploaded log file, then return a structured `AnalysisResult`.

### 9.1 Request format

Content type:

`multipart/form-data`

Required field:

| Field | Type | Required | Description |
|---|---|---|---|
| `file` | File upload | Yes | A supported `.log` or `.txt` file |

Example using `curl`:

```bash
curl -X POST \
  "http://127.0.0.1:8000/api/v1/analyze" \
  -H "accept: application/json" \
  -F "file=@sample.log"
```

The uploaded filename is display metadata only. It must never be trusted as a server filesystem path.

### 9.2 Upload validation

Before analysis, the application must validate:

- The upload field exists.
- The filename is present and has a supported extension.
- The file is within the configured size limit.
- The content can be decoded using the application's documented text-decoding policy.
- The content is not empty or otherwise invalid for processing.

The application must not trust the client-provided MIME type or filename alone to establish that the content is valid.

A supported extension does not guarantee that every line will be recognized by the parser.

### 9.3 Processing flow

The request follows this sequence:

1. Receive the uploaded file.
2. Validate the filename, extension and size.
3. Read the content using the documented decoding policy.
4. Parse supported log lines and preserve source line numbers.
5. Normalize parsed records.
6. Apply deterministic detection rules.
7. Generate evidence-backed troubleshooting recommendations.
8. Correlate related events only when supported by the available evidence.
9. Construct and validate the `AnalysisResult`.
10. Return the JSON response.
11. Clean up temporary files, if any were created.

AI processing is disabled in the initial MVP. No network configuration commands are executed.

### 9.4 Successful response

HTTP status: `200 OK`

The response must follow the `AnalysisResult` model defined in `DATA_MODELS.md`.

Required top-level fields:

| Field | Type |
|---|---|
| `analysis_id` | String |
| `status` | `COMPLETED` or `PARTIAL` |
| `filename` | String |
| `created_at` | Timezone-aware datetime |
| `summary` | `AnalysisSummary` |
| `events` | Array of `NetworkEvent` |
| `correlations` | Array of `Correlation` |
| `warnings` | Array of `AnalysisWarning` |
| `metadata` | `ProcessingMetadata` |

Each event must use the canonical event fields and structured evidence and recommendations defined in `DATA_MODELS.md`.

Each warning must be an object containing `code`, `message`, `severity` and `line_number`.

Each recommendation must be an object containing `recommendation_id`, `title`, `description`, `action_type`, `risk_level` and `requires_authorization`.

Plain-string warnings and plain-string recommendations are not valid substitutes for these structured objects.

### 9.5 Analysis status

The API supports these analysis statuses:

- `COMPLETED`: Analysis completed without a known limitation requiring partial status.
- `PARTIAL`: Results were produced, but processing limitations affected analysis coverage.

For example, unsupported lines may be skipped while supported lines are still analyzed. The API must communicate meaningful coverage limitations through warnings and the analysis status.

An analysis with zero detected events can still be `COMPLETED` if processing completed successfully. No detected events does not prove that the network is healthy.

## 10. Error Response Contract

Errors that prevent a valid `AnalysisResult` from being produced must use an HTTP error status.

The MVP should return a consistent JSON error body:

```json
{
  "error": {
    "code": "UNSUPPORTED_FILE_TYPE",
    "message": "Upload a supported .log or .txt file."
  }
}
```

### Error body fields

| Field | Type | Description |
|---|---|---|
| `error.code` | String | Stable machine-readable error code |
| `error.message` | String | Human-readable explanation |

This error envelope is separate from `AnalysisWarning`. Warnings describe limitations within an analysis response; errors describe requests that cannot be processed successfully or cannot produce a valid result.

### Expected HTTP errors

| HTTP status | Error code example | Condition |
|---|---|---|
| `400 Bad Request` | `INVALID_UPLOAD` | Missing or malformed upload information |
| `413 Content Too Large` | `FILE_TOO_LARGE` | Upload exceeds the configured size limit |
| `415 Unsupported Media Type` | `UNSUPPORTED_FILE_TYPE` | Unsupported file extension or content type, according to validation policy |
| `422 Unprocessable Entity` | `INVALID_FILE_CONTENT` | Content cannot be accepted under the documented input policy |
| `500 Internal Server Error` | `INTERNAL_ERROR` | Unexpected server-side failure |

FastAPI may produce its default validation-error response unless a custom exception handler is implemented. If the application adopts the error envelope above, custom handlers and tests must ensure that applicable error responses follow it consistently.

Do not expose Python tracebacks, secrets, local filesystem paths or sensitive log content in public error messages.

## 11. Summary and Count Consistency

The `summary` returned by the API must match the content of the same response.

**Line-count invariant**

`total_lines = parsed_lines + unparsed_lines`

**Event-count invariant**

`total_events = info_events + low_events + medium_events + high_events + critical_events`

**Correlation-count invariant**

`total_correlations = len(correlations)`

The API must not return counts calculated from a different request or an earlier analysis.

## 12. File Handling and Privacy

The initial MVP does not permanently store uploaded logs or analysis results.

Requirements:

- Process uploaded content only for the current request.
- Use safe temporary-file handling if disk-backed upload processing is required.
- Attempt temporary-file cleanup after both successful and failed processing.
- Do not use the uploaded filename directly to construct a filesystem path.
- Avoid logging full uploaded contents.
- Do not expose internal exceptions or sensitive content in error responses.
- Do not create an analysis-history database.
- Do not imply that `analysis_id` can be used to retrieve a result later.

The application must follow `SECURITY_AND_PRIVACY.md` for the detailed upload, retention and cleanup policy.

## 13. Timeout and Resource Limits

The MVP must enforce a maximum upload size of 5 MiB by default.

Additional safeguards should include:

- Bounded file reading and validation.
- Controlled handling of malformed or unusually long lines.
- Predictable parsing behavior for unsupported input.
- Appropriate exception handling for decoding and processing failures.
- Tests for empty, oversized, malformed and partially supported files.

Timeout limits and any additional line-length limits must be explicitly configured and documented before being treated as enforced guarantees.

## 14. API Security Boundaries

The MVP is intended for local development and demonstration.

- Bind to localhost by default during development.
- Do not expose the service publicly without a separate security review.
- Do not execute shell commands from uploaded log content.
- Do not allow uploaded text to control file paths or application configuration.
- Do not execute suggested troubleshooting actions.
- Do not treat log content as trusted instructions for an AI system.
- Do not claim authentication, multi-tenancy or production access control unless those features are actually implemented.

Any deployment beyond local development requires a separate review of authentication, authorization, network exposure, rate limiting and operational logging.

## 15. Testing Requirements

API tests must cover the following scenarios.

### Health and information

- `GET /health` returns the documented successful response.
- `GET /ready` accurately reflects readiness.
- `GET /api/v1/info` returns the actual configured upload limit and AI setting.

### Upload validation

- Valid `.log` file.
- Valid `.txt` file.
- Missing upload field.
- Unsupported extension.
- Empty file.
- File exceeding the configured size limit.
- Invalid or unsupported text encoding.
- Malformed log lines.
- Filename containing path-like characters.

### Analysis response

- Response follows the `AnalysisResult` schema.
- Status values use the documented uppercase enum values.
- Severity supports `INFO`, `LOW`, `MEDIUM`, `HIGH` and `CRITICAL`.
- Warnings are structured objects.
- Recommendations are structured objects.
- Evidence contains valid source line numbers.
- Event IDs are unique within the response.
- Correlations refer only to events in the same response.
- Summary counts satisfy all invariants.
- `created_at` is timezone-aware.
- `metadata.ai_used` is `false` in the initial MVP.
- No permanent analysis storage is created.

### Error handling

- Expected client errors return appropriate HTTP status codes.
- Unexpected exceptions do not expose tracebacks or uploaded content.
- Temporary-file cleanup is attempted after failures.

## 16. Deferred Features

The following are outside the initial MVP API contract:

- `GET /api/v1/analyses`
- `GET /api/v1/analyses/{analysis_id}`
- Saved analysis retrieval
- Saved analysis export
- Database-backed history
- User accounts and multi-tenancy
- Automated device connectivity checks
- Remote router access
- Automatic network configuration changes
- AI-generated detections
- Cloud AI integration

These features must not be described as implemented unless they are separately designed, built and tested.

## 17. Definition of Done

The API contract is ready for implementation when:

- [ ] The four MVP endpoints are defined.
- [ ] Upload format and size limits are explicit.
- [ ] The analysis response matches `DATA_MODELS.md`.
- [ ] Enum values are consistent across all documentation.
- [ ] Warnings and recommendations use structured objects.
- [ ] Error responses are distinct from analysis warnings.
- [ ] Summary counts and event references are validated.
- [ ] Temporary-file cleanup and privacy requirements are documented.
- [ ] Tests cover successful, partial and failed requests.
- [ ] Deferred features are not presented as existing capabilities.

**Final decision:** `API_SPECIFICATION.md` defines the HTTP interface. `DATA_MODELS.md` remains the single source of truth for the structure of analysis data.
