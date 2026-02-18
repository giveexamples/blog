---
title: "Setting Up Azure CLI and Terraform AzureAD Provider for Entra ID Management"
date: 2026-02-18
draft: false
description: "A comprehensive guide to installing Azure CLI, authenticating with Azure, obtaining access tokens, and configuring Terraform to manage Microsoft Entra ID resources"
weight: 2
tags: ["azure", "azure-cli", "terraform", "azuread", "entra-id", "authentication", "infrastructure-as-code", "access-token", "ms-graph"]
comments: true
---

## Introduction

Managing Microsoft Entra ID (formerly Azure Active Directory) resources requires proper authentication and tooling. Whether you're automating app registrations, configuring conditional access policies, or managing users and groups, Azure CLI and Terraform provide powerful infrastructure-as-code capabilities.

In this article, you'll learn:
- How to install and configure Azure CLI
- How to authenticate with `az login --allow-no-subscriptions`
- How to obtain Microsoft Graph access tokens
- How to configure Terraform's AzureAD provider
- What happens behind the scenes during authentication

## Why Azure CLI and Terraform?

**Azure CLI** provides direct command-line access to Azure and Entra ID resources. It's essential for:
- Quick resource queries and configurations
- Obtaining access tokens for testing
- Scripting and automation tasks
- Troubleshooting and debugging

**Terraform** offers declarative infrastructure management with:
- Version-controlled infrastructure definitions
- Reproducible deployments across environments
- State management and drift detection
- Team collaboration through shared state

Together, they form a powerful toolkit for managing cloud identity infrastructure.

## Prerequisites

Before starting, ensure you have:
- An Azure account (even without subscriptions)
- Administrator access to an Entra ID tenant
- macOS, Linux, or Windows with WSL2
- Basic understanding of command-line operations

## Step 1: Installing Azure CLI

Azure CLI is a cross-platform command-line tool for managing Azure resources.

### macOS Installation

Using Homebrew (recommended):

```bash
# Install Azure CLI
brew update && brew install azure-cli

# Verify installation
az --version
```

### Linux Installation

Using the installation script:

```bash
# Download and run the installation script
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify installation
az --version
```

For specific distributions:

**Ubuntu/Debian:**
```bash
curl -sL https://packages.microsoft.com/keys/microsoft.asc | \
    gpg --dearmor | \
    sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
    sudo tee /etc/apt/sources.list.d/azure-cli.list

sudo apt-get update
sudo apt-get install azure-cli
```

**RHEL/CentOS/Fedora:**
```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

sudo dnf install -y https://packages.microsoft.com/config/rhel/9/packages-microsoft-prod.rpm
sudo dnf install azure-cli
```

### Windows Installation

Using Windows Package Manager:

```powershell
# Install using winget
winget install -e --id Microsoft.AzureCLI

# Verify installation
az --version
```

