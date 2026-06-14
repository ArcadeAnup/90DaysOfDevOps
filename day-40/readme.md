# Day 40: Terraform Hands-On with Azure

## The Setup

### Install Terraform
```bash
# macOS
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

### Verify Installation
```bash
terraform version
# Terraform v1.5.0
```

## Create Azure Provider Configuration

### providers.tf

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

This tells Terraform: "I want to use Azure (azurerm provider)"

## Create Azure Resources

### main.tf

```hcl
# Variables
variable "resource_group_name" {
  description = "Name of the Azure resource group"
  type        = string
  default     = "my-resource-group"
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "storage_account_name" {
  description = "Name of the storage account"
  type        = string
  default     = "mystorageaccount"
}

# Create Resource Group
resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = "Dev"
    Project     = "90DaysOfDevOps"
  }
}

# Create Storage Account
resource "azurerm_storage_account" "storage" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  tags = {
    Environment = "Dev"
  }
}
```

### outputs.tf

```hcl
output "resource_group_id" {
  description = "ID of the created resource group"
  value       = azurerm_resource_group.rg.id
}

output "storage_account_id" {
  description = "ID of the created storage account"
  value       = azurerm_storage_account.storage.id
}

output "storage_account_name" {
  description = "Name of the created storage account"
  value       = azurerm_storage_account.storage.name
}
```

## Terraform Workflow

### Step 1: Initialize

```bash
terraform init
```

**Output:**
Initializing the backend...

Initializing provider plugins...

Reusing previous version of hashicorp/azurerm from the dependency lock file
Using previously-installed hashicorp/azurerm v3.50.0

Terraform has been successfully configured!

What happened:
- Downloaded Azure provider
- Created `.terraform/` directory
- Created `terraform.lock.hcl` (lock file for reproducibility)
- Ready to use Azure

### Step 2: Validate

```bash
terraform validate
```

**Output:**
Success! The configuration is valid.

Checks syntax and logic (doesn't talk to Azure yet).

### Step 3: Plan

```bash
terraform plan
```

**Output:**
Terraform will perform the following actions:
azurerm_resource_group.rg will be created

resource "azurerm_resource_group" "rg" {

id       = (known after apply)
location = "East US"
name     = "my-resource-group"
tags     = {

"Environment" = "Dev"
"Project"     = "90DaysOfDevOps"

}

}





azurerm_storage_account.storage will be created

resource "azurerm_storage_account" "storage" {

access_tier              = "Hot"
account_replication_type = "LRS"
account_tier             = "Standard"
id                       = (known after apply)
location                 = "East US"
name                     = "mystorageaccount"
resource_group_name      = "my-resource-group"

}



Plan: 2 to add, 0 to change, 0 to destroy.
Changes to Outputs:

resource_group_id     = (known after apply)
storage_account_id    = (known after apply)
storage_account_name  = "mystorageaccount"


**Key observations:**
- Shows exactly what will be created
- `+` means create
- `(known after apply)` means Azure will assign it
- Plan: 2 resources to create

### Step 4: Apply

```bash
terraform apply
```

**Terraform asks for confirmation:**
Do you want to perform these actions?

Terraform will perform the actions described above.

Only 'yes' will be accepted to approve.
Enter a value:

Type `yes` to proceed.

**Output:**
azurerm_resource_group.rg: Creating...

azurerm_resource_group.rg: Creation complete after 2s [id=/subscriptions/xxxx/resourceGroups/my-resource-group]

azurerm_storage_account.storage: Creating...

azurerm_storage_account.storage: Creation complete after 10s [id=/subscriptions/xxxx/storageAccounts/mystorageaccount]
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
Outputs:
resource_group_id = "/subscriptions/xxxx/resourceGroups/my-resource-group"

storage_account_id = "/subscriptions/xxxx/storageAccounts/mystorageaccount"

storage_account_name = "mystorageaccount"

**What happened:**
- Resource Group created
- Storage Account created in that RG
- Outputs displayed
- State file created (`terraform.tfstate`)

## Verify in Azure Portal

1. Go to Azure Portal
2. Search "Resource Groups"
3. See "my-resource-group" (created by Terraform)
4. Click it
5. See "mystorageaccount" Storage Account inside

Real Azure infrastructure. Created by code.

## State File

Terraform created `terraform.tfstate`:

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "serial": 2,
  "lineage": "abc123",
  "outputs": {
    "resource_group_id": {
      "value": "/subscriptions/xxxx/resourceGroups/my-resource-group",
      "type": "string"
    },
    "storage_account_name": {
      "value": "mystorageaccount",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "azurerm_resource_group",
      "name": "rg",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "/subscriptions/xxxx/resourceGroups/my-resource-group",
            "name": "my-resource-group",
            "location": "East US"
          }
        }
      ]
    }
  ]
}
```

