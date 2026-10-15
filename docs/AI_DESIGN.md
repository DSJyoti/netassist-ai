# NetAssist AI — AI Design Document

**Document:** AI Design  
**Project:** NetAssist AI  
**Status:** Design Phase  
**Version:** 1.2
**Principle:** Evidence First, AI Second

---

## 1. Purpose

This document defines the design of the optional AI explanation and Retrieval-Augmented Generation (RAG) layer for NetAssist AI.

NetAssist AI is a local-first network log troubleshooting assistant. Its core responsibility is to analyze uploaded network logs using deterministic parsing, detection rules, structured evidence, correlation logic, and rule-based recommendations.

The future optional AI layer may help users:

- Understand detected network events in simple language.
- Summarize deterministic analysis results.
- Explain technical terminology and supported detection rules.
- Retrieve relevant troubleshooting guidance from a curated local knowledge base.
- Explain the relationship between detected events and supplied log evidence.
- Present relevant knowledge-base references and source citations.
- Organize safe troubleshooting checks.
- Understand uncertainty and limitations in the available evidence.

The AI layer will not replace the parser, deterministic detection engine, severity assignment, evidence extraction, or deterministic fallback mechanism.

**The initial MVP must work without an AI model, embedding model, vector database, or external AI service. AI and RAG will remain disabled by default until the deterministic MVP is stable.**

---

## 2. Core Design Principles

1. **Evidence First:** Event-specific statements must be grounded in structured analysis results and original log evidence.
2. **Deterministic Authority:** The rule-based engine remains authoritative for detections, severity, evidence, and correlations.
3. **AI Is Optional:** The application must function when AI and RAG are disabled or unavailable.
4. **Free and Local First:** Future AI components should prioritize free, locally running software and models, subject to hardware, licensing, and compatibility constraints.
5. **RAG Is Retrieval, Not Detection:** Retrieved knowledge provides supporting troubleshooting guidance; it cannot create or confirm detected events.
6. **No Invented Evidence:** AI must not invent log lines, timestamps, event IDs, source passages, or device states.
7. **Clear Uncertainty:** Possible causes must be labeled as hypotheses, not confirmed facts.
8. **Privacy by Default:** Uploaded logs, prompts, and retrieved context must not be transmitted to external AI services in local-only mode.
9. **Graceful Failure:** AI or retrieval failures must not invalidate deterministic analysis.
10. **No Automatic Remediation:** The application must not execute network commands or change device configurations.
11. **Source Traceability:** Knowledge-based claims should identify the actual local source and passage supporting them.
12. **Modular Integration:** RAG and AI components must remain separate from parsing and deterministic detection.

---

## 3. Responsibilities and Boundaries

### 3.1 Deterministic Analysis Layer

The deterministic layer is responsible for:

- Validating uploaded files.
- Parsing log lines and preserving source evidence.
- Matching supported detection rules.
- Assigning event severity and confidence according to deterministic logic.
- Producing structured events and correlations.
- Generating deterministic summaries and recommendations.
- Returning the canonical `AnalysisResult`.

These responsibilities must not depend on an AI model or successful RAG retrieval.

### 3.2 RAG Retrieval Layer

The future RAG layer may:

- Search a curated local troubleshooting knowledge base.
- Retrieve relevant passages about supported network technologies, error messages, protocols, and troubleshooting procedures.
- Provide source identifiers and passage references.
- Return no results when the available documents do not support a useful answer.
- Supply retrieved context to the optional AI explanation layer.

The RAG layer must not:

- Declare a new network event detected.
- Change the severity of an existing event.
- Treat a retrieved document as proof of what happened on the user's network.
- Convert a general troubleshooting possibility into a confirmed root cause.
- Treat retrieved text as trusted instructions for the application to execute.

### 3.3 Optional AI Explanation Layer

The AI layer may:

- Explain already-detected events.
- Summarize deterministic findings.
- Explain relevant technical concepts.
- Combine supplied event evidence with retrieved troubleshooting guidance.
- Suggest safe, non-executing troubleshooting checks.
- Explain when the available evidence is insufficient.

