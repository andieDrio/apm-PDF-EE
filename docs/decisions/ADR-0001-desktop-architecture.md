# ADR-0001: Cross-platform desktop architecture

- **Status:** Accepted as the initial direction; implementation validation pending.
- **Date:** 2026-10-15.
- **Decision owners:** Project maintainer.

## Context

apm-PDF-EE targets macOS, Windows, and Linux. It requires a professional desktop UI, local file access, document-processing workers, offline OCR, optional local AI, and a strong security boundary around untrusted PDF inputs. The repository currently has documentation only.

## Decision

Use Tauri 2 with a React/TypeScript frontend and a Rust host as the leading desktop architecture candidate. Isolate document processing in worker processes and expose narrow, versioned, validated IPC contracts. Python is the leading candidate for document/OCR workers where mature library support justifies it. PDF.js is a candidate for viewing/rendering, not a full editing engine.

This decision authorizes a feasibility spike and scaffold design; it does not assert that dependencies are installed, compatible, or production-ready.

## Alternatives considered

- **Electron:** mature ecosystem, but a larger runtime and different security/packaging tradeoffs.
- **Python-only desktop UI:** may simplify worker integration but requires a separate UI/packaging evaluation for the intended polished cross-platform experience.
- **Browser-based application:** does not meet the initial desktop-first product direction without a desktop wrapper.

## Consequences

- Multiple toolchains and package managers must be pinned and maintained.
- Worker packaging and native library discovery require explicit cross-platform testing.
- IPC and filesystem permissions must be narrowly scoped.
- The desktop shell can evolve independently from the PDF engine behind internal contracts.
- The decision must be revisited if a Gate 01 spike shows unacceptable platform or packaging limitations.

## Validation required

- Build and launch a minimal shell on macOS Tahoe.
- Verify the same scaffold can be built and launched on selected Windows and Linux targets.
- Verify file dialog/path behavior, worker launch and shutdown, and restrictive IPC capabilities.
- Record exact dependency versions, licenses, build commands and known platform limitations.
