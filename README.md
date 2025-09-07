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

## Commands (quick reference)

### Frontend (React app)

```bash
cd react-app

# Install dependencies
npm install

# Start dev server (Vite)
npm run dev

# Build production assets
npm run build

# Preview the production build locally
npm run preview

# Lint the codebase
npm run lint
```

Optional Vite env (create `.env.local` inside `react-app/`):

```bash
echo 'VITE_API_BASE_URL=http://localhost:5678/webhook/robotina-webhook' >> react-app/.env.local
```

### Backend (Docker)

```bash
# From repository root
docker build -t robotina:latest .
docker run --rm \
  -v "$HOME/.oci:/root/.oci:ro" \
  -v "$(pwd)/config:/app/config:ro" \
  -v "$(pwd)/wallets:/app/wallets:ro" \
  -e ROBOTINA_CONFIG=/app/config/robotina.yaml \
  -e ROBOTINA_DB_PASSWORD \
  -e ROBOTINA_ATP_ADMIN_PASSWORD \
  robotina:latest
```

### Backend (local runtime example)

```bash
# Python virtualenv example
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt

# Required env
export ROBOTINA_CONFIG="$(pwd)/config/robotina.yaml"
export ROBOTINA_DB_PASSWORD='strong-db-password'
export ROBOTINA_ATP_ADMIN_PASSWORD='strong-atp-password'

# Explore CLI
robotina --help
```

### Robotina CLI (common examples)

```bash
# Discover tenancies and compartments
robotina inventory tenancies --all
robotina inventory compartments --tenancies prod,dev --regions us-ashburn-1,eu-frankfurt-1

# Databases: list + query
robotina db list --tenancies prod,dev --all-regions
robotina db query \
  --engine atps \
  --sql 'select sysdate from dual' \
  --tenancies prod \
  --regions eu-frankfurt-1,us-ashburn-1

# Compute: list + SSH command
robotina compute list --tenancies prod --regions us-ashburn-1 --compartments "*"
robotina compute ssh \
  --cmd 'uname -a' \
  --tenancies prod,dev \
  --regions us-ashburn-1,us-phoenix-1 \
  --selector 'env=prod'
```

### Webhook (n8n) request example

```bash
curl -sS -X POST http://localhost:5678/webhook/robotina-webhook \
  -H 'Content-Type: application/json' \
  -d '{
    "tenancy": "oacnativeproduction",
    "region": "us-ashburn-1",
    "realm": "oc1",
    "target_type": "databases",
    "identifier": "*",
    "action": "list",
    "params": {},
    "cache_refresh": false
  }'
```

---

## Backend architecture and automation (n8n + webhook + cache)

### Overview

- **n8n workflow**: Orchestrates Robotina backend actions via an HTTP webhook.
- **Webhook**: `POST http://localhost:5678/webhook/robotina-webhook`
- **Data sources**: OCI CLI (authoritative), Redis (cache of recent OCI CLI responses)
- **Local MCP server**: Exposed by n8n to allow local LLMs with MCP support to interact with OCI operations through Robotina tools.

### Backend stack

- **Automation**: n8n (self-hosted)
- **Cloud control plane**: Oracle Cloud Infrastructure via OCI CLI
- **Cache**: Redis (ephemeral responses from OCI CLI)
- **LLM integration**: Local MCP server (exposed by n8n) for MCP-capable local LLMs
- **API surface**: n8n webhook (HTTP POST)

### Webhook contract

Expected JSON payload:

```json
{
  "tenancy": "oacnativeproduction",
  "region": "us-ashburn-1",
  "realm": "oc1",
  "target_type": "tenancies | databases | compute | health",
  "identifier": "display-name | ocid | pattern",
  "action": "list | ssh | sql",
  "params": {},
  "cache_refresh": false
}
```

