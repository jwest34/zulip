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

## 2026-09-17 — Prompt 03 (switch to live Groq model, verify LLM path)

- **Actions:** Recorded prompt 03 into `docs/prompts/`. Confirmed the
  gitignored `zproject/dev-secrets.conf` holds `topic_summarization_api_key`
  (without displaying it). Made `TOPIC_SUMMARIZATION_MODEL` /
  `TOPIC_SUMMARIZATION_API_BASE` overridable via `get_secret` in
  `zproject/dev_settings.py` (defaults `openai/gpt-oss-120b` and Groq's base
  URL) and set `TOPIC_SUMMARIZATION_PARAMETERS = {"reasoning_effort": "low"}`.
  Restarted `run-dev`, verified the live summary endpoint as iago, ran the
  backend test, and committed. Detail in
  `docs/reports/report-03-llm-verify-2609171633.md`.
- **Findings:** Live summary of Denmark / "green server has been running"
  (6 msgs) returned HTTP 200 in ~1.44 s with a real Groq `openai/gpt-oss-120b`
  completion. `./tools/test-backend zerver.tests.test_message_summary` passed
  (2 tests). One operational snag: a leftover `webpack` held port 9994 after
  the restart, so `run-dev` self-terminated; fixed by clearing stale processes
  and relaunching via `setsid`. The Summarize button ("Summarize recent
  messages") is in the left-sidebar topic ⋮ popover
  (`left_sidebar_topic_actions_popover.hbs` → `topic_popover.ts` →
  `message_summary.get_narrow_summary`).
