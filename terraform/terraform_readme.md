# Terraform in This Project

## Purpose

This directory contains all infrastructure-as-code for the SOC homelab in Azure.

This document is a structural and operational guide for reviewers starting from a clean state.

It provisions:

- Resource group and virtual network.
- Subnets for SOC, AD, client, and Bastion.
- NSGs and subnet associations.
- Azure Bastion with tunneling enabled.
- Linux Splunk VM and two Windows VMs.
- Useful outputs (private IPs, domain, management notes).

## Why The Files Were Split

Initial implementation started with three files:

```text
terraform/
  main.tf
  variables.tf
  terraform.tfvars
```

As the design grew, those files became harder to maintain and review. The configuration was split into focused files so each concern has a clear home.

## Previous vs Current Structure

Previous structure:

```text
terraform/
  main.tf
  variables.tf
  terraform.tfvars
```

Current structure:

```text
terraform/
  compute.tf
  network.tf
  outputs.tf
  providers.tf
  security.tf
  terraform.tfvars
  variables.tf
```

## Creation Order and File Responsibility

The practical order used while expanding the Terraform design was:

1. `providers.tf`
   Initializes provider and Terraform requirements.

2. `variables.tf`
   Defines configurable inputs for the lab.

3. `network.tf`
   Creates resource group, VNet, and core subnets.

4. `security.tf`
   Adds Bastion subnet, Bastion host, NSGs, and rule associations.

5. `compute.tf`
   Defines NICs and VM resources (`vm-splunk`, `vm-ad`, `vm-client`).

6. `outputs.tf`
   Exposes key addresses and operational outputs.

7. `terraform.tfvars`
   Supplies environment-specific values for this lab.

## How To Run Terraform Here

Run from `terraform/`:

```bash
terraform init -upgrade
terraform fmt
terraform validate
az login --tenant <tenant-id>
terraform plan -out tfplan
terraform apply tfplan
```

Here are the outputs and platform screenshots after a successful apply:

Terraform Plan Output![Terraform plan output](../_assets/terraform_plan.png)

Terraform Apply Output![Terraform apply output](../_assets/terraform_apply.png)

Azure Platform![Azure platform](../_assets/azure_platform.png)

## Operational Recovery Note

This design assumes clean apply from scratch. If a VM later misbehaves (for example Bastion instability, guest OS lockup, reconnect failures), replace that VM only:

```bash
terraform plan -replace="azurerm_windows_virtual_machine.ad_vm" -out tfplan-replace
terraform apply tfplan-replace
```

Use the same pattern for other VMs by adjusting the resource target.

## Constraint Recorded During Design

No dedicated attacker VM is deployed in the current active design due to regional core/quota limits.

Simulation is intentionally run from `vm-client` to preserve detection pipeline validation with available resources.

## Future Improvements

1. Move secrets out of `terraform.tfvars` into secure secret handling.
2. Add remote state backend with locking (for example Azure Storage + state lock).
3. Add environment layering (`dev`, `test`) and variable validation rules.
4. Add CI checks for `terraform fmt`, `validate`, and policy gates.
5. Add optional attacker VM module that can be enabled only when quota permits.
