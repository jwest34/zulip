# Report 02 — Provision the Zulip dev environment (native WSL2)

**Date:** 2026-09-16
**Prompt:** 02 (provision)
**Repo:** `/home/gdpr/projects/hw01/zulip` (branch `main`)

## Environment

- **WSL2 distro:** Ubuntu 24.04.4 LTS (`lsb_release -ds`)
- **Kernel:** `6.6.87.2-microsoft-standard-WSL2`, x86_64
- **Disk free on `/` (before provision):** 953 GiB avail (3.6 GiB used of 1007 GiB)
- **Disk free on `/` (after provision + run-dev):** 948 GiB avail (8.3 GiB used of 1007 GiB) — provisioning consumed ~4.7 GiB
- No Vagrant, no Docker (native provision, as required).

## Commands run, in order

| # | Command | Result / elapsed |
|---|---|---|
| 1 | `mkdir -p docs/prompts docs/reports` + copy prompt 02 into `docs/prompts/` | ok |
| 2 | Created `CLAUDE.md`, `docs/dev-log.md` | ok |
| 3 | Verified `sudo -n true` | initially failed (password required); user enabled passwordless sudo via `/etc/sudoers.d/99-gdpr-nopasswd`, then OK |
| 4 | `./tools/provision` (attempt 1) | **FAILED**, exit 1, ~66 s |
| 5 | Fix: `cp /usr/local/bin/uv /home/gdpr/.local/bin/uv` (upgrade shadowing uv) | ok, `uv --version` → 0.12.5 |
| 6 | `./tools/provision` (attempt 2) | **SUCCEEDED**, exit 0, ~200 s (~3.3 min) |
| 7 | `source .venv/bin/activate` + `./tools/run-dev` (background → `var/log/run-dev.out`) | server up, PID 15544 |
| 8 | Poll `curl http://localhost:9991/` | HTTP 200 |
| 9 | `curl -X POST .../api/v1/dev_fetch_api_key --data-urlencode username=hamlet@zulip.com` | HTTP 200, `result: success` |

> Note on provision timing: the successful run was fast (~200 s) because attempt 1
> had already completed most of the download/install work before failing on the
> `uv` version check; only the remaining steps re-ran.

## Errors encountered and fixes (verbatim)

### Error 1 — `sudo` requires a password (blocked non-interactive provision)

```
sudo: a password is required
```

`./tools/provision` invokes `sudo` internally; a background/non-interactive
shell cannot supply a password. **Fix:** with the user's agreement, passwordless
sudo was enabled by the user in a real terminal:

```
echo "gdpr ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/99-gdpr-nopasswd && sudo chmod 440 /etc/sudoers.d/99-gdpr-nopasswd
```

### Error 2 — provision attempt 1 failed: stale `uv` shadowed the installed one

Provision installed `uv 0.12.5` to `/usr/local/bin`, but its version check ran
`uv --version` and got `0.11.21` from an older binary earlier in `PATH`
(`/home/gdpr/.local/bin/uv`, which `PATH` even listed twice). Verbatim tail:

```
+ out='uv 0.11.21 (x86_64-unknown-linux-gnu)'
+ [[ uv 0.11.21 (x86_64-unknown-linux-gnu) = \u\v\ \0\.\1\2\.\5\ \(* ]]

Subcommand of tools/lib/provision.py failed with exit status 1: sudo --preserve-env=PATH -- env http_proxy= https_proxy= no_proxy= scripts/lib/install-uv
Actual error output for the subcommand is just above this.

Provisioning failed (exit code 1)!
```

**Fix (non-destructive):** upgraded the stale binary in place rather than
deleting the user's tool:

```
cp /usr/local/bin/uv /home/gdpr/.local/bin/uv
```

After the fix, `uv --version` resolves to `uv 0.12.5` at both `~/.local/bin/uv`
and `/usr/local/bin/uv`. Re-running `./tools/provision` succeeded:

```
Zulip development environment setup succeeded!
```

## Does http://localhost:9991/ respond, and what does it show?

**Yes — HTTP 200.** The page is the Zulip dev environment login page (title
"Zulip Dev"), which offers direct dev login as the standard test users
(hamlet, iago, etc.). `run-dev` is listening on `127.0.0.1:9991` (PID 15544)
and webpack reports "compiled successfully".

## Did `dev_fetch_api_key` succeed?

**Yes.** `POST /api/v1/dev_fetch_api_key` with `username=hamlet@zulip.com`
returned **HTTP 200** with `"result":"success"` and an `api_key` field.
Per instructions, the key value was **not** recorded anywhere.

## How to stop and restart run-dev

`run-dev` is currently **running** (PID recorded in `var/log/run-dev.pid`,
logging to `var/log/run-dev.out`).

**Stop:**

```
kill "$(cat var/log/run-dev.pid)"
# or, if needed:
pkill -f tools/run-dev
```

**Restart (from repo root):**

```
source .venv/bin/activate
./tools/run-dev            # foreground; Ctrl-C to stop
# or backgrounded:
nohup ./tools/run-dev > var/log/run-dev.out 2>&1 & echo $! > var/log/run-dev.pid
```

Then wait for `curl -s http://localhost:9991/` to return HTTP 200.

## Things to know before the next step

- **Always activate the venv first:** `source .venv/bin/activate` from the repo
  root before any Zulip command; otherwise commands fail with a warning about
  the virtualenv not being active.
- **Passwordless sudo is now enabled** for user `gdpr` via
  `/etc/sudoers.d/99-gdpr-nopasswd`. Remove it with
  `sudo rm /etc/sudoers.d/99-gdpr-nopasswd` if you don't want it to persist.
- **`~/.local/bin/uv` was upgraded** from 0.11.21 to 0.12.5 to match the
  provisioned toolchain. If other projects pinned the old version, be aware.
- The summarization/LLM feature scaffolding already exists upstream (see
  report 01); dev config points at Groq and needs a
  `topic_summarization_api_key` in `zproject/dev-secrets.conf` (untracked) to
  actually call a model.
- Leave `run-dev` running for the next step, as instructed.
