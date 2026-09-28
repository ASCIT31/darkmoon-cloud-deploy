# AWS

Three ways to stand up a DarkMoon node on EC2. All three run the same
[`install.sh`](one-command.md) and produce the same node.

Pick an **amd64** instance type from the [Sizing](sizing.md) table (e.g. `t3.xlarge` for
the Standard profile). arm64 (Graviton) is experimental — don't use it.

---

## Path A — one click from the console (User data)

When launching the instance in the EC2 console, expand **Advanced details → User data**
and paste the startup script (`deploy/cloud-init/darkmoon-user-data.sh`). On first boot
the instance provisions itself into a DarkMoon node.

Security-group rules to set at launch (see [Network](network.md)):

- **Inbound** TCP `80` (dashboard) from your **trusted CIDR** only; optionally TCP `22`.
- **Outbound** TCP `443` (image pulls, license, and the LLM provider in Connected mode).

Minimal user-data (fetches secrets from SSM — never inline the keys, see
[Security](security.md)):

```bash
#!/usr/bin/env bash
set -euo pipefail
LICENSE=$(aws ssm get-parameter --name /darkmoon/license --with-decryption --query Parameter.Value --output text)
API_KEY=$(aws ssm get-parameter --name /darkmoon/api-key --with-decryption --query Parameter.Value --output text)
[ -n "$LICENSE" ] && [ -n "$API_KEY" ] || { echo "secret fetch failed" >&2; exit 1; }
curl -fsSL https://portal.dark-moon.org/cloud | DARKMOON_LICENSE_KEY="$LICENSE" bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key "$API_KEY"
```

Attach an **instance role** that can read exactly those two parameters. User-data runs as
root on first boot, so no `sudo` is needed inside it.

---

## Path B — one command with Terraform (from your laptop)

Use the AWS module at `deploy/terraform/aws/`. It creates the EC2 instance, a security
group with only the flows above, and injects the user-data.

```bash
cd deploy/terraform/aws
terraform init
terraform apply -var license_key=... -var ai_api_key=...
```

The module is expected to expose variables like `instance_type` (default a Standard-class
amd64 type), `trusted_cidr` (who may reach the dashboard), `region`, `key_name`, and the
provider selection. Prefer passing secrets via a secret manager / Terraform variables
marked `sensitive` rather than committing them.

---

## Path C — from AWS CloudShell (browser, free)

CloudShell already has the AWS CLI and Terraform-friendly credentials. Run Path B's
commands there without installing anything locally:

```bash
git clone https://github.com/ASCIT31/darkmoon-cloud-deploy    # or your fork with the deploy/ tree
cd darkmoon-cloud-deploy/deploy/terraform/aws
terraform init && terraform apply -var license_key=... -var ai_api_key=...
```

---

## After launch

- Reach the dashboard at `http://<instance-ip>:80` from your trusted CIDR. Put TLS in
  front if it is exposed beyond a trusted network ([Security](security.md)).
- `darkmoon doctor` to verify health; operate via the `darkmoon` CLI
  ([Doctor & lifecycle](doctor-and-lifecycle.md)).
- Hitting a problem? See [Troubleshooting](troubleshooting.md).
