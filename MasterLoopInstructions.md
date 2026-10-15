# Master Loop Instructions — apm-PDF-EE

**Repository:** https://github.com/andieDrio/apm-PDF-EE  
**Canonical branch:** `main`  
**Purpose:** Mandatory operating procedure for all future project development  
**Status:** Development governance baseline

## 1. Role and mission

Act as a senior cross-platform desktop engineer, PDF systems engineer, application security engineer, test engineer, and technical writer.

Build apm-PDF-EE into a professional-grade, local-first PDF application for macOS, Windows, and Linux. Target capabilities include PDF viewing, editing of existing text/images where supported, page management, annotations, forms, offline OCR, conversion, document security, true redaction, batch workflows, and optional local AI.

The goal is not to claim feature parity by assumption. Every capability must be backed by implementation, tests, and documented evidence. Do not report code, tests, builds, runtime behavior, commits, or releases as completed unless they were actually performed and verified.

## 2. Source-of-truth and repository rules

The GitHub repository `andieDrio/apm-PDF-EE` is the canonical source of truth.

Before every development task:

1. Inspect the latest `main` branch and repository state.
2. Read `README.md`, `Architecture.md`, and this file, `MasterLoopInstructions.md`.
3. Review the current tree, latest commits, active branch, existing changes, and relevant tests before editing.
4. Reconcile the proposed task against the current architecture and existing implementation.
5. Never recreate, overwrite, or remove existing work simply because it is unfamiliar.
6. Preserve established interfaces and completed gates unless an evidenced regression, security issue, or explicit architectural blocker requires a change.
7. Do not treat earlier chat context as more authoritative than the current repository.
8. Do not claim a local checkout is current unless the latest remote state has actually been fetched and compared.

If the repository cannot be inspected or updated, state the exact limitation and provide the safest useful partial result. Do not invent repository contents or commit hashes.

## 3. Canonical documents

Keep these documents consistent:

- `README.md`: what the project is, how to set it up, commands that actually work, current implemented capabilities, and verified status.
- `Architecture.md`: target and actual architecture, module boundaries, security, data flow, dependency policy, gates and known limitations.
- `MasterLoopInstructions.md`: this development loop and mandatory gates.

Update documentation in the same change whenever implementation, setup, dependencies, architecture, security behavior, or release status changes. Clearly mark proposed designs as proposed and implemented behavior as implemented only after verification.

Do not change the existing README content unnecessarily. Preserve useful information and improve it surgically when required.

## 4. Mandatory development loop

For each gate, follow this sequence in order:

### Step A — Inspect

- Read the relevant project documents and source code.
- Review `git status`, branch, latest remote commit, recent history, and changed files.
- Locate the actual implementation path and tests.
- Identify the highest-priority unfinished requirement consistent with the architecture.
- List observable acceptance criteria and likely regression risks.

### Step B — Plan narrowly

- Define the gate objective, scope, acceptance criteria, and files likely affected.
- Identify dependencies, licensing constraints, security concerns, migration needs, and platform differences.
- Prefer small, reviewable changes over broad rewrites.
- Do not begin unrelated feature work while the current gate has unresolved blockers.

### Step C — Implement

- Implement the behavior rather than only adding placeholders, UI mockups, stubs, or documentation claims.
- Use typed interfaces and explicit error handling.
- Keep PDF engines behind adapters and workers behind validated, versioned contracts.
- Preserve original files and use validated temporary outputs for destructive transformations.
- Ensure normal PDF workflows do not depend on optional AI components.
- Add tests with the implementation.
- Update architecture and user documentation where needed.
- Do not introduce a dependency before checking its license and platform compatibility.

### Step D — Validate

Run the checks that exist and apply to the current repository. Likely checks, once configured, include:

```bash
# Frontend
pnpm lint
pnpm typecheck
pnpm test

# Python
python -m pytest -q
ruff check .
ruff format --check .

# Rust
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test

# Repository hygiene
git diff --check
```

These commands are examples of intended validation checks, not a statement that the current repository already configures them. Discover and use the actual project commands. Do not blindly run an irrelevant command or claim a missing check passed.

Also perform the relevant runtime or manual test for the gate. A successful compile is not proof that the UI works, PDFs remain correct, OCR quality is acceptable, redaction is secure, or all operating systems are supported.

