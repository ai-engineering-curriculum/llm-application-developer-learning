# Chapter 2 — Managing keys and per-environment secrets

The endpoint in chapter 1 assumed `client = anthropic.AsyncAnthropic()` would find an API key somewhere. This chapter is about *where*, and — much more importantly — where it must never be. A leaked model-provider key is not an abstract security incident: it is a direct debit on your bank account, and the first attacker with a scraper is the one who spends it.

## Motivation

The two failure modes this chapter is written to prevent:

1. **A key committed to a repository.** The provider's key-scanning bots find it, someone else's scraper finds it, or a future employee `git log`s their way to it. The provider will usually rotate the key for you and email a rate-limit-abuse notice; the bill from before the rotation is yours. This is the mode that pays for a whole audit team once a year at a large company.
2. **A key printed into a log or a trace.** The key never went to GitHub. It went to Datadog, or CloudWatch, or a Sentry event body, or a stack trace in a Slack alert channel where 40 people have access. Every place a log stream is copied is now a place the key has to be rotated out of. Chapter 3 spends a whole section on redaction because this failure mode is easier to walk into than it looks.

Both are avoidable with a small amount of discipline applied from the first commit. Retrofitting either after the fact is possible but expensive; the cheaper move is to never make the mistake.

## The one rule

**No secret in the repository. Ever. Not in a comment, not in a `.env.example`, not in a test fixture, not in a docstring.** Secrets live in environment variables at runtime, sourced from a per-environment secret store. Anything else is a variation on "we will fix it later" that never gets fixed.

Two operational habits that make the rule sticking-able:

