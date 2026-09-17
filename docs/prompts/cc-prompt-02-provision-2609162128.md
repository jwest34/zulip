# Prompt 02 — provision the Zulip dev environment (native WSL2)

Run from the repo root. Do the steps below in order; do nothing not listed.

## 0. Record this prompt
`mkdir -p docs/prompts docs/reports` and copy this file into `docs/prompts/`
unchanged.

## 1. Project rules file
Create `CLAUDE.md` at the repo root with:
- Project: course assignment extending Zulip with two LLM features —
  (a) a recap of all unread messages with clickable links to the original
  messages; (b) topic-drift detection that suggests a better topic title.
  Deliverables: replace README.md with install/run instructions;
  `implementation.md` (≤500 words per feature, file/line pointers, demo-video
  link); final commit URL submitted on Canvas. Due Sept 18, 2026, 11:59 PM.
- Rules: never write API keys or secrets into any tracked file — the only
  place for them is `zproject/dev-secrets.conf`. Commit in small increments
  with descriptive messages. Never `git push` without asking. Ask before
  adding any Python or npm dependency. The dev server runs natively in this
  WSL2 shell (no Vagrant); after provisioning, server commands run from the
  repo root with `.venv` activated.
- Reporting: after every task, append an entry to `docs/dev-log.md`
  (date, prompt number, actions, findings) and write
  `docs/reports/report-NN-<slug>-$(date +%y%m%d%H%M).md`, then copy that
  report to `/mnt/c/Users/gdpr.WIND.000/Downloads/`. Print the report path as
  the final line.

Create `docs/dev-log.md` with entries for prompt 01 (inventory) and prompt 02
(this task).

## 2. Provision
Run `./tools/provision`. It needs sudo and typically takes 10–30 minutes.
On failure: read the error, consult `docs/development/setup-recommended.md`
(WSL2 section) and `docs/development/setup-advanced.md`, fix, re-run. Record
every error and fix verbatim. Do not install Vagrant or Docker.

## 3. Start and verify
Activate `.venv`, start `./tools/run-dev` in the background logging to
`var/log/run-dev.out`, and wait until `curl -s http://localhost:9991/`
returns the dev login page (it lists test users such as hamlet and iago).
Then confirm the API path:
`curl -s -X POST http://localhost:9991/api/v1/dev_fetch_api_key --data-urlencode username=hamlet@zulip.com`
Report success/failure only — never record the returned key anywhere.
Leave run-dev running.

## 4. Report
Write `docs/reports/report-02-provision-$(date +%y%m%d%H%M).md` with:
- Ubuntu version in WSL2 (`lsb_release -ds`); disk free before and after.
- Commands run, in order, with elapsed time for provision.
- Every error and its fix.
- Whether http://localhost:9991/ responds and what it shows.
- Whether dev_fetch_api_key succeeded (yes/no).
- How to stop and restart run-dev.
- Anything I should know before the next step.
Copy the report to `/mnt/c/Users/gdpr.WIND.000/Downloads/`.
Commit `CLAUDE.md`, `docs/`, and nothing else, locally. Do not push.
Print the report path as your final line.
