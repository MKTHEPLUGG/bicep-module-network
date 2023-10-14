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


### SETUP APP REGISTRATION WITH FEDERATED IDENTITY FOR AZURE LOGIN

Certainly! Here's a documentation on how to create an app registration with a federated reference for GitHub to use an environment as `int` or `prod` to authenticate with the app registration and give the app registration proper rights in ACR to post images:

## App Registration with Federated Reference for GitHub

### 1. Azure AD Workload Identity Federation
- This GitHub action acquires access tokens (JWTs) for federated Azure AD workload identities that have configured GitHub as Open ID Connect (OIDC) credential provider.
- The access tokens can be used for any kind of API access or usage, like Microsoft Graph.
- The retrieved access token is stored as environment variable `ACCESS_TOKEN` within the GitHub Action workflow and can also be consumed from PowerShell.
- **Requirements**:
  - Azure AD Tenant ID
  - Azure AD App Registration Client ID
  - Add the `id-token` permissions to your GitHub action.
- In Azure AD, you need to:
  - Create an app registration.
  - Add the desired permissions and grant admin consent.
  - Add federated credentials.
  - Specify your GitHub repository information.
  - Copy your tenant and client id.
  
  [Source](https://github.com/marketplace/actions/azure-ad-workload-identity-federation)

### 2. Configure GitHub Authentication in Azure App Service
- Configure Azure App Service or Azure Functions to use GitHub as an authentication provider.
- **Steps**:
  - Register your application with GitHub.
  - Sign in to the Azure portal and go to your application.
  - Follow the instructions for creating an OAuth app on GitHub.
  - On the application page, note the Client ID.
  - Under Client Secrets, generate a new client secret.
  - Add GitHub information to your application in Azure portal.
  - Select Authentication in the menu on the left.
  - Click Add identity provider.
  - Select GitHub in the identity provider dropdown.
  - Paste in the Client ID and Client secret values.
  
  [Source](https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-github)

### 3. Configuring OpenID Connect in Azure
- OpenID Connect (OIDC) allows your GitHub Actions workflows to access resources in Azure without needing to store the Azure credentials as long-lived GitHub secrets.
- **Steps**:
  - Add the Federated Credentials to Azure.
  - Create an Azure Active Directory application and a service principal.
  - Add federated credentials for the Azure Active Directory application.
  - Create GitHub secrets for storing Azure configuration.
  - Update your GitHub Actions workflow.
  - Add permissions settings for the token.
  - Use the `azure/login` action to exchange the OIDC token (JWT) for a cloud access token.
  
  [Source](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)

### Workflow Example:
```yaml
steps:
  - name: Checkout Step
    uses: actions/checkout@v3
  - name: Azure Login
    uses: Azure/login@v1.4.6
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  - name: Azure CLI Action
    uses: Azure/cli@v1.0.8
    with:
      inlineScript: |
        az account show
        echo ${{ github.ref_name }}
        az bicep publish --force --file ./module/network.bicep --target br:eruza123.azurecr.io/bicep/modules/network:v${{ github.ref_name }}
```

This documentation provides a comprehensive guide on setting up federated authentication with GitHub in Azure and using it in your workflows. Ensure you follow the steps and guidelines provided in the sources for a smooth setup.



Yes, the documentation provided outlines the process of setting up an app registration with federated reference in Azure to integrate with GitHub Actions. Here's a more concise breakdown:

### Setting up App Registration with Federated Reference for GitHub Actions:

1. **Azure AD Workload Identity Federation**:
   - Use GitHub action to acquire access tokens for federated Azure AD workload identities that have GitHub as an OIDC provider.
   - Store the retrieved access token as an environment variable `ACCESS_TOKEN` within the GitHub Action workflow.
   - Requirements:
     - Azure AD Tenant ID
     - Azure AD App Registration Client ID
     - Add the `id-token` permissions to your GitHub action.
   - In Azure AD:
     - Create an app registration.
     - Add permissions and grant admin consent.
     - Add federated credentials.
     - Specify your GitHub repository information.

2. **Configure GitHub Authentication in Azure App Service**:
   - Use Azure App Service or Azure Functions with GitHub as an authentication provider.
   - Register your application with GitHub and note the Client ID.
   - Generate a new client secret under Client Secrets.
   - Add GitHub information to your application in the Azure portal.

3. **Configuring OpenID Connect in Azure**:
   - Use OIDC to allow GitHub Actions workflows to access Azure resources without storing Azure credentials as long-lived GitHub secrets.
   - Add Federated Credentials to Azure.
   - Create an Azure Active Directory application and a service principal.
   - Add federated credentials for the Azure AD application.
   - Create GitHub secrets for storing Azure configuration.
   - Update your GitHub Actions workflow to:
     - Add permissions settings for the token.
     - Use the `azure/login` action to exchange the OIDC token for a cloud access token.

### Workflow Example:
```yaml
steps:
  - name: Checkout Step
    uses: actions/checkout@v3
  - name: Azure Login
    uses: Azure/login@v1.4.6
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  - name: Azure CLI Action
    uses: Azure/cli@v1.0.8
    with:
      inlineScript: |
        az account show
        echo ${{ github.ref_name }}
        az bicep publish --force --file ./module/network.bicep --target br:eruza123.azurecr.io/bicep/modules/network:v${{ github.ref_name }}
```

This setup allows you to use GitHub Actions with Azure, leveraging federated authentication. You can set an environment on the jobs based on the GitHub reference name. Ensure you follow the steps and guidelines provided for a successful setup.