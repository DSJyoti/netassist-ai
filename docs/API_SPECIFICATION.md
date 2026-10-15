# API_SPECIFICATION.md

**Project:** NetAssist AI — Intelligent Network Log Troubleshooting Assistant  
**Document Version:** 1.2
**Status:** MVP API Contract  
**Last Updated:** October 15, 2026

---

## 1. Purpose

This document defines the HTTP API contract for NetAssist AI, a local-first network log troubleshooting assistant.

The API accepts supported log files, validates uploaded content, processes logs using deterministic Python rules, and returns structured troubleshooting results.

This document must remain consistent with:

- `DATA_MODELS.md`
- `DETECTION_RULES.md`
- `ARCHITECTURE.md`
- `SECURITY_AND_PRIVACY.md`
- `TESTING_STRATEGY.md`
- `TECH_STACK.md`
- `PRD.md`

`DATA_MODELS.md` is the canonical source for event, correlation, warning, recommendation, summary and analysis-result structures. This document defines how those structures are exposed over HTTP.

## 2. API Design Principles

1. Use REST-style HTTP endpoints.
2. Validate uploads before processing.
3. Use deterministic detection rules in the initial MVP.
4. Return consistent JSON responses.
5. Preserve source evidence and original line numbers.
6. Distinguish validation errors from processing limitations.
7. Do not execute network commands or change device configuration.
8. Do not permanently store uploaded files or analysis results by default.
9. Do not require paid APIs or cloud AI services.
10. Keep the API modular and suitable for a future browser UI.
11. Return only data supported by the current analysis.
12. Keep optional AI/RAG functionality separate from deterministic detection.

## 3. Technology

| Component | Technology |
|---|---|
| Backend framework | FastAPI |
| Data validation | Pydantic v2 |
| Runtime | Python 3.12+ |
| ASGI server | Uvicorn |
| Analysis request | Multipart form upload |
| Response format | JSON |
| Testing | Pytest and HTTPX |

FastAPI's interactive API documentation may be used during local development.

## 4. Base URL and Versioning

The local development base URL is:

`http://127.0.0.1:8000`

The API prefix for analysis operations is:

`/api/v1`

The initial MVP provides exactly four application endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Check whether the application process is responding |
| `GET` | `/ready` | Check whether the application is ready to handle requests |
| `GET` | `/api/v1/info` | Return supported formats and application capabilities |
| `POST` | `/api/v1/analyze` | Upload and analyze a network log file |

Only these four application endpoints are included in the MVP contract. Framework-generated documentation routes are not additional application features.

## 5. Common Response Rules

- Successful responses use an appropriate `2xx` status code.
- Client errors use appropriate `4xx` status codes.
- Unexpected server-side failures use `5xx` status codes.
- Enum values use the exact values defined in `DATA_MODELS.md`.
- Datetime values must be timezone-aware and serialized in ISO 8601 format.
- JSON field names must match the canonical model fields.
- Nullable fields use `null` where the canonical model permits it.
- Collection fields such as `events`, `correlations` and `warnings` are arrays, including when empty.
- The API must not fabricate events merely to produce a non-empty response.
- Response metadata must describe the processing that actually occurred.

## 6. Endpoint: Health Check

**Method:** `GET`
**Path:** `/health`

### Purpose

Confirm that the application process is responding.

### Successful response

HTTP status: `200 OK`

```json
{
  "status": "ok"
}
```

### Requirements

- This endpoint provides a basic process-health check.
- It does not prove that every detection rule has been validated.
- It must not expose secrets, environment variables or uploaded log contents.

## 7. Endpoint: Readiness Check

**Method:** `GET`
**Path:** `/ready`

### Purpose

Report whether the application is ready to handle analysis requests.

### Successful response

HTTP status: `200 OK`

```json
{
  "status": "ready"
}
```

If a required application component fails initialization, the endpoint should return an appropriate non-success status, such as `503 Service Unavailable`.

### Requirements

- The initial MVP does not depend on a database, cloud service or external AI provider.
- Readiness must reflect actual application initialization.
- The endpoint must not report an unchecked dependency as healthy.
- Optional AI/RAG components must not prevent readiness when AI is disabled.

## 8. Endpoint: Application Information

**Method:** `GET`
**Path:** `/api/v1/info`

### Purpose

Provide information the browser UI needs before uploading a file.

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
| `supported_extensions` | Array of strings | Supported upload extensions |
| `max_upload_size_bytes` | Integer | Configured maximum upload size |
| `ai_enabled` | Boolean | Whether optional AI processing is enabled |
| `persistent_analysis_storage` | Boolean | Whether analysis results are permanently stored |

