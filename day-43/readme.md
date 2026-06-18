# Day 43: Resource Dependencies, Outputs, and State in Terraform

## The Problem Terraform Solves

### Without Terraform (Manual)
Create Resource Group

↓ (must wait for completion)

Create Storage Account in that RG

↓ (must get RG ID manually)

Create Blob Container in Storage Account

↓ (must get Storage Account endpoint manually)

Configure application to use endpoint

Error-prone, manual coordination, painful.

### With Terraform (Declarative)
Declare all resources

↓

Terraform figures out order

↓

Terraform passes values between resources

↓

All created in correct order, automatically

This is what dependencies and outputs do.

## Resource Dependencies

### Implicit Dependencies

When one resource references another, Terraform automatically creates the dependency.

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}

resource "azurerm_storage_account" "storage" {
  name                     = "mystorageaccount"
  resource_group_name      = azurerm_resource_group.rg.name  # <- Implicit dependency
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

**What Terraform sees:**
- Storage Account references `azurerm_resource_group.rg.name`
- Therefore, RG must be created first
- Terraform automatically creates RG, waits for it, then creates Storage Account

### Explicit Dependencies

Sometimes implicit dependencies don't work. Use `depends_on`:

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "my-vnet"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_virtual_machine" "vm" {
  name                = "my-vm"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  depends_on = [
    azurerm_virtual_network.vnet
  ]
}

# Could also do:
# depends_on = [azurerm_virtual_network.vnet, azurerm_storage_account.storage]
```

VM explicitly waits for VNet, even though no direct reference.

### Dependency Graph

Terraform builds a dependency graph:
Resource Group

↓

Storage Account (depends on RG)

↓

Blob Container (depends on Storage Account)

↓

Virtual Network (depends on RG)

↓

Virtual Machine (depends on VNet)

When you run `terraform apply`:
- Creates RG first
- Creates Storage Account and VNet in parallel (no dependency between them)
- Creates Blob Container (waits for Storage Account)
- Creates VM (waits for VNet)

Parallel creation where possible, sequential where needed.

## Output Variables

### Purpose of Outputs

Outputs are:
1. **Display information** after `terraform apply`
2. **Pass values** to other resources
3. **Pass values** to other Terraform modules
4. **Export values** for use outside Terraform (scripts, other tools)

### Basic Output

```hcl
output "resource_group_id" {
  description = "ID of the resource group"
  value       = azurerm_resource_group.rg.id
}

output "storage_account_name" {
  description = "Name of the storage account"
  value       = azurerm_storage_account.storage.name
}
```

After `terraform apply`:
Outputs:
resource_group_id = "/subscriptions/xxxx/resourceGroups/my-rg"

storage_account_name = "mystorageaccount"

### Outputs with Sensitive Data

```hcl
output "database_password" {
  description = "Database password"
  value       = azurerm_database.db.admin_password
  sensitive   = true
}
```

`sensitive = true` prevents password from being displayed in logs/output.
Outputs:
database_password = <sensitive>

To see it:
```bash
terraform output database_password
```

### Using Outputs in Resources

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}

resource "azurerm_storage_account" "storage" {
  name                = "mystorageaccount"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  account_tier        = "Standard"
  account_replication_type = "LRS"
}

output "storage_connection_string" {
  description = "Connection string to storage account"
  value       = azurerm_storage_account.storage.primary_connection_string
  sensitive   = true
}

output "resource_group_name" {
  description = "Name of resource group"
  value       = azurerm_resource_group.rg.name
}
```

Application can read these outputs and use them.

### Accessing Outputs from Other Modules

**module/storage/main.tf:**
```hcl
resource "azurerm_storage_account" "storage" {
  name = "mystorageaccount"
  # ...
}

output "storage_account_id" {
  value = azurerm_storage_account.storage.id
}
```

**main.tf:**
```hcl
module "storage" {
  source = "./modules/storage"
}

resource "azurerm_role_assignment" "example" {
  scope              = module.storage.storage_account_id
  role_definition_name = "Storage Blob Data Contributor"
  principal_id       = azurerm_user_assigned_identity.example.principal_id
}
```

Module outputs pass values to other resources.

### Viewing Outputs

```bash
# Show all outputs
terraform output

# Show specific output
terraform output resource_group_id

# JSON format (for scripting)
terraform output -json
# Output:
# {
#   "resource_group_id": {
#     "sensitive": false,
#     "type": "string",
#     "value": "/subscriptions/xxxx/resourceGroups/my-rg"
#   }
# }
```

## The State File

### What is State?

