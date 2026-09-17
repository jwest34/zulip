# Prompt 03 — switch to a live Groq model, place the key, verify the LLM path

Run from the repo root with `.venv` activated. Do the steps in order; do
nothing not listed. Background: Groq decommissioned `llama-3.3-70b-versatile`
on 2026-08-16, so Zulip's dev default 404s. Replacement: `openai/gpt-oss-120b`.

## 0. Record this prompt
Copy this file from `/mnt/c/Users/gdpr.WIND.000/Downloads/` into
`docs/prompts/` unchanged.

## 1. API key (secret — never print it, never cat the file, never copy it)
`zproject/dev-secrets.conf` already exists, written by `tools/setup-llm-secrets`,
and contains `topic_summarization_api_key`, `topic_summarization_model`, and
`topic_summarization_api_base`. Confirm only that
`git check-ignore zproject/dev-secrets.conf` prints the path and that
`grep -c '^topic_summarization_api_key' zproject/dev-secrets.conf` prints 1.
Do not display the file's contents.

## 2. Make model and base URL overridable from the same secrets file
In `zproject/dev_settings.py`, replace the two hard-coded
`TOPIC_SUMMARIZATION_MODEL` / `TOPIC_SUMMARIZATION_API_BASE` lines so that:
- `TOPIC_SUMMARIZATION_MODEL` = secret `topic_summarization_model`,
  default `"openai/gpt-oss-120b"`
- `TOPIC_SUMMARIZATION_API_BASE` = secret `topic_summarization_api_base`,
  default `"https://api.groq.com/openai/v1"`
- `TOPIC_SUMMARIZATION_PARAMETERS = {"reasoning_effort": "low"}`
Use the existing `get_secret` helper (see how `zproject/computed_settings.py`
or `zproject/config.py` provides it). Add a 2–3 line comment explaining that a
grader can point the server at any OpenAI-compatible provider by setting
`topic_summarization_api_key`, `topic_summarization_model`, and
`topic_summarization_api_base` in `zproject/dev-secrets.conf`.

## 3. Restart the server
Stop run-dev (`kill "$(cat var/log/run-dev.pid)"`), restart it in the
background as before, wait for `curl -s -o /dev/null -w '%{http_code}' http://localhost:9991/` to return 200.

## 4. Verify the LLM path via the API
- Get a dev API key for iago: `POST /api/v1/dev_fetch_api_key` with
  `username=iago@zulip.com`. Hold it in a shell variable only.
- Find one channel/topic with several messages (e.g. via
  `GET /api/v1/messages?anchor=newest&num_before=20&num_after=0&narrow=[{"operator":"channel","operand":"Verona"}]`).
- Call the existing summary endpoint:
  `GET /api/v1/messages/summary?narrow=<json narrow for that channel+topic>`
  with iago's key as HTTP basic auth (`-u iago@zulip.com:$KEY`).
- Record the HTTP status, elapsed time, and the returned summary text. If it
  fails, capture `var/log/run-dev.out` tail and the exact error, diagnose
  (wrong model id, permission `can_summarize_topics_group`, missing key,
  rate limit), fix if it is a config problem, and retry. Do not modify
  `zerver/` code in this prompt.

## 5. Backend test
Run `./tools/test-backend zerver.tests.test_message_summary` and record
pass/fail (it mocks the model; it proves the settings change broke nothing).

## 6. Secret scan and commit
`git status --short` must not list `zproject/dev-secrets.conf`.
`git diff | grep -i gsk_` must return nothing.
Commit `zproject/dev_settings.py`, `tools/setup-llm-secrets`, `docs/`, and
nothing else, locally.
Do not push.

## 7. Report
Write `docs/reports/report-03-llm-verify-$(date +%y%m%d%H%M).md` with:
- The exact `dev_settings.py` diff.
- The channel/topic used, HTTP status and elapsed time of the summary call,
  and the summary text returned.
- Any errors and fixes.
- Test result from step 5.
- Where the Summarize button is in the web UI so I can verify by clicking
  (as iago): which menu on a topic shows it, per `web/src/message_summary.ts`
  and the templates that reference it.
- The three secret names a grader must set, the Groq defaults, and that
  `./tools/setup-llm-secrets` writes all three.
Append to `docs/dev-log.md`. Copy the report to
`/mnt/c/Users/gdpr.WIND.000/Downloads/`. Print the report path as the final line.
