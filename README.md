# Automation-Config-Management-Procore-Plus-Lab

Automation &amp; Configuration Management — Ansible playbooks, Foreman remote execution and host registration, and Terraform installation for infrastructure-as-code workflows.

## Environment
CentOS Stream / RHEL-based VMs in the Procore-Plus lab (dev-app, stage-web, dev-performance hosts), managed as a production-style environment with Jira ticket tracking.

## Skills & Tools
Ansible (playbooks, host patterns), Foreman (remote jobs, host registration, SSH trust), Terraform, firewalld automation, compliance patching

## Tickets

| Ticket | Title | Documentation |
| --- | --- | --- |
| #50 | Create Tasks Using an Ansible Playbook (user + tmux) | [ticket-50-ansible-playbook-tasks.md](tickets/ticket-50-ansible-playbook-tasks.md) |
| #51 | Close Ports 80 and 443 via Ansible Playbook | [ticket-51-ansible-close-ports.md](tickets/ticket-51-ansible-close-ports.md) |
| TL-18 | Ansible: Create Shared Scripts Directory (webmasters, 775) | [ticket-tl18-ansible-shared-scripts-dir.md](tickets/ticket-tl18-ansible-shared-scripts-dir.md) |
| TS5-17 | Patch Dev Servers Using Ansible | [ticket-ts517-ansible-patch-dev-servers.md](tickets/ticket-ts517-ansible-patch-dev-servers.md) |
| T-34 | Foreman Remote Command: Create User rcamilo | [ticket-t34-foreman-remote-command.md](tickets/ticket-t34-foreman-remote-command.md) |
| TS5-33 | Register VMs to Foreman Server | [ticket-ts533-register-vms-foreman.md](tickets/ticket-ts533-register-vms-foreman.md) |
| #75 | Install Terraform on a Virtual Machine | [ticket-75-terraform-install.md](tickets/ticket-75-terraform-install.md) |

## About
Each ticket document includes the objective, environment details, step-by-step resolution, commands used, and verification/outcome. These reflect real hands-on system administration work completed in a lab modeled on production operations.
