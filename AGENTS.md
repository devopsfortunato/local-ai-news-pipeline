# AGENTS.md

The pipeline that builds **Local AI News** once a day and publishes it to GitHub
Pages. It runs unattended every morning, so anything merged to `main` is a
production change. `CONTRIBUTING.md` is the authority; this file is the fast
path for coding agents.

## Before you call a change done

1. Run the check CI runs:

   ```bash
   python3 .github/scripts/validate_repo.py
   ```

   It must exit 0 with every line `[OK]`. It checks Python syntax, bash syntax,
   `skills/_shared/sources.md` JSON blocks, and the skill name/directory
   invariant. Passing does **not** mean the change is safe — it means nothing a
   computer can check without secrets is broken.

2. If you changed behavior or the order of steps, update `docs/ARCHITETTURA.md`
   in the same change.

3. If you changed the shape of `edition.json`, update **every** consumer in the
   same change: `scripts/core/render.py`,
   `scripts/inject/inject_wire_ticker.py`,
   `scripts/content/make_making_of.py`,
   `scripts/core/update_headlines_history.py`.

## Hard rules

- **Never change `PIPELINE_PROVIDER` or `PIPELINE_MODEL` in `scripts/run.sh`**
  unless the task explicitly asks. The warning block above them documents why:
  a silent provider swap once ran a full edition on a paid backend.
- A skill's directory name must equal its frontmatter `name:` — `run.sh`
  resolves skills by that name.
- Steps communicate only through JSON files under `/tmp/lain/`. No step calls
  another directly.
- Keep the explanatory comment blocks in `run.sh` that document production
  incidents; preserve them when editing nearby code and add one when you fix a
  new production failure.
- **No secrets in the repo.** All inference goes through the self-hosted DGX
  Spark LiteLLM proxy, configured as the `litellm` provider in the Hermes
  profile. CI has no access to it.
- This repo is the pipeline, not the newspaper: the published output lives in
  the separate `local-ai-news` repo. Don't commit generated editions here.
- Push branches only (`card/<card-id>`); never push `main`. Open a PR and let
  the `Validate` check go green.

## Layout

| Path | What |
|---|---|
| `scripts/run.sh` | The daily orchestrator (bash); `cron_wrapper.sh` calls it |
| `scripts/{content,core,inject,maintenance}/` | Mechanical steps, plain Python/bash |
| `skills/<name>/SKILL.md` | Judgment steps, one skill per beat |
| `template/` | HTML/CSS for the newspaper |
| `docs/ARCHITETTURA.md` | What runs, in what order |
| `.github/scripts/validate_repo.py` | The static check CI runs |

This repo is a fork of `NTTLuke/lux-in-tenebris-pipeline`; keep the structure
upstream-compatible.