The initial upload limit is 5 MiB, or 5,242,880 bytes.

These values must reflect actual application configuration. If the configured limit or AI setting changes, this endpoint must report the updated value.

The initial MVP must return `ai_enabled: false` and `persistent_analysis_storage: false`.

## 9. Endpoint: Analyze Log File

**Method:** `POST`
**Path:** `/api/v1/analyze`

### Purpose

Validate, parse and analyze an uploaded log file, then return a structured `AnalysisResult`.

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
- The file does not exceed the configured 5 MiB limit.
- The content can be processed under the documented text-decoding policy.
- Empty or otherwise invalid content is handled consistently with the validation policy.

The application must not trust the client-provided MIME type or filename alone to establish that content is valid.

A supported extension does not guarantee that every line will be recognized by the parser.

The size limit must be enforced while reading the upload, not only after loading an arbitrarily large file into memory.

### 9.3 Processing flow

The request follows this sequence:

1. Receive the uploaded file.
2. Validate filename, extension and size.
3. Read content using the documented decoding policy.
4. Parse supported log lines and preserve original line numbers.
5. Normalize parsed records.
6. Apply implemented deterministic detection rules.
7. Generate evidence-backed recommendations.
8. Correlate events only when documented criteria are satisfied.
9. Construct and validate `AnalysisResult`.
10. Return the JSON response.
11. Clean up temporary files, if any were created.

AI processing is disabled in the initial MVP. The application must not execute network configuration commands.

### 9.4 Successful response

HTTP status: `200 OK`

The response must conform to `AnalysisResult` in `DATA_MODELS.md`.

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

Nested objects must use the canonical fields and types from `DATA_MODELS.md`.

Each warning must follow the canonical `AnalysisWarning` structure, including `code`, `message`, `severity` and `line_number` as defined there.

Each recommendation must use the canonical structured `Recommendation` model, including `recommendation_id`, `title`, `description`, `action_type`, `risk_level` and `requires_authorization`.

Plain-string warnings or recommendations are not valid substitutes for structured objects.

### 9.5 Analysis status

The API supports the following statuses:

- `COMPLETED`: Processing finished without a known limitation requiring partial status.
- `PARTIAL`: Results were produced, but a processing limitation affected analysis coverage.

The precise conditions for assigning these statuses must be implemented consistently with `DATA_MODELS.md` and `TESTING_STRATEGY.md`.

For example, unsupported lines may be skipped while supported lines are still analyzed. A meaningful coverage limitation should be reported through structured warnings and the appropriate status.

An analysis with zero detected events can still be `COMPLETED` if processing completed successfully. Zero detections do not prove that the network is healthy.

A request that fails before a valid `AnalysisResult` can be produced must return an HTTP error response instead.

## 10. Error Response Contract

Errors that prevent a valid `AnalysisResult` from being produced must use an HTTP error status.

The target error envelope is:

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

This error envelope is separate from `AnalysisWarning`.

Warnings describe limitations within a returned analysis. Errors describe requests that cannot be processed successfully or cannot produce a valid result.

### Expected HTTP errors

| HTTP status | Error code example | Condition |
|---|---|---|
| `400 Bad Request` | `INVALID_UPLOAD` | Missing or malformed upload information, if handled as a bad request |
| `413 Content Too Large` | `FILE_TOO_LARGE` | Upload exceeds the configured limit |
| `415 Unsupported Media Type` | `UNSUPPORTED_FILE_TYPE` | Unsupported file type under the documented validation policy |
| `422 Unprocessable Entity` | `INVALID_FILE_CONTENT` | Content cannot be accepted under the input policy |
| `500 Internal Server Error` | `INTERNAL_ERROR` | Unexpected server-side failure |

The implementation must choose and document the precise mapping for missing fields and malformed requests. FastAPI's default request-validation response may differ from the custom error envelope.

If a uniform error envelope is adopted, implement the required exception handlers and test the applicable validation, upload and server errors.

Do not expose Python tracebacks, secrets, local filesystem paths or sensitive log content in client-facing error messages.

## 11. Summary and Count Consistency

The returned `summary` must describe the same analysis represented by `events` and `correlations`.

The following invariants apply when the canonical model defines these fields accordingly:

**Line-count invariant**

`total_lines = parsed_lines + unparsed_lines`

Every source line must be accounted for consistently. If the parser uses another classification for blank or ignored lines, the canonical definition must specify how those lines are counted.

**Event-count invariant**

