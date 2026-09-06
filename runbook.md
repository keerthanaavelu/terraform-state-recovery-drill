markdown

# Terraform State Recovery Drill

A hands-on runbook documenting deliberate state-drift and state-corruption scenarios, and how each was diagnosed and recovered — using the `dev` environment from [terraform-envs](https://github.com/keerthanaavelu/terraform-envs).

## Baseline (Before Any Drill)

**`terraform plan`:**

No changes. Your infrastructure matches the configuration.

**`terraform state list`:**

module.vnet.azurerm_network_security_group.this
module.vnet.azurerm_subnet.this
module.vnet.azurerm_subnet_network_security_group_association.this
module.vnet.azurerm_virtual_network.this

Confirmed: dev environment is healthy and matches state before any drill begins.

---

## Drill 1: Resource Deleted Outside Terraform

_(to be documented)_

Drift detection:

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:

- create

Terraform will perform the following actions:

# module.vnet.azurerm_network_security_group.this will be created

- resource "azurerm_network_security_group" "this" {
  - id = (known after apply)
  - location = "eastus"
  - name = "nsg-dev"
  - resource_group_name = "tf_res_group1"
  - security*rule = [ + { + access = "Allow" + destination_address_prefix = "*" + destination*address_prefixes = [] + destination_application_security_group_ids = [] + destination_port_range = "22" + destination_port_ranges = [] + direction = "Inbound" + name = "allow-ssh" + priority = 100 + protocol = "Tcp" + source_address_prefix = "*" + source_address_prefixes = [] + source_application_security_group_ids = [] + source_port_range = "\*" + source_port_ranges = [] # (1 unchanged attribute hidden)
    },
    ]
    }

# module.vnet.azurerm_subnet_network_security_group_association.this will be created

- resource "azurerm_subnet_network_security_group_association" "this" {
  - id = (known after apply)
  - network_security_group_id = (known after apply)
  - subnet_id = "/subscriptions/7b1ed50d-1210-4a50-a492-d45e62644919/resourceGroups/tf_res_group1/providers/Microsoft.Network/virtualNetworks/vnet-dev/subnets/subnet-dev"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if
you run "terraform apply" now.
Releasing state lock. This may take a few moments...
PS E:\Devopsjourney\projects\terraform PE role\terraform-envs\dev>

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

→ Confirmed back to "No changes."

**Outcome:** Manual deletion was detected and cleanly reconciled by letting Terraform recreate the missing resources. Azure's built-in dependency protection (blocking direct NSG deletion while associated) is itself a useful safety net worth noting.

**Lesson:** `terraform plan`/`apply` alone is sufficient recovery when the goal is simply to restore what should exist — `import`/`state mv` are reserved for cases where you want to _adopt_ existing real-world resources without destroying/recreating them (see Drill 2, 3).

## Drill 2: Resource Created Outside Terraform, Adopted via Import

**Scenario:** Someone manually creates a resource in Azure that should be managed by Terraform.

**Setup:**

- Manually created `nsg-manual` (Network Security Group) in `tf_res_group1` via Azure Portal — untouched by Terraform.
- Retrieved its resource ID via `az network nsg show`.
- Wrote a matching (not-yet-applied) resource block in `dev/imported.tf`:

```hcl
  resource "azurerm_network_security_group" "manual" {
    name                = "nsg-manual"
    location            = "eastus"
    resource_group_name = "tf_res_group1"
  }
```

**Import:**
→ "No changes." Config matched the imported resource with no drift.

→ Confirmed `azurerm_network_security_group.manual` now tracked in state.

**Outcome:** Manually-created resource successfully adopted into Terraform management with zero disruption — no destroy, no recreate.

**Lesson:** `terraform import` requires exact resource ID formatting per provider (case-sensitive segments) — a small but common gotcha. Also, the `.tf` resource block must be written _before_ import; import only populates state, not code.
