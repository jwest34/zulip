# Report 03 — Switch to a live Groq model, place the key, verify the LLM path

**Date:** 2026-09-17
**Prompt:** 03 (llm-verify)
**Repo:** `/home/gdpr/projects/hw01/zulip` (branch `main`)

Background: Groq decommissioned `llama-3.3-70b-versatile` on 2026-08-16, so
Zulip's dev default 404s. Replacement model: `openai/gpt-oss-120b`.

## Secret confirmation (step 1)

Without displaying the file:

- `git check-ignore zproject/dev-secrets.conf` → `zproject/dev-secrets.conf` (ignored).
- `grep -c '^topic_summarization_api_key' zproject/dev-secrets.conf` → `1`.

## `dev_settings.py` diff (step 2)

```diff
diff --git a/zproject/dev_settings.py b/zproject/dev_settings.py
index e4d0abd55b..297a4fe70d 100644
--- a/zproject/dev_settings.py
+++ b/zproject/dev_settings.py
@@ -2,6 +2,7 @@ import os
 import pwd
 
 from scripts.lib.zulip_tools import deport
+from zproject.config import get_secret
 from zproject.settings_types import SCIMConfigDict
 
 ZULIP_ADMINISTRATOR = "desdemona+admin@zulip.com"
@@ -232,8 +233,15 @@ DEMO_ORG_DEADLINE_DAYS = 30
 if external_host_env is None and not IS_DEV_DROPLET:
     USING_CAPTCHA = True
 
-TOPIC_SUMMARIZATION_MODEL = "llama-3.3-70b-versatile"
-TOPIC_SUMMARIZATION_API_BASE = "https://api.groq.com/openai/v1"
+# A grader can point the server at any OpenAI-compatible provider by
+# setting topic_summarization_api_key, topic_summarization_model, and
+# topic_summarization_api_base in zproject/dev-secrets.conf; the defaults
+# below use Groq's OpenAI-compatible API.
+TOPIC_SUMMARIZATION_MODEL = get_secret("topic_summarization_model", "openai/gpt-oss-120b")
+TOPIC_SUMMARIZATION_API_BASE = get_secret(
+    "topic_summarization_api_base", "https://api.groq.com/openai/v1"
+)
+TOPIC_SUMMARIZATION_PARAMETERS = {"reasoning_effort": "low"}
 # Defaults based on groq's pricing for Llama 3.3 70B Versatile 128k.
 # https://groq.com/pricing/
 OUTPUT_COST_PER_GIGATOKEN = 590
```

Model and API base are now read from the secrets file (`get_secret`), with the
Groq defaults baked in. `TOPIC_SUMMARIZATION_PARAMETERS` sets
`{"reasoning_effort": "low"}`.

## Server restart (step 3)

Stopped `run-dev`, restarted it, and confirmed
`curl -s -o /dev/null -w '%{http_code}' http://localhost:9991/` → **200**.

## LLM path verification (step 4)

- Dev API key fetched for `iago@zulip.com` (`POST /api/v1/dev_fetch_api_key`),
  held only in a shell variable (never printed/stored).
- Chose the busiest topic among recent public-channel messages:
  **channel `Denmark`, topic `green server has been running` (6 messages).**
- Called `GET /api/v1/messages/summary` with a narrow for that channel+topic,
  authenticated as iago via HTTP basic auth (`-u iago@zulip.com:$KEY`).

**Result:**

| Field | Value |
|---|---|
| HTTP status | **200** |
| Elapsed (`time_total`) | **1.44 s** |
| `result` | `success` |

**Summary text returned:**

> Zoe introduces the motivation for public‑private key pairs and references an
> "empathic algorithm" for erasure coding. The Imported User suggests improving
> responsiveness as a quick win. Zoe then poetically describes herself as a
> muse‑inspired poet. Cordelia critiques a solution's shortcomings, noting that
> sensor networks can be made linear‑time, secure, and "fuzzy," and proposes a
> hierarchical database approach, while Othello adds that without superblocks
> the understanding of write‑ahead logging might never have emerged.

