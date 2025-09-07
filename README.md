## Robotina 2.0

A cross-tenancy, multi-region operations assistant for Oracle Cloud Infrastructure (OCI).

Robotina makes it easy to discover, query, and operate on your OCI estate across multiple tenancies, regions, and compartments. It inventories resources and safely executes fleet-wide actions such as SQL queries across databases and remote SSH commands on compute instances.

### Key capabilities

- **Tenancy discovery**: Fetch tenancy details including subscribed regions and compartment hierarchies for each configured tenancy.
- **Databases (DBaaS and ATP-S)**:
  - Discover all database instances across all regions, compartments, and tenancies.
  - Retrieve rich metadata for DBaaS and Autonomous Transaction Processing – Serverless (ATP-S).
  - Execute SQL queries across multiple databases spanning different tenancies and regions.
- **Compute Instances**:
  - Inventory instances across tenancies, regions, and compartments.
  - Execute remote SSH commands on targets in different tenancies, regions, and compartments (with optional Bastion support).

---

## Quick start

### Prerequisites

- **OCI access per tenancy** (choose one per tenancy):
  - API key profile in `~/.oci/config` (recommended for local runs)
  - or Instance/Dynamic Group + IAM policies (recommended for running inside OCI)
- **Permissions** in each tenancy (least-privilege examples below)
- **Local tooling**:
  - Git, OpenSSH, OpenSSL
  - Docker (optional, recommended), or a local runtime (e.g., Python 3.11+)
- **Network reachability** to targets (DB/SSH) from where Robotina runs
- **Secrets** for DB users and SSH keys stored securely as environment variables/files

### OCI credentials: multi-profile setup

Robotina supports multiple tenancies by using multiple profiles in `~/.oci/config`. Create one profile per tenancy.

```ini
[DEFAULT]
user=ocid1.user.oc1..aaaa...
fingerprint=aa:bb:cc:dd:ee:ff:...
key_file=~/.oci/robotina.pem
tenancy=ocid1.tenancy.oc1..aaaa...
region=us-ashburn-1

[prod]
user=ocid1.user.oc1..prod...
tenancy=ocid1.tenancy.oc1..prod...
region=eu-frankfurt-1
key_file=~/.oci/robotina-prod.pem
fingerprint=...

[dev]
user=ocid1.user.oc1..dev...
tenancy=ocid1.tenancy.oc1..dev...
region=us-phoenix-1
key_file=~/.oci/robotina-dev.pem
fingerprint=...
```

Alternatively, when running inside OCI, use a Dynamic Group + Policies to grant a compute instance or function the required permissions (see IAM policies below).

### IAM policies (least-privilege starting point)

Create or reuse a group (for user/API key usage) or a dynamic group (for instance principals). Then add policies in each tenancy Robotina should manage. Start with read/inspect access and expand if needed.

```
Allow group RobotinaAdmins to inspect compartments in tenancy
Allow group RobotinaAdmins to read all-resources in tenancy

# Optional granularity examples
Allow group RobotinaAdmins to read instance-family in tenancy
Allow group RobotinaAdmins to read database-family in tenancy
Allow group RobotinaAdmins to use bastion-family in tenancy
```

For Dynamic Groups (if running Robotina on OCI compute), scope the principal and use the same policies by replacing `group` with `dynamic-group`.

### Database access requirements

- **DBaaS (VM/BM/Exadata)**:
  - Reachable listener (default port 1521) or private access via VPN/FASTCONNECT
  - Service name(s) or connect strings
  - A database user with least privileges for your queries
- **ATP-S**:
  - Wallet ZIP downloaded for each ATP-S instance (or use Database Tools secrets)
  - `ADMIN` or least-privilege app user; store passwords securely

### SSH access requirements (compute)

- SSH private key(s) with access to target instances
- Username(s) (e.g., `opc`, `ubuntu`, `ec2-user`, or custom)
- Optional: OCI Bastion for private instances; ensure `bastion` use permissions

---

## Configuration

Create a configuration file to enumerate tenancies, regions, and execution defaults. The default path is `config/robotina.yaml`. Example:

