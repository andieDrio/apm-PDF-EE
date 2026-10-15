# ADR-0003: Local-first processing and worker isolation

- **Status:** Accepted as a design requirement; implementation validation pending.
- **Date:** 2026-10-15.
- **Decision owners:** Project maintainer.

## Context

The suite will process potentially sensitive PDFs and untrusted embedded content. OCR and local AI can generate sensitive derived data. Network transmission must not be implicit.

## Decision

Core viewing, supported editing, page operations, OCR, security inspection, and export must work offline after required dependencies and data are installed. Remote inference and telemetry are not enabled by default. Document processing should occur in constrained worker processes where practical, with typed requests, resource limits, narrow filesystem access, and no network access by default.

The source file must remain intact until a validated output is committed. Redaction is a separate destructive transformation with post-export verification.

## Consequences

- The application must manage local language packs, model files, caches, indexes and cleanup.
- Workers need robust crash, cancellation and timeout behavior.
- Platform-specific process isolation must be evaluated individually.
- Local AI remains optional and must not block normal PDF workflows.
- Offline certificate/revocation checks may be incomplete and must be reported honestly.

## Validation required

- Run core workflows with network access disabled and monitor for attempted external requests.
- Verify source hashes remain unchanged after failed jobs.
- Test worker termination, timeout, cancellation and cleanup.
- Test redaction against extracted text, metadata, annotations, attachments and relevant objects.
- Test AI prompt-injection fixtures and ensure document content cannot invoke arbitrary tools.
