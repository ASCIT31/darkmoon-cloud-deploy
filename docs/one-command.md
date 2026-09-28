# One command — the cloud installer

This is Path 1: you have a fresh Linux VM or bare-metal host and an SSH session. One
line turns it into a DarkMoon node.

```bash
curl -fsSL https://portal.dark-moon.org/cloud | sudo DARKMOON_LICENSE_KEY=<KEY> bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key sk-ant-...
```

## What it does

The URL serves `deploy/cloud-install.sh`. Piped into `sudo bash`, it runs three steps in
order:

1. **Installs Docker** — Docker Engine + Compose v2, via the official
   `get.docker.com` convenience script.
2. **Runs `install.sh` non-interactively** — it fetches the DarkMoon installer and runs
   it with the flags you passed after `--`. `install.sh` **requires the Pro license**
   (via `DARKMOON_LICENSE_KEY` or as the first positional argument) and a provider
   selection.
3. **Runs `darkmoon doctor`** — verifies runtime, containers, images, license, AI
   provider, Privacy Gateway, ports, disk and memory before handing back.

Everything after `bash -s --` is passed straight through to `install.sh`.

## The license (required)

DarkMoon Pro will not install without a valid license. Provide it either as an
environment variable (recommended, shown above) or as the **first positional argument**
to `install.sh`:

```bash
# env var form (preferred)
sudo DARKMOON_LICENSE_KEY=<KEY> bash -s -- --provider anthropic --model claude-opus-4-6 --api-key sk-ant-...

# first-positional form
sudo bash -s -- <KEY> --provider anthropic --model claude-opus-4-6 --api-key sk-ant-...
```

Each fresh VM consumes a **device/machine slot** on the license (machine-code
fingerprint). See [Doctor & lifecycle](doctor-and-lifecycle.md#license--slots) before
you script the creation of many ephemeral VMs.

## Provider flags

Pick exactly one AI backend. See [AI modes](ai-modes.md) for what each means.

### Connected (a hosted LLM provider)

```
--provider <name>      # e.g. anthropic
--model   <model-id>   # e.g. claude-opus-4-6
--api-key <key>        # e.g. sk-ant-...
```

### A native Anthropic endpoint (explicit URL/model/key)

```
--anthropic-url   <base-url>
--anthropic-model <model-id>
--anthropic-key   <key>
```

> Use the native `anthropic` path (not an OpenAI-compatible shim) when you want
> Anthropic prompt caching honoured.

### Local (on-node inference, no LLM egress)

```
--local
--local-engine ollama|llama.cpp
--local-url    <endpoint>       # e.g. http://127.0.0.1:11434
--local-model  <model-id>
```

Local mode needs extra RAM/VRAM per model — see [Sizing](sizing.md) and
[AI modes](ai-modes.md).

## Common combinations

```bash
# Connected — hosted provider
curl -fsSL https://portal.dark-moon.org/cloud | sudo DARKMOON_LICENSE_KEY=<KEY> bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key sk-ant-...

# Private — customer LLM endpoint via the native anthropic path
curl -fsSL https://portal.dark-moon.org/cloud | sudo DARKMOON_LICENSE_KEY=<KEY> bash -s -- \
  --anthropic-url https://llm.internal.example.com --anthropic-model claude-opus-4-6 --anthropic-key <KEY>

# Local — on-node inference, no LLM egress
curl -fsSL https://portal.dark-moon.org/cloud | sudo DARKMOON_LICENSE_KEY=<KEY> bash -s -- \
  --local --local-engine ollama --local-url http://127.0.0.1:11434 --local-model <model-id>
```

## After it finishes

- Open the dashboard on the node's `ui_port` (default `80`). Restrict that port to
  trusted CIDRs — see [Security](security.md).
- Re-run diagnostics any time with `darkmoon doctor`.
- Operate the node through the `darkmoon` CLI, not the raw containers — see
  [Doctor & lifecycle](doctor-and-lifecycle.md).

## Security reminder for automation

When you script this (user-data, Terraform, CI), **do not inline the license key or the
API key in plaintext** where instance metadata or the console can read them. Fetch them
from the cloud secret manager at boot. See [Security](security.md).

## Review before you pipe

Piping a URL into `sudo bash` executes remote code as root. If your policy requires
review first, download and read it, then run it:

```bash
curl -fsSL https://portal.dark-moon.org/cloud -o cloud-install.sh
less cloud-install.sh
sudo DARKMOON_LICENSE_KEY=<KEY> bash cloud-install.sh --provider anthropic --model claude-opus-4-6 --api-key sk-ant-...
```
