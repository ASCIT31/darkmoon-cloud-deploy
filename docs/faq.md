# FAQ

## Do you sell a hardware box?

**No.** DarkMoon Pro is software-defined and self-hosted. There is no proprietary
appliance, no server, no dongle, no OVA and no custom OS to flash. You bring a Linux
machine — a cloud VM or bare metal — and one command turns it into a dedicated DarkMoon
node. See [Overview](overview.md).

## Which clouds do you support?

**Any of them, and bare metal.** The node is Docker images installed by a script, so the
*same* deployment works on AWS, GCP, Azure, OVH and on your own hardware. There is no
per-cloud fork. See [AWS](aws.md), [GCP](gcp.md), [Azure](azure.md), [OVH](ovh.md).

## Why isn't DarkMoon on the AWS / GCP / Azure marketplace?

Because it doesn't need to be, and a marketplace would work against the things that make
DarkMoon easy to run:

- **Cloud-agnostic by design.** One mechanism (`install.sh`) works everywhere. A
  marketplace listing is per-cloud: you'd maintain a separate packaged image and
  publishing pipeline for each provider, and the product would look tied to that cloud.
- **Free and no lock-in.** The deployment tooling here is free and open. You keep full
  control of the host, the network and the data. A marketplace inserts the provider
  between you and the software.
- **Marketplaces add overhead and take a cut.** Per-cloud packaging, review cycles and a
  revenue share add cost and friction without changing what you actually run.

A marketplace listing **could come later, purely for discoverability** — a convenient
"launch" button for people who prefer it. It would be an *additional* front door, never a
requirement, and it would install the very same node you get today with one command.

## Do I need a special build or a vendor SDK?

No. There is no vendor SDK to link against and no cloud-specific build. The node talks to
your chosen LLM backend (or runs a local model). See [AI modes](ai-modes.md).

## Do I need a GPU?

Only for **Local** inference (running a model on the node). Connected and Private modes do
inference off the node and never need a GPU. See [AI modes](ai-modes.md).

## Is my data sent to a model provider?

Only in **Connected** mode, and only after the **Privacy Gateway** minimizes and
tokenizes it. In **Private** mode the model traffic stays on infrastructure you control;
in **Local** mode there is no LLM egress at all. Connected is data-minimized, *not*
offline. See [AI modes](ai-modes.md).

## Is it licensed / commercial?

Yes. DarkMoon Pro is licensed software (Cryptolens + a runtime guard). You need a valid
license key to install and run it. The license has device slots, and each fresh VM
consumes one — plan for that if you spin up many ephemeral nodes. See
[Doctor & lifecycle](doctor-and-lifecycle.md#license--slots). *This documentation
repository* is free (CC BY 4.0); the engine is not.

## Activation says "device limit reached" — what do I do?

The key is valid; all its device slots are in use (a previous activation, or a floating
lease, still holds the slot). Free one yourself: in the
[client portal](https://portal.dark-moon.org) open **Devices & activation slots** and
release the machine code shown in your activation error (`darkmoon doctor` prints it too),
then re-run activation. Floating slots also release themselves within about an hour. See
[Troubleshooting](troubleshooting.md).

## Can I run it fully air-gapped?

Not as a validated configuration today. Local mode removes *LLM* egress, but install and
update still need Docker Hub, and Pro still needs periodic license validation
(cache-tolerant). See [AI modes](ai-modes.md).

## How do I keep it running?

Operate it through the `darkmoon` CLI: `doctor` (diagnose/repair), `update`
(transactional, rolls back on failure), `repair` / `repair --full`. Don't manage the raw
containers by hand. See [Doctor & lifecycle](doctor-and-lifecycle.md).

## How much machine do I need?

Pick a [profile](sizing.md) (Minimum / Standard / Performance / Industrial) and map it to
your cloud's instance type from the table. Use **amd64** — arm64 is experimental.