```yaml
tenancies:
  - name: prod
    profile: prod              # Profile name from ~/.oci/config
    tenancy_ocid: ocid1.tenancy.oc1..aaaa...
    regions: [us-ashburn-1, eu-frankfurt-1]
    compartments:
      - name: all
        selector: "*"          # discover all compartments

  - name: dev
    profile: dev
    tenancy_ocid: ocid1.tenancy.oc1..bbbb...
    regions: [us-phoenix-1]
    compartments:
      - name: platform
        selector: ocid1.compartment.oc1..cccc...

databases:
  dbaas:
    default_port: 1521
    auth:
      username: ROBOTINA
      password_env: ROBOTINA_DB_PASSWORD

  atps:
    wallet_dir: wallets/       # Directory containing extracted ATP-S wallets
    users:
      - username: ADMIN
        password_env: ROBOTINA_ATP_ADMIN_PASSWORD

compute:
  ssh:
    default_user: opc
    private_key_path: ~/.ssh/robotina.pem
    bastion:
      enabled: false

execution:
  concurrency: 8
  retry:
    attempts: 2
    backoff_ms: 500
```

Environment variables for secrets (example):

```bash
export ROBOTINA_DB_PASSWORD='strong-db-password'
export ROBOTINA_ATP_ADMIN_PASSWORD='strong-atp-password'
export ROBOTINA_CONFIG="$(pwd)/config/robotina.yaml"
```

---

## Install and run

Robotina can run in a container or directly on your machine. Choose one path below.

### Option A: Docker

```bash
# Build
docker build -t robotina:latest .

# Run with OCI config and Robotina config mounted
docker run --rm \
  -v "$HOME/.oci:/root/.oci:ro" \
  -v "$(pwd)/config:/app/config:ro" \
  -v "$(pwd)/wallets:/app/wallets:ro" \
  -e ROBOTINA_CONFIG=/app/config/robotina.yaml \
  -e ROBOTINA_DB_PASSWORD \
  -e ROBOTINA_ATP_ADMIN_PASSWORD \
  robotina:latest
```

### Option B: Local environment (example: Python runtime)

```bash
# Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -U pip
pip install -r requirements.txt

# Configure environment
export ROBOTINA_CONFIG="$(pwd)/config/robotina.yaml"
export ROBOTINA_DB_PASSWORD='strong-db-password'
export ROBOTINA_ATP_ADMIN_PASSWORD='strong-atp-password'

# First run
robotina --help
```

Note: The exact runtime and commands may differ based on implementation language. The Docker path works regardless of local language runtime.

---

## Typical workflows

Commands shown below are illustrative. Names and flags may vary depending on the CLI implementation.

### 1) Discover tenancies, regions, and compartments

```bash
robotina inventory tenancies --all
robotina inventory compartments --tenancies prod,dev --regions us-ashburn-1,eu-frankfurt-1
```

### 2) Inventory and query databases (DBaaS + ATP-S)

```bash
# List all DBs across tenancies/regions
robotina db list --tenancies prod,dev --all-regions

# Execute a SQL statement across ATP-S instances in prod
robotina db query \
  --engine atps \
  --sql 'select sysdate from dual' \
  --tenancies prod \
  --regions eu-frankfurt-1,us-ashburn-1
```

### 3) Inventory and operate on compute instances via SSH

```bash
# List compute instances
robotina compute list --tenancies prod --regions us-ashburn-1 --compartments "*"

# Run a remote command across matching instances (with filters/selectors)
robotina compute ssh \
  --cmd 'uname -a' \
  --tenancies prod,dev \
  --regions us-ashburn-1,us-phoenix-1 \
  --selector 'env=prod'
```

---

## Security guidance

- Use least-privilege IAM policies and limit the scope to required compartments.
- Store secrets in environment variables or secret managers; avoid committing them to git.
- Prefer OCI Bastion for private instance access and avoid exposing SSH over the internet.
- Rotate API keys and DB credentials regularly.

---

## Troubleshooting

- **Auth errors**: Verify the selected `profile` exists in `~/.oci/config` and that the key, fingerprint, and tenancy OCID are correct.
- **Permission denied**: Confirm IAM policies are in place in each tenancy and that compartments are in scope.
- **Network timeouts**: Ensure your runner has network access to DB/compute targets; consider Bastion or private endpoints.
- **ATP-S connectivity**: Confirm wallet directory is mounted and `tnsnames.ora` entries match your targets.
- **SSH failures**: Validate the private key path, username, and that the instance’s `authorized_keys` includes the public key.

---

## Roadmap (high level)

- Fine-grained selectors for targets (tags, shapes, lifecycle state)
- Pluggable credential providers (OCI Vault, HashiCorp Vault)
- Safe execution guards and dry-run for fleet operations
- Exporters for metrics and audit logs

---

## Disclaimer

This is a first-draft README intended to capture the vision and setup for Robotina 2.0. Command names and configuration fields may evolve as the implementation solidifies.

