# Troubleshooting

Use the [quick start](quickstart.md#render-and-validate) to prepare the pinned Template checkout
and register the standalone project before trying these fixes. In the commands below,
`$render_project` points to the registered checkout of the committed source you are troubleshooting.

## Literal `{{TOKEN_NAME}}` appears in the rendered PDF

**Cause.** A placeholder in `docs/manuscript/*.md` was not substituted because
`scripts/z_generate_manuscript_variables.py` was not (re)run after the
analysis stage, or the token is not defined in `src/template_code_project/core/manuscript_variables.py::generate_variables()`.

**Fix.**

1. Run hydration in the registered checkout.
   ```bash
   (cd "$render_project" && "$template_root/.venv/bin/python" scripts/z_generate_manuscript_variables.py)
   ```
2. Inspect the resolved values.
   ```bash
   jq . "$render_project/output/data/manuscript_variables.json"
   ```
3. [Render and validate](quickstart.md#render-and-validate) with the pinned Template checkout.
4. If the token is still literal, add it to `generate_variables()` in
   `src/template_code_project/core/manuscript_variables.py`.

## `FileNotFoundError`: missing `optimization_results.csv` during variable hydration

**Cause.** `scripts/z_generate_manuscript_variables.py` runs in strict mode by default (`require_analysis_outputs=True`) and requires `output/data/optimization_results.csv` from the analysis stage.

**Fix.**

1. Run analysis first:
   ```bash
   uv run python scripts/optimization_analysis.py
   ```
2. Re-run variable hydration:
   ```bash
   uv run python scripts/z_generate_manuscript_variables.py
   ```
3. For intentional early drafts only (tokens may fall back to `"N/A"`):
   ```bash
   uv run python scripts/z_generate_manuscript_variables.py --allow-draft
   ```

## Edited `docs/manuscript/config.yaml` but the figures or PDF didn't change

**Cause.** Stage 03 reads the registered source and existing analysis results.
A change to the experiment config requires updated analysis.

**Fix.** Register the updated committed source using the [quick start](quickstart.md#render-and-validate).
Generate the evidence in that project, then render and validate it from the pinned Template checkout.

```bash
(
  cd "$render_project" &&
  "$template_root/.venv/bin/python" scripts/optimization_analysis.py &&
  "$template_root/.venv/bin/python" scripts/z_generate_manuscript_variables.py
)
(
  cd "$template_root" &&
  uv run --no-sync python scripts/pipeline/stage_03_render.py --project working/template_code_project &&
  uv run --no-sync python scripts/pipeline/stage_04_validate.py --project working/template_code_project
)
```

## `Figure ???` in the rendered PDF

**Cause.** The Pandoc-crossref `[@fig:label]` reference does not match
any `{#fig:label}` anchor, or the figure inclusion was not resolved
before compilation.

**Fix.**

1. Verify `docs/manuscript/03_results.md` contains the image with the
   correct anchor:
   ```markdown
   ![Caption.](../../output/figures/convergence_plot.png){#fig:convergence}
   ```
2. Verify the prose reference uses the same label:
   ```markdown
   See [@fig:convergence] for the trajectories.
   ```
3. Re-run the analysis and render stages.

## BibTeX / citation errors during PDF rendering

**Symptom.** `xelatex` reports `Citation 'foo' on page X undefined` or
`bibtex` flags a malformed entry.

**Fix.**

1. Look at the LaTeX log for the exact line:
   ```bash
   grep -nE "Citation|undefined|Error" \
     output/pdf/_combined_manuscript.log
   ```
2. Check the entry exists in `docs/manuscript/references.bib` and is well-formed.
3. Ensure all required BibTeX fields are present (`author`, `title`,
   `year`, plus type-specific fields).

## Analysis script aborts with a Python error

**Fix.**

1. Re-run with the full traceback visible:
   ```bash
   uv run python scripts/optimization_analysis.py 2>&1 | tee /tmp/analysis.log
   ```
2. Check the output directory exists and is writable.
3. Validate `docs/manuscript/config.yaml`:
   ```bash
   uv run python -c "import yaml; yaml.safe_load(open('docs/manuscript/config.yaml'))"
   ```

## `uv` command not found

**Fix.** Follow the [checksum-verified uv installation instructions](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/operational/build/dependency-management.md#installing-uv) for macOS or Linux.

On macOS, you can also use Homebrew.

```bash
brew install uv
```

Make sure `~/.local/bin` (or the install location) is on your `PATH`.

## Coverage gate fails (under 90%)

**Fix.**

1. Find which lines are uncovered from the standalone checkout root.
   ```bash
   uv run --locked --extra dev pytest tests/ --cov --cov-config=pyproject.toml --cov-report=term-missing -v
   ```
2. Add tests covering the missing branches.
3. Re-run until coverage ≥ 90% and all tests pass.

## `optimization_analysis.py` fails to import `optimizer`

**Symptom.** `ModuleNotFoundError: No module named 'optimizer'`.

**Fix.** Run from the repository root and let the workspace handle paths:

```bash
uv run python scripts/optimization_analysis.py
```

## Sluggish test execution

**Cause.** Performance benchmarks use real timing with multiple
repetitions; on a slow machine they can dominate the suite.

**Fix.** From the standalone checkout root, skip them for fast iteration.

```bash
uv run --locked --extra dev pytest tests/ -k "not Performance"
```

For a parallel run, include pytest-xdist.

```bash
uv run --locked --extra dev --with pytest-xdist pytest tests/ -n auto
```

## YAML parse error in `docs/manuscript/config.yaml`

**Common mistakes.**

- Indentation uses tabs instead of spaces.
- Trailing commas (JSON style) are invalid in YAML.
- Unclosed quotes or brackets.

**Fix.** Validate the file before running:

```bash
uv run python -c "import yaml; yaml.safe_load(open('docs/manuscript/config.yaml'))"
```

## PDF Rendering fails: `mmdc` could not find Chrome

**Symptom:** the pipeline reaches **PDF Rendering** and fails (authoritative
`output/reports/pipeline_report.json` shows `"PDF Rendering" -> failed`)
even though tests, analysis, and per-section slide PDFs passed. The error
contains:

```
❌ Rendering error generating combined PDF: mmdc failed for
inline_mermaid_0001_...: Error: Could not find Chrome (ver. 131.0.6778.204).
```

**Cause:** a manuscript section embeds a ```mermaid``` block (this project's
convention is Mermaid-for-all-diagrams). The combined-PDF render shells out
to `mmdc`, which needs a **pinned** `chrome-headless-shell` in the Puppeteer
cache. Slide PDFs do not invoke `mmdc`, so they still succeed — that
asymmetry is the tell.

**Fix:** install the pinned headless Chrome once (reversible; lands in
`~/.cache/puppeteer/`):

```bash
npx --yes puppeteer browsers install chrome-headless-shell@131.0.6778.204
# or the line mmdc/puppeteer resolves by default:
npx --yes puppeteer browsers install chrome-headless-shell
(cd "$template_root" && uv run --no-sync python scripts/pipeline/stage_03_render.py --project working/template_code_project)
```

Environment provisioning, not a manuscript defect — CI provisions it; a
fresh clone does not. See
[rendering_pipeline.md](rendering_pipeline.md#prerequisite-mermaid-diagrams-need-chrome-headless-shell).

## Tests report PASSED but ran 0 tests / 0.0% coverage

**Symptom:** `scripts/pipeline/stage_01_test.py --project templates/template_code_project` prints
`Project: ✓ PASSED (0/0 tests, 0.0% coverage)` and exits 0, while the
documented direct command runs the current project suite with the measured
coverage shown in the repo canonical facts.

**Cause:** the aggregate runner resolves the interpreter from
`projects/templates/template_code_project/.venv`. A `.venv` made by `uv venv` without
`uv sync --extra dev` lacks `pytest`, so pytest collects nothing. **A green exit with
zero collected tests is not a pass.**

**Fixes:**

1. Run the coverage gate directly from the standalone checkout root.
   ```bash
   uv run --locked --extra dev pytest tests/ --cov --cov-config=pyproject.toml --cov-fail-under=90
   ```
2. For the parent Template runner, install the test dependencies in the
   exemplar directory with `uv sync --extra dev`.
3. Always confirm **collected > 0 AND coverage ≥ 90%** — never exit code
   alone. (The runner now falls back to the workspace interpreter and
   hard-fails a 0-collected or below-threshold run.)

In the standalone checkout, coverage paths are relative to the project root
and come from [`pyproject.toml`](../pyproject.toml). For parent Template
validation, see the [fork guide](standalone.md#validation-commands).

## See also

- [`output_conventions.md`](output_conventions.md) — output directory layout and regeneration rules.
- [`rendering_pipeline.md`](rendering_pipeline.md) — the four phases and their config controls.
- [`syntax_guide.md`](syntax_guide.md) — manuscript token and cross-reference syntax.
- [`quickstart.md`](quickstart.md) — basic run commands.
- [`faq.md`](faq.md) — frequently asked questions.
