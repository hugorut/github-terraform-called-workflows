# Reusable Called Workflows for Terraform

Reusing workflows avoids duplication. This makes workflows easier to maintain and allows you to create new workflows
more quickly by building on the work of others, just as you do with actions.

Workflow reuse also promotes best practice by helping you to use workflows that are well designed, have already been
tested, and have been proved to be effective. Your organization can build up a library of reusable workflows that can
be centrally maintained.

## Reusing Workflows

Rather than copying and pasting from one workflow to another, you can make workflows [reusable](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows). You and anyone with access to the reusable workflow can then call the reusable workflow from another workflow.

### Azure Usage

```yaml
name: Development

on:
  push:
    branches:
      - main

jobs:
  global_infra:
    name: Global
    uses: lzysh/github-terraform-called-workflows/.github/workflows/azure-plan-and-apply-called.yml@v1.0.0
    with:
      arm_subscription_id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
      checkout_ref: ${{ github.ref }}
      github_env: "Development Infrastructure: Global"
      terraform_version: 1.1.7
      tf_plan_args: -var-file=tfvars/dev.tfvars
      tf_state_storage_account: tfprojectpreprod
      tf_workspace: global-infra-development
      working_dir: global
    secrets:
      arm_client_id: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_CLIENT_ID }}
      arm_client_secret: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_CLIENT_SECRET }}
      gpg_passphrase: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      tf_state_access_key: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_STORAGE_KEY }}

  eastus_infra:
    name:  "Infra: eastus"
    uses: lzysh/github-terraform-called-workflows/.github/workflows/azure-plan-and-apply-called.yml@v1.0.0
    with:
      arm_subscription_id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
      checkout_ref: ${{ github.ref }}
      github_env: "Development Infrastructure: Regional - eastus"
      terraform_version: 1.1.7
      tf_plan_args: -var-file=tfvars/eastus-dev.tfvars
      tf_state_storage_account: tfprojectpreprod
      tf_workspace: eastus-infra-development
      working_dir: regional
    secrets:
      arm_client_id: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_CLIENT_ID }}
      arm_client_secret: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_CLIENT_SECRET }}
      gpg_passphrase: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      tf_state_access_key: ${{ secrets.AZURE_APPLICATION_PRE_PROD_GITHUB_STORAGE_KEY }}
```

### AWS Usage

```yaml
name: Development

on:
  push:
    branches:
      - main

jobs:
  global_infra:
    name: Global
    uses: lzysh/github-terraform-called-workflows/.github/workflows/aws-plan-and-apply-called.yml@v1.0.0
    with:
      checkout_ref: ${{ github.ref }}
      github_env: "Development Infrastructure: Global"
      region: us-east-1
      terraform_version: 1.1.7
      tf_dynamodb_table: github-actions-terraform-state-db
      tf_plan_args: -var-file=tfvars/dev.tfvars
      tf_state_bucket: github-actions-terraform-state-1
      tf_state_bucket_region: us-east-1
      tf_workspace: global-infra-development
      working_dir: global
    secrets:
      gpg_passphrase: ${{ secrets.GPG_PASSPHRASE }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      access_key_id: secrets.AWS_APLICATION_PRE_PROD_GITHUB_ACCESS_KEY_ID
      secret_access_key: secrets.AWS_APLICATION_PRE_PROD_GITHUB_SECRET_ACCESS_KEY

  us_east_1_infra:
    name:  "Infra: us-east-1"
    uses: lzysh/github-terraform-called-workflows/.github/workflows/aws-plan-and-apply-called.yml@v1.0.0
    with:
      checkout_ref: ${{ github.ref }}
      github_env: "Development Infrastructure: Regional - us-east-1"
      region: us-east-1
      terraform_version: 1.1.7
      tf_dynamodb_table: github-actions-terraform-state-db
      tf_plan_args: -var-file=tfvars/us-east-1-dev.tfvars
      tf_state_bucket: github-actions-terraform-state-1
      tf_state_bucket_region: us-east-1
      tf_workspace: us-east-1-infra-development
      working_dir: regional
    secrets:
      gpg_passphrase: ${{ secrets.GPG_PASSPHRASE }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      access_key_id: secrets.AWS_APLICATION_PRE_PROD_GITHUB_ACCESS_KEY_ID
      secret_access_key: secrets.AWS_APLICATION_PRE_PROD_GITHUB_SECRET_ACCESS_KEY
```

### GCP Usage

```yaml
name: Acceptance

on:
  push:
    branches:
      - master

jobs:
  global_infra:
    name: "Global"
    uses: lzysh/github-terraform-called-workflows/.github/workflows/gcp-plan-and-apply-called.yml@v1.0.0
    with:
      checkout_ref: ${{ github.ref }}
      github_env: "Acceptance Infrastructure: Global"
      terraform_version: 1.1.7
      tf_plan_args: -var-file=tfvars/acc.tfvars
      tf_state_bucket: tools-prod-pre-prod_tf_state
      tf_workspace: services-global-infra-acc
      working_dir: global
    secrets:
      gpg_passphrase: ${{ secrets.GPG_PASSPHRASE }}
      service_account_key: ${{ secrets.GCP_UTILS_PRE_PROD_GITHUB_SA_KEY }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
      tf_plan_secret_args: -var="bridgecrew_token=${{ secrets.BRIDGECREW_TOKEN }}"

  europe_west1_infra:
    name: "Infra: europe-west1"
    uses: lzysh/github-terraform-called-workflows/.github/workflows/gcp-plan-and-apply-called.yml@v1.0.0
    needs: global_infra
    with:
      checkout_ref: ${{ github.ref }}
      github_env: "Acceptance Infrastructure: Regional - europe-west1"
      terraform_version: 1.1.7
      tf_plan_args: -var-file=tfvars/europe-west1-acc.tfvars
      tf_state_bucket: tools-prod-pre-prod_tf_state
      tf_workspace: services-europe-west1-infra-acc
      working_dir: regional
    secrets:
      gpg_passphrase: ${{ secrets.GPG_PASSPHRASE }}
      service_account_key: ${{ secrets.GCP_UTILS_PRE_PROD_GITHUB_SA_KEY }}
      ssh_key: ${{ secrets.LZYSH_RO_SSH_PRIV_KEY }}
```