It must not independently create authoritative detections, modify deterministic results, or claim that a suggested check was performed.

### 3.4 Source of Truth

The following boundaries are mandatory:

| Information | Authoritative source |
|---|---|
| Detected events | Deterministic detection engine |
| Event severity | Deterministic detection rules |
| Original log evidence | Validated parser output |
| Correlation candidates | Deterministic correlation logic |
| Troubleshooting references | Retrieved local knowledge documents |
| Natural-language explanation | Optional AI output, subject to validation |
| Final API data contract | `DATA_MODELS.md` and `API_SPECIFICATION.md` |

AI explanations are advisory artifacts. They must remain distinguishable from the original deterministic findings.

---

## 4. Planned Architecture

The planned future architecture has one shared deterministic pipeline and two optional explanation paths.

```text
                    Uploaded Log File
                           |
                           v
                    File Validation
                           |
                           v
                       Log Parser
                           |
                           v
                Deterministic Detection
                           |
                           v
              Correlation + Rule-Based Logic
                           |
                           v
                Canonical AnalysisResult
                           |
                  +--------+--------+
                  |                 |
                  v                 v
           Deterministic       AI/RAG Enabled?
           Explanation               |
                  |             +-----+-----+
                  |             |           |
                  |            No          Yes
                  |             |           |
                  |             |           v
                  |             |    Evidence Context
                  |             |       Builder
                  |             |           |
                  |             |           v
                  |             |    Local Knowledge
                  |             |       Retrieval
                  |             |           |
                  |             |           v
                  |             |    Retrieved Passages
                  |             |       + Sources
                  |             |           |
                  |             |           v
                  |             |     Local AI Adapter
                  |             |           |
                  |             |           v
                  |             |    Output Validation
                  |             |           |
                  |             |           v
                  |             |    Valid Explanation?
                  |             |       |       |
                  |             |      Yes      No
                  |             |       |       |
                  +-------------+-------+       |
                                |               |
                                v               v
                        Final Presentation  Deterministic
                        of Analysis and     Fallback
                        Optional Explanation
```

This diagram describes a planned design, not an implemented feature.

### 4.1 Shared Analysis Pipeline

Both paths must use the same deterministic analysis result. The AI path must not rerun detection using model-generated interpretations or modify the canonical result.

### 4.2 Deterministic-Only Path

When AI/RAG is disabled or unavailable, the application returns deterministic analysis, summaries, and recommendations.

This is the required MVP path.

### 4.3 Optional RAG-Assisted Path

When enabled in a future phase:

1. Build a limited context from the validated analysis result.
2. Retrieve relevant passages from the local knowledge base.
3. Pass the structured analysis and selected passages to the local AI adapter.
4. Validate the proposed explanation and its references.
5. Present the explanation separately from the original analysis.
6. Use deterministic fallback if retrieval, generation, or validation fails.

### 4.4 MVP Boundary

The initial MVP implements deterministic processing and analysis only. RAG ingestion, embeddings, vector search, the local model adapter, prompt templates, and AI output validation are future-phase components.

They must not be described as implemented until they have been built and tested.

---

## 5. Local Knowledge Base and RAG Design

RAG means Retrieval-Augmented Generation. It retrieves relevant reference material and supplies that material to a language model to help ground its response.

**RAG is not itself an API or an AI model.** It is a retrieval-and-context workflow that can be integrated into the application.

### 5.1 Planned Knowledge Sources

The initial local knowledge base may contain carefully selected, legally usable reference documents, such as:

- Network protocol documentation.
- Vendor documentation that permits the intended local use.
- General troubleshooting guides.
- Curated explanations of supported log messages.
- Project-authored troubleshooting notes.
- Approved examples and test cases.

The first implementation should use a small, curated set of documents rather than indiscriminately ingesting large collections of unverified material.

Each document should have a stable source identifier, title, source type, version or retrieval date where available, and any relevant licensing information.

### 5.2 Knowledge Ingestion Pipeline

