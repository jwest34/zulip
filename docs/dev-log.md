# Dev log

## 2026-09-16 — Prompt 01 (inventory)

- **Actions:** Inventoried the repository read-only (git state, Zulip
  server markers, existing LLM/summarization code, secrets handling,
  dependencies, prior modifications, host environment). Saved the report
  to `docs/reports/report-01-inventory-2609162101.md`.
- **Findings:** This is the Zulip server (v12.0-dev+git, API feature
  level 511), a fork of `jwest34/zulip`, clean tree on `main`. Upstream
  Zulip's built-in topic-summarization feature already exists in full
  (backend actions/views/tests, frontend `message_summary.ts` +
  `topic_summary.hbs`, and all `TOPIC_SUMMARIZATION_*` settings; dev
  defaults to Groq's `llama-3.3-70b-versatile`). `openai` is a
  dependency (in `pyproject.toml`); `litellm` is not present.
  `zproject/dev-secrets.conf` does not exist yet but is gitignored; the
  code reads `topic_summarization_api_key` via `get_secret`. Host is
  WSL2 (Ubuntu 24.04), no Vagrant/Docker installed; python 3.12.3,
  node v22.23.0.

## 2026-09-16 — Prompt 02 (provision the dev environment)

- **Actions:** Recorded prompt 02 into `docs/prompts/`. Created root
  `CLAUDE.md` (assignment brief, rules, reporting convention) and this
  dev log. Provisioned the native WSL2 dev environment via
  `./tools/provision`, started `./tools/run-dev`, and verified the dev
  login page and `dev_fetch_api_key` API path. Full details in
  `docs/reports/report-02-provision-*.md`.
- **Findings:** Provision first failed (exit 1) because a stale `uv 0.11.21`
  in `~/.local/bin` shadowed the `0.12.5` provision installed to
  `/usr/local/bin`, failing the version check; fixed by upgrading
  `~/.local/bin/uv` in place, after which `./tools/provision` succeeded
  (~200 s). Also required enabling passwordless sudo (user did this in a real
  terminal). `run-dev` starts cleanly on `127.0.0.1:9991`; the dev login page
  (Zulip Dev, test users) returns HTTP 200 and `dev_fetch_api_key` returns
  success. Full detail in `docs/reports/report-02-provision-2609162218.md`.
