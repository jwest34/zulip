# Repository Inventory — `/home/gdpr/projects/hw01/zulip`

## 1. Basics
- **pwd:** `/home/gdpr/projects/hw01/zulip`
- **git remote -v:** `origin  https://github.com/jwest34/zulip` (fetch + push) — a personal **fork**, no `upstream` remote configured
- **Current branch:** `main`
- **Last commit:** `db7ae5fb831d653d340457a0b63150bfb7893d61 Tue Sep 15 23:33:31 2026 +0530 settings: Use consistent case-folding in bot owner sort.`
- **Working tree:** **clean** (`git status --short` empty)

## 2. Is this the Zulip server? — YES
| Marker | Status |
|---|---|
| `zproject/` | PRESENT |
| `zerver/` | PRESENT |
| `web/src/` | PRESENT |
| `tools/run-dev` | PRESENT |
| `Vagrantfile` | PRESENT |
| `manage.py` | PRESENT |

- **ZULIP_VERSION:** `12.0-dev+git`
- **API_FEATURE_LEVEL:** `511`

## 3. LLM / summarization code — ALREADY PRESENT ⚠️
| File | Status |
|---|---|
| `zerver/actions/message_summary.py` | **EXISTS** |
| `zerver/views/message_summary.py` | **EXISTS** |
| `zerver/tests/test_message_summary.py` | **EXISTS** |
| `web/src/message_summary.ts` | **EXISTS** |
| `web/templates/topic_summary.hbs` | **EXISTS** |

`grep TOPIC_SUMMARIZATION zproject/*.py`:
```
dev_settings.py:235: TOPIC_SUMMARIZATION_MODEL = "llama-3.3-70b-versatile"
dev_settings.py:236: TOPIC_SUMMARIZATION_API_BASE = "https://api.groq.com/openai/v1"
prod_settings_template.py:823: # TOPIC_SUMMARIZATION_MODEL = "meta-llama/Meta-Llama-3-8B-Instruct"
prod_settings_template.py:827: # TOPIC_SUMMARIZATION_API_BASE = "https://router.huggingface.co/v1"
prod_settings_template.py:831: # TOPIC_SUMMARIZATION_PARAMETERS = {}
default_settings.py:768: TOPIC_SUMMARIZATION_MODEL: str | None = None
default_settings.py:769: TOPIC_SUMMARIZATION_API_BASE: str | None = None
default_settings.py:770: TOPIC_SUMMARIZATION_PARAMETERS: dict[str, Any] = {}
computed_settings.py:1325: # Which API key to use will be determined based on TOPIC_SUMMARIZATION_MODEL.
computed_settings.py:1326: TOPIC_SUMMARIZATION_API_KEY = get_secret("topic_summarization_api_key", None)
```
**This is upstream Zulip's built-in topic-summarization feature** — the full machinery already exists (defaults to Groq's `llama-3.3-70b-versatile` in dev). Note the dev model expects a **Groq** API base, not OpenAI.

## 4. Secrets handling
- `zproject/dev-secrets.conf`: **ABSENT** (not yet created)
- **Gitignored:** YES — `.gitignore:22` = `/zproject/dev-secrets.conf` (also referenced in a comment at line 5)
- `grep api_key/API_KEY --include=*.conf`: **no matches** (no `.conf` files with keys tracked)
- Relevant secret name expected by code: `topic_summarization_api_key` (read via `get_secret`)

## 5. Dependencies
- **openai:** present — `pyproject.toml:184: "openai"` (not in `requirements/*.txt`)
- **litellm:** **not found** anywhere in requirements or pyproject

## 6. Prior modifications
- `git log --oneline -15`: all commits are standard Zulip development (event-type pydantic refactors, settings tweaks) — **no obvious local/custom homework commits** on top.
- **Non-upstream docs files:** cannot be determined precisely — no `upstream` remote is configured to diff against. Nothing stands out in the tree.
- `README.md` history: normal upstream commits (`docs: Replace Twitter (X) references with Bluesky.`, etc.)
- `implementation.md`: **does not exist**

## 7. Environment
- **Host OS:** `Linux wind 6.6.87.2-microsoft-standard-WSL2 ... x86_64` (WSL2)
- **vagrant:** not installed
- **docker:** not installed (Docker Desktop WSL integration not active)
- **python3:** `3.12.3`
- **node:** `v22.23.0`
- **Vagrant machine status:** not checked — `vagrant` binary is not installed, so `vagrant status` cannot run (Vagrantfile exists but Vagrant isn't available)

---

**Key takeaways:**
1. This is a Zulip 12.0-dev fork (`jwest34/zulip`), clean tree, on `main`.
2. **The topic/message summarization feature already exists in full** (backend actions/views/tests, frontend TS + template, and all `TOPIC_SUMMARIZATION_*` settings). Dev config points at **Groq**, and the dependency present is `openai` (no `litellm`).
3. `dev-secrets.conf` doesn't exist yet but is properly gitignored; the code expects a `topic_summarization_api_key` secret.
4. No Vagrant/Docker available on this WSL2 host — provisioning would run natively or need one of those installed (not done, per your instruction).

No changes were made.