- **Pre-commit hook that scans for secrets** — `gitleaks` (<https://github.com/gitleaks/gitleaks>), `detect-secrets` (<https://github.com/Yelp/detect-secrets>), or the provider's own scanning if you use GitHub's Advanced Security. Reference for GitHub's built-in secret scanning: <https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning>.
- **Rotate on every suspicious event, not on a schedule.** If you think a key might have leaked — someone shared their screen, a laptop went missing, a log store had an incident — rotate first, investigate second. Rotation is cheap; not-rotating-and-being-wrong is not.

## Environment variables at runtime, not in code

The pattern the SDKs already assume:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# or
export OPENAI_API_KEY="sk-..."
```

Both SDKs read the corresponding environment variable if you construct the client with no arguments. That is not laziness — it is *the* pattern. It keeps the code identical across dev, staging, and prod; only the environment differs.

Two anti-patterns to unlearn:

- **`client = Anthropic(api_key="sk-ant-...")` with the key literal in code.** Even if that literal comes from a helper that reads `os.environ`, the *pattern* invites someone to hard-code the value "just for a debug session" and forget to remove it. Never take the string as an argument in your own module. If you construct the client yourself, do it once in a `providers.py`-shaped module and let the SDK read the env var.
- **Reading the env var and then printing it "to confirm it worked."** The one line that puts the key in every log stream. If you want to confirm the key loaded, check that the first character is `s` and the length is what you expect — or make a real API call and let the 401 tell you.

## Per-environment secret stores

"Environment variables at runtime" leaves open one question: where do those variables come from? In production, from a **secret store**. Never from the shell history of the developer who deployed.

The concrete options:

- **Cloud-provider secret managers.** AWS Secrets Manager (<https://docs.aws.amazon.com/secretsmanager/>), GCP Secret Manager (<https://cloud.google.com/secret-manager/docs>), Azure Key Vault (<https://learn.microsoft.com/en-us/azure/key-vault/secrets/>). Read at pod-start / task-start into an env var. IAM controls who can read what.
- **Kubernetes Secrets** (<https://kubernetes.io/docs/concepts/configuration/secret/>). The default for Kubernetes-hosted workloads. Note the default is unencrypted-at-rest in etcd unless you enable encryption at rest — verify before shipping. See <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>.
- **HashiCorp Vault** (<https://developer.hashicorp.com/vault/docs>). Provider-independent, self-hosted or hosted. Common in on-prem or multi-cloud shops.
- **Platform-as-a-service secret pages.** Fly, Render, Railway, Vercel, Heroku all expose a "secrets" or "environment variables" page that stores the value encrypted at rest and injects it into your process at boot. Fine for a first feature; you graduate off it when you outgrow the platform.
- **1Password / Doppler / Infisical** as a developer-secrets sync layer. Optional; useful when the same developer works across many projects.

The question is not which of these you use — any of them is fine for a first feature. The question is: **is there a per-environment separation, and is the dev key different from the prod key?** If the same `ANTHROPIC_API_KEY` is used for local development and for the customer-facing endpoint, one leaks the other.

## The local-development story: `.env` files

For local development the standard shape is a `.env` file at the repo root, loaded into the process's environment before the code runs. The library ecosystem is well-worn:

- **Python — `python-dotenv`.** Loads `.env` into `os.environ` at import. Reference: <https://github.com/theskumar/python-dotenv>.
- **Node — `dotenv`.** Same shape. Reference: <https://github.com/motdotla/dotenv>.

Three inflexible rules:

1. **`.env` is in `.gitignore` from the first commit.** If it is not in `.gitignore` before you first `git add`, you have already made a mistake. Fixing it means rotating the key, not just deleting the file.
2. **`.env.example` (committed) documents the *variable names*, not the values.** The file exists so a new contributor knows which env vars to populate. Every value is a placeholder — `ANTHROPIC_API_KEY=your-key-here`. Never a real one.
3. **The application reads env vars, not `.env` files.** `.env` is a *developer convenience* for populating env vars locally. Production reads from the secret store into env vars; the application code does not know the difference. If your handler `open(".env")`s directly, you have coupled local development to production semantics.

The pattern:

```python
# providers.py
import os
from anthropic import AsyncAnthropic

# In development, whatever loaded ANTHROPIC_API_KEY into os.environ
# (python-dotenv, direnv, your shell) is not this module's problem.
# The SDK reads ANTHROPIC_API_KEY on its own.
_client: AsyncAnthropic | None = None

def get_client() -> AsyncAnthropic:
    global _client
    if _client is None:
        # Fail loudly at first use if the key is missing.
        if not os.environ.get("ANTHROPIC_API_KEY"):
            raise RuntimeError(
                "ANTHROPIC_API_KEY is not set. See docs/local-setup.md."
            )
        _client = AsyncAnthropic(timeout=15.0)
    return _client
```

Note two things: the module fails loudly on missing config (instead of returning a client that will 401 on every request), and it never touches the key value directly — the SDK reads it from the environment.

## Non-secret config: the same discipline, softer

Not every value in `providers.py` is a secret. `MODEL = "claude-opus-4-7"`, `INPUT_TOKEN_LIMIT = 8_000`, `SYSTEM_PROMPT = "..."` are configuration, not secrets. Two patterns work:

- **Small config module** (`config.py`) with constants, overridden by env vars where sensible (`MODEL = os.environ.get("MODEL", "claude-opus-4-7")`). Good enough for a first feature.
- **`pydantic-settings`.** A typed settings model with env-var and `.env` loading built in. Nice when the config grows past ~10 knobs. Reference: <https://docs.pydantic.dev/latest/concepts/pydantic_settings/>.

Two rules from experience:

- **Non-secret config that differs per environment lives in the same store as secrets.** The prod model, the prod input-token limit, and the prod prompt version are all things you want changeable without a code deploy. Chapter 4 makes this concrete for model selection.
- **The system prompt is not a secret, but it should be versioned.** Store the prompt text next to your code (a `.md` or `.txt` file), reference it by content hash in traces (chapter 3), and swap it via a config value the same way you swap models. Prompt changes are a common source of regression; treating the prompt as code makes the diff reviewable.

## What the SDK does with the key

Both SDKs send the key on every request in an `Authorization` header (OpenAI) or a provider-specific header like `x-api-key` (Anthropic). Two consequences:

- **HTTPS everywhere.** Any middleman between your process and the provider that terminates TLS sees the key. Every hop from your service to the provider must be TLS. The SDK does this by default; do not talk yourself into a `verify=False` "just for debugging."
- **Debug proxies see the key.** If you run through `mitmproxy` / Charles / Fiddler to debug a request, that tool logs the header value. Rotate the key when you are done, and never run a debug proxy against a production key.

## Rotating keys

Two-key rotation is the shape most secret stores support. The moves:

1. **Provision a new key** on the provider's dashboard. Both keys are active.
2. **Write the new key** to the secret store; on the next process start, the app reads the new one.
3. **Confirm** by watching request counts on both keys' dashboards. When the old key stops seeing traffic, revoke it.

Two things people forget the first time:

- **Rotate on secret-store change, not just on incident.** Any time someone with access to the secret store leaves the team, rotate every secret they could have read. It is boring and it is the whole reason for the two-key pattern.
- **Do not rotate all providers at once during an incident.** If Anthropic and OpenAI both fail simultaneously right after you rotated both keys, you cannot tell which rotation caused it. Rotate one, wait, then the other.

## What secrets look like in this feature

By the end of the module you will have exactly a handful of things in the secret store per environment:

- `ANTHROPIC_API_KEY` (or `OPENAI_API_KEY` — pick one for the feature; do not straddle both without a reason).
- `MODEL` — the model id the endpoint calls. Chapter 4 makes this a flag.
- `SYSTEM_PROMPT_VERSION` — the version of the prompt this environment is running. The prompt text itself lives in the repo, indexed by version.
- `INPUT_TOKEN_LIMIT`, `OUTPUT_TOKEN_LIMIT` — the budget knobs from chapter 1.
- Optional: `INTERNAL_API_TOKEN` for caller auth (chapter 1's one-paragraph auth story).

That is the whole list for a first feature. Anything else in your secret store is either non-secret (move it to a config file) or leaking across boundaries (split it per environment).

## Common mistakes

- **`ANTHROPIC_API_KEY=...` in a shell script committed to the repo.** The shell script "for running the app locally" is the file people forget is in git. Same rule as `.env`: not committed.
- **Sharing one team key across all developers.** Every developer with the shared key can spend from the shared budget, and rotating on offboarding rotates for everyone. Give each developer their own dev-tier key. Providers make this easy.
- **Logging the whole request body of the outgoing HTTP call.** Common when adding "quick observability" via an HTTP interceptor. The `Authorization` header goes with it. Chapter 3's redaction section covers this in detail — but the first-order defence is not to log outbound headers at all.
- **Reading the key on every request.** Not wrong, just wasteful — the SDK caches the value on the client. Construct the client once at startup (or lazily on first use) and reuse it.
- **A test that hits the real provider with a real key on every run.** Fine occasionally as an integration test, disastrous as a unit test. Use recorded responses (`vcrpy`, `pytest-httpx`) for the fast tests and gate the real-provider tests behind an env-var switch.

## Summary

- No secret in the repository. Ever. Pre-commit scanning enforces the rule.
- Environment variables at runtime, sourced from a per-environment secret store. Dev key ≠ prod key.
- `.env` for local development, in `.gitignore` from the first commit; `.env.example` documents variable *names*, not values.
- Fail loudly at startup on missing secrets; do not return a broken client that will 401 later.
- Rotate on any suspicious event and on any secret-store access change. Two-key rotation is the default shape.
- Log nothing that includes an API key — outbound-header redaction is a rule, not a suggestion. Chapter 3 makes this operational.

The next chapter takes the traces we have been alluding to and makes them a real, redacted, per-request record you can ship to any log sink.
