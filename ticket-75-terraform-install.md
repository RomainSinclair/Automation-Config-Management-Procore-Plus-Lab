# Ticket #75 — Install Terraform on Virtual Machine

*Category: Infrastructure / Terraform (IaC)*

| **Field** | Value |
| --- | --- |
| **Ticket #** | 75 |
| **Title** | Install Terraform on Virtual Machine |
| **Category** | Infrastructure / Terraform (IaC) |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |

## Objective

Install Terraform on dev-app using the HashiCorp RHEL repository and create a VM template of dev-performance in vSphere so it can be used to deploy a new production server using Terraform.

## Requirements

- Terraform installation on dev-app server

- Terraform Template — create a VM template of dev-performance in vSphere

- Verify terraform -version and initialization

## Environment Details

- VM: dev-app

- Tool: Terraform (HashiCorp)

- Platform dependency: vSphere (for VM template creation)

## Implementation Steps

Step 1 — Install required utilities:

```bash
sudo dnf -y install dnf-plugins-core curl unzip
```
Step 2 — Add the HashiCorp RHEL repository:

```bash
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```
Step 3 — Install Terraform:

```bash
sudo dnf -y install terraform
```
Step 4 — Verify the installation:

```bash
terraform -version
```
which terraform

Step 5 — Initialize a working directory:

```bash
terraform init
```
Step 6 — Validate configuration (if .tf files present):

```bash
terraform validate
terraform plan
```
Step 7 — In vSphere, create a VM template from dev-performance:

Right-click dev-performance-rs1 > Clone to Template

Select datastore DS-01, confirm template creation

## Troubleshooting & Root Cause Analysis

Terraform was not available in the default CentOS Stream / RHEL system repositories. The HashiCorp RHEL repository had to be added manually via dnf config-manager before the package became available.

The VM template piece was identified as a vSphere administration action. Terraform would consume the template (using vsphere_virtual_machine data source) rather than create it solely from within the Linux guest. The template is created through the vSphere Client UI.

```bash
terraform init requires a properly configured main.tf with provider definitions before it will succeed. A minimal vsphere provider block is needed for vSphere-targeted Terraform work.
```
## Validation & Testing

```bash
terraform -version
```
- which terraform

```bash
terraform init
terraform validate
terraform plan
```
## Key Lessons Learned

- Terraform is not in the default RHEL repos — always add the HashiCorp repo first

- terraform init must be run before validate or plan — it downloads the required providers

- VM templates in vSphere are created via the UI (Clone to Template) not from the Terraform provider itself

## Screenshots

***Figure 1: vSphere Client — Clone Virtual Machine to Template wizard, storage selection (DS-01)***

***Figure 2: vSphere — VM template creation confirmation***

***Figure 3: dev-app terminal — Terraform installed and version verified***
