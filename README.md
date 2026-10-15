# apm-PDF-EE

**PDF Enterprise Edition** is a proposed local-first, cross-platform desktop PDF suite for macOS, Windows, and Linux. The product goal is to combine professional PDF viewing and editing, offline OCR, document security, batch processing, and optional local AI in one application.

> **Current status:** Architecture and development-governance baseline. The repository currently defines the target design and acceptance gates; the application implementation, build tooling, installers, and runtime behavior have not yet been established or validated.

## Product direction

- PDF viewing, text search, page organization, annotations, and forms.
- Existing text and image editing where the selected engine can preserve document fidelity, with explicit warnings for reconstruction and unsupported cases.
- Offline OCR and searchable PDF generation.
- Document inspection, sanitization, and verifiable permanent redaction.
- Batch processing with progress, cancellation, and safe output handling.
- Optional local AI for summaries and evidence-linked document Q&A.
- Local-first processing, with no automatic document upload or mandatory cloud service.

## Canonical project documents

- [Architecture.md](Architecture.md) — target architecture, module boundaries, security, data flow, dependency policy, and implementation gates.
- [MasterLoopInstructions.md](MasterLoopInstructions.md) — mandatory inspect → plan → implement → validate → review → commit development loop.
- [Security threat model](docs/threat-model/ThreatModel.md) — assets, trust boundaries, threat register, mitigations, and release security gate.
- [Dependency and licensing policy](docs/licensing/DependencyPolicy.md) — dependency/model review and release compliance.
- [Test and release matrix](docs/requirements/TestMatrix.md) — cross-platform test coverage and required evidence.
- [ADR-0001: Desktop architecture](docs/decisions/ADR-0001-desktop-architecture.md).
- [ADR-0002: PDF engine and licensing](docs/decisions/ADR-0002-pdf-engine-and-licensing.md).
- [ADR-0003: Local-first security](docs/decisions/ADR-0003-local-first-security.md).

## Development environment

The primary development host is a MacBook Pro running macOS Tahoe. macOS is the initial local validation environment; Windows and Linux support must be verified independently before the project can claim cross-platform readiness.

Application source code, dependency lockfiles, setup commands, and build/test scripts will be added after the baseline and scaffold decisions are complete. Do not assume that example commands in the architecture documents are already available.

## Architecture direction

The leading candidate is Tauri 2 with React/TypeScript, a Rust host, and isolated document-processing workers. PDF.js is a viewer/rendering candidate, not a full PDF editor. PDF engine selection remains open until real-document fidelity testing and license review are complete.

The repository's current [MIT license](LICENSE) does not automatically relicense third-party libraries, native binaries, OCR language data, fonts, or AI model weights. See the dependency policy before approving any bundled component.

## Implementation gates

1. **Gate 00:** Architecture, security, licensing, and repository baseline.
2. **Gate 01:** Cross-platform desktop shell and CI.
3. **Gate 02:** PDF viewer and document model.
4. **Gate 03:** PDF editing engine feasibility spike.
5. **Gate 04:** Editing and export.
6. **Gate 05:** Offline OCR and conversion.
7. **Gate 06:** Security, redaction, and signatures.
8. **Gate 07:** Offline AI and document intelligence.
9. **Gate 08:** Production hardening and release.

A gate is complete only after its acceptance criteria and validation evidence are recorded. See [MasterLoopInstructions.md](MasterLoopInstructions.md).

## Validation status

No application build, lint run, automated test suite, installer, or cross-platform runtime check is claimed at this baseline stage. Validation results must be added only after the corresponding checks are actually executed.
