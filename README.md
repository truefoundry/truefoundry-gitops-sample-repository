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
```

We have a top level folder for each cluster. Inside the cluster folder, we have the cluster configuration file and a folder for workspaces. The workspaces folder contains the workspace configuration files and the applications folder which contains the application configuration files.

## File Naming Convention

Each file in the clusters folder should follow the naming convention of `<cluster-name>.yaml`. The cluster name should be the same as the folder name.

The workspaces folder should contain a folder for each workspace. The folder name should be the same as the workspace name. The workspace configuration file should follow the naming convention of `<workspace-name>.yaml`.

The applications folder contains the spec of all the applications. Each application should be in a separate file and follow the naming convention of `<application-name>.yaml`.

## Github actions

We will use Github actions to validate the configuration files and also apply them on every push. To set up the actions, we will need to create a Github repository secret with the name `TFY_API_KEY` and the value of the API key which you can generate from the Truefoundry dashboard.  

Before running the actions, please 

There are primarily 2 Github actions in the .github/workflows folder:

1. **dry_run_on_pr.yaml** - This action is triggered on every pull request and subsequent changed to the PR. It validated the following things on every PR:

 - The filename matches the name field in the yaml file
 - It executes apply in dry-run mode for the changed file. 

2. **apply_on_merge.yaml** - This action is triggered on every merge to the main branch. It applied all the changed filed in the commit and deleted the configurations for the deleted files.
