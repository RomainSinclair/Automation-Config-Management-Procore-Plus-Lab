# Automation & Configuration Management — Procore-Plus Lab

Ansible playbooks, Foreman remote execution and host registration, and Terraform installation for infrastructure-as-code workflows.

## Environment

CentOS Stream / RHEL-based VMs (`dev-app`, `stage-web`, `dev-performance` hosts) managed as a production-style environment with Jira ticket tracking, monitoring, and change control.

## Skills & Tools

Ansible (playbooks, host patterns, idempotent tasks) · Foreman (remote jobs, host registration, SSH trust) · Terraform · firewalld automation · compliance patching

## Tickets

| Ticket | Title | Documentation |
| --- | --- | --- |
| #50 | Create Tasks Using an Ansible Playbook (user + tmux) | [view](tickets/ticket-50-ansible-playbook-tasks.md) |
| #51 | Close Ports 80 and 443 via Ansible Playbook | [view](tickets/ticket-51-ansible-close-ports.md) |
| #18 | Ansible: Create Shared Scripts Directory (webmasters, 775) | [ticket-18-ansible-shared-scripts-dir.md](tickets/ticket-18-ansible-shared-scripts-dir.md) |
| #17 | Patch Dev Servers Using Ansible | [ticket-17-ansible-patch-dev-servers.md](tickets/ticket-17-ansible-patch-dev-servers.md) |
| #34 | Foreman Remote Command: Create User rcamilo | [ticket-34-foreman-remote-command.md](tickets/ticket-34-foreman-remote-command.md) |
| #33 | Register VMs to Foreman Server | [view](tickets/ticket-33-register-vms-foreman.md) |
| #75 | Install Terraform on a Virtual Machine | [view](tickets/ticket-75-terraform-install.md) |

## Highlights

- **Security hardening via automation (#51):** Wrote an Ansible playbook to close ports 80/443 through firewalld across target hosts, avoiding manual per-server changes.
- **Fleet management (#17, #33):** Patched multiple dev servers by host pattern and registered VMs to Foreman under SSH trust for centralized configuration management.
- **Remote execution (#34):** Used Foreman remote commands to provision a new user account without direct SSH access to the target host.

## About

Each ticket document includes the objective, environment details, step-by-step commands, troubleshooting/root-cause notes, and verification of the outcome.
