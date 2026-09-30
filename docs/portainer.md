# DarkMoon in Portainer

Deploy the **open source (GPL-3.0) DarkMoon CLI** as a Portainer *App Template* or
*Custom Template*, from the same Compose the project already ships. No box, no
marketplace, no per-cloud fork.

> **What you get:** the terminal-driven (TUI) DarkMoon engine. There is **no web
> GUI** in this stack. You drive DarkMoon from the console of the `opencode`
> container. The DarkMoon Pro web dashboard and the remediation-to-PR agent are
> **not** part of this open source stack.

## Honest privilege notice

DarkMoon is an offensive-security toolbox, so the stack asks for real host
privileges. Deploy it **only on a host you control**:

- `/var/run/docker.sock` is mounted into both containers — this is **root-equivalent
  control of the Docker host**.
- the `darkmoon` service runs with `network_mode: host` — full host network access,
  needed for LAN scanning. Chrome DevTools port **9222** (used by the browser tool)
  is therefore reachable directly on the host.
- `cap_add: NET_RAW, NET_ADMIN` — raw sockets for nmap-style scanning.
- containers run as **root** (`user: "0:0"`).

## Option A — add it as a Custom Template URL (recommended)

This installs DarkMoon straight from this repository, always current.

1. In Portainer, go to **App Templates → Settings** (or **Custom Templates**).
2. Set the templates URL to:

   ```
   https://raw.githubusercontent.com/ASCIT31/darkmoon-cloud-deploy/master/portainer-template.json
   ```

3. Back on **App Templates**, pick **DarkMoon (open source CLI)**.
4. Fill the LLM prompts (see below) and **Deploy the stack**.

Portainer clones this repo and deploys [`portainer/docker-compose.yml`](../portainer/docker-compose.yml).

## Option B — paste the Compose as a Web-editor stack

If you would rather not add a template URL: **Stacks → Add stack → Web editor**,
paste the contents of [`portainer/docker-compose.yml`](../portainer/docker-compose.yml),
add the environment variables below, and deploy.

## LLM provider

DarkMoon needs a model. The template is preset for a **cloud** provider and prompts
for three variables:

| Variable | Meaning | Example |
| --- | --- | --- |
| `OPENROUTER_PROVIDER` | provider name | `anthropic`, `openai`, `openrouter` |
| `OPENCODE_MODEL` | model id | `claude-opus-4-6`, `gpt-4o` |
| `OPENROUTER_API_KEY` | provider API key | `sk-...` |

To run a **local model** (Ollama, llama.cpp) or an **on-prem Anthropic-compatible**
endpoint instead, edit the stack's environment after deploy and set the matching
variables (`OPENCODE_LOCAL_MODE=true` + `OPENCODE_LOCAL_PROVIDER_ID` /
`OPENCODE_LOCAL_BASE_URL` / `OPENCODE_LOCAL_MODEL`, or `ANTHROPIC_BASE_URL` /
`ANTHROPIC_MODEL` / `ANTHROPIC_API_KEY`). See [AI modes](ai-modes.md).

## Use it

After deploy, open the **console** of the `opencode` container (Portainer →
Containers → `opencode` → Console → `/bin/bash`, or attach to it), and start a
DarkMoon session. Reports and sessions persist in the `darkmoon_reports` and
`darkmoon_sessions` volumes.

## Links

- Source (GPL-3.0): https://github.com/ASCIT31/Dark-Moon
- Cloud deployment docs: [README](../README.md)