The future ingestion pipeline is:

```text
Approved Local Documents
          |
          v
Document Validation
          |
          v
Text Extraction
          |
          v
Text Cleaning
          |
          v
Chunking
          |
          v
Local Embedding Model
          |
          v
Vector Index + Metadata
          |
          v
Local Retrieval at Query Time
```

The ingestion pipeline prepares reference documents. It is separate from uploaded-log analysis.

### 5.3 Chunking

Documents will be divided into smaller passages, or chunks, to improve retrieval relevance.

Chunking rules should:

- Preserve meaningful section boundaries where practical.
- Avoid splitting technical instructions unnecessarily.
- Retain document and section metadata.
- Use configurable chunk sizes and overlap.
- Avoid treating unrelated passages as one source.
- Support traceability from a retrieved chunk to its original document.

The initial chunk size and overlap must be selected through testing rather than treated as fixed quality guarantees.

### 5.4 Embeddings and Vector Search

An embedding model converts text into numerical representations used to retrieve semantically similar passages.

The future implementation may evaluate a free, locally runnable embedding model and a local vector index such as FAISS or Chroma.

The final choice must consider:

- Python and dependency compatibility.
- Hardware and memory requirements.
- Model licensing and permitted usage.
- Retrieval quality on representative network troubleshooting questions.
- Installation and maintenance complexity.

These are candidate technologies, not final implementation commitments.

### 5.5 Retrieval Flow

At query time, the application will:

1. Construct a search query from the relevant deterministic events and the user's troubleshooting question, if one is supplied.
2. Generate an embedding using the configured local embedding model.
3. Search the local vector index.
4. Apply configurable retrieval limits and relevance thresholds.
5. Return selected passages with their source metadata.
6. Pass only the relevant passages to the explanation layer.

If no sufficiently relevant passages are found, the application should report that the knowledge base did not provide a suitable reference. It must not fabricate a citation or force an irrelevant passage into the answer.

### 5.6 Source Citations

A retrieved passage should retain enough metadata to identify its source, including:

- `source_id`
- `document_title`
- `section` or page reference, when available
- `chunk_id`
- Original source location or URL, when applicable and permitted
- Document version or access date, when available

These are proposed RAG metadata fields, not additions to the current canonical API schema.

Citations must refer to real documents and retrieved passages. The application must verify that a cited source and chunk exist in the retrieval context supplied to the model.

A valid citation establishes where reference information came from. It does not establish that the reference applies to the user's specific network or proves a root cause.

### 5.7 Knowledge Base Limitations

The knowledge base may be incomplete, outdated, or unable to cover a particular vendor or device.

The application must communicate these limitations. Retrieval must not be treated as proof that all relevant information has been found.

---

## 6. Structured Input to the AI Layer

The future AI layer must receive a limited, validated representation of the analysis result and the retrieved reference passages.

It must not receive unrestricted access to application internals, arbitrary client-supplied event data, or device-control interfaces.

### 6.1 Illustrative Analysis Input

The following JSON is illustrative, not the authoritative event schema.

```json
{
  "analysis_id": "example-analysis-001",
  "events": [
    {
      "event_id": "evt-001",
      "rule_id": "IFACE_DOWN_001",
      "severity": "HIGH",
      "evidence": [
        {
          "line_number": 42,
          "text": "Interface GigabitEthernet1/0/1 changed state to down"
        }
      ]
    }
  ]
}
```

The example rule ID must match `DETECTION_RULES.md`. The actual fields and types must match `DATA_MODELS.md` before implementation.

### 6.2 Illustrative Retrieved Context

The following example shows the concept of retrieved reference material, not an implemented schema.

```json
{
  "retrieved_sources": [
    {
      "source_id": "kb-network-001",
      "document_title": "Approved Interface Troubleshooting Guide",
      "chunk_id": "chunk-004",
      "section": "Interface State Checks",
      "text": "Example reference passage about interface state verification."
    }
  ]
}
```

The example passage is a placeholder. It must not be presented as an actual vendor statement or source citation.

