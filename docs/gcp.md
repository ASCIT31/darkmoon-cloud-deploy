# GCP

Three ways to stand up a DarkMoon node on Compute Engine. All three run the same
[`install.sh`](one-command.md) and produce the same node.

Pick an **amd64** machine type from the [Sizing](sizing.md) table (e.g. `e2-standard-4`
for the Standard profile). arm64 (Axion / `t2a`) is experimental — don't use it.

---

## Path A — one click from the console (Startup script)

When creating the VM in the Compute Engine console, open **Automation → Startup script**
(stored as the `startup-script` metadata key) and paste the startup script
(`deploy/cloud-init/darkmoon-user-data.sh`). On first boot the VM provisions itself into a
DarkMoon node.

Firewall rules (VPC firewall + a network tag on the VM; see [Network](network.md)):

- **Inbound** TCP `80` (dashboard) from your **trusted source range** only; optionally
  TCP `22`.
- **Outbound** TCP `443` (image pulls, license, and the LLM provider in Connected mode).
  Egress is allowed by default on GCP unless you've locked it down.

Minimal startup script (fetches secrets from Secret Manager — never inline the keys, see
[Security](security.md)):

```bash
#!/usr/bin/env bash
set -euo pipefail
LICENSE=$(gcloud secrets versions access latest --secret=darkmoon-license)
API_KEY=$(gcloud secrets versions access latest --secret=darkmoon-api-key)
[ -n "$LICENSE" ] && [ -n "$API_KEY" ] || { echo "secret fetch failed" >&2; exit 1; }
curl -fsSL https://portal.dark-moon.org/cloud | DARKMOON_LICENSE_KEY="$LICENSE" bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key "$API_KEY"
```

Give the VM a **service account** with `roles/secretmanager.secretAccessor` on exactly
those secrets. Startup scripts run as root, so no `sudo` is needed.

---

## Path B — one command with Terraform (from your laptop)

Use the GCP module at `deploy/terraform/gcp/`. It creates the Compute Engine instance, the
VPC firewall rule(s) with only the flows above, and injects the startup script via
metadata.

```bash
cd deploy/terraform/gcp
terraform init
terraform apply -var license_key=... -var ai_api_key=...
```

Expected variables include `machine_type` (default a Standard-class amd64 type),
`project`, `zone`, `trusted_cidr`, and the provider selection.

---

## Path C — from Google Cloud Shell (browser, free)

Cloud Shell ships `gcloud` and Terraform with your credentials already wired. Run Path B
there:

```bash
git clone https://github.com/ASCIT31/darkmoon-cloud-deploy    # or your fork with the deploy/ tree
cd darkmoon-cloud-deploy/deploy/terraform/gcp
terraform init && terraform apply -var license_key=... -var ai_api_key=...
```

---

## After launch

- Reach the dashboard at `http://<instance-ip>:80` from your trusted range. Put TLS in
  front if it is exposed beyond a trusted network ([Security](security.md)).
- `darkmoon doctor` to verify health; operate via the `darkmoon` CLI
  ([Doctor & lifecycle](doctor-and-lifecycle.md)).
- Hitting a problem? See [Troubleshooting](troubleshooting.md).
