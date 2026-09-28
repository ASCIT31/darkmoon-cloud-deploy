# Network — flows to open in the security group / firewall

A DarkMoon node needs very little open. The guiding rule: **one inbound port (the
dashboard), plus outbound HTTPS**. Everything else is optional.

## The minimum

**Inbound**

| Flow | Proto / Port | Required |
| --- | --- | --- |
| Operator browser → dashboard | TCP `ui_port` (default **80**) | yes |
| Operator → SSH (management) | TCP **22** | optional |

**Outbound**

| Flow | Proto / Port | Required |
| --- | --- | --- |
| Node → Docker Hub (image pulls) | TCP **443** | at install/update |
| Node → license validation | TCP **443** | yes (Pro) |
| Node → LLM provider (Connected mode) | TCP **443** | conditional |
| Node → customer LLM endpoint (Private mode) | 443 / custom | conditional |
| Scanner → authorized targets | any TCP/UDP the scan needs | yes (to run scans) |
| Node → DNS / NTP | UDP **53** / UDP **123** | recommended |

## Read this carefully

- **Only one inbound application port.** Open TCP `ui_port` (default `80`) to reach the
  dashboard, and optionally SSH `22` for management. Nothing else inbound is required.
  Restrict the dashboard port to trusted CIDRs — see [Security](security.md).
- **Outbound HTTPS 443** covers three things: Docker Hub image pulls, DarkMoon license
  validation, and — *only in Connected mode* — calls to the LLM provider.
- **In Local AI mode, no LLM traffic leaves the node.** Inference runs over loopback
  (`127.0.0.1`). You still need outbound 443 at install/update (registry + license), but
  no model data egresses.
- **The three LLM flows are mutually exclusive.** Exactly one applies, depending on the
  [AI mode](ai-modes.md) you chose: Connected → provider over 443; Private → your
  endpoint; Local → none.
- **The scanner reaches your authorized targets** on whatever ports the assessment
  needs. Scope this outbound rule to the target ranges you are authorized to test.

## Per-cloud terms

The same flows, different console vocabulary:

- **AWS** — Security Group (inbound/outbound rules). See [AWS](aws.md).
- **GCP** — VPC firewall rules + network tags. See [GCP](gcp.md).
- **Azure** — Network Security Group (NSG). See [Azure](azure.md).
- **OVH** — security groups / network rules. See [OVH](ovh.md).

## Full flow matrix

This page is the cloud-focused summary. The complete DarkMoon **network-flows** matrix —
every source/destination/port/direction with the exact conditions — lives in the
official DarkMoon documentation (the Deployment / Network Requirements section on
`docs.dark-moon.org`). Hand that matrix to a firewall team when you need the exhaustive
version.
