# Truefoundry Sample Gitops Repo

This is a sample repo for Truefoundry's Gitops workflow. It comprises of the folder structure to keep the cluster, workspace and application configurations in a single repository. It also includes github actions to validate the configuration files on every PR and apply the changes on merge to the main branch.

## Folder Structure

The folder structure is as follows:

```
access/
├── externalauth/
│   ├── identies/
│   │   └── customer1.yaml
│   └── identityproviders/
│       └── custom-oidc.yaml
├── roles/
│   └── custom.yaml
├── teams/
│   ├── backend.yaml
│   └── frontend.yaml
└── virtualaccounts/
    └── virtualaccount1.yaml

clusters/
├── cluster1/
│   ├── cluster1.yaml
│   └── workspaces/
│       └── workspace1/
│           ├── applications/
│           │   └── app1.yaml
│           └── workspace1.yaml
└── cluster2/
    ├── cluster2.yaml
    └── workspaces/
        ├── applications/
        │   └── sample-app.yaml
        └── workspace1.yaml

gateway/
├── agent/
│   └── github-agent.yaml
├── controls/
│   ├── budget.yaml
│   ├── data-access.yaml
│   ├── gateway-data-routing-config.yaml
│   ├── guardrails-control.yaml
│   ├── rate-limiting-config.yaml
│   └── routing-config.yaml
├── guardrails/
│   └── openai-guardrail.yaml
├── mcp-servers/
│   └── mcp1.yaml
└── models/
    └── openai-model.yaml

modelregistry/
└── custom-model1.yaml

platform/
├── environments/
│   ├── devlopment.yaml
│   └── production.yaml
├── integrations/
│   └── custom.yaml
└── polices/
    ├── mutate-policy.yaml
    └── validate-policy.yaml

repositories/
└── ml-agent.yaml
```

The repository contains the following top-level folders:

- **access/** - Identity, token, teams and roles configuration
- **clusters/** - Cluster configurations with nested workspaces and applications
- **gateway/** - AI Gateway configurations including models, guardrails, and MCP servers
- **modelregistry/** - You can save your custom models for them to use
- **platform/** - All cluster level configurations like intregrations, policies, environments
- **repositories/** - Create Repository for different usecases

## File Naming Convention

### Access
The access folder contains four subfolders:
- **externalauth/** - Identies Configurations
- **roles/** - Role configuration where each file follow the naming convention of `<role-name>.yaml`
- **teams/** - Team configuration where each file follow the naming convention of `<team-name>.yaml`
- **virtualaccounts/** - Virtual account configuration where each file follow the naming convention of `<virtual-account-name>.yaml`

#### ExternalAuth
The externalauth folder contains two subfolders:
- **identities/** - External Identity Configuration where each file follow the naming convention of `<identities-name>.yaml`
- **identityproviders/** - External Identity Provider Configuration where each file follow the naming convention of `<identity-provider-name>.yaml`

##### Identities
This folder have yamls for External Identities that refers to a client or tenant.

##### IdentityProviders
This folder have yamls for External Identity providers where you can add your details for the providers.

#### Roles
This folder have yamls for the roles that you can create to attach to a virtual account or a user.

#### Teams
Each team file in the teams folder should follow the naming convention of `<team-name>.yaml`. The team name in the file should match the filename.

#### Virtual Accounts
Each virtual account file in the virtualaccounts folder should follow the naming convention of `<virtualaccount-name>.yaml`. The virtual account name in the file should match the filename.

### Clusters
Each file in the clusters folder should follow the naming convention of `<cluster-name>.yaml`. The cluster name should be the same as the folder name.

The workspaces folder should contain a folder for each workspace. The folder name should be the same as the workspace name. The workspace configuration file should follow the naming convention of `<workspace-name>.yaml`.

The applications folder contains the spec of all the applications. Each application should be in a separate file and follow the naming convention of `<application-name>.yaml`.

### Gateway
The gateway folder contains three subfolders:
- **models/** - Each model file should follow the naming convention of `<model-name>.yaml`
- **guardrails/** - Each guardrail file should follow the naming convention of `<guardrail-name>.yaml`
- **mcp-servers/** - Each MCP server file should follow the naming convention of `<mcp-server-name>.yaml`

#### Models
This folder contain yamls for each model which can be custom as well.

#### Guardrails
This folder contain yamls for guardrails which are basically rules for the requests.

#### MCP-Servers
This folder contains MCP servers yamls for each of the MCP Server which can be the external ones or a custom added one.

### ModelRegistry
The Storage for Custom Model in the Registry to save and use them for future use and plug and play. Each file follow the naming convention of `<model-registry-name>.yaml`.

### Platform
The platform folder contains three subfolders:
- **environments/** - Create Environment tags to attached to the resources where each file follow the naming convention of `<environment-name>.yaml`
- **integrations/** - Integration provider configuration where each file follow the naming convention of `<integration-name>.yaml`
- **polices/** - Create Validate & Mutate Policy for AI Gateway requests where each file follow the naming convention of `<policy-name>.yaml`

#### Environments
This folder contains yamls for each environment tag to apply those tag on a resource to segregate them.

#### Integrations
Each integration file in the integrations folder should follow the naming convention of `<integration-name>.yaml`. The integration name in the file should match the filename.

#### Policies
This folder contains yamls for each policy that we have where a policy can be attached to the entities for restricting or allowing them for specific conditions with mostly of two types validate and mutate.

### Repositories
The Repositories folder you can have yamls for the different repositories that you need for your use of different types where you can store artifacts and images. Each file follow the naming convention of `<repository-name>.yaml`

## Github actions

We will use Github actions to validate the configuration files and also apply them on every push. To set up the actions, we will need to create the following Github repository secrets:
- `TFY_HOST` - Your TrueFoundry host URL  
- `TFY_API_KEY` - The API key which you can generate from the TrueFoundry dashboard

There are primarily 2 Github actions in the .github/workflows folder:

1. **dry_run_on_pr.yaml** - This action is triggered on every pull request and subsequent changed to the PR. It validated the following things on every PR:

 - The filename matches the name field in the yaml file
 - It executes apply in dry-run mode for the changed file. 

2. **apply_on_merge.yaml** - This action is triggered on every merge to the main branch. It applied all the changed filed in the commit and deleted the configurations for the deleted files.
