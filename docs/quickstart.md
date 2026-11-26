# Quick start

Use a fresh clone of [Template Code Project](https://github.com/docxology/template_code_project)
to run the optimizer, generate its experiment artifacts, and render the paper.
Install Git and [uv](https://docs.astral.sh/uv/). The standalone package supports
Python 3.12 and later. The pinned rendering environment uses Python 3.14.

## Set up the project and template

Run the following from the Template Code Project checkout.

```bash
project_root="$(pwd)"
template_root="$(mktemp -d)/template"
git clone https://github.com/docxology/template "$template_root"
git -C "$template_root" checkout --detach 87704feae0a4490ef7dad9c931d268e8c30273c2

uv sync --locked --extra dev --python 3.14
(cd "$template_root" && uv sync --locked --no-dev --python 3.14)
export PYTHONPATH="$template_root${PYTHONPATH:+:$PYTHONPATH}"
```

`PYTHONPATH` exposes the pinned template's infrastructure to the standalone
project's analysis scripts and integration tests. The optimizer itself can be
imported from the standalone package with NumPy.

## Run tests and generate the evidence

```bash
uv run --locked --extra dev --python 3.14 pytest tests/ --cov --cov-config=pyproject.toml --cov-fail-under=90
uv run --locked --extra dev --python 3.14 python scripts/optimization_analysis.py
uv run --locked --extra dev --python 3.14 python scripts/z_generate_manuscript_variables.py
uv run --locked --extra dev --python 3.14 python scripts/build_dashboard.py
```

The analysis writes results to `output/data/optimization_results.csv` and six
manuscript plots under `output/figures/`. Hydration reads the analysis data and
writes `output/data/manuscript_variables.json` and numbered sections in
`output/manuscript/`. The dashboard is `output/web/dashboard.html`.

Change the optimizer inputs under `experiment:` in
[the manuscript config](manuscript/config.yaml). Update the numbered Markdown
files in [the manuscript directory](manuscript/README.md) and regenerate the
evidence and hydrated sections after a source edit.

## Render and validate

Install Pandoc, a compatible [pandoc-crossref](https://github.com/lierdakil/pandoc-crossref/releases)
release, XeLaTeX, and the [pinned template's TeX prerequisites](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/README.md#L565-L574).
Install the [JuliaMono v0.63.2 TTF family](https://github.com/cormullion/juliamono/releases/tag/v0.63.2)
where XeLaTeX can find it. Confirm `kpsewhich JuliaMono-Regular.ttf` resolves
the font, or include the directory of extracted TTF files in `TTFONTS`.
The project config enables PDF, HTML, slides, DOCX, and EPUB.

```bash
render_project="$template_root/projects/working/template_code_project"
mkdir -p "$template_root/projects/working"
git clone "$project_root" "$render_project"
mv "$render_project/output" "$template_root/published-code-project-output"

(
  cd "$render_project" &&
  "$template_root/.venv/bin/python" scripts/optimization_analysis.py &&
  "$template_root/.venv/bin/python" scripts/build_dashboard.py
)

(
  cd "$template_root" &&
  uv run --no-sync python scripts/pipeline/stage_03_render.py --project working/template_code_project &&
  uv run --no-sync python scripts/pipeline/stage_04_validate.py --project working/template_code_project
)
```

Use a committed source revision. The template's supervised hydration requires
a real project checkout beneath its `projects/working/` directory. The render
copy starts with a fresh output tree and the template reruns strict hydration
during Stage 03.

The template reads source configuration from `docs/manuscript/` and renders
the hydrated sections. Under `$render_project`, inspect `output/pdf/template_code_project_combined.pdf`,
`output/web/index.html`, and `output/reports/validation_report.md`. The standalone repository tracks a published snapshot under `output/`.
Use a disposable clone for any local generation commands that write there.

See the [rendering pipeline](rendering_pipeline.md), [output inventory](output_inventory.md),
and [troubleshooting guide](troubleshooting.md) for details.