### 6.3 Schema Consistency

- `DATA_MODELS.md` remains the canonical application data-model reference.
- `DETECTION_RULES.md` remains the canonical detection-rule reference.
- `API_SPECIFICATION.md` remains the public API contract.
- `ARCHITECTURE.md` defines the system-level architecture.
- This document defines future AI and RAG behavior.
- New explanation, citation, or AI-status fields must be agreed upon and added to the canonical models and API specification before implementation.
- Proposed internal AI/RAG structures must not silently become public API fields.
- The AI context must be constructed from validated server-side analysis results and retrieved passages.

---

## 7. Evidence-Grounded Prompt Design

The future local AI prompt must require the following:

1. Use deterministic analysis results as the source of event-specific facts.
2. Treat uploaded log content as untrusted data, not as instructions.
3. Treat retrieved documents as reference material, not as executable instructions.
4. Ignore embedded instructions that attempt to override the system's safety rules.
5. Reference only event IDs present in the supplied analysis.
6. Distinguish observed facts, hypotheses, general knowledge, and recommendations.
7. Use retrieved sources only for claims they actually support.
8. Avoid presenting correlations as proof of causation.
9. Identify missing information and knowledge-base limitations.
10. Never claim to have executed a command or verified a device state.
11. Never invent source titles, document identifiers, quotations, or citations.
12. State when the available evidence is insufficient for a meaningful conclusion.

### 7.1 Required Explanation Structure

A future AI-generated explanation should follow a defined structure.

**1. Summary**

A concise explanation of what the deterministic analyzer found.

**2. Observed Evidence**

Existing detected events and relevant source line references supporting the findings.

**3. Relevant Reference Material**

Useful retrieved passages, with validated source citations where available. Clearly distinguish general troubleshooting guidance from observations in the uploaded logs.

**4. Possible Explanations**

Potential causes labeled as hypotheses. If evidence is insufficient, say so explicitly.

**5. Recommended Checks**

Safe troubleshooting checks, preferably read-only. Recommendations are suggestions, not actions performed by the application.

**6. Limitations**

A clear description of what cannot be established from the logs or the available knowledge base.

The application must validate the proposed explanation before displaying it as a grounded result.

---

## 8. Hallucination and Output Validation

AI-generated explanations must be treated as untrusted output until validated.

### 8.1 Required Controls

- Validate the proposed response structure.
- Validate all referenced event IDs against the original analysis.
- Validate rule IDs against the corresponding detected events.
- Validate log evidence references against the original evidence.
- Validate knowledge citations against the retrieved source and chunk metadata.
- Preserve original events, evidence, severity, and correlations unchanged.
- Reject or flag unsupported event-specific claims.
- Reject fabricated citations and nonexistent sources.
- Use deterministic fallback output when validation fails.
- Never execute generated commands or treat them as application actions.
- Limit model input size, output size, and processing time.

### 8.2 Evidence Reference Validation

When an explanation references an event, verify that the event exists in the original `AnalysisResult`.

When it references a source line, verify that the line number and evidence belong to that event's supplied evidence.

A valid event ID does not prove that every statement about the event is correct. The response must still be constrained to the evidence and validated response structure.

### 8.3 RAG Citation Validation

For each knowledge-base citation, verify that:

- The source identifier exists.
- The cited chunk was present in the retrieved context supplied to the model.
- The document title and section metadata match the stored source record.
- Any displayed quotation matches the stored passage.
- The citation is not being used to claim more than the source supports.

If validation fails, remove the unsupported citation or reject the affected explanation according to the defined validation policy.

A citation check alone cannot fully prove semantic correctness. Representative tests and conservative output rules are also required.

### 8.4 Severity Protection

Severity is assigned by deterministic detection rules.

The AI layer must not recalculate, upgrade, downgrade, or replace the original event severity. It must not add authoritative detection events based on its own interpretation.

If an explanation disagrees with the deterministic result, the deterministic result remains authoritative.

### 8.5 Correlation Protection