Or download the MSI installer from [Microsoft's official page](https://docs.microsoft.com/cli/azure/install-azure-cli-windows).

### Verify Installation

Check that Azure CLI is installed correctly:

```bash
az --version
```

**Expected output:**
``` text
azure-cli                         2.57.0

core                              2.57.0
telemetry                          1.1.0

Dependencies:
msal                              1.26.0
azure-mgmt-resource               23.1.0b2

Python location '/usr/local/Cellar/azure-cli/2.57.0/libexec/bin/python'
Extensions directory '/Users/username/.azure/cliextensions'

Python (Darwin) 3.11.7 (main, Dec  8 2023, 14:20:09) [Clang 15.0.0 (clang-1500.1.0.2.5)]
```

## Step 2: Authenticate with Azure CLI

Azure CLI supports multiple authentication methods. The most common is interactive browser-based login.

### Basic Login

Standard login for users with Azure subscriptions:

```bash
az login
```

This command:
1. Opens your default web browser
2. Redirects to Microsoft's login page
3. Prompts for credentials (username/password + MFA if enabled)
4. Returns authentication tokens to Azure CLI

### Login Without Subscriptions

If your account has access to Entra ID but no Azure subscriptions, use:

```bash
az login --allow-no-subscriptions
```

**Why use this flag?**
- Your account might have Entra ID admin roles but no subscription access
- You're managing identity resources (users, groups, app registrations) only
- You need to work with Microsoft Graph API without Azure resource management

**Example output:**
```json
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "12345678-1234-1234-1234-123456789abc",
    "id": "N/A(tenant level account)",
    "isDefault": true,
    "managedByTenants": [],
    "name": "N/A(tenant level account)",
    "state": "Enabled",
    "tenantId": "12345678-1234-1234-1234-123456789abc",
    "user": {
      "name": "user@example.com",
      "type": "user"
    }
  }
]
```

Notice that `id` shows `"N/A(tenant level account)"` instead of a subscription ID.

### Service Principal Authentication

For automation and CI/CD pipelines, use service principal authentication:

```bash
az login --service-principal \
  --username <app-id> \
  --password <password-or-cert> \
  --tenant <tenant-id>
```

### Managed Identity Authentication

When running on Azure resources (VMs, App Service, Container Instances):

```bash
az login --identity
```

## Step 3: Working with Authentication Context

### List Authenticated Accounts

View all logged-in accounts:

```bash
az account list
```

### Show Current Account

Display the currently active account:

```bash
az account show
```

**Example output:**
```json
{
  "environmentName": "AzureCloud",
  "homeTenantId": "12345678-1234-1234-1234-123456789abc",
  "id": "N/A(tenant level account)",
  "isDefault": true,
  "managedByTenants": [],
  "name": "N/A(tenant level account)",
  "state": "Enabled",
  "tenantId": "12345678-1234-1234-1234-123456789abc",
  "user": {
    "name": "user@example.com",
    "type": "user"
  }
}
```

### Switch Between Tenants

If you have access to multiple tenants:

```bash
az login --tenant <tenant-id> --allow-no-subscriptions
```

## Step 4: Obtaining Microsoft Graph Access Tokens

Access tokens are required to call Microsoft Graph API directly or test API integrations.

### Get Microsoft Graph Token

Obtain an access token for Microsoft Graph:

```bash
az account get-access-token --resource-type ms-graph
```

**Example output:**
```json
{
  "accessToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1...",
  "expiresOn": "2026-02-18 15:30:00.000000",
  "expires_on": 1708271400,
  "subscription": "N/A(tenant level account)",
  "tenant": "12345678-1234-1234-1234-123456789abc",
  "tokenType": "Bearer"
}
```

### Extract Token for Use

Extract just the token value:

```bash
# Save to variable
TOKEN=$(az account get-access-token --resource-type ms-graph --query accessToken -o tsv)

# Verify it's set
echo $TOKEN
```

### Use Token with Microsoft Graph API

Test the token by calling Microsoft Graph:

```bash
# Get current user profile
curl -H "Authorization: Bearer $TOKEN" \
  https://graph.microsoft.com/v1.0/me

# List all users (requires appropriate permissions)
curl -H "Authorization: Bearer $TOKEN" \
  https://graph.microsoft.com/v1.0/users

# Get application registrations
curl -H "Authorization: Bearer $TOKEN" \
  https://graph.microsoft.com/v1.0/applications
```

### Get Token for Custom Resources

For custom APIs or specific Azure resources:

```bash
# For a custom API
az account get-access-token --resource api://my-custom-api

# For Azure Resource Manager
az account get-access-token --resource https://management.azure.com/

# For Azure Key Vault
az account get-access-token --resource https://vault.azure.net
```

### Token Properties and Claims

Decode your token to understand its claims (use [jwt.io](https://jwt.io) or jwt CLI):

```bash
# Install jwt-cli (optional)
brew install mike-engel/jwt-cli/jwt-cli

# Decode token
echo $TOKEN | jwt decode -

# View specific claims
echo $TOKEN | jwt decode - | jq '.payload.scp'  # Scopes
echo $TOKEN | jwt decode - | jq '.payload.roles'  # Roles
echo $TOKEN | jwt decode - | jq '.payload.aud'  # Audience
```

**Typical Microsoft Graph token claims:**
```json
{
  "aud": "https://graph.microsoft.com",
  "iss": "https://sts.windows.net/12345678-1234-1234-1234-123456789abc/",
  "iat": 1708267800,
  "nbf": 1708267800,
  "exp": 1708271700,
  "scp": "User.Read profile openid email",
  "sub": "AAAAAAAAAAAAAAAAAAAAAAAAAg",
  "tid": "12345678-1234-1234-1234-123456789abc",
  "upn": "user@example.com"
}
```

## Step 5: Understanding Azure CLI Authentication Storage

When you run `az login`, Azure CLI stores authentication information locally.

### Authentication Storage Location

All authentication data is stored in `~/.azure/`:

```bash
ls -la ~/.azure/
```

**Directory structure:**
``` text
~/.azure/
├── accessTokens.json     # OAuth access tokens and refresh tokens
├── azureProfile.json     # Account and subscription information
├── clouds.config         # Cloud environment configurations
├── config                # CLI configuration settings
├── logs/                 # Operation logs
└── cliextensions/        # Installed CLI extensions
```

### Key Files Explained

#### 1. `accessTokens.json`

Stores OAuth tokens for authenticated sessions.

```bash
cat ~/.azure/accessTokens.json
```

**Structure:**
```json
[
  {
    "tokenType": "Bearer",
    "expiresOn": "2026-02-18T15:30:00.000000Z",
    "resource": "https://management.core.windows.net/",
    "accessToken": "eyJ0eXAiOiJKV1QiLC...",
    "refreshToken": "0.AXcA...",
    "expiresIn": 3599,
    "_clientId": "04b07795-8ddb-461a-bbee-02f9e1bf7b46",
    "_authority": "https://login.microsoftonline.com/12345678-1234-1234-1234-123456789abc",
    "userId": "user@example.com"
  },
  {
    "tokenType": "Bearer",
    "expiresOn": "2026-02-18T15:30:00.000000Z",
    "resource": "https://graph.microsoft.com",
    "accessToken": "eyJ0eXAiOiJKV1QiLC...",
    "refreshToken": "0.AXcA...",
    "expiresIn": 3599,
    "_clientId": "04b07795-8ddb-461a-bbee-02f9e1bf7b46",
    "_authority": "https://login.microsoftonline.com/12345678-1234-1234-1234-123456789abc",
    "userId": "user@example.com"
  }
]
```

**Important:**
- Contains both access tokens and refresh tokens
- Access tokens expire (typically 1 hour)
- Refresh tokens allow obtaining new access tokens without re-authentication
- **This file contains sensitive data** - protect it with appropriate file permissions

#### 2. `azureProfile.json`

Stores account and subscription metadata.

```bash
cat ~/.azure/azureProfile.json
```

**Structure:**
```json
{
  "installationId": "abc12345-6789-0def-1234-567890abcdef",
  "subscriptions": [
    {
      "id": "N/A(tenant level account)",
      "name": "N/A(tenant level account)",
      "state": "Enabled",
      "user": {
        "name": "user@example.com",
        "type": "user"
      },
      "isDefault": true,
      "tenantId": "12345678-1234-1234-1234-123456789abc",
      "environmentName": "AzureCloud",
      "homeTenantId": "12345678-1234-1234-1234-123456789abc",
      "managedByTenants": []
    }
  ]
}
```

**Contains:**
- Current account information
- Tenant ID
- Subscription details (or "N/A" for tenant-only access)
- Default subscription selection

#### 3. `config`

Stores CLI configuration preferences.

```bash
cat ~/.azure/config
```

**Example:**
```ini
[core]
output = json
collect_telemetry = false
only_show_errors = false

[cloud]
name = AzureCloud

[defaults]
location = eastus
group = my-resource-group
```

### Security Considerations

#### File Permissions

Check and secure your Azure directory:

```bash
# Check permissions
ls -la ~/.azure/

# Secure the directory (should be 700)
chmod 700 ~/.azure

# Secure token files (should be 600)
chmod 600 ~/.azure/accessTokens.json
chmod 600 ~/.azure/azureProfile.json
```

#### Token Refresh Mechanism

Azure CLI automatically refreshes expired access tokens using refresh tokens:

1. Access token expires (1 hour typically)
2. CLI detects expiration on next command
3. Uses refresh token to obtain new access token
4. Updates `accessTokens.json` automatically
5. Refresh token itself has longer lifetime (90 days or more)

#### Logout and Token Cleanup

When you logout, tokens are removed:

```bash
# Logout from all accounts
az logout

# Logout specific account
az logout --username user@example.com

# Verify tokens are cleared
cat ~/.azure/accessTokens.json
```

## Step 6: Configuring Terraform AzureAD Provider

Now that you're authenticated with Azure CLI, let's configure Terraform to manage Entra ID resources.

### Install Terraform

If you haven't installed Terraform:

**macOS:**
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

**Linux:**
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

**Verify installation:**
```bash
terraform --version
```

### Understanding AzureAD Provider Authentication

The Terraform AzureAD provider supports multiple authentication methods:

1. **Azure CLI** (Recommended for local development)
2. **Service Principal with Client Secret**
3. **Service Principal with Client Certificate**
4. **Managed Identity** (for Azure-hosted Terraform runs)

### Method 1: Azure CLI Authentication (Recommended)

This method uses your existing `az login` session.

**`main.tf`**
```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.47"
    }
  }
}

# The provider will automatically use Azure CLI authentication
provider "azuread" {
  # No explicit authentication configuration needed
  # Uses the currently logged-in Azure CLI account
}

# Get current client (the authenticated identity)
data "azuread_client_config" "current" {}

# Output the authenticated identity information
output "current_user" {
  value = {
    tenant_id   = data.azuread_client_config.current.tenant_id
    object_id   = data.azuread_client_config.current.object_id
    client_id   = data.azuread_client_config.current.client_id
  }
}
```

**Initialize and test:**
```bash
terraform init
terraform plan
```

### Method 2: Service Principal Authentication

For CI/CD pipelines and automation, use a service principal.

#### Step 1: Create Service Principal

```bash
# Create service principal
az ad sp create-for-rbac \
  --name "terraform-entra-automation" \
  --role "Directory.ReadWrite.All" \
  --scopes /providers/Microsoft.Graph
```

**Output:**
```json
{
  "appId": "12345678-abcd-1234-abcd-123456789abc",
  "displayName": "terraform-entra-automation",
  "password": "super-secret-password-here",
  "tenant": "12345678-1234-1234-1234-123456789abc"
}
```

#### Step 2: Grant API Permissions

The service principal needs Microsoft Graph permissions:

```bash
# Get the service principal object ID
SP_OBJECT_ID=$(az ad sp show --id <appId> --query id -o tsv)

# Grant Microsoft Graph permissions (requires admin consent)
az ad app permission add \
  --id <appId> \
  --api 00000003-0000-0000-c000-000000000000 \
  --api-permissions \
    19dbc75e-c2e2-444c-a770-ec69d8559fc7=Role \
    1bfefb4e-e0b5-418b-a88f-73c46d2cc8e9=Role

# Grant admin consent
az ad app permission admin-consent --id <appId>
```

#### Step 3: Configure Terraform

**`main.tf`**
```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.47"
    }
  }
}

provider "azuread" {
  client_id     = var.client_id
  client_secret = var.client_secret
  tenant_id     = var.tenant_id
}
```

**`variables.tf`**
```hcl
variable "client_id" {
  description = "Service Principal Application (Client) ID"
  type        = string
  sensitive   = true
}

variable "client_secret" {
  description = "Service Principal Client Secret"
  type        = string
  sensitive   = true
}

variable "tenant_id" {
  description = "Azure AD Tenant ID"
  type        = string
}
```

**`terraform.tfvars`** (add to `.gitignore`):
```hcl
client_id     = "12345678-abcd-1234-abcd-123456789abc"
client_secret = "super-secret-password-here"
tenant_id     = "12345678-1234-1234-1234-123456789abc"
```

**Better: Use environment variables:**
```bash
export TF_VAR_client_id="12345678-abcd-1234-abcd-123456789abc"
export TF_VAR_client_secret="super-secret-password-here"
export TF_VAR_tenant_id="12345678-1234-1234-1234-123456789abc"

terraform plan
```

### Method 3: Managed Identity (Azure-hosted)

When running Terraform on Azure resources:

**`main.tf`**
```hcl
provider "azuread" {
  use_msi   = true
  tenant_id = var.tenant_id
}
```

## Step 7: Creating Your First Entra ID Resource

Let's create a complete example that provisions an application registration.

**`main.tf`**
```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.47"
    }
  }
}

provider "azuread" {
  # Uses Azure CLI authentication automatically
}

# Get current configuration
data "azuread_client_config" "current" {}

# Create an application registration
resource "azuread_application" "example" {
  display_name = "My Terraform Managed App"

  # Configure identifier URIs
  identifier_uris = ["api://my-terraform-app"]

  # Configure sign-in audience
  sign_in_audience = "AzureADMyOrg"

  # Define required resource access (API permissions)
  required_resource_access {
    # Microsoft Graph
    resource_app_id = "00000003-0000-0000-c000-000000000000"

    # User.Read delegated permission
    resource_access {
      id   = "e1fe6dd8-ba31-4d61-89e7-88639da4683d"
      type = "Scope"
    }
  }

  # Configure web redirect URIs
  web {
    redirect_uris = ["https://localhost:8080/callback"]

    implicit_grant {
      access_token_issuance_enabled = true
      id_token_issuance_enabled     = true
    }
  }

  # Define API scopes
  api {
    oauth2_permission_scope {
      admin_consent_description  = "Allow the application to access resources"
      admin_consent_display_name = "Access resources"
      enabled                    = true
      id                         = "00000000-0000-0000-0000-000000000001"
      type                       = "User"
      user_consent_description   = "Allow the application to access resources on your behalf"
      user_consent_display_name  = "Access resources"
      value                      = "user_impersonation"
    }
  }
}

# Create service principal for the application
resource "azuread_service_principal" "example" {
  client_id = azuread_application.example.client_id

  app_role_assignment_required = false
}

# Create a client secret
resource "azuread_application_password" "example" {
  application_id = azuread_application.example.id
  display_name   = "Terraform Managed Secret"

  end_date = "2027-02-18T00:00:00Z"
}

# Outputs
output "application_id" {
  value       = azuread_application.example.client_id
  description = "Application (Client) ID"
}

output "tenant_id" {
  value       = data.azuread_client_config.current.tenant_id
  description = "Azure AD Tenant ID"
}

output "client_secret" {
  value       = azuread_application_password.example.value
  description = "Client secret value"
  sensitive   = true
}

output "identifier_uri" {
  value       = "api://my-terraform-app"
  description = "Application identifier URI"
}
```

### Apply the Configuration

```bash
# Initialize Terraform
terraform init

# Review the execution plan
terraform plan

# Apply the configuration
terraform apply

# View outputs
terraform output

# View sensitive outputs
terraform output -raw client_secret
```

### Verify in Azure Portal

After applying, verify in the Azure Portal:
1. Navigate to **Azure Portal** → **Microsoft Entra ID**
2. Go to **App registrations**
3. Find "My Terraform Managed App"
4. Review the configuration matches your Terraform code

## Step 8: Managing State and Collaboration

### Local State

By default, Terraform stores state in `terraform.tfstate`:

```bash
ls -la
# terraform.tfstate
# terraform.tfstate.backup
```

**Important:** Never commit state files to version control as they contain sensitive data.

### Remote State with Azure Storage

For team collaboration, use remote state:

**`backend.tf`**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "entra-id.tfstate"
  }
}
```

**Setup storage:**
```bash
# Create resource group
az group create --name terraform-state-rg --location eastus

# Create storage account
az storage account create \
  --name tfstateaccount \
  --resource-group terraform-state-rg \
  --location eastus \
  --sku Standard_LRS

# Create container
az storage container create \
  --name tfstate \
  --account-name tfstateaccount
```

## Common Issues and Troubleshooting

### Issue 1: "No subscriptions found"

**Error:**
```
ERROR: No subscriptions found for user@example.com.
```

**Solution:**
Use `--allow-no-subscriptions` flag:
```bash
az login --allow-no-subscriptions
```

### Issue 2: "Insufficient privileges"

**Error:**
```
(Authorization_RequestDenied) Insufficient privileges to complete the operation.
```

**Solution:**
Ensure your account has appropriate Entra ID roles:
```bash
# Check current role assignments
az ad signed-in-user show --query '{UPN:userPrincipalName, ObjectId:id}'

# Required roles for Terraform operations:
# - Application Administrator
# - Cloud Application Administrator
# - Global Administrator
```

### Issue 3: Token Expired

**Error:**
```
AADSTS70043: The refresh token has expired due to inactivity.
```

**Solution:**
Re-authenticate:
```bash
az logout
az login --allow-no-subscriptions
```

### Issue 4: Provider Configuration Not Found

**Error:**
```
Error: Failed to instantiate provider "azuread"
```

**Solution:**
Ensure you're logged in and provider is configured:
```bash
# Verify login
az account show

# Re-initialize Terraform
rm -rf .terraform .terraform.lock.hcl
terraform init
```

### Issue 5: "Use of undeclared resource"

**Error:**
```
Error: Reference to undeclared resource
```

**Solution:**
Ensure resource names match between resource blocks and references.

## Best Practices

### 1. Use Azure CLI for Local Development

Azure CLI authentication is perfect for development:
- No credentials in code
- Easy account switching
- Automatic token refresh

### 2. Use Service Principals for Production

For CI/CD and production:
- Create dedicated service principals
- Grant least-privilege permissions
- Rotate secrets regularly
- Use Azure Key Vault for secret storage

### 3. Organize Terraform Code

Structure your Terraform projects:

```
entra-terraform/
├── main.tf              # Provider and main resources
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── backend.tf           # Remote state configuration
├── terraform.tfvars     # Variable values (gitignored)
├── modules/             # Reusable modules
│   ├── app-registration/
│   ├── security-group/
│   └── conditional-access/
└── environments/        # Environment-specific configs
    ├── dev/
    ├── staging/
    └── production/
```

### 4. Use Variables for Reusability

Parameterize your configurations:

```hcl
variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

resource "azuread_application" "example" {
  display_name = "myapp-${var.environment}"
}
```

### 5. Implement State Locking

Prevent concurrent modifications:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "entra-id.tfstate"
    use_azuread_auth     = true
  }
}
```

### 6. Version Control Your Infrastructure

Always use Git:
```bash
# .gitignore
.terraform/
*.tfstate
*.tfstate.backup
terraform.tfvars
.terraform.lock.hcl
```

### 7. Document Your Resources

Add descriptions and comments:

```hcl
resource "azuread_application" "api" {
  display_name = "Production API"

  # This application serves as the backend API
  # for our customer-facing mobile application
  description = "Backend API for mobile app authentication"
}
```

## Security Best Practices

### Protect Credentials

1. **Never commit secrets:**
   - Use `.gitignore` for `terraform.tfvars`
   - Use environment variables
   - Use Azure Key Vault

2. **Secure local files:**
   ```bash
   chmod 600 ~/.azure/accessTokens.json
   chmod 600 terraform.tfvars
   ```

3. **Rotate secrets regularly:**
   ```bash
   # Rotate service principal secret
   az ad sp credential reset --id <app-id>
   ```

### Audit and Monitoring

Enable logging:

```bash
# Enable Azure CLI logging
az config set core.collect_telemetry=false
az config set core.only_show_errors=false

# View Azure CLI logs
tail -f ~/.azure/az.log
```

Monitor Terraform operations:
```bash
# Enable detailed logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform-debug.log

terraform apply
```

## Next Steps

You now have a complete setup for managing Entra ID resources with Azure CLI and Terraform. You can:
- Create application registrations
- Configure authentication flows
- Manage users and groups
- Implement conditional access policies
- Automate identity governance

For more context on identity concepts, check out:
- [Understanding OAuth 2.0 and OIDC]({{< ref "oauth2-oidc-saml" >}})

## Further Reading

- [Azure CLI Documentation](https://learn.microsoft.com/cli/azure/)
- [Terraform AzureAD Provider Documentation](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs)
- [Microsoft Graph API Reference](https://learn.microsoft.com/graph/api/overview)
- [Microsoft Entra ID Documentation](https://learn.microsoft.com/entra/identity/)

---

**Questions or improvements?** Leave a comment below or open an issue on the blog repository.
