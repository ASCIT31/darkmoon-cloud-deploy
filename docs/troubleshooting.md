# Troubleshooting — cloud edge cases, symptom → fix

The workflow is always the same: **problem → `darkmoon doctor` → diagnosis → repair or
documented action**. Run Doctor first; it usually names the exact cause. This page focuses
on the *cloud-provisioning* edge cases you hit before or around that.

## Edge-case matrix

| Symptom | Likely cause | Fix (doctor-first) |
| --- | --- | --- |
| `install.sh` aborts immediately: "license required" / activation error | No `DARKMOON_LICENSE_KEY` (or empty first arg). Pro will not install without a license. | Pass the key via env var or first positional arg. If it was fetched from a secret manager, confirm the fetch returned a value (see below). |
| Activation fails: slot / machine limit reached | License **device slots exhausted** — too many fresh VMs registered their machine-code. | Free slots in the license dashboard; reuse long-lived nodes; release the slot on teardown. See [Doctor & lifecycle](doctor-and-lifecycle.md#license--slots). |
| Images won't pull / "no matching manifest" / exec format error | You launched an **arm64** instance. arm64 is experimental. | Recreate on an **amd64** instance type (see the [Sizing](sizing.md) table — all listed types are amd64). |
| One-liner fails at the Docker step | Docker install failed — no egress to `get.docker.com` / `download.docker.com`, or an unsupported distro. | Confirm outbound 443 (see [Network](network.md)); use a supported Linux; if Docker is pre-installed, the installer detects it. Then re-run. |
| Dashboard unreachable from your browser | Security group / firewall has **no inbound rule for `ui_port` (80)**, or the port is bound only to a private subnet. | Open TCP 80 (or your `ui_port`) to your trusted CIDR (see per-cloud pages); confirm the node has a reachable address; `darkmoon doctor` for a bound/unhealthy core. |
| Install hangs / fails pulling images or validating license | **No outbound egress** (locked-down VPC, no NAT/IGW). | Provide outbound 443 to Docker Hub + licensing (see [Network](network.md)). Fully air-gapped is not validated (see [AI modes](ai-modes.md)). |
| Agents run but get no model responses (Connected/Private) | LLM provider unreachable: missing/invalid API key, or no egress to the endpoint. | `darkmoon doctor` reports the provider check (key + TCP probe). Fix the key or open egress to the provider (see [AI modes](ai-modes.md)). |
| `curl` to the installer fails / wrong content | Wrong region has no egress, or a proxy/interception is in the path. | Confirm outbound 443 and DNS; try the [review-first](one-command.md#review-before-you-pipe) download form to inspect what you got. |
| Cloud-init / user-data "did nothing" on first boot | Wrong field (data pasted in the wrong console box), or the OS image ships no cloud-init. | Check the boot/cloud-init log; use the correct field for your cloud (see below); use an image with cloud-init. |
| Boot-time secret fetch failed — installer got an empty license/key | The VM identity can't read the secret (missing role/binding), wrong secret name, or wrong region/endpoint. | Grant least-privilege read to the VM identity on the exact secret; verify the name/region; **fail the script if the fetch is empty** rather than calling `install.sh` with a blank value. See [Security](security.md). |
| Node was fine, now unhealthy after a redeploy | Image drift, restart-loop, unhealthy container, or Privacy Gateway socket missing. | `darkmoon doctor` → `darkmoon doctor --fix`; if it persists, `darkmoon repair` then `darkmoon repair --full` (data kept). See [Doctor & lifecycle](doctor-and-lifecycle.md). |
| Disk filling up | Evidence retention outgrew the volume (Doctor warns at ≥90%). | Grow the volume / free space, or move up a [profile](sizing.md). |
| GPU not detected (Local AI only) | GPU/driver stack not present. GPU is only for Local inference. | Fix the GPU driver/toolkit; if you are Connected/Private you do not need a GPU (see [AI modes](ai-modes.md)). |

## Where the user-data field lives (the "did nothing" case)

- **AWS EC2** → *Advanced details → User data*
- **GCP Compute Engine** → *Automation → Startup script* (metadata `startup-script`)
- **Azure VM** → *Advanced → Custom data*
- **OVH Public Cloud** → cloud-init `user_data`

Paste the script into that field, not into the SSH-key or tags box. See your cloud's page:
[AWS](aws.md), [GCP](gcp.md), [Azure](azure.md), [OVH](ovh.md).

## When in doubt

1. `darkmoon doctor` — read the named diagnosis.
2. `darkmoon doctor --fix` — let it repair the safe classes.
3. `darkmoon repair` / `darkmoon repair --full` — force-recreate, data kept.
4. Check the boot/cloud-init log for anything that failed *before* `install.sh` ran.
