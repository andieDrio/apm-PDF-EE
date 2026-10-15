# Security Threat Model — apm-PDF-EE

**Status:** Baseline for design and implementation; controls are not yet implemented or verified.
**Scope:** Local-first desktop PDF application for macOS, Windows, and Linux.
**Review cadence:** At every security-relevant architecture change and before each production release.

## 1. Security objectives

1. Preserve confidentiality of local documents, OCR text, search indexes, prompts, model data, and derived artifacts.
2. Preserve integrity of source documents and exported outputs.
3. Process malformed or hostile documents without granting them arbitrary code execution, filesystem access, or network access.
4. Make destructive operations, especially redaction and sanitization, verifiable.
5. Avoid silently executing document actions or contacting external services.
6. Report security findings accurately without claiming that static inspection proves a file is safe.

## 2. Assets

- Original PDFs and other input documents.
- Exported documents and revision history.
- Extracted text, OCR output, thumbnails, temporary files, and search indexes.
- Local AI models, embeddings, prompts, responses, and source passages.
- Encryption passwords and signing material handled by the application.
- User settings, audit events, job history, and file paths.
- Application binaries, worker executables, native libraries, language packs, and update artifacts.

## 3. Trust boundaries

- **Untrusted input boundary:** every imported PDF, image, attachment, font, embedded file, and extracted string.
- **UI-to-host boundary:** desktop UI commands must be schema-validated and capability-limited.
- **Host-to-worker boundary:** worker requests and responses must be validated; worker crashes must not corrupt host state.
- **Worker-to-filesystem boundary:** workers should receive only the input/output paths and capabilities needed for a job.
- **AI boundary:** document text is untrusted data and must never be treated as trusted system instructions.
- **Dependency/update boundary:** downloaded binaries, OCR language data, and model files require source, integrity, and license verification.
- **OS boundary:** file permissions, process isolation, and packaging behavior differ by platform and must be tested per OS.

## 4. Threat register

| ID | Threat | Required mitigation | Verification evidence |
|---|---|---|---|
| T-01 | Malformed PDF exploits a parser or exhausts resources | Isolated worker where practical; patched dependencies; page, size, time, memory and decompression limits | Malformed corpus, timeout and resource-limit tests |
| T-02 | Embedded JavaScript, launch action, or external link executes unexpectedly | Never auto-execute document actions; require explicit user action for external navigation | Fixture tests and runtime inspection |
| T-03 | PDF content triggers local AI prompt injection | Treat all extracted content as untrusted; delimit evidence; restrict tools and filesystem/network capabilities | Adversarial prompt-injection corpus |
| T-04 | Sensitive data leaks through network requests | Local-first defaults; no remote inference or telemetry by default; monitor network behavior | Offline tests with network blocked and connection monitoring |
| T-05 | Path traversal or symlink race overwrites an unintended file | Canonicalize/validate destinations, use safe temp directories, avoid trusting document-supplied paths | Cross-platform path tests |
| T-06 | Failed transformation corrupts or replaces the source | Source preservation, temporary output, validation, atomic replacement where supported | Failure injection and source hash checks |
| T-07 | Cosmetic redaction leaves recoverable text or attachments | Dedicated redaction operation; inspect text, annotations, metadata, attachments and relevant objects after export | Redaction leakage test corpus |
| T-08 | Logs or crash reports expose document contents | Structured, sanitized logs; no document text, secrets, or prompts by default | Log assertions and privacy review |
| T-09 | Malicious or incompatible dependency/model is bundled | Pinned dependencies, license inventory, hashes, vulnerability review, provenance | Release inventory/SBOM and compliance checklist |
| T-10 | Local index exposes deleted or stale document contents | Explicit retention/deletion, rebuildable indexes, lifecycle cleanup | Deletion and index consistency tests |
| T-11 | Signature trust is overstated or signed file is altered | Distinguish cryptographic integrity from certificate trust; preserve original; warn on modifications | Signed fixtures and documented validation cases |
| T-12 | Worker compromise escapes its intended boundary | Least privilege, constrained IPC, no shell interpolation, no network by default, OS-specific sandboxing where feasible | Threat-specific isolation tests per platform |
| T-13 | External update or model download is tampered with | Explicit source, TLS, integrity hashes/signatures where available, user-visible provenance and license | Tamper/hash mismatch tests |
| T-14 | Unsupported PDF feature is silently dropped | Capability detection, warnings, preservation tests, documented limitations | Round-trip tests for forms, annotations, bookmarks and attachments |

## 5. Security design requirements

- PDF parsing and conversion must not run with unnecessary privileges. Use worker-process isolation where practical; document any platform where a stronger sandbox is unavailable.
- Workers must not expose a general-purpose network listener. If a local transport is unavoidable, bind narrowly, authenticate it, validate origin/request data, and document its threat model.
- Never interpolate document-controlled strings into shell commands. Prefer argument arrays and fixed executable paths.
- Apply configurable limits to input size, page count, object/attachment counts, decompression, runtime, memory and batch concurrency.
- Avoid automatic execution of JavaScript, launch actions, embedded executables, or remote resources.
- Do not follow document links automatically. Display destination information and require an explicit user action.
- Store temporary files with restrictive permissions where supported and remove them after completion or cancellation.
- Preserve the source document until the output passes relevant structural and content validation.
- For redaction, verify the produced artifact rather than trusting the renderer or the appearance of the page.
- For encryption and signing, do not log passwords, private keys, or sensitive certificate material.
- Provide clear warnings for unsigned/untrusted certificates, invalid signatures, unsupported encryption, and transformations that may invalidate signatures.
- Keep security checks independent from optional AI services.
- Do not label a document “safe” solely because a scan found no known suspicious structures.

## 6. Privacy contract

Default behavior must be local. The application must not transmit documents, extracted text, OCR results, embeddings, prompts, model responses, or usage telemetry to remote services without explicit, informed configuration.

- Core workflows must work offline once application dependencies and selected language/model data are installed.
- Optional network-dependent functionality must be visibly identified and disabled by default.
- Search indexes, AI history, caches, and temporary artifacts require a documented retention and deletion policy.
- Diagnostic reports must omit document content and secrets by default.
- Tests must verify that offline workflows do not attempt remote requests.

## 7. Residual risks and limitations

- Parser vulnerabilities can exist in any third-party PDF engine; dependency updates and isolation reduce but do not eliminate risk.
- OS sandboxing and packaging differ across macOS, Windows, and Linux.
- Static structural inspection cannot prove a PDF is benign.
- Redaction verification can be incomplete if a library does not expose all relevant object types; unsupported cases must fail closed or be explicitly warned.
- Digital-signature trust may require certificate-chain and revocation data that are not available offline. Report offline limitations accurately.
- Local AI may hallucinate or be manipulated by document content; responses require evidence and user review.

## 8. Release security gate

A release candidate cannot be marked security-ready until the applicable hostile-input tests, redaction leakage tests, offline/network tests, dependency/license review, file-integrity tests, and platform-specific packaging checks have recorded results. Skipped or unavailable tests must be called out explicitly.
