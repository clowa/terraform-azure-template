# GitHub Actions

**DISCLAIMER:** The current pipeline uses AzureAD service principals for authentication. There is a newer OIDC based authentication method for GitHub Actions you might want to setup on your own. This pipeline currently doesn't implement this authentication.

## Choose your pipeline

There are two pipelines templates available:

`tpl_terraform_basic.yaml`:

A basic pipeline that doesn't enforce the changes commented to the PR, but is less complex and mostly suitable for small projects with less parallel changes.

`tpl_terraform_enforce_plan.yaml`:

A pipeline that enforces the changes commented to the PR by using `tfplan` artifacts from a previous GitHub Action run. This pipeline is more complex to setup because it requires a Repo Scoped Personal Access Token (PAT) but suitable for bigger projects with parallel changes.

### The basic Pipeline

Features:

- Run Code validation
- Allow approval enforcement via Pull Requests
- Comment terraform plan changes to the PR

Limitations:

- Does not enforce the commented changes on merge. To work around this you could merge `main` into your branch before merging.

Required GitHub Secrets:

- `ARM_CLIENT_ID`: The client id of the service principal used to authenticate against Azure.
- `ARM_CLIENT_SECRET`: The client secret of the service principal used to authenticate against Azure.
- `ARM_TENANT_ID`: The tenant id of the service principal used to authenticate against Azure.
- `ARM_SUBSCRIPTION_ID`: The subscription id to deploy the infrastructure to.

### The advanced Pipeline

Features:

- Same as the basic pipeline _and ..._
- Enforce commented changes on merge

Limitations:

- Requires a Repo Scoped Personal Access Token (PAT) with `repo` scope. This token is used to access the `tfplan` artifact of a previous GitHub Action run of the related PR. The token is stored as a GitHub secret and can be used by anyone with write access to the repository. Therefore it is recommended to create a new GitHub account with only write access to the repository and use this account to create the PAT. The PAT is only used to access the `tfplan` artifact of the related PR and nothing else.

Required GitHub Secrets:

- `ARM_CLIENT_ID`: The client id of the service principal used to authenticate against Azure.
- `ARM_CLIENT_SECRET`: The client secret of the service principal used to authenticate against Azure.
- `ARM_TENANT_ID`: The tenant id of the service principal used to authenticate against Azure.
- `ARM_SUBSCRIPTION_ID`: The subscription id to deploy the infrastructure to.
- `REPO_SCOPED_TOKEN`: The repo scoped personal access token (PAT) used to access the `tfplan` artifact of the related PR.

## Pipeline setup

1. Create a azure app registration mostly you will do this via [Azure Portal](https://portal.azure.com/#home), but you can also use [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac).
2. If you use Azure Portal add a secret to the app registration.
3. Write down `Application (client) ID`, the value of the generated secret/password, `Directory (tenant) ID` and `Subscription ID`.
4. Create the [GitHub Action Secrets](https://docs.github.com/de/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) required by your pipeline. Secrets that are the same for all environments can be created as repository secrets. Secrets that are different for each environment should be created as environment secrets.

### Add another stage

1. [Create a new GitHub environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment) in the repository settings.
2. Ether [setup a new pipeline](#pipeline-setup) or reuse the service principal of the existing stages and add the required GitHub Action Secrets for the new stage.
3. Copy the file `terraform-dev.yaml.example` and modify the lines marked with `#! CHANGE ME` to match your environment.

## Nice to know

### Run code validation locally

Terraform allows you to run code validation locally even without permission to access remote resources. You can do so by running the following commands.

1. Initialize terraform without the backend:

   ```bash
   terraform init -backend=false
   ```

2. Run code formatter:

   ```bash
   terraform fmt -recursive
   ```

3. Run code validation:

   ```bash
   terraform validate
   ```
