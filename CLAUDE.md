# CLAUDE.md — Course assignment: LLM features for Zulip

## Project

Course assignment extending Zulip with two LLM-powered features:

- **(a) Unread recap:** a recap of all unread messages, with clickable
  links back to the original messages.
- **(b) Topic-drift detection:** detect when a conversation has drifted
  and suggest a better topic title.

### Deliverables

- Replace `README.md` with install/run instructions.
- `implementation.md` — ≤500 words per feature, with file/line pointers
  and a demo-video link.
- Final commit URL submitted on Canvas.
- **Due: Sept 18, 2026, 11:59 PM.**

## Rules

- **Never write API keys or secrets into any tracked file.** The only
  place for secrets is `zproject/dev-secrets.conf` (gitignored).
- Commit in small increments with descriptive messages.
- **Never `git push` without asking.**
- **Ask before adding any Python or npm dependency.**
- The dev server runs **natively in this WSL2 shell (no Vagrant)**.
  After provisioning, server commands run from the repo root with
  `.venv` activated.

## Reporting

After every task:

1. Append an entry to `docs/dev-log.md` (date, prompt number, actions,
   findings).
2. Write `docs/reports/report-NN-<slug>-$(date +%y%m%d%H%M).md`.
3. Copy that report to `/mnt/c/Users/gdpr.WIND.000/Downloads/`.
4. Print the report path as the final line.
