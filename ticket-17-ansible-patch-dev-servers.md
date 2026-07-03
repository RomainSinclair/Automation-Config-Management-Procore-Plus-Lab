
| **Field** | Value |
| --- | --- |
| **Ticket #** | 17 |
| **Title** | Patch Dev Servers Using Ansible** *Ansible Playbook · yum update · Host Pattern · Compliance Patching |
| **Category** | Ansible Automation |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |
| **Target Servers** |  dev-app, dev-performance |




## 1. Objective

In accordance with company policy, all development servers must be patched regularly. This task uses an Ansible playbook from the dev-ansible control node to automate patching of dev-app and dev-performance simultaneously.

> **  Task Requirements** Successful dev server patching via Ansible playbook Route: dev-ansible → dev-app AND dev-performance Playbook location: /opt/ansible/patching/dev-patch.yml Screenshots of playbook execution required

## 2. Why Ansible for Patching? The Engineer's Rationale

| **Concept** | **Explanation** |
| --- | --- |
| Idempotency | Run the playbook 10 times — same result. yum update only applies missing patches; no duplicate installs. |
| Multi-host Parallel | One command patches dev-app AND dev-performance simultaneously — saves time vs. SSH-ing into each server. |
| Auditability | The YAML playbook is version-controllable evidence of exactly what was patched, when, and on which hosts. |
| Safe Exclusions | The playbook excludes kernel, kernel-*, and rpm* from updates — prevents accidental kernel upgrades that could cause boot failures. |

## 3. Step-by-Step Execution

### Step 1 — SSH into the Ansible Control Node

```bash
# Connect to the Ansible control node ssh <user>@dev-ansible.procore.prod1
# Navigate to the patching playbook directory cd /opt/ansible/patching ls *.yml
# list available playbooks
```

> **  Screenshot 1: SSH login to dev-ansible and navigation to /opt/ansible/patching**

### Step 2 — Run the Patching Playbook (First Attempt)

```bash
# Run the dev-patch playbook with privilege escalation ansible-playbook dev-patch.yml -K
# -K prompts for BECOME (sudo) password
```

> **  Troubleshooting: Host Pattern Mismatch** Error: [WARNING]: Invalid characters were found in group names but not replaced        [WARNING]: Could not match supplied host pattern, ignoring: dev-[Initials]        Skipping: no hosts matched Cause: The playbook hosts: field contains the placeholder "dev-[Initials]" which was never updated to match the actual group name defined in /etc/ansible/hosts. Fix: Open dev-patch.yml and update the hosts: value to match your actual inventory group name (e.g., dev-tl, dev-ts5, etc.).

> **  Screenshot 2: First playbook run showing WARNING host pattern mismatch and skipping output**

### Step 3 — Fix Host Pattern in Playbook

```bash
# Open the playbook to fix the hosts field vim dev-patch.yml
# BEFORE (wrong — placeholder not replaced):
# hosts: dev-[Initials]
# AFTER (correct — matches inventory group):
# hosts: dev-ts5
# Verify the playbook contents cat dev-patch.yml
```

> **  Engineer's Note** Check /etc/ansible/hosts to confirm the correct group name. The hosts: value in the playbook MUST exactly match a group defined in the inventory file.

> **  Screenshot 3: vim dev-patch.yml showing corrected hosts field and playbook YAML structure**

### Step 4 — Re-run Playbook and Confirm Success

```bash
# Re-run patching playbook after fixing host pattern ansible-playbook dev-patch.yml -K
# Expected PLAY RECAP:
# dev-app-ts5.procore.prod1    : ok=2  changed=1  unreachable=0  failed=0
# dev-performance-ts5.procore.prod1 : ok=2  changed=1  unreachable=0  failed=0
```

> **  Screenshot 4: Successful PLAY RECAP showing ok=2 changed=1 failed=0 on both hosts**

## 4. Verification Matrix

| **#** | **Check Item** | **How to Verify** | **Status** |
| --- | --- | --- | --- |
| 1 | SSH'd into dev-ansible control node | Terminal showing dev-ansible prompt |  Done |
| 2 | Navigated to /opt/ansible/patching/ | pwd output confirmed correct directory |  Done |
| 3 | Identified host pattern mismatch in dev-patch.yml | WARNING message in first run output |  Done |
| 4 | Updated hosts: value to match inventory group | cat dev-patch.yml shows correct group name |  Done |
| 5 | Playbook ran successfully on dev-app | PLAY RECAP: failed=0 for dev-app |  Done |
| 6 | Playbook ran successfully on dev-performance | PLAY RECAP: failed=0 for dev-performance |  Done |

## 5. Command Quick Reference

```bash
# ── CONNECT ───────────────────────────────────────────────────────────── ssh <user>@dev-ansible.procore.prod1 cd /opt/ansible/patching
# ── CHECK INVENTORY ────────────────────────────────────────────────────── cat /etc/ansible/hosts
# find your group name
# ── FIX PLAYBOOK ───────────────────────────────────────────────────────── vim dev-patch.yml
# update: hosts: <your-group-name>
# ── DRY RUN (no changes) ───────────────────────────────────────────────── ansible-playbook dev-patch.yml -K --check
# ── EXECUTE ────────────────────────────────────────────────────────────── ansible-playbook dev-patch.yml -K
```

> **Ticket 17 · Patch Dev Servers Using Ansible · Procore-Plus Lab***  │  Assignee: Romain Sinclair · PROCORE Infrastructure Team*
