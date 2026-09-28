# OVH

Three ways to stand up a DarkMoon node on OVH Public Cloud. All three run the same
[`install.sh`](one-command.md) and produce the same node.

Pick an **amd64** instance flavor from the [Sizing](sizing.md) table (e.g. `b2-30` for the
Standard profile). Stick to amd64 flavors — arm64 is experimental.

---

## Path A — one click at instance creation (cloud-init user_data)

When creating the instance in the OVH Public Cloud console (or via the OpenStack API),
provide the startup script (`deploy/cloud-init/darkmoon-user-data.sh`) as the cloud-init
**`user_data`**. On first boot the instance provisions itself into a DarkMoon node.

Network rules (OVH security groups / network; see [Network](network.md)):

- **Inbound** TCP `80` (dashboard) from your **trusted CIDR** only; optionally TCP `22`.
- **Outbound** TCP `443` (image pulls, license, and the LLM provider in Connected mode).

Minimal user_data:

```bash
#!/usr/bin/env bash
set -euo pipefail
# OVH Public Cloud has no first-party managed secret store; see docs/security.md for options.
# Fetch LICENSE and API_KEY from your chosen secret source here, then:
[ -n "${LICENSE:-}" ] && [ -n "${API_KEY:-}" ] || { echo "secrets not provided" >&2; exit 1; }
curl -fsSL https://portal.dark-moon.org/cloud | DARKMOON_LICENSE_KEY="$LICENSE" bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key "$API_KEY"
```

> **Secrets on OVH.** There is no OVH-native Key Vault equivalent. Do **not** inline the
> license/API key in `user_data` (it is readable from instance metadata). Options: run an
> open-source secrets manager you control, fetch from a secret store in another cloud over
> 443, or provision keys out-of-band by running the [one-liner interactively](one-command.md)
> over SSH. See [Security](security.md).

---

## Path B — one command with Terraform (from your laptop)

Use the OVH module at `deploy/terraform/ovh/` (OVH exposes an OpenStack-compatible API, so
the module uses the OVH/OpenStack providers). It creates the instance, the network rules
with only the flows above, and injects the cloud-init `user_data`.

```bash
cd deploy/terraform/ovh
terraform init
terraform apply -var license_key=... -var ai_api_key=...
```

Expected variables include `flavor` (default a Standard-class amd64 flavor), `region`,
`trusted_cidr`, and the provider selection. Configure OVH API credentials
(application key/secret/consumer key) and, where applicable, your OpenStack `openrc`
values as the module documents.

---

## Path C — from a browser shell

OVH does not offer a first-party browser CloudShell like the big three. Run Path B from
any machine with Terraform and your OVH credentials — your laptop, a jump host, or a
throwaway VM. The commands are identical to Path B.

---

## After launch

- Reach the dashboard at `http://<instance-ip>:80` from your trusted CIDR. Put TLS in
  front if it is exposed beyond a trusted network ([Security](security.md)).
- `darkmoon doctor` to verify health; operate via the `darkmoon` CLI
  ([Doctor & lifecycle](doctor-and-lifecycle.md)).
- Hitting a problem? See [Troubleshooting](troubleshooting.md).
