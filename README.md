# TrueFoundry sample GitOps repository

Sample repo for TrueFoundry GitOps: cluster, workspace, application, gateway, and access configuration in one place. CI runs **`tfy apply`** in dry-run on pull requests and applies changes when you merge (or push) to **`main`**.

All manifests live under **`truefoundry/`** so a single directory can be passed to the CLI (`tfy apply --dir truefoundry`).

## Folder structure

```
truefoundry/
├── access/                    # Identity providers, teams, roles, virtual accounts, secret groups
│   ├── identityproviders/
│   ├── roles/
│   ├── secretgroups/
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
├── ai-gateway/                # Gateway: models, guardrails, MCP, skills, prompts, controls, agents
│   ├── agents/
│   ├── controls/
│   ├── guardrails/
│   ├── mcp-servers/
│   ├── models/
│   ├── prompts/
│   ├── skills/
│   └── virtual-models/
└── repositories/              # Repository definitions
```

### Top-level areas under `truefoundry/`

| Path | Purpose |
|------|--------|
| **access/** | Identity providers, roles, secret groups, teams, virtual accounts |
| **ai-deployment/** | Clusters, nested workspaces & applications, environments, policies, custom models |
| **ai-gateway/** | AI Gateway: models, virtual models, guardrails, MCP servers, skills, prompts, controls, agents |
| **repositories/** | Repository specs (artifacts, images, etc.) |

## File naming conventions

### Access

- **identityproviders/** — one YAML per identity provider (`<name>.yaml`).
- **roles/** — `<role-name>.yaml`.
- **secretgroups/** — `<secret-group-name>.yaml`.
- **teams/** — `<team-name>.yaml`; typically align `.name` with the filename.
- **virtualaccounts/** — `<virtual-account-name>.yaml`; typically align `.name` with the filename.

### AI deployment

- **clusters/&lt;cluster&gt;/** — cluster manifest, with its role bindings after a `---`; cluster name usually matches the folder.
- **workspaces/** — per-workspace folder + `<workspace-name>.yaml`, with its role bindings after a `---`.
- **applications/** — one YAML per application (`<application-name>.yaml`).
- **environments/** — `<environment-name>.yaml`.
- **integrations** (under deployment if used) — `<integration-name>.yaml`.
- **polices/** — `<policy-name>.yaml` (validate / mutate).
- **modelregistry/** — `<model-registry-name>.yaml`.

### AI gateway

- **models/**, **virtual-models/**, **guardrails/**, **mcp-servers/** — one YAML per resource, with its role bindings after a `---`; name aligned with filename where applicable.
- **skills/**, **prompts/** — one YAML per skill or prompt; each names the `ml_repo` it is stored in (see `repositories/`).
- **controls/** — gateway control configs (routing, rate limits, budgets, guardrail rules, data access, logging, OTEL, global settings).
- **agents/** — agent definitions.

### Repositories

- **repositories/** — `<repository-name>.yaml`.

## Access control (role bindings)

Access to a resource is managed with **`role-binding`** manifests. A role binding lives in the **same YAML file as its resource**, as a second document after a `---` separator:

```yaml
name: openai-model
type: provider-account/openai
# ... the resource ...
---
type: role-binding
name: pa-openai-model-manager-4p
subjects:
  - type: team
    name: everyone
permissions:
  - resourceType: provider-account
    resourceFqn: <TENANT_NAME>:openai:openai-model
    role: provider-account-manager
```

- `collaborators` on a resource is the older, outdated way to grant access. It still works, but the samples use role bindings instead.
- To convert an existing repo, run `tfy migrate-gitops -d truefoundry`. It rewrites each file's `collaborators` into role bindings in place; review the diff, then apply.

## Things to know before applying

- **Apply `repositories/` first on a new tenant.** Prompts and skills are stored in an ML repo, and `tfy apply` does not wait for the repo to be created first: `tfy apply -f truefoundry/repositories/<repository-name>.yaml`.
- **Some gateway configs are tenant-wide.** `otel-config.yaml`, `global-settings.yaml` and `data-access.yaml` have one copy per tenant, so applying them replaces the existing settings.

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
  - `tfy apply --dir truefoundry --dry-run --show-diff --diffs-only --sync --ref origin/<base_branch>`
- **Meaning:** Only manifests that **differ from the PR base branch** are simulated; output includes diffs. No separate YAML/name validation step — invalid manifests surface as CLI errors.

### 2. `apply_on_merge.yaml`

- **When:** `push` to **`main`**.
- **What it does:**
  - `tfy apply --dir truefoundry --diffs-only --sync --ref "${{ github.event.before }}"`
- **Meaning:** Applies only what changed in **that push** relative to the previous tip of `main` (typical merge or direct push).

> **Note:** Workflows must exist on the **default branch** for PR-triggered jobs to run. First-time workflow PRs need that file merged (or present) on the default branch.

---

## CI: Bitbucket Pipelines

`bitbucket-pipelines.yml` mirrors the same idea:

| Pipeline | When | Command |
|----------|------|--------|
| **TFY dry-run** | Pull requests | `tfy apply --dir truefoundry --dry-run --show-diff --diffs-only --sync --ref "$BITBUCKET_PR_DESTINATION_COMMIT"` |
| **TFY apply** | Push to **main** | `tfy apply --dir truefoundry --diffs-only --sync --ref "HEAD^"` |

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
