# TrueFoundry sample GitOps repository

Sample repo for TrueFoundry GitOps: cluster, workspace, application, gateway, and access configuration in one place. CI runs **`tfy apply`** in dry-run on pull requests and applies changes when you merge (or push) to **`main`**.

All manifests live under **`truefoundry/`** so a single directory can be passed to the CLI (`tfy apply --dir truefoundry`).

## Folder structure

```
truefoundry/
├── access/                    # Identity, teams, roles, virtual accounts
│   ├── externalauth/
│   │   ├── identities/
│   │   └── identityproviders/
│   ├── roles/
│   ├── teams/
│   └── virtualaccounts/
├── ai-deployment/             # Clusters, workspaces, apps, envs, policies, model registry
│   ├── applications/
│   │   ├── jobs/
│   │   └── services/
│   ├── clusters/
│   │   └── <cluster>/
│   │       └── workspaces/
│   ├── environments/
│   ├── modelregistry/
│   └── polices/               # validate / mutate policies
├── ai-gateway/                # Gateway: models, guardrails, MCP, controls, agents
│   ├── agent/
│   ├── controls/
│   ├── guardrails/
│   ├── mcp-servers/
│   └── models/
├── integrations/              # Integration manifests (repo-level)
└── repositories/              # Repository definitions
```

### Top-level areas under `truefoundry/`

| Path | Purpose |
|------|--------|
| **access/** | External auth, identities, identity providers, roles, teams, virtual accounts |
| **ai-deployment/** | Clusters, nested workspaces & applications, environments, policies, custom models |
| **ai-gateway/** | AI Gateway: models, guardrails, MCP servers, controls, agents |
| **integrations/** | Integration provider configs |
| **repositories/** | Repository specs (artifacts, images, etc.) |

## File naming conventions

### Access

- **externalauth/identities/** — one YAML per external identity (`<name>.yaml`).
- **externalauth/identityproviders/** — one YAML per provider (`<name>.yaml`).
- **roles/** — `<role-name>.yaml`.
- **teams/** — `<team-name>.yaml`; typically align `.name` with the filename.
- **virtualaccounts/** — `<virtual-account-name>.yaml`; typically align `.name` with the filename.

### AI deployment

- **clusters/&lt;cluster&gt;/** — cluster manifest; cluster name usually matches the folder.
- **workspaces/** — per-workspace folder + `<workspace-name>.yaml`.
- **applications/** — one YAML per application (`<application-name>.yaml`).
- **environments/** — `<environment-name>.yaml`.
- **integrations** (under deployment if used) — `<integration-name>.yaml`.
- **polices/** — `<policy-name>.yaml` (validate / mutate).
- **modelregistry/** — `<model-registry-name>.yaml`.

### AI gateway

- **models/**, **guardrails/**, **mcp-servers/** — one YAML per resource; name aligned with filename where applicable.
- **controls/** — gateway control configs (routing, budget, data access, etc.).
- **agent/** — agent definitions.

### Repositories

- **repositories/** — `<repository-name>.yaml`.

---

## CI: GitHub Actions

Configure these **repository secrets**:

| Secret | Description |
|--------|-------------|
| **TFY_HOST** | TrueFoundry host URL |
| **TFY_API_KEY** | API key from the TrueFoundry dashboard |

Workflows (`.github/workflows/`):

### 1. `dry_run_on_pr.yaml`

- **When:** `pull_request` — `opened`, `synchronize`.
- **What it does:** Installs the TrueFoundry CLI and runs:
  - `tfy apply --dir truefoundry --dry-run --show-diff --diffs-only --ref origin/<base_branch>`
- **Meaning:** Only manifests that **differ from the PR base branch** are simulated; output includes diffs. No separate YAML/name validation step — invalid manifests surface as CLI errors.

### 2. `apply_on_merge.yaml`

- **When:** `push` to **`main`**.
- **What it does:**
  - `tfy apply --dir truefoundry --diffs-only --ref "${{ github.event.before }}"`
- **Meaning:** Applies only what changed in **that push** relative to the previous tip of `main` (typical merge or direct push).

> **Note:** Workflows must exist on the **default branch** for PR-triggered jobs to run. First-time workflow PRs need that file merged (or present) on the default branch.

---

## CI: Bitbucket Pipelines

`bitbucket-pipelines.yml` mirrors the same idea:

| Pipeline | When | Command |
|----------|------|--------|
| **TFY dry-run** | Pull requests | `tfy apply --dir truefoundry --dry-run --show-diff --diffs-only --ref "$BITBUCKET_PR_DESTINATION_COMMIT"` |
| **TFY apply** | Push to **main** | `tfy apply --dir truefoundry --diffs-only --ref "HEAD^"` |

Set secured variables **`TFY_API_KEY`** and **`TFY_HOST`**. The pipeline removes **`bitbucket-pipelines.yml`** before apply so the pipeline file is not treated as a manifest.

---

## Local usage

```bash
pip install -U truefoundry
export TFY_HOST=...
export TFY_API_KEY=...

# Dry-run entire tree vs current branch tip
tfy apply --dir truefoundry --dry-run --show-diff

# Apply everything under truefoundry (no git diff filter)
tfy apply --dir truefoundry
```

Use **`--diffs-only`** and **`--ref <commit>`** when you only want changed files (same as CI).
