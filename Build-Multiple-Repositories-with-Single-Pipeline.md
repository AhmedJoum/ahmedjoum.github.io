
# Build Multiple Repositories with Single Pipeline

## Introduction

This section discuss how to build multiple Repositories with single pipeline. this could be needed when the feature you deliver may effect multiple code repositories and you need to make sure that all builds together.

## The `resources` section on the Pipeline

To tell the pipeline about the other repositories that need to be build, you need to add the `resources` section before starting `steps` or `jobs` sections. lets see the `resources` section structure:

```yml
resources:
  repositories:
  - repository: <RepositoryAliasName>
    type: git
    name: <AzureDevOps-ProjectName>/<RepositoryName>
    ref: <BranchName>
```

The `resources` section contain sub-section `repositories`, which contains one or multiple repository definition.

For each repository we need to define the following:

- `repository` the value will be an alias name for the repository, this name will be used on other pipeline step `checkout`.
- `type`: define where the repository is:
  - `git`: if the repository is a git repository in the same Azure DevOps server and on the same organization as the pipeline.
  - For more types check [this document](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/multi-repo-checkout?view=azure-devops#specify-multiple-repositories)
- `name`: it's a combination of project name on Azure DevOps and the repository name.
- `ref`: the branch that will check the code from.

## The `checkout` Section in the Pipeline

After defining the values of `resources.repositories`, you have to checkout the code, we will do that inside the `steps` section:

```yml
##
steps:
- checkout: <RepositoryAliasName>
##
```

The `checkout` key takes one value which is the alias name that we gave to the repository on `resources.repositories`.

### **Notes That:**

- you have to define and check even the repository that contains the pipeline.
- First time you run the pipeline after adding the repository and checkout, it will ask for give permition to access the banch.
  <img width="872" alt="pipeline-need-permission-first-checkout-branch" src="https://user-images.githubusercontent.com/11056447/185391346-aaddcf5a-0efc-4d48-9895-3a82d5d6233e.png">

  just click **View** and then **Permit**
  for more information about pipeline limit job Autherization check the following [link](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/access-tokens?view=azure-devops&tabs=yaml).

## Archive the build Results

After the `checkout` completed, the build task can start, the checkout step will download the source code in a folder with the same name of the repository name.
Therefore, the build will be done in that folder.

You can build multible repositories in one step if and only if both repositories are using the same framework.

After the build, you can copy or archive the build result to Artifact

```yml
- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '<RepositoryName>/<BuildingPath>'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)/<ProjectName>.$(Build.BuildNumber).zip'
    replaceExistingArchive: true

```

Here we done a step for archiving one of the projects, as you notice, Repository Name must be added as a part of the building result path.

Same can be for the copy task.
