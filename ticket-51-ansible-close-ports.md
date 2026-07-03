# Ticket #51 — Closing Ports 80 and 443 via Ansible Playbook

*Category: Infrastructure / Ansible Automation*

| **Field** | Value |
| --- | --- |
| **Ticket #** | 51 |
| **Title** | Closing Ports 80 and 443 via Ansible Playbook |
| **Category** | Infrastructure / Ansible Automation |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |

## Objective

Create and run an Ansible playbook to close (remove) firewall ports 80 and 443 on dev-app using firewalld, automating the firewall rule change rather than applying it manually.

## Requirements

- VM: dev-app

- Ansible playbook that removes ports 80 and 443 from firewalld

- Use the ansible.posix.firewalld module

## Environment Details

- VM: dev-app (ansible managed node)

- Ansible control node: dev-ansible

- Service: firewalld

## Implementation Steps

Step 1 — Create the playbook (e.g. close_http_https_ports.yml):

```bash
vi close_http_https_ports.yml
```
Playbook content:

---

- name: Close ports 80 and 443 on dev-app

  hosts: dev_rs1

  become: yes

  tasks:

    - name: Remove port 80/tcp from firewalld

```bash
ansible.posix.firewalld:
```
        port: 80/tcp

        permanent: true

        state: disabled

    - name: Remove port 443/tcp from firewalld

```bash
ansible.posix.firewalld:
```
        port: 443/tcp

        permanent: true

        state: disabled

    - name: Reload firewalld

      service:

        name: firewalld

        state: reloaded

Step 2 — Run the playbook:

```bash
ansible-playbook close_http_https_ports.yml
```
Step 3 — Validate ports are closed on dev-app:

```bash
sudo firewall-cmd --list-ports
sudo firewall-cmd --list-services
ss -tulnp | egrep ':80|:443'
```
## Troubleshooting & Root Cause Analysis

The key advantage of this ticket was using Ansible to automate the firewall change rather than applying it manually on each host. The ansible.posix.firewalld module applies both the permanent rule and optionally reloads firewalld in one step.

When using permanent: true, firewalld writes the rule to the persistent config but does not reload the running config automatically — a separate reload task is needed to apply the change to the running state.

If the ports were not previously open, the playbook may report 'already in desired state' — this is not an error.

## Validation & Testing

```bash
sudo firewall-cmd --list-ports
sudo firewall-cmd --list-services
ss -tulnp | egrep ':80|:443'
```
- Expected: port 80 and 443 not listed, no listeners on those ports

## Key Lessons Learned

- Use permanent: true AND a reload task together — permanent writes the config, reload applies it

- ansible.posix.firewalld is idempotent — running it multiple times will not cause duplicate rules

- Automating firewall changes with Ansible is preferred over manual ssh+firewall-cmd for consistency across multiple servers