(The dev test messages are Lorem-ipsum-like, so the summary is nonsensical in
content — but it is a genuine live completion from Groq's `openai/gpt-oss-120b`,
which confirms the end-to-end LLM path works.)

## Errors encountered and fixes

1. **`run-dev` failed to come back up after the first restart attempt.**
   A leftover `webpack` child (PID from the old `run-dev`) kept holding
   internal port 9994, so the freshly started `run-dev` couldn't bind it and
   its supervisor shut everything down ("Received interrupt signal"). The
   settings change itself was fine — startup logged "System check identified
   no issues" with no traceback.
   **Fix:** killed the stray `webpack`/`run-dev` processes, confirmed ports
   9991–9996 were free, and relaunched `run-dev` fully detached via `setsid`
   so it survives background-task teardown. Server then returned 200.

2. **`GET /api/v1/messages` returned non-JSON on the first try.** The raw
   `narrow=[...]` embedded in the URL wasn't URL-encoded.
   **Fix:** used `curl -G --data-urlencode 'narrow=...'`. Not a server issue.

No `zerver/` code was modified in this prompt.

## Backend test (step 5)

```
./tools/test-backend zerver.tests.test_message_summary
Ran 2 tests in 1.728s
OK
```

**PASS** (2 tests). The suite mocks the model, so this confirms the settings
change didn't break the summary code path.

## Where the Summarize button is in the web UI (verify by clicking as iago)

- **Menu:** the **topic actions popover** — hover a topic in the **left
  sidebar** and click the **⋮ (three-dot) menu**. The item is labeled
  **"Summarize recent messages."**
- **Template:** `web/templates/popovers/left_sidebar/left_sidebar_topic_actions_popover.hbs`
  — the `<a class="sidebar-popover-summarize-topic …">` menu item, shown only
  when `show_ai_features` **and** `can_summarize_topics` are true.
- **Handler:** `web/src/topic_popover.ts` binds a click on
  `.sidebar-popover-summarize-topic` to
  `message_summary.get_narrow_summary(stream_id, topic_name)` in
  `web/src/message_summary.ts`, which calls the same
  `/api/v1/messages/summary` endpoint verified above.
- **Visibility for iago:** iago is a realm owner, so
  `can_summarize_topics_group` is satisfied; `show_ai_features` requires the
  server to have summarization configured (model + key), which it now does.

## Grader configuration (step 7 summary)

A grader points the server at any OpenAI-compatible provider by setting three
secrets in `zproject/dev-secrets.conf`:

| Secret name | Purpose | Groq default (this repo) |
|---|---|---|
| `topic_summarization_api_key` | API key (secret; never committed) | (provided by grader) |
| `topic_summarization_model` | Model id | `openai/gpt-oss-120b` |
| `topic_summarization_api_base` | OpenAI-compatible base URL | `https://api.groq.com/openai/v1` |

`./tools/setup-llm-secrets` writes all three (it prompts for the key with
hidden input and defaults `LLM_MODEL`/`LLM_API_BASE` to the Groq values above;
both are overridable via env vars, e.g.
`LLM_MODEL=gpt-4o-mini LLM_API_BASE=https://api.openai.com/v1 ./tools/setup-llm-secrets`).

## Secret hygiene

- `git status --short` does **not** list `zproject/dev-secrets.conf` (gitignored).
- `git diff | grep -i gsk_` returns nothing; `tools/setup-llm-secrets` contains
  no hardcoded key (it reads the key via `read -s`).

## How to stop / restart run-dev

```
kill "$(cat var/log/run-dev.pid)"      # or: pkill -f tools/run-dev
# restart:
source .venv/bin/activate
setsid bash -c './tools/run-dev > var/log/run-dev.out 2>&1' < /dev/null &
```
