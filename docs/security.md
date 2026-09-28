# Security — keep secrets out of instance metadata

The single most important rule when you automate a cloud deployment:

> **Do not inline the license key or the API key in user-data.**

User-data / custom-data / startup-scripts are **readable from the instance metadata
service and from the cloud console** by anyone with access to the instance. A key pasted
there is a key leaked. Instead, **fetch secrets at boot from the cloud secret manager**.

## Use the cloud secret manager

Store the DarkMoon license key and the LLM API key in the managed secret store, grant the
VM an identity that can read exactly those secrets, and fetch them in the startup script
just before invoking the installer.

### AWS — SSM Parameter Store or Secrets Manager

Attach an instance role with least-privilege read on the specific parameters/secrets.

```bash
LICENSE=$(aws ssm get-parameter --name /darkmoon/license --with-decryption --query Parameter.Value --output text)
API_KEY=$(aws ssm get-parameter --name /darkmoon/api-key --with-decryption --query Parameter.Value --output text)
# or Secrets Manager:
# LICENSE=$(aws secretsmanager get-secret-value --secret-id darkmoon/license --query SecretString --output text)
```

### GCP — Secret Manager

Grant the VM service account `roles/secretmanager.secretAccessor` on the specific secrets.

```bash
LICENSE=$(gcloud secrets versions access latest --secret=darkmoon-license)
API_KEY=$(gcloud secrets versions access latest --secret=darkmoon-api-key)
```

### Azure — Key Vault

Enable a system-assigned managed identity on the VM and grant it *get* on the secrets.

```bash
TOKEN=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net" | jq -r .access_token)
LICENSE=$(curl -s -H "Authorization: Bearer $TOKEN" "https://<vault>.vault.azure.net/secrets/darkmoon-license?api-version=7.4" | jq -r .value)
API_KEY=$(curl -s -H "Authorization: Bearer $TOKEN" "https://<vault>.vault.azure.net/secrets/darkmoon-api-key?api-version=7.4" | jq -r .value)
```

### OVH

OVH Public Cloud has no first-party managed secret store equivalent. Options: run an
open-source secrets manager you control, fetch from a secret store in another cloud over
443, or provision the keys out-of-band (e.g. an operator SSHes in and runs the one-liner
[interactively](one-command.md)) rather than baking them into `user_data`.

Then hand the fetched values to the installer via the environment and flags:

```bash
curl -fsSL https://portal.dark-moon.org/cloud | sudo DARKMOON_LICENSE_KEY="$LICENSE" bash -s -- \
  --provider anthropic --model claude-opus-4-6 --api-key "$API_KEY"
```

## Host & network hardening

- **Dedicated host.** Run DarkMoon on a machine that does nothing else. It executes
  offensive tooling; do not co-locate it with unrelated workloads.
- **Lock the dashboard.** Restrict the `ui_port` (default 80) to trusted CIDRs in the
  security group / firewall. Do not expose it to `0.0.0.0/0`.
- **TLS in front.** If the dashboard is reachable beyond a trusted network, terminate TLS
  in front of it (a reverse proxy / load balancer with a certificate). The node speaks
  HTTP on `ui_port`; do not expose that in the clear over the internet.
- **Minimal inbound.** Only the dashboard port (and optionally SSH 22). See
  [Network](network.md).
- **Patch the host.** Keep the OS and Docker patched. Use `darkmoon update` for the
  DarkMoon stack itself (see [Doctor & lifecycle](doctor-and-lifecycle.md)).
- **Scope the scanner outbound** to the target ranges you are authorized to test.

## Don't echo secrets into logs

Startup scripts often run under logging. Avoid `set -x` around the lines that handle the
license/API key, and prefer environment variables over positional arguments so keys do
not show up in a process list or a shell trace.
