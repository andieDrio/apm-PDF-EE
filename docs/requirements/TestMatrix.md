# Test and Release Matrix — apm-PDF-EE

**Status:** Planned validation baseline; no application test suite is configured yet.
**Primary development host:** MacBook Pro running macOS Tahoe.
**Target platforms:** macOS, Windows, Linux.

## 1. Validation principles

- Report only checks that were actually run.
- Separate automated tests, manual runtime checks, and external platform checks.
- Do not treat successful compilation as proof of document fidelity or security.
- Preserve reproducible fixtures and avoid committing confidential user documents.
- Mark each check PASS, FAIL, SKIPPED, NOT RUN, or UNVERIFIED with evidence.

## 2. Platform matrix

| Area | macOS Tahoe | Windows (supported version to be selected) | Linux (supported distributions to be selected) |
|---|---|---|---|
| Desktop startup/shutdown | Primary local validation | CI/clean-machine validation required | CI/clean-machine validation required |
| Open/save dialogs and paths | Required | Required | Required |
| Worker process lifecycle | Required | Required | Required |
| PDF rendering and editing | Golden corpus | Golden corpus | Golden corpus |
| OCR executable/language packs | Offline test | Offline test | Offline test |
| File permissions/temp cleanup | Required | Required | Required |
| Offline/network behavior | Required | Required | Required |
| Installer and clean install | Required | Required | Required |
| Accessibility/keyboard | Required | Required | Required |
| Performance baseline | Record hardware/software | Record runner/hardware | Record runner/hardware |

Windows versions and Linux distributions must be chosen and documented before Gate 01's platform acceptance criteria are finalized. macOS Tahoe is the primary development environment, not evidence of cross-platform compatibility.

## 3. Test layers

### Unit tests
Cover pure domain logic, page coordinate transformations, validation, job-state transitions, cancellation semantics, error mapping, settings, and permissions.

### Contract tests
Validate typed requests and responses across UI, native host, workers, persistence and engine adapters. Reject invalid schemas, unsupported protocol versions and oversized payloads.

### Integration tests
Exercise real PDF files through open, inspect, edit, OCR, export, validation, cancellation, failure cleanup, and persistence.

### Golden-document and visual regression tests
Maintain fixtures for:
- Digitally generated text PDFs.
- Embedded and subset fonts.
- Mixed text, vector, and raster content.
- Scanned and skewed documents.
- Multilingual pages, including representative English and Filipino samples.
- Tables, columns, unusual encodings and rotated text.
- Forms, annotations, bookmarks, links, attachments and metadata.
- Encrypted PDFs and signed PDFs.
- Large documents and malformed/adversarial inputs.

Record expected extracted text, page count, page boxes, key metadata, annotation/form preservation, and rendered output. Use documented visual comparison tolerances; investigate unexpected differences instead of automatically updating baselines.

### Security tests
Cover malformed inputs, decompression/resource exhaustion, embedded actions, file-path attacks, unexpected network attempts, log leakage, worker crashes, temporary-file cleanup, and redaction leakage.

### End-to-end tests
Verify user workflows through the packaged desktop UI: open, search, edit, save as, OCR, inspect, redact, verify output, and batch-process documents.

### Performance tests
Measure first-page render, navigation latency, memory use, OCR throughput, batch-job behavior, cancellation responsiveness, and optional AI inference latency. Record hardware, OS, input corpus and test method.

## 4. Gate-specific validation

| Gate | Required evidence |
|---|---|
| 00 — Baseline | Threat model, dependency policy, ADRs, test matrix, canonical docs consistency |
| 01 — Desktop shell | App starts on each target OS, IPC validation, worker lifecycle, basic CI |
| 02 — Viewer | Render/search/navigation tests, source preservation, large-file responsiveness |
| 03 — Editing spike | Document corpus results, edit fidelity, known limitations, license decision |
| 04 — Editing/export | Structural and visual regression, forms/annotations/page operations |
| 05 — OCR | Offline execution, language-pack checks, quality measurements, batch failures |
| 06 — Security | Hostile input, redaction leakage, sanitization and signature test results |
| 07 — AI | Offline execution, evidence grounding, prompt injection, model terms and resource measurements |
| 08 — Release | Clean installs, platform tests, notices/inventory, vulnerability and release artifact review |

## 5. Initial local validation on macOS Tahoe

Before application scaffolding exists, documentation changes can be reviewed for Git whitespace and consistency, but frontend, Python, Rust, build, and runtime tests are NOT APPLICABLE / NOT RUN. Do not initialize toolchains or create application source files before the repository baseline and Gate 00 decisions are reviewed.

After Gate 01 selects and configures the toolchains, record the exact versions and use the committed project scripts. Likely checks include frontend lint/typecheck/tests, Python tests and Ruff, Rust formatting/Clippy/tests, packaging, and Git diff checks. Use only commands that actually exist in the repository.

## 6. Release evidence record

For every release candidate, record:
- Commit SHA and clean/dirty working-tree state.
- OS version, architecture, hardware and toolchain versions.
- Exact commands, exit codes and test summaries.
- Platforms and test categories skipped, with reasons.
- PDF corpus version and known limitations.
- Dependency inventory, license notices, vulnerability review and exceptions.
- Packaging/install results and offline behavior.
- Open blockers and risk acceptance, if any.

No release should be described as cross-platform validated unless the required evidence exists for all supported platforms.
