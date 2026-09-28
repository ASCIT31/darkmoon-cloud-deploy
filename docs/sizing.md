# Sizing — profiles mapped to instance types

DarkMoon is a single-node appliance: you scale **up** (a bigger instance), not out. Pick
the profile that matches the workload you actually run — assets × agents × tool
executions × parallelism × campaign complexity × evidence retention — and treat the
figures as sizing estimates with margin, not hard floors.

## Profiles → instance size

| Profile | vCPU / RAM | AWS | GCP | Azure | OVH |
| --- | --- | --- | --- | --- | --- |
| **Minimum** | 2 / 8 | `t3.large` | `e2-standard-2` | `Standard_B2ms` | `b2-15` |
| **Standard** | 4 / 16 | `t3.xlarge` | `e2-standard-4` | `Standard_D4s_v5` | `b2-30` |
| **Performance** | 8 / 32 | `t3.2xlarge` | `e2-standard-8` | `Standard_D8s_v5` | `b2-60` |
| **Industrial** | 16 / 64 | `m6i.4xlarge` | `e2-standard-16` | `Standard_D16s_v5` | `b2-120` |

**Which to pick**

- **Minimum** — a single low-parallelism campaign against a handful of hosts.
- **Standard** — one campaign at normal parallelism. A good default.
- **Performance** — high concurrency or larger campaigns.
- **Industrial** — sustained edge node with long evidence retention.

## Architecture: amd64 only

Use **amd64 (x86-64)** instance types. **arm64 is experimental** — do not select arm64
instances (Graviton, Ampere/Axion, `Dpsv5`/`b3` ARM, etc.) for a production node. Every
instance type in the table above is amd64.

## Disk

Provision an SSD/NVMe root or data volume sized to your evidence retention:

- Minimum: ~40 GB SSD
- Standard: ~80 GB SSD
- Performance: ~160 GB NVMe
- Industrial: ~250 GB+ NVMe

Disk is where campaign evidence accumulates. `darkmoon doctor` warns at ≥90% used.

## Local AI adds memory

A GPU is **only** for **Local** inference (on-node model). Connected and Private modes
do inference off the node and never need a GPU. If you run Local mode, add memory
(RAM, or VRAM if you use a GPU) on top of the base profile, per model size:

| Model size | Extra memory |
| --- | --- |
| 7B | +8 GB |
| 13B | +16 GB |
| 33B | +32 GB |

So a Standard node (4 / 16) running a 13B local model wants roughly 16 + 16 = 32 GB —
step up to a Performance-class instance, or add a GPU with enough VRAM. See
[AI modes](ai-modes.md).

## Notes

- These figures are engineering estimates derived from component specs and existing
  data, not a fresh execution benchmark.
- Start one profile above your best guess if you are unsure; it is cheaper than
  re-provisioning (and re-provisioning consumes a license slot — see
  [Doctor & lifecycle](doctor-and-lifecycle.md#license--slots)).
