# Azure

Three ways to stand up a DarkMoon node on an Azure VM. All three run the same
[`install.sh`](one-command.md) and produce the same node.

Pick an **amd64** VM size from the [Sizing](sizing.md) table (e.g. `Standard_D4s_v5` for
the Standard profile). arm64 (`Dpsv5`/`Dplsv5` ARM) is experimental — don't use it.

---

## Path A — one click from the portal (Custom data)

When creating the VM, go to the **Advanced** tab → **Custom data** and paste the startup
script (`deploy/cloud-init/darkmoon-user-data.sh`). On first boot (via cloud-init) the VM
provisions itself into a DarkMoon node.

Network Security Group (NSG) rules (see [Network](network.md)):

- **Inbound** TCP `80` (dashboard) from your **trusted CIDR** only; optionally TCP `22`.
- **Outbound** TCP `443` (image pulls, license, and the LLM provider in Connected mode).

Minimal custom-data (fetches secrets from Key Vault via the VM's managed identity — never
inline the keys, see [Security](security.md)):

```bash
#!/usr/bin/env bash
set -euo pipefail
TOKEN=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net" | jq -r .access_token)
LICENSE=$(curl -s -H "Authorization: Bearer $TOKEN" "https://<vault>.vault.azure.net/secrets/darkmoon-license?api-version=7.4" | jq -r .value)
API_KEY=$(curl -s -H "Authorization: Bearer $TOKEN" "https://<vault>.vault.azure.net/secrets/darkmoon-api-key?api-version=7.4" | jq -r .value)
[ -n "$LICENSE" ] && [ -n "$API_KEY" ] || { echo "secret fetch failed" >&2; exit 1; }
curl -fsSL https://portal.dark-moon.org/cloud | DARKMOON_LICENSE_KEY="$LICENSE" bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key "$API_KEY"
```

Enable a **system-assigned managed identity** on the VM and grant it *get* on those
secrets. Custom-data runs as root via cloud-init.

---

## Path B — one command with Terraform (from your laptop)

Use the Azure module at `deploy/terraform/azure/`. It creates the VM, an NSG with only the
flows above, and injects the custom-data.

```bash
cd deploy/terraform/azure
terraform init
terraform apply -var license_key=... -var ai_api_key=...
```

Expected variables include `vm_size` (default a Standard-class amd64 size),
`resource_group`, `location`, `trusted_cidr`, and the provider selection.

---

## Path C — from Azure Cloud Shell (browser, free)

Azure Cloud Shell ships the Azure CLI and Terraform with your credentials wired. Run
Path B there:

```bash
git clone https://github.com/ASCIT31/darkmoon-cloud-deploy    # or your fork with the deploy/ tree
cd darkmoon-cloud-deploy/deploy/terraform/azure
terraform init && terraform apply -var license_key=... -var ai_api_key=...
```

---

## After launch

- Reach the dashboard at `http://<vm-ip>:80` from your trusted CIDR. Put TLS in front if
  it is exposed beyond a trusted network ([Security](security.md)).
- `darkmoon doctor` to verify health; operate via the `darkmoon` CLI
  ([Doctor & lifecycle](doctor-and-lifecycle.md)).
- Hitting a problem? See [Troubleshooting](troubleshooting.md).
