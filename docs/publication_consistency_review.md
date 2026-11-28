# Publication consistency review

## Optimization claims

The current source distinguishes converged, iteration-capped and non-finite runs. For a positive-definite quadratic, contraction requires `0 < alpha < 2/lambda_max`; the unit-Hessian factor `abs(1-alpha)` is not monotonic across the configured sweep. Accuracy applies to converged rows. Scalar objective-evaluation stability is distinct from gradient-descent convergence.

## Reader verification on November 28, 2026

Use the [standalone quick start](quickstart.md) for the locked test suite and
pinned Template rendering commands. The reviewed merge at `18a65e55` passed
its project tests and all nine Stage 04 checks for PDF, HTML, slides, DOCX,
and EPUB. Page 23 of the 24-page PDF contains the References heading and all
eight bibliography entries. A focused text-bound check found the rendered
infrastructure paths within the page edge. [Reader verification](https://github.com/docxology/template_code_project/pull/19)

The 14-file `output/web/` inventory contains 11 HTML pages and their shared
source-font asset at `output/web/assets/source-web-fonts.js`. The inventory
totals 2,494,920 bytes. The complete native inventory contains 67 files and
totals 13,315,699 bytes.

Replacing the bundled Code Project output subtree on Template candidate
`bd887754` with that native inventory projects 229,193,951 tracked output bytes.
That leaves 5,687,073 bytes under Template’s repository-wide 224 MiB ceiling.
The coordinated publication check measures the exact final set across all
bundled examples. [Template output guard](https://github.com/docxology/template/blob/bd887754553cc79a127c3ac05c15b2b42b8a7c60/infrastructure/project/git_guards.py#L361-L400)

The v3.0.0 release handoff includes the final DOI-bound metadata, reviewed
reader assets, and the verification for their exact source revision.
