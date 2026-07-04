# Ticket #50 — Create Tasks Using Ansible Playbook

*Category: Infrastructure / Ansible Automation*

| **Field** | Value |
| --- | --- |
| **Ticket #** | 50 |
| **Title** | Create Tasks Using Ansible Playbook |
| **Category** | Infrastructure / Ansible Automation |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |

## Objective

Create and run an Ansible playbook targeting dev-app that creates a local user rsinclair, expires that user's password immediately, and installs the tmux package.

## Requirements

- VM: dev-app (managed node)

- Ansible playbook must: create local user rsinclair, expire rsinclair's password, install tmux

- Validate: user exists, password is expired, tmux is installed

## Environment Details

- VM: dev-app (ansible managed node)

- Ansible control node: dev-ansible

- Package: tmux

- User: rsinclair

## Implementation Steps

Step 1 — On the Ansible control node, create the playbook file (e.g. create_user_and_install_tmux.yml):

```bash
vi create_user_and_install_tmux.yml
```
Playbook content:

---

- name: Create user, expire password, install tmux

  hosts: dev_rs1

  become: yes

  tasks:

    - name: Create local user tfleming

      user:

        name: rsinclair

        state: present

    - name: Expire rsinclair password

      command: chage -d 0 tfleming

    - name: Install tmux

      dnf:

        name: tmux

        state: present

Step 2 — Run the playbook:

```bash
ansible-playbook create_user_and_install_tmux.yml
```
Step 3 — Validate results on dev-app:

id rsinclair

```bash
sudo chage -l rsinclair
```
tmux -V

## Troubleshooting & Root Cause Analysis

The main validation point was confirming that tmux -V returned a version string, confirming installation success.

The chage -d 0 command sets the last password change date to epoch (Jan 1, 1970), which forces a password change at next login — this is the correct way to expire a password immediately in RHEL/CentOS.

The Ansible user module creates the user with default settings; the command module runs chage as an ad-hoc system command since Ansible's user module does not expose a direct password_expire_date parameter in this version.

## Validation & Testing

id rsinclair

```bash
sudo chage -l rsinclair
```
- tmux -V

- Expected: user rsinclair exists, password expired (Last password change: Jan 01, 1970), tmux version printed

## Key Lessons Learned

- chage -d 0 <user> immediately expires the password — the user must change it at next login

- Use become: yes in Ansible playbooks for tasks that require root privileges

- Ansible's dnf module is preferred over yum on RHEL 8/9 and CentOS Stream