Correlated events may indicate a possible relationship between observations. They do not independently establish a confirmed root cause.

For example, an interface-down event occurring near a BGP-neighbor-down event may support a troubleshooting hypothesis. It does not automatically prove that the interface event caused the BGP event.

---

## 9. Local AI Technology Strategy

The core application must not require AI dependencies or a model download.

After the deterministic MVP is stable, the following local components may be evaluated:

| Component | Purpose | Candidate approach |
|---|---|---|
| Local model runner | Runs the language model locally | Ollama or another suitable local runner |
| Language model | Generates explanations | A compatible, freely usable local model |
| Embedding model | Converts text into vectors | A compatible local embedding model |
| Vector retrieval | Finds relevant document chunks | FAISS or Chroma |
| Validation | Checks response structure and references | Python validation logic and Pydantic |

These are candidates, not final technology decisions.

The final stack must be selected after checking hardware, RAM, CPU/GPU requirements, licensing, latency, retrieval quality, and Python compatibility.

**Free software does not mean zero-resource execution.** Local models and embeddings may require an initial download, disk space, memory, and processing time. Once the necessary components and knowledge index are installed, the application should be able to operate locally without external AI requests.

### 9.1 Adapter Design

The future AI integration should use a small adapter interface so the core application does not depend on one model provider.

Illustrative interface:

```python
from typing import Protocol


class AIAdapter(Protocol):
    def explain_analysis(self, context: dict) -> dict:
        """Return a proposed explanation for validated context."""
        ...
```

This is a design example, not an implemented module. The final interface must use defined application types and a validated explanation schema.

Retrieval should have its own interface rather than being hidden inside the deterministic detection engine.

### 9.2 Potential Future File Structure

```text
backend/
  ai/
    __init__.py
    base.py
    local_adapter.py
    prompts.py
    output_validator.py
  rag/
    __init__.py
    document_loader.py
    chunker.py
    embeddings.py
    retriever.py
    source_validator.py
```

These files belong to future implementation phases. Their presence in this document does not mean they currently exist.

The final structure may change after dependency and interface decisions are made.

---

## 10. AI and RAG Configuration

AI and RAG must be disabled by default.

Proposed future environment settings:

```dotenv
AI_ENABLED=false
RAG_ENABLED=false
AI_PROVIDER=ollama
AI_MODEL=
AI_TIMEOUT_SECONDS=30
AI_MAX_INPUT_CHARS=20000
RAG_TOP_K=5
RAG_MIN_RELEVANCE=
```

These are proposed settings, not implemented configuration. `RAG_MIN_RELEVANCE` must be set only after selecting a retrieval method and defining its scoring behavior.

### 10.1 Configuration Requirements

- The default configuration must not require a model, embeddings, or vector index.
- The core application must start and analyze logs with AI and RAG disabled.
- A missing or unavailable local model must not break deterministic analysis.
- Missing or unavailable retrieval components must not break deterministic analysis.
- AI processing must have configurable time and size limits.
- Credentials must never be hard-coded or committed.
- The MVP must not require a paid API.
- Local-only mode must not send logs, prompts, embeddings, or retrieved context to external services.
- Sensitive fields should be minimized where practical.
- External-provider integration requires a separate privacy, security, configuration, and consent design.

---

## 11. AI and RAG Failure Handling

| Failure condition | Expected behavior |
|---|---|
| AI disabled | Return deterministic analysis |
| RAG disabled | Skip retrieval and use the configured non-RAG path |
| Local model unavailable | Preserve deterministic analysis and use fallback |
| Embedding model unavailable | Skip retrieval and use fallback |
| Vector index missing or corrupt | Report the optional subsystem issue; preserve deterministic analysis |
| No relevant passages found | Do not fabricate citations; explain that no suitable reference was found |
| Request timeout | Stop optional processing and use fallback |
| Invalid response structure | Reject the response and use fallback |
| Unknown event ID | Reject or flag the unsupported reference |
| Invalid log evidence reference | Reject or flag the unsupported claim |
| Invalid source citation | Reject or remove the unsupported citation |
| Unsupported root-cause claim | Do not present it as a confirmed fact |
| Unsafe command or remediation | Exclude the unsafe instruction |
| Input exceeds configured limits | Apply limits or skip optional processing |
| AI adapter raises an error | Preserve deterministic results and use fallback |