`total_events = info_events + low_events + medium_events + high_events + critical_events`

All returned events must be included exactly once in the applicable severity count.

**Correlation-count invariant**

`total_correlations = len(correlations)`

Counts must be computed from the current analysis, not from another request or previous result.

If any invariant conflicts with the canonical definitions in `DATA_MODELS.md`, resolve the canonical model first rather than silently changing API behavior.

## 12. File Handling and Privacy

The initial MVP does not permanently store uploaded logs or analysis results.

Requirements:

- Process uploaded content only for the current request.
- Use safe temporary-file handling if disk-backed processing is required.
- Attempt temporary-file cleanup after successful and failed processing.
- Never use the uploaded filename directly to construct a filesystem path.
- Avoid logging full uploaded content.
- Do not expose sensitive content in error responses.
- Do not create a database-backed analysis history.
- Do not imply that `analysis_id` can retrieve a result later.
- Keep retention behavior consistent with `SECURITY_AND_PRIVACY.md`.

## 13. Timeout and Resource Limits

The initial MVP must enforce a maximum upload size of 5 MiB.

Additional safeguards should include:

- Bounded upload reading and validation.
- Controlled handling of malformed or unusually long lines.
- Predictable parsing of unsupported input.
- Appropriate exception handling for decoding and processing failures.
- Tests for empty, oversized, malformed and partially supported files.

Timeout and line-length limits must be explicitly configured before they are described as enforced guarantees.

## 14. API Security Boundaries

The MVP is intended for local development and demonstration.

- Bind to localhost by default during development.
- Do not expose the service publicly without a separate security review.
- Do not execute shell commands from uploaded log content.
- Do not allow uploaded text to control file paths or application configuration.
- Do not execute suggested troubleshooting actions.
- Do not treat log content as trusted AI instructions.
- Do not claim authentication, multi-tenancy or production access control unless those features are implemented and tested.

Any deployment beyond local development requires a separate review of authentication, authorization, network exposure, rate limiting and operational logging.

## 15. API Testing Requirements

Tests must cover the following scenarios.

### Health and information

- `GET /health` returns the documented response.
- `GET /ready` accurately reflects readiness.
- `GET /api/v1/info` returns the actual configured upload limit and AI setting.

### Upload validation

- Valid `.log` file.
- Valid `.txt` file.
- Missing upload field.
- Unsupported extension.
- Empty file.
- File exceeding the configured limit.
- Invalid or unsupported text encoding.
- Malformed log lines.
- Filename containing path-like characters.

### Analysis response

- Response conforms to `AnalysisResult`.
- Status uses the canonical enum.
- Severity supports `INFO`, `LOW`, `MEDIUM`, `HIGH` and `CRITICAL`.
- Warnings and recommendations use structured objects.
- Evidence preserves valid original source line numbers.
- Event IDs are unique within the response.
- Correlations refer only to events in the same response.
- Summary counts satisfy the canonical invariants.
- `created_at` is timezone-aware.
- `metadata.ai_used` is `false` in the initial MVP.
- No permanent analysis storage is created.

### Error handling

- Client errors use documented status codes and error codes.
- Unexpected exceptions do not expose tracebacks or uploaded content.
- Temporary-file cleanup is attempted after failures.
- The error envelope is consistent wherever the application contract requires it.

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

These features must not be described as implemented unless separately designed, built and tested.

Optional local RAG and AI explanations may be introduced in a later phase. They must not change deterministic detections or require changes to the public response schema without a documented model and API contract update.

## 17. Definition of Done

The API contract is ready for implementation when:

- [ ] The four MVP endpoints are defined.
- [ ] Upload format and size limits are explicit.
- [ ] The analysis response matches `DATA_MODELS.md`.
- [ ] Enum values are consistent across documentation.
- [ ] Warnings and recommendations use structured objects.
- [ ] HTTP errors are distinct from analysis warnings.
- [ ] Summary counts and event references are validated.
- [ ] Temporary-file cleanup and privacy requirements are documented.
- [ ] Tests cover successful, partial and failed requests.
- [ ] Deferred features are not presented as existing capabilities.
- [ ] FastAPI's actual validation and exception behavior is covered by tests.

## Final Decision

`API_SPECIFICATION.md` defines the HTTP interface. `DATA_MODELS.md` remains the single source of truth for analysis data structures and invariants.

The initial MVP exposes four application endpoints and processes supported log uploads through deterministic analysis. AI remains disabled, uploaded logs and analysis results are not permanently stored by default, and errors, warnings and partial-analysis limitations must be reported distinctly.