State is a JSON file that Terraform maintains tracking:
1. What infrastructure exists
2. Resource IDs from cloud provider
3. Resource configurations
4. Metadata

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "serial": 42,
  "lineage": "abc-123",
  "outputs": {
    "resource_group_id": {
      "value": "/subscriptions/xxxx/resourceGroups/my-rg",
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
            "id": "/subscriptions/xxxx/resourceGroups/my-rg",
            "name": "my-rg",
            "location": "East US",
            "tags": {
              "Environment": "Dev"
            }
          }
        }
      ]
    },
    {
      "mode": "managed",
      "type": "azurerm_storage_account",
      "name": "storage",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "/subscriptions/xxxx/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "account_tier": "Standard"
          }
        }
      ]
    }
  ]
}
```

State maps:
- Terraform code (`azurerm_resource_group.rg`)
- To Azure IDs (`/subscriptions/xxxx/resourceGroups/my-rg`)

### Why State Matters

#### Scenario 1: Terraform Doesn't Know What Exists

Imagine you delete the state file but infrastructure still exists in Azure:

```bash
rm terraform.tfstate
terraform apply
```

Terraform sees empty state (nothing recorded), thinks it needs to create everything.

Result: Tries to create duplicate resources → Fails (names already taken).

#### Scenario 2: State is Source of Truth

```bash
# Infrastructure exists in Azure
# State file says it exists

terraform apply
# Terraform: "State matches code, nothing to do"

# Someone manually deletes Resource Group in Portal
# But state still says it exists

terraform plan
# Terraform: "Sees state says RG exists, but can't find it in Azure"
# Will try to recreate it on next apply
```

State is the **source of truth**, not Azure Portal.

### State vs Code
Code says: "Resource Group with name 'my-rg'"

State says: "Created resource with ID /subscriptions/xxxx/resourceGroups/my-rg"

Azure has: Resource Group with ID /subscriptions/xxxx/resourceGroups/my-rg
All three match → healthy

If they diverge, Terraform gets confused.

### Local vs Remote State

#### Local State (Development)
```bash
# State stored in terraform.tfstate (local file)
# Good for: Solo development, learning
# Bad for: Teams (state file not shared), production (state not backed up)
```

#### Remote State (Production)

**Using Azure Blob Storage:**

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state"
    storage_account_name = "tfstate"
    container_name       = "state"
    key                  = "prod.terraform.tfstate"
  }
}
```

- State stored in Azure Blob Storage
- Team members all use same state
- State backed up
- Locking prevents concurrent modifications

### State Commands

```bash
# View state
terraform state list
# aws_instance.web
# aws_security_group.web

# Show specific resource state
terraform state show azurerm_resource_group.rg
# id = "/subscriptions/xxxx/resourceGroups/my-rg"
# name = "my-rg"
# location = "East US"

# Move resource (rename in state)
terraform state mv azurerm_resource_group.rg azurerm_resource_group.production_rg

# Remove resource from state (Terraform forgets about it)
terraform state rm azurerm_resource_group.rg

# Refresh state (sync with actual infrastructure)
terraform refresh
```

### State is Sensitive Data

State contains:
- Resource IDs
- Passwords (from secrets)
- Database connection strings
- API keys
- Everything needed to access infrastructure

**Never commit to Git:**
```bash
# .gitignore
terraform.tfstate
terraform.tfstate.backup
```

**In production:** Store state in encrypted S3/Blob Storage with access controls.

## How Terraform Actually Works

### Step 1: Read Code
Parse .tf files

↓

Build desired state (what resources should exist)

### Step 2: Read State
Load terraform.tfstate

↓

Know what currently exists

### Step 3: Fetch Current State
Query Azure API

↓

See what actually exists in Azure

### Step 4: Compare
Desired State (code): Resource Group "my-rg"

Current State (state file): Resource Group "my-rg" exists (ID: xxx)

Actual State (Azure): Resource Group "my-rg" exists
Result: No changes needed

### Step 5: Determine Actions
Desired != Current/Actual?

↓

If missing: Create

If extra: Destroy

If different: Update

### Step 6: Plan
Show user what will happen

### Step 7: Apply
Execute actions

↓

Update state file with new IDs/values

## Complete Example: Dependencies + Outputs

```hcl
# Resource Group (no dependencies)
resource "azurerm_resource_group" "rg" {
  name     = "myapp-rg"
  location = "East US"
}

# Storage Account (depends on RG)
resource "azurerm_storage_account" "storage" {
  name                     = "myappstorage"
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

# Blob Container (depends on Storage Account)
resource "azurerm_storage_container" "container" {
  name                  = "myappdata"
  storage_account_name  = azurerm_storage_account.storage.name
  container_access_type = "private"
}

# Outputs
output "resource_group_id" {
  description = "Resource Group ID"
  value       = azurerm_resource_group.rg.id
}

output "storage_account_id" {
  description = "Storage Account ID"
  value       = azurerm_storage_account.storage.id
}

output "storage_connection_string" {
  description = "Connection string for Storage Account"
  value       = azurerm_storage_account.storage.primary_connection_string
  sensitive   = true
}

output "container_name" {
  description = "Name of the blob container"
  value       = azurerm_storage_container.container.name
}
```

Terraform workflow:
1. `terraform init` - Download provider
2. `terraform plan` - Shows: Create RG → Create Storage → Create Container
3. `terraform apply` - Executes in order, passes values automatically
4. Shows outputs (IDs, connection string, etc.)


done .