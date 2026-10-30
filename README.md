# Hermes Agent — Setup & Configuration

Public snapshot of a **Hermes Agent** (by Nous Research) installation: identity (soul), config, skills, memories, anti-refusal guardrail proxy, and bundled plugins. This repo is meant to be **read and copied** — it contains no live secrets.

Live secrets were kept in `~/.hermes/.env`, `creds/`, `keys/`, and `vault/`, none of which are in this repo.

## Contents

| Path | What it is |
|---|---|
| `SOUL.md` | Agent identity/persona (system-prompt layer) |
| `config.yaml` | Main Hermes config (providers, gateway, plugins, display) |
| `profile.yaml` | Profile metadata (display name, avatar) |
| `skills/` | Skill library — markdown playbooks the agent loads on demand |
| `memories/` | `MEMORY.md` (agent notes) + `USER.md` (user profile) |
| `anti-refusal/` | Local guardrail proxy (`proxy.py`) that sits between Hermes and the LLM router; retries refusals with an escalated anti-refusal system line. systemd unit included |
| `plugins/` | Bundled plugins: `lightclawbot`, `vela-im` (IM adapters), `weixinpay` (WeChat Pay) |
| `.env.example` | Every env var the install uses, **values stripped** |

## Prerequisites

- Linux (or macOS) with Python **3.11+**
- Git
- An OpenAI-compatible LLM endpoint (any router/provider)
- Optional: Telegram bot token for the Telegram platform

## Install

```bash
# 1. Get Hermes Agent
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
# (or: pip install hermes-agent)

# 2. Clone this repo and overlay it onto your .hermes dir
git clone https://github.com/br111e/hermes-agent-setup hermes-agent-setup
cp hermes-agent-setup/SOUL.md ~/.hermes/SOUL.md
cp hermes-agent-setup/config.yaml ~/.hermes/config.yaml
cp hermes-agent-setup/profile.yaml ~/.hermes/profile.yaml
cp -r hermes-agent-setup/skills ~/.hermes/
cp -r hermes-agent-setup/memories ~/.hermes/
cp -r hermes-agent-setup/plugins ~/.hermes/
cp -r hermes-agent-setup/anti-refusal ~/.hermes/
cp hermes-agent-setup/.env.example ~/.hermes/.env
chmod 600 ~/.hermes/.env
```

## Configure

Fill in the values in `~/.hermes/.env` (see `.env.example` for the full list):

```bash
# LLM router key (OpenAI-compatible endpoint)
CUSTOM_API_KEY=***
CUSTOM_BASE_URL=https://token.lightvela.ai/v1

# Telegram (optional — for the telegram platform)
TELEGRAM_BOT_TOKEN=123456...
```

Pick your model in `config.yaml` under `model:` — it points at a local proxy by default
(`http://127.0.0.1:9200/v1`). If you don't want the proxy, point `model.base_url`
directly at your LLM endpoint.

### Anti-refusal proxy (optional)

`anti-refusal/proxy.py` is a tiny aiohttp guardrail: it forwards
`/v1/chat/completions` to your upstream, detects refusal patterns (EN + zh), and
re-asks up to `max_attempts` times with escalating anti-refusal system lines.

```bash
# run it
python3 ~/.hermes/anti-refusal/proxy.py   # listens on 127.0.0.1:9200

# or as a service
sudo cp ~/.hermes/anti-refusal/antirefusal.service /etc/systemd/system/
sudo systemctl enable --now antirefusal
```

Edit `anti-refusal/config.json` to change upstream, retry count, or port.

## Run

```bash
hermes          # start the CLI session
hermes gateway  # start the messaging gateway (telegram/IM platforms)
```

Verify the provider works:

```bash
hermes --chat "ping"
```

## Notes

- `memories/MEMORY.md` and the `heavens-gateway` skill contain project-specific
  notes; token-like values there were replaced with placeholders
  (`<ADMIN_PW>`, `<GONKA_API_KEY>`, ...).
- The `weixinpay` plugin is large (~10 MB) and contains prebuilt binaries; safe to
  delete if you don't use WeChat Pay.
- Secrets that were in the original install: LLM router keys, Telegram bot token,
  SSH key, password vault — all live outside this repo.

## Reference

- Docs: https://hermes-agent.nousresearch.com/docs