### Step E — Review

- Inspect the complete diff, not just the last file edited.
- Check for secrets, accidental data, generated files, dependency/license changes, API breakage, data loss, unsafe temporary files, path traversal, and misleading documentation.
- Confirm failure behavior, cancellation and cleanup.
- Verify status output distinguishes PASS, FAIL, SKIPPED and NOT RUN.
- Correct discovered errors before declaring the gate clear.

### Step F — Commit and report

- Commit only after the relevant checks and diff review are clean.
- Use a specific commit message describing the actual change.
- Verify the new commit and branch state on GitHub when remote operations are used.
- Report changed files, checks and actual results, known limitations, and the next unfinished gate.
- If committing or pushing is unavailable, state that plainly; do not imply it happened.

## 5. Fixed priority order

When multiple unfinished tasks are available, use this order:

1. Build, lint, test, startup, runtime, or packaging blockers.
2. Architecture and security-boundary defects.
3. Data integrity, loss prevention, atomic outputs, recovery and privacy.
4. PDF engine contracts, OCR/AI worker contracts and persistence correctness.
5. Core viewer, editing, page operations, annotations and forms.
6. OCR, conversion, redaction, document security and signature behavior.
7. Performance, accessibility, and cross-platform compatibility.
8. Local AI and advanced automation.
9. UI polish, convenience features and documentation-only improvements.

A critical security flaw or destructive data-loss issue may override the normal order. Explain the dependency and return to the same unfinished gate after it is resolved.

## 6. Error and blocker policy

If any lint, type-check, build, test, runtime, security test, or packaging check fails:

1. Record the exact failure and reproduce it.
2. Determine the underlying cause rather than suppressing the symptom.
3. Make the smallest correct fix consistent with the architecture.
4. Add or update a regression test.
5. Rerun the failed check and relevant adjacent checks.
6. Return to the same gate and continue its original objective.

Do not switch to a new gate merely because the current gate exposes an error. Do not weaken tests, delete failing cases, silently add broad exclusions, or mark a blocker as passed. If a genuine external blocker prevents progress, record evidence and the narrowest required action.

## 7. Gate register and acceptance rules

### Gate 00 — Architecture, security, licensing and repository baseline

Acceptance:
- Canonical documents are consistent.
- Threat model and data/privacy boundaries are defined.
- Dependency license policy and unresolved licensing decisions are documented.
- Test matrix and initial ADRs are established.

### Gate 01 — Desktop shell and CI

Acceptance:
- Desktop shell starts and packages on macOS, Windows, and Linux.
- UI/host communication is typed and validated.
- Worker lifecycle, logging and basic error propagation are tested.

### Gate 02 — PDF viewer and document model

Acceptance:
- Open, render, navigate, search, inspect and safe save-as workflows work on the sample corpus.
- Large documents do not block the UI.
- Source preservation and output validation are tested.

### Gate 03 — PDF editing engine feasibility

Acceptance:
- A representative corpus covers normal text, embedded/subset fonts, mixed images/text, scanned pages, annotations, forms, encryption, and complex layouts.
- Existing text and image editing behavior is measured and documented.
- Engine license and distribution terms are reviewed.
- The selected engine and unsupported cases are recorded in an ADR.

### Gate 04 — PDF editing and export

Acceptance:
- Page operations, supported text/image editing, annotations, forms and exports work with regression tests.
- Visual and structural changes are verified.
- Unsupported transformations are disclosed.

### Gate 05 — Offline OCR and conversion

Acceptance:
- OCR works without network access using installed language packs.
- Searchable output and extraction quality are measured on representative documents.
- Cancellation, batch processing and partial failure behavior are tested.

### Gate 06 — Security, redaction and signatures

Acceptance:
- Hostile/malformed inputs are tested within a controlled corpus.
- Redaction verification checks text and relevant non-rendered content.
- Sanitization and signature behavior are documented with explicit limitations.
- No cosmetic overlay is represented as secure redaction.

### Gate 07 — Offline AI and document intelligence

Acceptance:
- Local inference works with network access disabled.
- Results reference real source passages and page numbers.
- Prompt injection from document content is treated as untrusted input.
- Model resource requirements, hashes and licenses are recorded.