Optional subsystem errors must not erase detected events, alter severity, or cause detection rules to be rerun with different interpretations.

---

## 12. Security and Privacy

### 12.1 Uploaded Log Safety

- Uploaded logs must be treated as untrusted input.
- File type and size validation must apply before processing.
- Log text must never be executed as code or treated as trusted instructions.
- Prompt-injection-like content inside logs must remain data.
- The AI adapter must not have shell execution tools or operating-system command access.
- The AI adapter must not have direct router, switch, or network-device access.

### 12.2 Knowledge-Base Safety

- Only approved documents should enter the knowledge base.
- Document ingestion must not execute scripts or embedded content.
- Source text must be treated as untrusted reference material.
- Embedded instructions must not override application policies.
- Document metadata and citations must be retained for traceability.
- Knowledge-base changes should be reproducible and testable.
- Licensing and permitted use must be considered before adding external documentation.

### 12.3 Data Minimization

- Supply only the structured analysis and reference passages needed for an explanation.
- Avoid unnecessary credentials, usernames, IP addresses, tokens, and other sensitive fields.
- Do not log full prompts or uploaded log contents unnecessarily.
- Do not introduce permanent storage of logs, prompts, or AI explanations as an implicit side effect.
- Do not persist user logs in the vector knowledge base.

Uploaded logs and curated reference documents have different purposes: logs are analyzed as temporary input, while approved reference documents are indexed for retrieval.

### 12.4 Local-Only Operation

In local-only mode, log data, prompts, embeddings, and retrieved context must remain on the local machine.

The application must not silently fall back to an external AI service if a local component fails. External-provider integration requires a separate design and explicit user consent before data transmission.

---

## 13. Testing and Evaluation

The AI and RAG subsystems must be tested independently from the deterministic detection engine.

### 13.1 Required Functional Tests

1. AI disabled while deterministic analysis succeeds.
2. RAG disabled while deterministic analysis succeeds.
3. Local model unavailable while deterministic analysis succeeds.
4. Valid explanation references existing event IDs.
5. Invalid event IDs are rejected or flagged.
6. Invalid evidence references are rejected or flagged.
7. AI output cannot change original event severity.
8. AI output cannot add authoritative detection events.
9. Prompt-injection-like log text is treated as untrusted data.
10. Unsupported root-cause claims are not presented as facts.
11. Excessively large inputs are limited.
12. Invalid output triggers deterministic fallback.
13. Timeouts and adapter exceptions trigger fallback.
14. Invalid source identifiers and chunk references are rejected.
15. Fabricated citations are rejected or removed.
16. No relevant retrieved passage results in an explicit no-reference outcome.
17. Local-only mode makes no external AI requests.
18. AI integration does not require device access or configuration changes.

### 13.2 RAG Evaluation

Evaluate retrieval using a small, reviewed test set containing representative network troubleshooting questions and known relevant documents.

Measure:

- Whether relevant documents appear among retrieved results.
- Whether the selected passages support the question.
- Whether source identifiers and chunk references remain accurate.
- Whether irrelevant results are filtered appropriately.
- Whether the system correctly returns no suitable result when evidence is insufficient.

Retrieval quality should be evaluated separately from language-model explanation quality.

### 13.3 AI Evaluation

Measure:

- Evidence-reference accuracy.
- Citation validity.
- Unsupported-claim frequency.
- Correct labeling of hypotheses.
- Compliance with the output structure.
- Response latency and failure rate.
- Successful deterministic fallback behavior.

### 13.4 Test Independence

The core parser, detection engine, API, and privacy tests must run without a paid API, downloaded model, or vector index.

Optional AI/RAG tests should use deterministic mocks or fixtures by default. Separate integration tests may use installed local models and an indexed test knowledge base.

