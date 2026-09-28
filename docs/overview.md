# Overview — no box, no marketplace, one mechanism everywhere

## The question this answers

> "Do you sell hardware boxes?"

No. DarkMoon Pro is **software-defined and self-hosted**. You bring the machine — a
cloud VM or a bare-metal server — and turn it into a dedicated DarkMoon node. There is
no OVA to import, no ISO to flash, no custom operating system, no proprietary appliance
chassis, and no vendor SDK to compile against.

Because the node is "just" Docker images installed by a script, the *same* deployment
works on **any** cloud and on bare metal. There is no per-cloud fork, no cloud-specific
build and no marketplace listing you have to buy through.

## One mechanism, three entry points

Everything below converges on one installer (`install.sh`) producing one single-node
stack. The three "paths" differ only in *where you start from*:

1. **[One command](one-command.md)** — you already have a Linux VM or bare-metal host,
   you have an SSH session, and you paste one line. `cloud-install.sh` installs Docker,
   runs `install.sh`, then runs `darkmoon doctor`.

2. **One click from the cloud console** — when you launch the VM, you paste a startup
   script into the console's user-data / custom-data / startup-script field. On first
   boot the VM provisions itself into a DarkMoon node. See your cloud's page:
   [AWS](aws.md), [GCP](gcp.md), [Azure](azure.md), [OVH](ovh.md).

3. **One command from your laptop or CloudShell (Terraform)** — per-cloud Terraform
   modules create the VM, the firewall/security-group and inject the same user-data.
   `terraform init && terraform apply`. Runs equally from a browser CloudShell.

## The single-node model

DarkMoon Pro today is a **single-node** appliance: one machine runs the full stack
(the `opencode` core + the `darkmoon` scanner + the Privacy Gateway). You scale **up**
(a bigger instance), not **out** (there is no clustering or fleet in the product). Sizing
is therefore a choice of instance type — see [Sizing](sizing.md).

## What lands on the machine

- The packaged **Docker images** (the core and the scanner), pulled from Docker Hub.
- The **`install.sh`** installer that builds and wires the stack — no hand-editing of
  `docker-compose.yml`.
- The **`darkmoon`** CLI (`doctor`, `update`, `repair`, `restart`) for lifecycle.
- The **Privacy Gateway**, which minimizes/tokenizes sensitive data before it reaches a
  model in Connected mode.

## What does *not* land on the machine

- **No proprietary hardware.** DarkMoon ships no box, server or dongle. You provide the
  machine.
- **No VM image / no custom OS.** You install onto a stock Linux distribution you
  already run.
- **No vendor SDK.** The node talks to your LLM provider (or runs a local model); there
  is nothing to link into your own code.

## Prerequisites at a glance

- A **Linux** VM or host, **amd64** (arm64 is experimental — do not use it in
  production; see [Sizing](sizing.md)).
- **Root / sudo** on that machine (the installer sets up Docker).
- A valid **DarkMoon Pro license key** (`DARKMOON_LICENSE_KEY`).
- **Outbound HTTPS (443)** at least at install/update time (image pulls, license
  validation) and, in Connected mode, to your LLM provider.
- An AI backend decision — see [AI modes](ai-modes.md).

Next: [One command](one-command.md).
