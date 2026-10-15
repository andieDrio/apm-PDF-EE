# Architecture — Advanced PDF Intelligence Suite

**Repository:** [andieDrio/apm-PDF-EE](https://github.com/andieDrio/apm-PDF-EE)  
**Product name:** apm-PDF-EE (PDF Enterprise Edition)  
**Document status:** Proposed architecture baseline — not proof of implementation  
**Last updated:** 2026-10-15

## 1. Purpose and authority

This document defines the target architecture for apm-PDF-EE: a cross-platform desktop application for professional PDF reading, editing, OCR, document intelligence, security inspection, and batch processing.

The supported desktop platforms are macOS, Windows, and Linux. The product is local-first: core document workflows must not require a cloud account or internet access after required application components, OCR language data, and optional models have been installed.

This is an architectural plan. Features, interfaces, commands, tests, and modules described here are not considered implemented until code exists and the implementation has been validated.

### Canonical project documents

- `README.md`: project overview, setup instructions, supported features, and current verified status.
- `Architecture.md`: system boundaries, data flow, security, and technical decisions.
- `MasterLoopInstructions.md`: mandatory development loop, gate rules, and acceptance criteria.
- `LICENSE`: repository-level license and the starting point for project licensing review.

If documents conflict, security and data-integrity requirements take precedence. Update all affected documents together when a decision changes.

## 2. Product goals

1. Provide dependable PDF viewing, search, page management, annotation, forms, and export.
2. Support editing of existing text and images where the PDF structure permits it, with explicit handling of unsupported layouts and font limitations.
3. Make scanned PDFs searchable with local OCR.
4. Provide optional local AI for summaries, evidence-linked question answering, and structured extraction without automatically uploading content.
5. Support true redaction and security inspection rather than cosmetic overlays.
6. Process batches with cancellation, progress, error reporting, and safe output handling.
7. Keep core features available offline and ship supported builds for all three desktop platforms.
8. Prefer open-source foundations while reviewing every direct dependency, transitive dependency, native binary, font, language pack, and AI model license before distribution.

## 3. Architectural principles

- **Local-first:** no telemetry, remote inference, document upload, or external content access without explicit policy and user consent.
- **Security by boundary:** treat every input PDF as untrusted; constrain parsing and conversion workers.
- **Engine adapters:** feature code depends on internal, versioned contracts, not directly on vendor-specific APIs.
- **Data integrity:** never silently overwrite the only source copy; write to a temporary output, validate, then atomically commit when supported by the filesystem.
- **Honest capability reporting:** disclose when editing requires reconstruction, font substitution, flattening, or unsupported content loss.
- **Deterministic core:** ordinary PDF processing must work when AI is unavailable.
- **Observable processing:** long-running work has job IDs, structured state, progress, cancellation behavior, and useful sanitized errors.
- **Cross-platform parity:** platform-specific behavior is isolated behind adapters and verified on each supported OS.
- **Minimal privilege:** UI code cannot execute arbitrary shell commands or access unrestricted filesystem paths.
- **Testable change:** each feature has unit, integration, regression, and where relevant security tests.

## 4. Target system context

```text
┌─────────────────────────────────────────────────────────┐
│ Desktop UI: Tauri + React + TypeScript                   │
│ Viewer · Editing workspace · OCR · Forms · Security · AI │
└────────────────────────┬────────────────────────────────┘
                         │ Typed, validated IPC
┌────────────────────────▼────────────────────────────────┐
│ Native Host / Application Services                       │
│ Permissions · File picker · Job orchestration · Logging  │
│ Worker lifecycle · Settings · Update/package integration│
└───────────┬────────────────────┬────────────────────────┘
            │                    │
┌───────────▼──────────┐ ┌───────▼────────────────────────┐
│ Document Workers     │ │ Local Persistence              │
│ PDF engine adapters  │ │ SQLite · FTS · Job history     │
│ OCR/conversion       │ │ Model registry · Audit events  │
│ Security inspection  │ │ No default storage of PDF blobs│
│ Optional local AI    │ └────────────────────────────────┘
└──────────────────────┘
            │
┌───────────▼─────────────────────────────────────────────┐
│ User-controlled files, temporary workspace, outputs,    │
│ OCR language data and explicitly installed local models │
└─────────────────────────────────────────────────────────┘
```

This diagram is the target design, not the current repository structure. The initial repository contains only the existing `README.md` and `LICENSE`; implementation structure must be introduced in reviewed gates.

## 5. Proposed technology choices

These are candidates, not dependency approvals. Versions must be selected, locked, and tested before use.

| Area | Candidate | Notes |
|---|---|---|
| Desktop shell | Tauri 2 + Rust | Native packaging and controlled host capabilities |
| UI | React + TypeScript | Modular desktop interface and typed UI contracts |
| Viewer | PDF.js | Rendering/viewing foundation; not a complete content editor |
| Structural PDF operations | pypdf or other license-compatible adapter | Page operations and basic document transformations |
| Advanced PDF editing | Engine selected after capability and license spike | Must prove existing text/image edit behavior against difficult samples |
| OCR | OCRmyPDF + Tesseract, subject to license and packaging review | Searchable text layer, local language packs |
| Local persistence | SQLite | Settings, job metadata, indexes and audit events |
| Search | SQLite FTS initially | Indexes must be rebuildable and removable |
| Local AI | Optional runtime adapter | Model/runtime selection follows hardware, privacy, quality, and license evaluation |
| Validation | pytest, frontend tests, Rust tests, golden-document tests | Exact tooling will follow the selected project scaffold |

### Licensing decision gate

The repository currently contains an MIT license. The project must not assume that every third-party library or model can be redistributed under MIT. Review the full dependency tree and model/data terms before packaging.

PyMuPDF may be available under AGPL-3.0 or a commercial licensing arrangement; do not make it a mandatory component until the owner has explicitly approved the licensing implications. OCR components, JavaScript packages, system libraries, codecs, fonts and model weights require their own inventory. Generate and maintain a third-party notices file before the first distributable build.

## 6. Logical modules and boundaries

### 6.1 Desktop presentation

Owns navigation, document tabs, viewer state, selection UI, editing tools, previews, accessibility, and user-facing errors. It must not implement PDF parsing or run arbitrary subprocesses.

### 6.2 Native host

Owns application startup/shutdown, explicit filesystem access, permission-scoped IPC, worker lifecycle, platform integration, settings paths, and controlled updates. All IPC payloads must be validated against versioned schemas.

### 6.3 Document model and PDF adapters

Own document identifiers, source/revision relationships, page coordinates, editing operations, export options, validation and capability reporting. Engine-specific differences remain behind adapters.

### 6.4 PDF worker

Performs inspection, rendering support where applicable, page operations, text/image edits, annotation operations, forms, conversion, encryption, and output verification. Treat input documents as untrusted and apply resource limits.

### 6.5 OCR worker

Performs image preprocessing, orientation correction, deskew, OCR, language-pack selection, searchable-PDF generation and extraction. Must run offline and report language/model availability, confidence where supported, and page-level failures.

### 6.6 Security worker

Inspects document actions, metadata, embedded files, links, attachments and other supported structures. Sanitization and redaction are explicit operations. Redaction must remove underlying sensitive content and verify the output rather than merely draw a covering rectangle.

### 6.7 Local AI worker

Optional service for local model inference, indexing, embeddings, summaries, question answering and field extraction. It must not be required to open or edit documents. Treat PDF content as untrusted data, not model instructions. Results should cite actual page and passage evidence and distinguish extracted facts from model-generated interpretation.

### 6.8 Job orchestration

Defines accepted job types, state transitions, progress events, cancellation, retries where safe, temporary artifacts, and terminal outcomes. Retries must not duplicate destructive effects. Each operation declares whether it can resume safely.

### 6.9 Persistence

Use SQLite initially for metadata, jobs, settings, search indexes and audit events. Store PDFs as normal files by default. Indexes and derived data must be rebuildable and removable. Use schema migrations and transactional writes.

## 7. PDF editing capability model

PDF content is positioned graphics, not necessarily a reflowable document model. The application must support multiple explicit strategies:

1. **Native object edit:** modify a supported existing text or image object.
2. **Text-region reconstruction:** rebuild a region when the original text cannot be safely patched.
3. **Overlay/replacement:** add replacement content while clearly warning that the original may remain underneath.
4. **OCR reconstruction:** add searchable text to scanned pages and support only those further edits validated by the engine.
5. **Page/image replacement:** replace a page or image when object-level editing is not reliable.

The interface must disclose font substitution, altered geometry, reconstruction, flattening, unsupported annotations, and other material fidelity changes. Overlay content is never evidence of secure redaction.

## 8. Data model

Initial entities are proposed; schema details require migration and privacy review.

- `documents`: stable ID, canonical path, file size, content hash, metadata and indexing state.
- `document_revisions`: revision ID, parent/source relationship, output hash, timestamp and operation.
- `jobs`: operation, status, progress, lifecycle timestamps, cancellation and sanitized error details.
- `job_artifacts`: generated outputs and validation reports.
- `annotations`: supported annotations with document/page references.
- `document_indexes`: extracted text, page mapping and indexing metadata.
- `ai_sessions`: local conversation metadata and evidence references, with retention controls.
- `audit_events`: security-sensitive operations and export/redaction events, avoiding unnecessary sensitive content.
- `model_registry`: local model identifier, file path, hash, version, terms/license and installation state.
- `settings`: user preferences and resource limits.

Do not store passwords, secrets, or entire document text in logs. Document indexes may themselves contain sensitive data; protect them using appropriate filesystem permissions and provide a clear delete/rebuild workflow.

## 9. Processing lifecycle

```text
Select file
  → Validate path, file type, size and resource limits
  → Inspect PDF structure in a constrained worker
  → Create job and revision metadata
  → Execute the selected engine operation
  → Validate output structurally and, where relevant, visually
  → If validation fails: preserve source, report failure, clean safe temporaries
  → If validation succeeds: commit output atomically where supported
  → Record hashes, job outcome and required audit event
  → Show output and any limitations to the user
```

Recommended job states:

`QUEUED → RUNNING → VALIDATING → SUCCEEDED`

Alternative terminal states are `FAILED` and `CANCELLED`. Interrupted jobs may be resumed only when the operation's documented semantics make that safe.

## 10. Security requirements

- Treat every PDF as untrusted, regardless of source.
- Disable automatic execution of PDF JavaScript, launch actions and external actions.
- Do not permit parser/conversion workers to access the network by default.
- Apply configurable file-size, page-count, decompression, embedded-file, memory and timeout limits.
- Isolate parsing and conversion in worker processes where practical.
- Use least-privilege process permissions and narrowly scoped IPC.
- Validate destination paths and guard against path traversal and symlink-related mistakes.
- Use safe temporary directories and clean up after completion/cancellation.
- Preserve originals until the output has passed validation.
- Protect local AI against prompt injection in document text and embedded content.
- Do not log full document text, extracted credentials, or other sensitive contents.
- Verify redaction against extracted text, metadata, annotations, attachments and relevant underlying objects.
- Preserve signed originals; disclose when any modification invalidates or affects a signature.
- Keep security limitations explicit; do not claim complete malware detection from structural inspection alone.

## 11. Offline and privacy contract

Core viewer, page operations, supported editing, OCR, security processing and export must work without an internet connection once required components are installed.

- No document or prompt upload by default.
- No mandatory remote AI provider.
- Model downloads are user-initiated or explicitly configured, with hash verification and license notices.
- Optional network-dependent features are off by default and visibly identified.
- Tests must monitor network attempts during representative offline workflows.
- Provide a way to remove document indexes, AI session records, temporary files and installed optional models.

## 12. Testing and release quality

Required test layers:

- Unit tests for domain logic, page geometry, status transitions, error translation and permission checks.
- Contract tests between UI, host, workers and engine adapters.
- Integration tests for file handling, persistence, cancellation, cleanup and recovery.
- Golden-document corpus covering digitally generated PDFs, scanned PDFs, mixed text/images, unusual fonts, forms, annotations, bookmarks, encryption, large files and malformed inputs.
- Structural and visual regression comparisons with documented tolerances.
- Security tests for hostile PDFs, resource exhaustion, embedded actions, path handling, privacy leakage and redaction leakage.
- Offline tests with network access blocked.
- Cross-platform clean-install and smoke tests on macOS, Windows and Linux.
- Performance tests measuring latency, memory, OCR throughput, batch behavior and AI resource use.
- Dependency/license inventory and vulnerability review for release candidates.

A passing unit-test suite alone does not establish PDF fidelity, security, or cross-platform readiness.

## 13. Implementation gates

| Gate | Scope | Required exit evidence |
|---|---|---|
| 00 | Architecture, threat model, licensing and repository baseline | Reviewed documents, license policy, dependency decision register, test matrix |
| 01 | Desktop shell and CI | Development builds start on all target OSes; typed IPC and worker supervision tests |
| 02 | Viewer and document model | Open/render/search/navigation tests pass; source preservation verified |
| 03 | PDF editing feasibility spike | Real-document corpus evaluated; existing text/image edit fidelity and licenses documented |
| 04 | Editing and export | Page operations, annotations, forms, validated save/export and regression evidence |
| 05 | OCR and conversion | Offline language packs, searchable output, quality measurements and batch tests |
| 06 | Security, redaction and signatures | Adversarial tests and redaction leakage checks pass; limitations documented |
| 07 | Local AI | Offline operation, source-grounded answers, model licensing and resource measurements |
| 08 | Production hardening | Cross-platform packaging, accessibility, performance, SBOM/license review and release checks |

Gates must be completed in order unless a documented dependency requires a narrow prerequisite change. A gate is not complete until its acceptance evidence is recorded and the code/document changes have been inspected and committed.

## 14. Architecture decision records

Record major decisions under `docs/decisions/`. Each ADR should include context, decision, alternatives, implications, licensing impact, security impact, evidence, and supersession status.

Initial required decisions:

- Desktop shell and UI stack.
- PDF editing engine and license compatibility.
- OCR packaging and supported languages.
- Worker protocol and process isolation.
- Persistence and index retention.
- Local model runtime and model distribution terms.
- Cross-platform installer/update strategy.
- Project distribution license and third-party notice policy.

## 15. Definition of done

A change is complete only when:

1. Requirements and architecture remain consistent.
2. Implementation and applicable tests are present.
3. Lint, type-check, build, unit and integration tests pass where configured.
4. Relevant failure paths and security implications are tested.
5. Actual runtime behavior is inspected on the applicable platforms.
6. Dependencies and licensing changes are documented.
7. `README.md` and related documentation reflect verified status.
8. `git diff --check` and a complete diff review show no unintended changes.
9. The change is committed with a meaningful message.
10. Status reporting distinguishes passed, failed, skipped and unverified checks.

## 16. Current status

At the time this baseline was drafted, repository inspection showed `README.md` and `LICENSE` on the default branch. This document defines the target architecture. It does not claim that application modules, automated tests, build tooling, installers, or any of the described product features have been implemented.
