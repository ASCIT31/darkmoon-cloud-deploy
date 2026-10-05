# Doctor & lifecycle — operate the node, not the containers

DarkMoon gives you SaaS-like operations with self-hosted control. **Operate the node
through the `darkmoon` CLI, not by poking the raw containers.** The commands are safe,
transactional and reversible in ways manual `docker` commands are not.

## `darkmoon doctor` — the first reflex

After boot, and whenever anything looks wrong, run:

```bash
darkmoon doctor          # diagnose host + stack (report only)
darkmoon doctor --fix    # also auto-repair the safe problem classes
darkmoon doctor --yes    # non-interactive (assume yes)
```

Doctor inspects:

- **Docker runtime** — daemon present/reachable, Compose v2 available, compose file
  present.
- **Both containers** (the `opencode` core + the `darkmoon` scanner) — present, running,
  healthy; detects absent / restart-loop / unhealthy states.
- **Images** — version drift (local vs remote digest).
- **License** — key present and free of activation errors.
- **AI provider** — API key present + a TCP probe of the configured base URL.
- **Privacy Gateway** — the unix socket is present.
- **Ports** — the UI port is available / correctly bound.
- **Host resources** — disk (warn at ≥90% used), memory (warn under the floor).

`--fix` only auto-repairs the mechanical classes (restart-loop → pull+up, stopped → up,
unhealthy → restart) and **backs up config first**. Everything else (version drift,
license, provider, ports, disk/RAM) is report-only because the fix is a decision, not a
restart.

## `darkmoon update` — transactional updates

```bash
darkmoon update
# 1. backup      — snapshot the current config
# 2. compose pull
# 3. compose up  — recreate with the new images
# 4. wait-healthy
# 5. rollback    — automatically, if health does not come back
```

A bad pull cannot leave you with a broken node: on health failure it rolls back. Backups
cover **config only** — the sealed data volume (evidence, campaigns) is never copied
around during an update.

## `darkmoon repair` — force-recreate

```bash
darkmoon repair          # force-recreate the stack (data kept)
darkmoon repair --full   # down, pull, re-run install.sh reusing license/config
```

Repair keeps your data: the sealed data volume is never touched, and `repair --full`
reuses the existing license and config when it re-runs `install.sh`.

## `darkmoon restart`

```bash
darkmoon restart         # restart the stack
```

## License & slots

DarkMoon Pro is licensed (Cryptolens + a runtime guard). Be aware of how the licensing
interacts with cloud automation:

- **Each fresh VM consumes a device/machine slot.** The license has a finite number of
  device slots, and every node registers a **machine-code fingerprint** on activation.
- **Re-provisioning many ephemeral VMs can exhaust slots.** If your automation creates and
  destroys nodes frequently (autoscaling experiments, short-lived CI runners, blue/green
  churn), you can burn through the slot count. Prefer long-lived nodes, or release slots
  when you tear a node down.
- **Free a slot yourself from the client portal.** If activation fails with
  `device limit reached`, open the [client portal](https://portal.dark-moon.org) →
  **Devices & activation slots**, and release the machine code shown in your activation
  error (`darkmoon doctor` prints this node's machine code too). Then re-run activation.
  No need to email support.
- **Floating activations are not listed as devices.** The guard uses *floating*
  activation, and floating leases are not shown in the dashboard's node-locked device
  list; they release themselves automatically within about an hour, or you release one
  immediately by its machine code in the portal.
- **Floating / offline behavior, at a high level.** The license supports floating
  activation (slots can be reclaimed and re-issued rather than being permanently pinned to
  a dead VM) and an **offline validation cache** so a node keeps working through brief
  licensing-endpoint outages (cache-tolerant). Design your automation to reclaim slots on
  teardown rather than relying on the cache indefinitely.

`darkmoon doctor` surfaces license/activation problems in its report; see
[Troubleshooting](troubleshooting.md).

## Golden rule

> Problem → `darkmoon doctor` → diagnosis → `darkmoon` repair/update or the documented
> action. Reach for raw `docker` commands last, not first.