State maps Terraform code to Azure resources.

## Desired State vs Actual State

### Initial State
- Desired: Resource Group + Storage Account
- Actual: (nothing exists)
- Terraform: Creates both

### Run terraform apply again
```bash
terraform apply
```

**Output:**
No changes. Your infrastructure matches the configuration.

- Desired: Resource Group + Storage Account
- Actual: Resource Group + Storage Account
- Terraform: Nothing to do

### Change Configuration

Edit `main.tf`:
```hcl
variable "location" {
  default     = "West US"  # Changed from East US
}
```

Run `terraform plan`:
~ resource "azurerm_resource_group" "rg" {

~ location = "East US" -> "West US"

name     = "my-resource-group"

}
Plan: 0 to add, 1 to change, 0 to destroy.

Shows what will change. Run `terraform apply` to update.

## Destroy Infrastructure

Clean up when done:

```bash
terraform destroy
```

**Asks for confirmation:**
Do you really want to destroy all resources?

Terraform will destroy all your managed infrastructure.
Enter a value:

Type `yes`.

**Output:**
azurerm_resource_group.rg: Destroying... [id=/subscriptions/xxxx/resourceGroups/my-resource-group]

azurerm_resource_group.rg: Destruction complete after 3s
Destroy complete! Resources: 2 destroyed.

Resources deleted from Azure. State file cleared.

## Full Workflow Recap

terraform init

↓

Download provider, create .terraform/
terraform plan

↓

Preview: "Will create Resource Group and Storage Account"
terraform apply

↓

Execute: Create both resources in Azure
Verify

↓

Check Azure Portal: Resources exist
terraform destroy

↓

Delete: Clean up resources


## Key Insights from Day 40

1. **Code controls infrastructure** - No Azure Portal clicking
2. **Preview before applying** - `terraform plan` shows exactly what will happen
3. **Idempotent** - Running `terraform apply` twice = safe, only changes needed
4. **State management** - Terraform tracks what exists
5. **Reproducible** - Same code = same infrastructure every time
6. **Clean destruction** - `terraform destroy` removes everything properly

## Directory Structure
project/

├── main.tf           # Resources

├── variables.tf      # Input variables

├── outputs.tf        # Outputs

├── terraform.tfstate # State (don't commit to Git)

└── .terraform/       # Provider plugins (auto-generated)

## Important Notes

### Don't Commit State File
```bash
# .gitignore
terraform.tfstate
terraform.tfstate.backup
.terraform/
```

State contains sensitive data. In production, store in S3/Azure Blob Storage with locking.

### Authenticate with Azure

```bash
# Login to Azure
az login

# Terraform will use Azure CLI credentials
```

Or use service principal for CI/CD.

## Next Steps with Terraform

1. **Multiple environments** - dev/staging/prod with different variables
2. **Modules** - Reusable infrastructure components
3. **Remote state** - Team collaboration
4. **CI/CD integration** - Automated infrastructure changes
5. **Drift detection** - Alert when actual state differs from code

## Realization

Day 39 = Terraform theory (what it is)
Day 40 = Terraform practice (how to use it)

Now infrastructure is code. Version controlled. Reproducible. Auditable. This is production DevOps.

---

**Progress: 40/90 days complete**

