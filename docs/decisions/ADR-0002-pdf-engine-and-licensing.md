# ADR-0002: PDF engine selection and license boundary

- **Status:** Open — engine selection intentionally deferred.
- **Date:** 2026-10-15.
- **Decision owners:** Project maintainer.

## Context

The product requires viewing, page operations, annotations, forms, OCR, true redaction, and editing existing text and images. No single candidate should be assumed to meet all requirements. PDF fidelity and dependency licensing are both release-critical.

## Decision

Do not select a single mandatory full-editing engine until a representative document corpus has been tested and the exact distribution license has been reviewed.

Use an internal PDF engine adapter. PDF.js is a viewer/rendering candidate; pypdf is a structural-manipulation candidate; OCRmyPDF/Tesseract are OCR candidates subject to bundle and language-data review. PyMuPDF is not approved as a mandatory dependency until the project owner resolves its AGPL/commercial licensing implications.

## Required feasibility corpus

- Ordinary text with common fonts.
- Embedded and subset fonts.
- Mixed vector/raster pages.
- Complex columns, rotated text and unusual encodings.
- Scanned documents.
- Existing images and text objects.
- Forms, annotations, bookmarks, links and attachments.
- Encrypted and digitally signed files.
- Malformed and large documents.

## Acceptance measures

- Text/image edit success rate and known failure modes.
- Layout and font fidelity after save/reopen.
- Preservation of forms, annotations, bookmarks, metadata and attachments where required.
- Structural and visual regression results.
- Performance and memory use.
- Offline packaging and OS compatibility.
- License, notice, transitive dependency and redistribution review.

## Consequences

Engine selection may require more than one library behind a stable adapter. Some PDFs will require reconstruction or page replacement; the UI must disclose limitations. The project must not claim general Acrobat-equivalent editing until the corpus and regression evidence support that claim.

## Revisit trigger

Close this ADR only after Gate 03 has documented test results, licensing review, selected engine(s), and rejected alternatives.