Notes:
- **target_type**: Selects the resource domain for the action.
- **identifier**: Matches the target(s) by display name, OCID, or pattern.
- **action**: The operation to perform (e.g., inventory list, remote SSH, or SQL query).
- **params**: Action-specific options (e.g., SQL text, SSH command, filters).

### Caching behavior

- **cache_refresh = true**: n8n calls the OCI CLI to fetch fresh data, updates Redis, and returns the fresh result.
- **cache_refresh = false**: n8n attempts to read from Redis first. On cache miss, it falls back to OCI CLI, populates Redis, and returns the result.
- **Redis usage**: n8n uses Redis as a temporary cache for OCI CLI responses.

### Example request

```bash
curl -sS -X POST http://localhost:5678/webhook/robotina-webhook \
  -H 'Content-Type: application/json' \
  -d '{
    "tenancy": "oacnativeproduction",
    "region": "us-ashburn-1",
    "realm": "oc1",
    "target_type": "databases",
    "identifier": "*",
    "action": "list",
    "params": {},
    "cache_refresh": false
  }'
```

### Frontend integration (React)

- The frontend is a React app that interacts with the n8n webhook above.
- The React client maintains a secondary cache in `localStorage` to speed up repeated interactions.
- Suggested keying: hash of the request payload (plus an optional TTL) to balance freshness versus responsiveness.
- See "Frontend tech stack (Robotina 2.0)" below for details.

### Frontend tech stack (Robotina 2.0)

- Framework: React + TypeScript + Vite
- UI: TailwindCSS + shadcn/ui (Radix under the hood)
- Charts: Recharts
- Routing: React Router
- State: Zustand (lightweight global)
- Server state / caching: TanStack Query
- Forms & validation: react-hook-form + zod
- HTTP: native fetch with a tiny wrapper (typed)
- Env: import.meta.env (Vite) with VITE_-prefixed vars
- Testing: Vitest + Testing Library
- Lint/Format: ESLint + Prettier

#### Details and usage

##### Framework: React + TypeScript + Vite

- Description: Modern React app scaffolded with Vite for fast dev server and optimized builds, with TypeScript for strong types.
- Why useful here: Enables a responsive, type-safe UI for cross-tenancy operations with quick iteration speed.
- How to use (in `react-app/`):
  - Dev: `npm run dev`
  - Build: `npm run build`; Preview: `npm run preview`
- Example (typed component):

```tsx
// react-app/src/components/Hello.tsx
import React from 'react';

type HelloProps = { name: string };
export function Hello({ name }: HelloProps) {
  return <div>Hello, {name}</div>;
}
```

##### UI: TailwindCSS + shadcn/ui (Radix under the hood)

- Description: Utility-first CSS (Tailwind) plus accessible headless primitives (Radix) wrapped as ready-to-use components (shadcn/ui).
- Why useful here: Consistent, accessible UI for complex forms, tables, dialogs, and command palettes without heavy custom CSS.
- How to use:
  - Tailwind classes in JSX; import shadcn/ui components as needed.
- Example:

```tsx
// react-app/src/components/Toolbar.tsx
import { Button } from "@/components/ui/button";

export function Toolbar() {
  return (
    <div className="flex items-center gap-2 p-2 border-b">
      <Button>Run</Button>
      <Button variant="secondary">Refresh</Button>
    </div>
  );
}
```

##### Charts: Recharts

- Description: Composable charting library for React based on D3 under the hood.
- Why useful here: Visualizes inventory counts, regional health, query timings.
- Example:

```tsx
// react-app/src/components/DbLatencyChart.tsx
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

export function DbLatencyChart({ data }: { data: Array<{ ts: string; ms: number }> }) {
  return (
    <ResponsiveContainer width="100%" height={240}>
      <LineChart data={data}>
        <XAxis dataKey="ts" />
        <YAxis />
        <Tooltip />
        <Line type="monotone" dataKey="ms" stroke="#0ea5e9" />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

##### Routing: React Router

- Description: Client-side routing for React.
- Why useful here: Separates views like Tenancies, Databases, Compute, and Health.
- Example:

```tsx
// react-app/src/main.tsx (excerpt)
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Databases from "./routes/Databases";