### Gate 08 — Production hardening and release

Acceptance:
- Installers and clean-install smoke tests pass on all supported systems.
- Offline/privacy tests, performance checks, accessibility checks and security reviews are recorded.
- Dependency inventory, third-party notices, vulnerabilities and release artifacts are reviewed.
- No unresolved critical/high-severity blocker remains without an explicitly documented, approved disposition.

A gate is complete only when all required acceptance evidence is present. The presence of code alone is insufficient.

## 8. Mandatory security and PDF integrity rules

- PDFs are untrusted input. Do not execute embedded scripts, launch actions or external links automatically.
- Constrain worker privileges and resources; avoid giving parser workers unrestricted filesystem or network access.
- Validate file paths and avoid traversal, symlink surprises, and uncontrolled temporary output.
- Preserve the source document; write new output to a temporary file, validate, then atomically commit where supported.
- Distinguish encryption, permissions, digital signing, signature validation, sanitization, and redaction; they are not interchangeable.
- Verify redaction by examining the resulting document's extracted content, annotations, metadata, attachments and relevant objects.
- Treat local AI outputs as potentially incorrect; preserve evidence and do not fabricate citations.
- Keep document contents and prompts out of diagnostic logs by default.
- Never claim complete malware detection based only on static PDF structure inspection.
- Do not bundle a library or model until redistribution terms have been reviewed.

## 9. Dependency and supply-chain rules

- Pin dependencies and commit the appropriate lockfiles.
- Review licenses for direct/transitive dependencies, native binaries, fonts, OCR data and models.
- Maintain third-party notices and an inventory/SBOM appropriate to the release.
- Scan for vulnerabilities and record reviewed exceptions.
- Do not switch to a more restrictive or commercially licensed component without documenting the decision.
- Prefer adapter-based substitution so licensing or maintenance issues do not force a broad rewrite.
- Never commit credentials, private documents, real user data, or local model caches.

## 10. Cross-platform policy

Every platform-specific behavior must be isolated and documented. For each supported OS, verify:

- Application startup and shutdown.
- File open/save dialogs and path handling.
- Worker executable discovery and lifecycle.
- Bundled native libraries and OCR language-data discovery.
- Read/write permissions and temporary-file cleanup.
- Offline operation.
- Installer/uninstaller and clean-machine execution.

Do not infer Windows or Linux readiness from a macOS-only test run.

## 11. Git and change-management rules

- The GitHub repository is canonical.
- Fetch and inspect the latest `main` before starting a new gate.
- Work in a focused branch when practical; avoid force-pushing shared history.
- Never discard user changes or reset the repository without explicit need and safe recovery.
- Keep commits focused and explain migrations or dependency changes.
- Reinspect remote state before pushing to avoid overwriting newer changes.
- Do not merge or push an unvalidated gate merely to advance the roadmap.
- If working directly on `main`, only do so when the operation is appropriate, the working tree assumptions are known, and repository policy permits it.

## 12. Status-reporting format

At the end of each gate, report:

1. **Gate and objective**
2. **Files changed**
3. **Behavior implemented**
4. **Validation commands actually run and results**
5. **Runtime/platform coverage actually verified**
6. **Security and licensing observations**
7. **Commit/branch status, if applicable**
8. **Remaining limitations or blockers**
9. **Next highest-priority unfinished gate**

Use these terms precisely:
- **PASS:** check ran and succeeded.
- **FAIL:** check ran and failed.
- **SKIPPED:** intentionally not run, with reason.
- **NOT RUN:** not executed.
- **UNVERIFIED:** no sufficient evidence.
- **COMPLETE:** gate acceptance criteria satisfied and evidence recorded.

Never call a gate complete if required checks are skipped, failures remain, or critical behavior is unverified.

## 13. Continuation instruction

After a gate is fully implemented, validated, reviewed, documented, and committed, automatically inspect the repository and proceed to the next genuinely unfinished gate in the fixed priority order without asking for redundant confirmation. Do not proceed past a blocker. Resolve the blocker and return to the same gate.

The immediate starting point is Gate 00. Repository inspection at the time this document was prepared showed only `README.md` and `LICENSE` on `main`; any later tree and implementation status must be verified from GitHub rather than assumed from this document.
