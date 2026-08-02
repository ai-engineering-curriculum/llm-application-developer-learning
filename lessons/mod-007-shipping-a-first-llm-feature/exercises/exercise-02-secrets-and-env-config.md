# Exercise 02 — Secrets and per-environment config

Paired with [chapter 2 — managing keys and per-environment secrets](../02-secrets-and-per-environment-config.md).

**Estimated effort:** 1.5 hours.

## Objective

Take the endpoint from exercise 01 and put its configuration on a defensible footing. By the end of the exercise: (a) no secret lives in the repo, and a pre-commit hook enforces the rule; (b) the app reads its configuration from environment variables (loaded from `.env` in dev, from a real secret store in "prod-shape"); (c) startup fails loudly with a specific message when a required secret is missing; and (d) the "prod-shape" configuration is materially different from dev — a different key, a different model default, a different budget ceiling.

## Problem statement

Extend `first-feature/` from exercise 01:

1. **Introduce `.env` / `.env.example`.**
   - `.env` (in `.gitignore`) has your real dev values.
   - `.env.example` (committed) has the *variable names* with placeholder values (`ANTHROPIC_API_KEY=your-key-here`, `MODEL=claude-opus-4-7`, etc.). Never a real one.
2. **Load `.env` at process start** (Python: `python-dotenv`, Node: `dotenv`). Do not read the file inside the request handler — env vars are the abstraction the code sees.
3. **Add a `config.py` (or `pydantic-settings` equivalent) that centralises every configurable value.** At minimum: `MODEL`, `INPUT_TOKEN_LIMIT`, `OUTPUT_TOKEN_LIMIT`, `SYSTEM_PROMPT_VERSION`. The system prompt *text* lives in a versioned file (`prompts/summarise-v1.md`); the config's `SYSTEM_PROMPT_VERSION` selects which file to load.
4. **Add a `providers.py` that lazily constructs the SDK client** and raises a specific, actionable `RuntimeError` if the key is missing at first use — including which env var was missing and where to look (a `docs/local-setup.md` link).
5. **Set up a pre-commit hook that scans for secrets.** `gitleaks` (<https://github.com/gitleaks/gitleaks>) or `detect-secrets` (<https://github.com/Yelp/detect-secrets>) with a `.pre-commit-config.yaml`. Reference: <https://pre-commit.com/>.
6. **Add a `dev`-vs-`prod` shape.** In this module you do not need a real cloud secret store; simulate the split with two files: `.env` (dev values) and `.env.prod.example` (documented prod values, no real key). The point is that the app has no code paths that only work in one environment — the env vars are the whole configuration surface.
7. **Ensure no key or secret ever reaches stdout.** If your code contains a line like `print(os.environ)` or `logger.info(f"key loaded: {key}")`, remove it. If it happens by accident later (a stack trace with locals), chapter 3 will catch it — but the first defence is not writing the line.

## Requirements

- **`.env` is in `.gitignore`** as the first commit of the exercise. If it is not, you have made the exact mistake this exercise exists to prevent — rotate the key you exposed and redo the commit.
- **`.env.example` is committed** and lists every env var the app reads, with placeholder values.
- **The gitleaks / detect-secrets hook runs on every commit** and blocks the commit when it finds a secret pattern. Confirm the hook fires by staging a fake `sk-...` string, attempting to commit, and observing the block.
- **`providers.py` never accepts an `api_key` argument** — the SDK reads it from the environment. Any temptation to add a `for testing` argument is refused.
- **Startup with a missing `ANTHROPIC_API_KEY`** (or the OpenAI equivalent) fails with a clear error: `"ANTHROPIC_API_KEY is not set. See docs/local-setup.md."` and a non-zero exit code. Not a 500 on the first request; not a 401 from the SDK — a startup-time failure with a fixable message.
- **The system prompt text is not in `config.py`.** It lives in `prompts/summarise-v1.md`. `config.py` selects the version; the loader reads the file. This sets up chapter 4's flag-swap of prompt versions.
- **A short `docs/local-setup.md`** explains how a new contributor gets a dev key and populates `.env`. One page, five to ten lines.
- **The full test in acceptance criteria below runs green** locally before you move on.

## Starter guidance

- Do not overreach into a real cloud secret store for this exercise. The point is the *shape* — env vars as the boundary, secret store as the source. AWS Secrets Manager, GCP Secret Manager, Kubernetes Secrets, Vault, and Doppler all inject into env vars in exactly this shape; you can adopt one later without changing the app.
- Every provider gives you a dedicated dev key at no cost or trivial cost. Use it. The chapter's "dev key ≠ prod key" rule is not negotiable, even for a solo project.
- If the pre-commit setup feels heavier than the value it produces, remember: it is not for *this* commit, it is for the commit six months from now when someone new to the repo pastes a key into a debug script.
- `pydantic-settings` reference: <https://docs.pydantic.dev/latest/concepts/pydantic_settings/>. `python-dotenv` reference: <https://github.com/theskumar/python-dotenv>.
- Anthropic API keys reference: <https://docs.anthropic.com/en/api/getting-started>. OpenAI API keys reference: <https://platform.openai.com/docs/api-reference/authentication>.

## Acceptance criteria

- `git log -p .env` shows nothing — the file has never been committed.
- `git ls-files .env.example` returns the file — it *has* been committed.
- Running `pre-commit run --all-files` on a repo that includes a staged `sk-fake-01234567890` string exits non-zero with a gitleaks / detect-secrets finding. Removing the string and re-running succeeds.
- Unsetting `ANTHROPIC_API_KEY` (`unset ANTHROPIC_API_KEY && python -m uvicorn main:app`) exits with `"ANTHROPIC_API_KEY is not set. See docs/local-setup.md."` on the first request (or, if you check at startup, on process start). The SDK's own 401 does not surface.
- Changing `MODEL` in `.env` and restarting the process changes the model the endpoint uses. Confirm by watching the request in the provider's dashboard.
- Changing `SYSTEM_PROMPT_VERSION` from `summarise-v1` to `summarise-v2` (create a `prompts/summarise-v2.md` with a different one-liner) and restarting produces visibly different summaries.
- `grep -r "sk-ant" .` and `grep -r "sk-proj" .` (or your provider's key prefix) return no matches from committed files — only from `.env` and its equivalents, which are ignored.
- `docs/local-setup.md` exists, is under a page, and a colleague following it end-to-end can bring the app up in under ten minutes.

## Stretch goals

- **Set up `direnv`.** `direnv` (<https://direnv.net/>) auto-loads `.env`-shaped files on `cd`. Nice when you have multiple projects with different keys.
- **Add a `make check-config` target** that verifies every env var listed in `.env.example` is populated in the current environment, and prints a legible report of what is missing. Useful before deploys — turn a "silent 401 in production" into a "config check failed in CI."
- **Wire a real hosted secret store, even in dev.** Doppler, 1Password Secrets Automation, Infisical, or any cloud provider's manager. Point `.env` at the CLI's `run` command (`doppler run -- python -m uvicorn main:app`) so the app never touches the source file. This is what the "dev key rotation" story looks like in a small team.
- **Two-key rotation dry run.** Provision a *second* Anthropic key on the dashboard, put it in `.env.next`, and add a `providers.py` code path that reads it from an alternate env var name. Confirm that both keys work for one request each. Delete the second key when you are done. This rehearses the emergency rotation exercise you hope never to run for real.
