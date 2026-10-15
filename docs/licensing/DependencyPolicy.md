# Dependency and Licensing Policy — apm-PDF-EE

**Status:** Proposed project policy; not legal advice.  
**Project license currently in repository:** MIT.  
**Policy owner:** Project maintainer.

## 1. Objective

Keep the application usable locally without a mandatory subscription and prefer open-source components. Avoid accidental license incompatibility, unreviewed redistribution obligations, and undocumented model/data terms.

The repository's MIT license applies to this repository's own covered material. It does not automatically relicense third-party dependencies, fonts, OCR language data, native libraries, or AI models.

## 2. Default selection rules

1. Prefer actively maintained projects with clear upstream licensing and security processes.
2. Prefer permissive licenses compatible with the intended distribution model where technical capability is adequate.
3. Do not reject a reciprocal license automatically; first document its obligations and decide whether the project can comply.
4. Do not introduce a commercial-only or source-available dependency without explicit approval and an architecture decision record.
5. Keep dependencies behind adapters when feasible to reduce lock-in.
6. Pin versions and commit lockfiles for all package managers.
7. Review direct and transitive dependencies, native binaries, codecs, fonts, OCR language packs, model runtimes, and model weights.
8. Do not infer a model's license from the runtime license or a repository's marketing description.
9. Do not bundle assets whose redistribution terms are unclear.
10. Re-review licenses when versions, build flags, distribution method, or product licensing changes.

## 3. Preliminary candidate register

This register is a starting point, not a completed legal clearance.

| Component | Intended use | Preliminary licensing consideration | Decision |
|---|---|---|---|
| Tauri 2 / Rust ecosystem | Desktop host and process boundary | Review Tauri, plugins, Rust crates, and platform binaries individually | Candidate; audit before pinning |
| React | UI | MIT | Candidate |
| TypeScript | UI language/tooling | Apache-2.0 | Candidate |
| PDF.js | PDF rendering/viewing | Apache-2.0 | Candidate; verify exact distribution and notices |
| pypdf | Basic structural PDF operations | BSD-3-Clause | Candidate; test feature coverage |
| OCRmyPDF | OCR orchestration | MPL-2.0; review file-level obligations and bundled dependencies | Candidate; compliance review required |
| Tesseract | OCR engine | Apache-2.0; inspect native dependencies and language-data terms | Candidate; package audit required |
| PyMuPDF | Advanced PDF manipulation candidate | AGPL-3.0 or commercial licensing path, depending on use/distribution terms | **Not approved as mandatory** pending explicit licensing decision |
| SQLite | Local metadata/index database | Public-domain dedication for SQLite core; review bundled wrappers | Candidate |
| Local AI runtime | Optional local inference | Runtime, backend, model weights, tokenizer, and model-specific terms may differ | No model approved yet |

This table does not guarantee that a specific version or distribution bundle is compliant. Confirm the authoritative license files and notices for the exact versions chosen.

## 4. Approval process for a dependency

Before adding a dependency:

1. Record its name, exact version, source URL, purpose, direct/transitive status, and platform coverage.
2. Record the SPDX license identifier where known and link the authoritative license.
3. Identify any exceptions, dual-license terms, attribution, source-offer, notice, patent, trademark, network-use, or redistribution requirements.
4. Review the dependency tree and bundled binary/data assets.
5. Check maintenance activity, known vulnerabilities, and relevant security advisories.
6. Test installation and operation on the supported target platforms.
7. Record the decision in `docs/decisions/`.
8. Update the lockfile, dependency inventory, and `THIRD_PARTY_NOTICES.md` when applicable.

If license terms are unclear, do not bundle the component until the ambiguity is resolved.

## 5. AI model and data policy

For each model or data package, record:

- Publisher and source.
- Exact model identifier and revision.
- Download URL and integrity hash.
- Model license and acceptable-use restrictions.
- Whether commercial use, redistribution, and bundling are allowed.
- Required notices and attribution.
- Expected hardware, memory and storage footprint.
- Whether the application downloads it or the user imports it.

A permissively licensed inference runtime does not imply permissively licensed model weights.

## 6. Distribution policy decision

The existing repository license is MIT. This policy does not change it.

Before the first distributable build, the project owner must explicitly decide whether the application will remain MIT/permissive-compatible or whether components with reciprocal obligations may be included under a compliance plan. Any change to the project's own license requires a separate, deliberate decision.

## 7. Release compliance checklist

- [ ] Direct and transitive dependencies inventoried.
- [ ] License identifiers and authoritative license texts checked.
- [ ] Native binaries, fonts, codecs, OCR language data and AI models reviewed.
- [ ] Required notices and attribution included.
- [ ] Vulnerability review performed and exceptions documented.
- [ ] Lockfiles committed and dependency versions reproducible.
- [ ] No unapproved commercial or source-available dependency bundled.
- [ ] Model hashes, versions and terms recorded.
- [ ] Third-party notice file matches the actual release bundle.
- [ ] Final artifact reviewed before publication.

## 8. Disclaimer

This policy is an engineering control, not a substitute for qualified legal advice where licensing obligations are uncertain.
