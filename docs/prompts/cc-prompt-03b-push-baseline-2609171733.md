# Prompt 03b — inventory local changes, secret-scan, first push to the fork

Run from the repo root. Do the steps in order; nothing else. Make no code
changes in this prompt.

## 0. Record this prompt
Copy this file from `/mnt/c/Users/gdpr.WIND.000/Downloads/` into `docs/prompts/`.

## 1. Confirm prompt 03's commit exists
`git log --oneline -5`. If `zproject/dev_settings.py` and
`tools/setup-llm-secrets` are not yet committed, commit them now (with
`docs/`) as prompt 03 step 6 specified.

## 2. Inventory everything that differs from the fork's snapshot
`git fetch origin` then:
- `git log --oneline origin/main..HEAD` (our commits)
- `git diff --stat origin/main..HEAD` (every file we added/changed)
- `git status --short` (anything uncommitted — should be empty apart from
  ignored files; confirm `zproject/dev-secrets.conf` does not appear)
Include all three outputs verbatim in the report.

## 3. Secret scan (must pass before pushing)
- `git diff origin/main..HEAD | grep -i -E 'gsk_|api_key *= *[A-Za-z0-9]'` → must be empty
- `git log -p origin/main..HEAD | grep -i 'gsk_'` → must be empty
- `git ls-files | grep -i secrets` → must show nothing except, if present,
  upstream's `zproject/dev-secrets.conf` **is not** tracked; report the result
If any scan is non-empty, STOP, do not push, and report what was found.

## 4. Push
`git push origin main`. If it fails on authentication, do not retry blindly:
capture the exact error and report which credential method is needed
(GitHub no longer accepts passwords over HTTPS — a personal access token via
`git credential` / Git Credential Manager, `gh auth login`, or an SSH remote).
Do not create tokens or change the remote yourself.

## 5. Verify
`git fetch origin && git status -sb` must show `main...origin/main` with no
"ahead" count. Report the commit hash at `origin/main` and the URL
`https://github.com/jwest34/zulip/commit/<hash>`.

## 6. Report
Write `docs/reports/report-03b-push-baseline-$(date +%y%m%d%H%M).md` with the
step 2 outputs, the step 3 scan results, whether the push succeeded (and the
exact error if not), and the step 5 hash and URL. Append `docs/dev-log.md`.
Commit `docs/` locally (this report will be pushed with the next push; don't
push again now). Copy the report to `/mnt/c/Users/gdpr.WIND.000/Downloads/`.
Print the report path as the final line.