const router = createBrowserRouter([
  { path: "/", element: <App /> },
  { path: "/databases", element: <Databases /> },
]);

// <RouterProvider router={router} /> in the root
```

##### State: Zustand (lightweight global)

- Description: Minimal global state without boilerplate.
- Why useful here: Share small app state (selected tenancy/region, UI filters) without Redux overhead.
- Example:

```ts
// react-app/src/state/useSelectionStore.ts
import { create } from "zustand";

type SelectionState = {
  tenancy: string | null;
  setTenancy: (t: string | null) => void;
};

export const useSelectionStore = create<SelectionState>((set) => ({
  tenancy: null,
  setTenancy: (t) => set({ tenancy: t }),
}));
```

##### Server state / caching: TanStack Query

- Description: Data fetching, caching, revalidation, and request deduping.
- Why useful here: Keeps webhook-driven data fresh and consistent across views with retries and cache.
- Example:

```tsx
// react-app/src/app/query.tsx
import { QueryClient, QueryClientProvider, useQuery } from "@tanstack/react-query";

const client = new QueryClient();
export function QueryRoot({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>;
}

export function useTenancies() {
  return useQuery({
    queryKey: ["tenancies"],
    queryFn: async () => {
      const res = await fetch("/api/tenancies");
      return res.json();
    },
  });
}
```

##### Forms & validation: react-hook-form + zod

- Description: Performant form state with schema validation.
- Why useful here: Safe command inputs (SQL text, SSH options) with instant feedback.
- Example:

```tsx
// react-app/src/components/SqlForm.tsx
import { useForm } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({ sql: z.string().min(1), tenancy: z.string().min(1) });
type FormData = z.infer<typeof schema>;

export function SqlForm({ onSubmit }: { onSubmit: (v: FormData) => void }) {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({ resolver: zodResolver(schema) });
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("sql")} placeholder="select sysdate from dual" />
      {errors.sql && <span>Required</span>}
      <button type="submit">Run</button>
    </form>
  );
}
```

##### HTTP: native fetch with a tiny wrapper (typed)

- Description: Use Web Fetch with a small typed helper for JSON requests.
- Why useful here: Minimal dependency surface; typed responses.
- Example:

```ts
// react-app/src/utils/http.ts
export async function httpJson<T>(input: RequestInfo, init?: RequestInit): Promise<T> {
  const res = await fetch(input, { ...init, headers: { "Content-Type": "application/json", ...(init?.headers || {}) } });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<T>;
}

// Usage
// const data = await httpJson<MyType>(`/api/databases`, { method: "POST", body: JSON.stringify(payload) });
```

##### Env: import.meta.env (Vite) with VITE_-prefixed vars

- Description: Build-time injected env vars.
- Why useful here: Configure webhook base URL, feature flags per environment.
- Example:

```ts
// react-app/src/config.ts
export const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "/api";
```

##### Testing: Vitest + Testing Library

- Description: Unit/component tests with a Jest-like runner and DOM utilities.
- Why useful here: Validates UI logic for risky operations (SQL/SSH) without hitting real backends.
- Example:

```tsx
// react-app/src/components/__tests__/Hello.test.tsx
import { render, screen } from "@testing-library/react";
import { Hello } from "../Hello";

it("renders name", () => {
  render(<Hello name="Robotina" />);
  expect(screen.getByText(/Robotina/)).toBeInTheDocument();
});
```

##### Lint/Format: ESLint + Prettier

- Description: Linting and formatting for consistent, error-free code.
- Why useful here: Keeps a fast-moving UI codebase readable and safe.
- How to use (in `react-app/`):
  - Lint: `npm run lint`
  - Format: `npm run format`

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

