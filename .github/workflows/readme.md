## Documentation for `Publish Module` Workflow

### Overview:
The "Publish Module" workflow is designed to automate the process of publishing a Bicep module to Azure Container Registry when there's a push to tags in the repository. This is particularly useful for automating the deployment of infrastructure as code (IaC) using Bicep in the Azure ecosystem.

### Key Components:

#### 1. Permissions:
   - `id-token`: This permission allows the workflow to write ID tokens.
   - `contents`: This permission allows the workflow to read the contents of the repository.

#### 2. Triggers:
   - `on.push.tags`: This means the workflow is triggered on any push that includes a tag (`*` denotes any tag).
   - `workflow_dispatch`: Allows manual triggering of the workflow from the GitHub Actions tab.

#### 3. Jobs:

**a. `build` Job:**

   - **Environment**: Production (`prd`)
   - **Runner**: The job runs on the latest version of Ubuntu.
   - **Condition**: This job runs only if the push is made to a tag.
   
**Steps within the `build` Job:**
   
   1. **Checkout Step**: Checks out the code from the repository using the `actions/checkout@v3` action.
   
   2. **Azure Login**: Logs into Azure using the provided `client-id`, `tenant-id`, and `subscription-id` which are fetched from the repository's secrets.
   
   3. **Azure CLI Action**: Executes a set of Azure CLI commands.
      - `az account show`: Displays details about the currently logged-in Azure account.
      - `echo ${{ github.ref_name }}`: Prints the tag name that triggered this workflow.
      - `az bicep publish`: Publishes the `network.bicep` file to the specified Azure Container Registry with the version tagged as the triggering tag name.

### Notes:
- Make sure to set the necessary secrets (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`) in your repository's secrets section for the Azure login step to work.
- The workflow differentiates its permissions approach from that of Azure DevOps, providing a more granular and inline control.

By integrating this workflow, you can ensure that your Bicep modules are automatically versioned and published to Azure Container Registry with every new tag, thus streamlining your infrastructure deployment processes.


## Documentation for `GitVersion Tag` Workflow

### Overview:
The "GitVersion Tag" workflow is designed to automatically tag the repository based on GitVersion when there's a push to the main branch. This helps in automating the versioning of your codebase using GitVersion.

### Key Components:

#### 1. Permissions:
   - `id-token`: This permission allows the workflow to write ID tokens.
   - `contents`: Permission to write the contents of the repository. This is crucial for a workflow that is tagging the repo.

#### 2. Triggers:
   - `on.push.branches`: This workflow is triggered when there's a push to the `main` branch.
   - `workflow_dispatch`: Provides the ability to manually trigger the workflow from the GitHub Actions tab.

#### 3. Jobs:

**a. `build` Job:**

   - **Runner**: The job runs on the latest version of Ubuntu.

**Steps within the `build` Job:**
   
   1. **Checkout Step**: Checks out the code from the repository using the `actions/checkout@v3` action. The `fetch-depth: 0` option ensures a complete history is fetched, which can be essential for tools that rely on the git history, like GitVersion.
   
   2. **Tag the repo**: This step uses a custom GitHub action located in the `.github/actions/gvtag` directory of the repository. This action is responsible for tagging the repository based on GitVersion.

### Notes:
- The section for `pull_request` is commented out, which means this workflow will not be triggered on pull requests to the `main` branch unless uncommented.
- The permissions approach provides a more granular and inline control compared to Azure DevOps.
- Ensure that the custom action (`gvtag`) in `.github/actions/gvtag` is correctly set up to utilize GitVersion for tagging.

Incorporating this workflow into your repository can help automate the versioning process, making sure that every push to the main branch is appropriately tagged based on GitVersion conventions.