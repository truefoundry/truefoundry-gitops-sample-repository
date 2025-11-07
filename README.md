# Truefoundry Sample Gitops Repo

This is a sample repo for Truefoundry's Gitops workflow. It comprises of the folder structure to keep the cluster, workspace and application configurations in a single repository. It also includes github actions to validate the configuration files on every PR and apply the changes on merge to the main branch.

## Folder Structure

The folder structure is as follows:

```
clusters/
├── cluster1/
│   ├── cluster1.yaml
│   └── workspaces/
│       └── workspace1/
│           ├── workspace1.yaml
│           └── applications/
│               └── app1.yaml
└── cluster2/
    ├── cluster2.yaml
    └── workspaces/
        └── workspace1/
            ├── workspace1.yaml
            └── applications/
                └── sample-app.yaml

gateway/
├── models/
│   └── openai-model.yaml
├── guardrails/
│   └── openai-guardrail.yaml
└── mcp-servers/
    └── mcp1.yaml

integrations/
└── custom.yaml

teams/
├── backend.yaml
└── frontend.yaml

virtualaccounts/
└── virtualaccount1.yaml
```

The repository contains the following top-level folders:

- **clusters/** - Cluster configurations with nested workspaces and applications
- **gateway/** - AI Gateway configurations including models, guardrails, and MCP servers
- **integrations/** - Integration provider configuration
- **teams/** - Team configuration
- **virtualaccounts/** - Virtual account configuration

## File Naming Convention

### Clusters
Each file in the clusters folder should follow the naming convention of `<cluster-name>.yaml`. The cluster name should be the same as the folder name.

The workspaces folder should contain a folder for each workspace. The folder name should be the same as the workspace name. The workspace configuration file should follow the naming convention of `<workspace-name>.yaml`.

The applications folder contains the spec of all the applications. Each application should be in a separate file and follow the naming convention of `<application-name>.yaml`.

### Gateway
The gateway folder contains three subfolders:
- **models/** - Each model file should follow the naming convention of `<model-name>.yaml`
- **guardrails/** - Each guardrail file should follow the naming convention of `<guardrail-name>.yaml`
- **mcp-servers/** - Each MCP server file should follow the naming convention of `<mcp-server-name>.yaml`

### Integrations
Each integration file in the integrations folder should follow the naming convention of `<integration-name>.yaml`. The integration name in the file should match the filename.

### Teams
Each team file in the teams folder should follow the naming convention of `<team-name>.yaml`. The team name in the file should match the filename.

### Virtual Accounts
Each virtual account file in the virtualaccounts folder should follow the naming convention of `<virtualaccount-name>.yaml`. The virtual account name in the file should match the filename.

## Github actions

We will use Github actions to validate the configuration files and also apply them on every push. To set up the actions, we will need to create the following Github repository secrets:
- `TFY_HOST` - Your TrueFoundry host URL  
- `TFY_API_KEY` - The API key which you can generate from the Truefoundry dashboard

There are primarily 2 Github actions in the .github/workflows folder:

1. **dry_run_on_pr.yaml** - This action is triggered on every pull request and subsequent changed to the PR. It validated the following things on every PR:

 - The filename matches the name field in the yaml file
 - It executes apply in dry-run mode for the changed file. 

2. **apply_on_merge.yaml** - This action is triggered on every merge to the main branch. It applied all the changed filed in the commit and deleted the configurations for the deleted files.