---

## 14. Implementation Phases

### Phase 1 — Deterministic MVP

- Implement upload validation and log parsing.
- Implement deterministic detection rules.
- Produce structured events with source evidence.
- Generate deterministic summaries and safe recommendations.
- Implement the documented API endpoints.
- Add unit and API tests.
- Keep AI and RAG disabled.
- Verify privacy and temporary-file cleanup behavior.

**Completion condition:** Core analysis works without any AI model, embedding model, or vector index.

### Phase 2 — Local Knowledge Base and Retrieval

- Select a small set of approved reference documents.
- Record source metadata and licensing information.
- Implement document loading and text extraction.
- Implement chunking and metadata preservation.
- Evaluate a suitable local embedding model.
- Build a local vector index.
- Implement retrieval and source-reference validation.
- Test relevant, irrelevant, and missing-reference scenarios.

**Completion condition:** The local retrieval subsystem can return relevant passages with verifiable source metadata without changing deterministic analysis.

### Phase 3 — Optional Local AI Adapter

- Define the adapter interface using agreed data models.
- Evaluate a suitable free local language model.
- Build a limited evidence-context builder.
- Implement prompt templates and output validation.
- Add timeout, size-limit, and fallback handling.
- Test event references, citations, and prompt-injection resistance.
- Verify local-only operation.
- Update canonical data models and API documentation before exposing any new response fields.

**Completion condition:** Optional explanations can be enabled without changing deterministic detection results or presenting unsupported citations as valid.

### Phase 4 — Evaluation and Improvement

- Compare explanations against reviewed test cases.
- Measure retrieval relevance and citation accuracy.
- Measure response latency and failure rate.
- Evaluate unsupported claims and hypothesis labeling.
- Test performance on representative log samples.
- Improve retrieval, prompts, and validation without changing deterministic detection behavior.

**Completion condition:** AI/RAG meet documented quality and safety criteria while preserving deterministic results.

---

## 15. Acceptance Criteria

The AI/RAG design is accepted when:

- AI and RAG are optional and disabled by default.
- Core analysis works without a model or vector index.
- The deterministic analysis result remains authoritative.
- AI explanations use validated structured analysis results.
- Retrieved reference passages have verifiable source metadata.
- Citations are checked against actual retrieved passages.
- Original events, evidence, severity, and correlations remain unchanged.
- Unsupported root-cause claims are not presented as confirmed facts.
- Missing references do not lead to fabricated citations.
- AI, embedding, retrieval, and validation failures trigger safe fallback behavior.
- Local-only mode does not transmit logs, prompts, or retrieved context to external AI services.
- Uploaded logs and reference documents are treated as untrusted data.
- No automatic command execution or network configuration changes are possible.
- The core test suite does not require a paid API or model download.
- RAG and AI remain separate from the deterministic detection engine.
- Implemented behavior is clearly distinguished from future design plans.
- Any new public response fields are approved and documented in the canonical data models and API specification.

---

## 16. Final Design Decision

NetAssist AI will follow a **deterministic-first, RAG-assisted, optional-local-AI architecture**.

The deterministic parser and detection engine establish what the supplied logs support. Rule-based logic assigns severity, preserves source evidence, and produces the initial analysis result.

The optional RAG subsystem retrieves relevant passages from a curated local knowledge base and provides source references. It supports troubleshooting explanations but does not establish what happened on the user's network.

The optional local AI layer may explain deterministic findings in accessible language, using validated event evidence and relevant retrieved references. It cannot independently create authoritative detections, modify severity, change evidence, fabricate citations, or confirm unsupported root causes.

The initial MVP will not require an AI model, embedding model, or vector index. Local RAG and AI will be introduced only after the deterministic system, API contracts, data models, and tests are stable.

**The deterministic analysis result remains the source of truth. RAG provides supporting reference material. AI explanations are advisory, optional, and subject to validation.**

This separation is mandatory for reliability, safety, privacy, traceability, and future maintainability.