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
