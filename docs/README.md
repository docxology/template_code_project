# docs/ — Project Documentation

> **Operational rulebook** for the template_code_project exemplar

**Quick Reference:** [Agent Instructions](agent_instructions.md) | [Architecture](architecture.md) | [Testing](testing_philosophy.md) | [Rendering](rendering_pipeline.md) | [Style](style_guide.md) | [Syntax](syntax_guide.md) | [Index](AGENTS.md)

Start with the [quick start](quickstart.md) for setup, verification, and
rendering. See the [manuscript index](manuscript/README.md), [standalone
guide](standalone.md), and [project backlog](TODO.md) for source, reuse,
and maintenance.

## Purpose

The `docs/` directory groups setup, project rules, manuscript instructions, and publication evidence. The [agent index](AGENTS.md) records the rules for this checkout.

## Contents

| File | Purpose | Audience |
|---|---|---|
| [`agent_instructions.md`](agent_instructions.md) | 7 hard rules for AI agents; verification checklist | AI agents, all developers |
| [`architecture.md`](architecture.md) | Layer table, dependency direction, forbidden patterns, how-to-add-algorithm | Developers |
| [`testing_philosophy.md`](testing_philosophy.md) | Zero-mock policy, test-file inventory, coverage mechanics, the gate-vs-exit-code rule (live counts → [`COUNTS.md`](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/_generated/COUNTS.md)) | Developers, testers |
| [`rendering_pipeline.md`](rendering_pipeline.md) | Mermaid/chrome prerequisite; 4-phase manuscript→PDF pipeline; config.yaml controls | Content authors, developers |
| [`style_guide.md`](style_guide.md) | 7 rules: Zero-Mock, Infrastructure Delegation, Thin Orchestrator, Show-Not-Tell, Explicit Paths, Type Hints, Error Messages | Developers |
| [`syntax_guide.md`](syntax_guide.md) | Markdown links, LaTeX refs, all `{{VARIABLE}}` tokens, figure label registry | Content authors |
| [`output_conventions.md`](output_conventions.md) | Output directory layout, what's disposable, regeneration rules | Developers |
| [`output_inventory.md`](output_inventory.md) | Full pipeline artifact inventory (generator + stage) | Developers |
| [`publication_consistency_review.md`](publication_consistency_review.md) | Verified reader outputs, layout checks, and the tracked output budget | Authors, maintainers |
| [`forking_guide.md`](forking_guide.md) | Fork workflow and drift-check guidance | Developers, agents |
| [`troubleshooting.md`](troubleshooting.md) | Diagnosed failures incl. `mmdc`/Chrome and "PASSED but 0 tests", with fix commands | Developers |
| [`quickstart.md`](quickstart.md) | Minimal run commands to first deliverable | New users |
| [`faq.md`](faq.md) | Recurring questions: architecture, testing, manuscript | All |
| [`AGENTS.md`](AGENTS.md) | Technical index of this `docs/` folder; verification commands | Developers, agents |

## Quick Navigation

### Before Modifying Any Code

1. Read **[Agent Instructions](agent_instructions.md)** — 7 rules and the verification checklist
2. Read **[Architecture](architecture.md)** — understand layer boundaries before touching file structure
3. Read **[Testing Philosophy](testing_philosophy.md)** — understand zero-mock constraint before writing tests

### Before Editing Manuscript Files

1. Read **[Rendering Pipeline](rendering_pipeline.md)** — understand the 4-phase pipeline
2. Read **[Syntax Guide](syntax_guide.md)** — complete `{{VARIABLE}}` token list and figure label registry

### Before Writing Source Code

1. Read **[Style Guide](style_guide.md)** — 7 rules covering mocks, imports, error messages, type hints

## Using this exemplar as a reference for a new project

Use this standalone checkout as a reference for tested research code,
project-local evidence, and the `docs/manuscript/` layout.

| Invariant | Where it's taught | How it's enforced |
|---|---|---|
| Thin-orchestrator: `scripts/` only I/O + orchestration, logic in `src/`/`infrastructure/` | [`architecture.md`](architecture.md), [`style_guide.md`](style_guide.md) | code review + `src/` infra-import grep |
| Zero mocks: real data, `tmp_path`, `pytest-httpserver` | [`testing_philosophy.md`](testing_philosophy.md) | The test-source check below |
| ≥90% project coverage on `src/` | [`testing_philosophy.md`](testing_philosophy.md) | `--cov-fail-under=90` (canonical command below) |
| `docs/manuscript/config.yaml` is the single source of run policy | [`rendering_pipeline.md`](rendering_pipeline.md) | rendering infra |
| Deterministic outputs (fixed seeds); everything in `output/` regeneratable | [`output_conventions.md`](output_conventions.md) | reproducibility checks |

Follow the [standalone guide](standalone.md) when adapting this checkout.
Template documents the bundled-project workflow in its
[new project guide](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/guides/new-project-setup.md).

**Prerequisites to know before referencing the render path:** combined-PDF
rendering of any ```mermaid``` block needs `chrome-headless-shell`
(`npx --yes puppeteer browsers install chrome-headless-shell`); the per-project
test gate is the **direct** command below (a green `scripts/pipeline/stage_01_test.py` exit with
0 collected tests is not a pass). Both are detailed in
[`troubleshooting.md`](troubleshooting.md) and [`rendering_pipeline.md`](rendering_pipeline.md).

**Code vs. prose — pick the right reference.** Use **`template_code_project`**
when the deliverable is computed (algorithms, figures from data, numerical
results). Use the sibling **[`template_prose_project`](https://github.com/docxology/template_prose_project)**
when the deliverable is editorial (readability/structure/citation review of
written prose). Both share the identical structural contract above; they
differ only in what `src/` computes and what the pipeline checks.

## Verification Commands

Run these commands from the standalone checkout root after the [quick start](quickstart.md).

```bash
# Tests pass + coverage ≥90%
uv run --locked --extra dev pytest tests/ --cov --cov-config=pyproject.toml --cov-fail-under=90

# Scan test modules for mocking utilities.
grep -rEn --include='*.py' 'unittest\.mock|MagicMock|@patch|create_autospec' tests/

# Check the pure optimizer and invariant modules for direct SDK imports.
grep -nE '^(from|import) infrastructure' src/template_code_project/core/{optimizer,invariants}.py
```

For each grep check, status 1 means no matches and status 2 means a read error.

## See Also

- [../AGENTS.md](../AGENTS.md) — Full project documentation (API reference, known issues, complete directory map)
- [../README.md](../README.md) — Project quick start
- [../manuscript/AGENTS.md](manuscript/AGENTS.md) — Manuscript directory rules and `{{VARIABLE}}` protocol
- [output_conventions.md](output_conventions.md) — Output directory structure and regeneration
- [../../../docs/](https://github.com/docxology/template/tree/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/) — Repository-level documentation hub (measured file/subdirectory counts live in `docs/_generated/COUNTS.md`)
