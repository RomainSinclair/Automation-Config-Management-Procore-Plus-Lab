> **PROCORE-PLUS LAB** **Ticket 18: Ansible: Create Shared Scripts Directory** *Ansible Playbook · Directory Provisioning · Webmasters Group · Permissions 775*

| **Ticket ID**   18 | **Reporter**   Procore Plus |
| --- | --- |
| **Project**   PROCORE-Plus Lab | **Assignee**   Romain Sinclair |
| **Type**   Task | **Resolved**   October 3, 2025 |
| **Priority**   Medium | **Control Node**   dev-ansible |
| **Status**   Done ✓ | **Target Servers**   dev-app, dev-performance |

## 1. Objective

The webmasters team requires a shared scripting directory on all development servers for collaboration. Instead of manually creating directories on each server, Ansible automates provisioning to ensure consistency, repeatability, and speed across the infrastructure.

> **📋  Task Requirements** Directory: /opt/scripts/<yourusername>/ on BOTH dev-app and dev-performance Owner: <yourusername> (your Linux account) Group: webmasters Permissions: 775 (rwxrwxr-x) Ansible route: dev-ansible → dev-app AND dev-performance

## 2. Understanding Permission 775

| **Octet** | **Who** | **Access (rwx)** | **Why** |
| --- | --- | --- | --- |
| 7 (rwx) | Owner (you) | Read + Write + Execute | Full control over your own directory |
| 7 (rwx) | Group (webmasters) | Read + Write + Execute | Team members can collaborate and add scripts |
| 5 (r-x) | Others | Read + Execute only | Others can use scripts but cannot modify them |

## 3. Step-by-Step Execution

### Step 1 — SSH into dev-ansible (Control Node)

```bash
# Connect to Ansible control node ssh <user>@dev-ansible.procore.prod1
# Navigate to playbook directory cd /opt/ansible/patching pwd
# confirm: /opt/ansible/patching
```

> **💡  Engineer's Note** All Ansible work must originate from the control node. Running ansible-playbook from a managed node (dev-app, dev-performance) would fail — those servers don't have the inventory or SSH keys needed to reach other hosts.

> **📸  Screenshot 1: SSH login to dev-ansible and cd /opt/ansible/patching**

### Step 2 — Create the Ansible Playbook

```bash
# Create the playbook file
sudo vim create_webmasters_shareddir_<initials>.yml
# Playbook contents: --- - name: Create scripts directory for <user>   hosts: dev-<initials>   become: yes   tasks:     - name: Create /opt/scripts/{{ ansible_user }}/ directory       file:         path: "/opt/scripts/{{ ansible_user }}/"         state: directory         owner: "{{ ansible_user }}"         group: webmasters         mode: "0775"
```

> **💡  Engineer's Note** {{ ansible_user }} is an Ansible magic variable — it automatically uses the SSH username connecting to each host. This makes the playbook reusable across different users without hardcoding names. become: yes is required to create directories in /opt/ which is root-owned.

> **📸  Screenshot 2: vim editor showing complete playbook YAML and cat output confirming contents**

### Step 3 — Execute and Verify

```bash
# Run the playbook with BECOME password ansible-playbook create_webmasters_shareddir_<initials>.yml -K
# Expected PLAY RECAP:
# dev-app    : ok=2  changed=1  unreachable=0  failed=0
# dev-perf   : ok=2  changed=1  unreachable=0  failed=0
# Verify on dev-app ssh <user>@dev-app.procore.prod1 ls -ld /opt/scripts/<username>/
# Expected: drwxrwxr-x. 2 <user> webmasters 6 Oct 3 00:00 /opt/scripts/<user>/
```

> **📸  Screenshot 3: ansible-playbook PLAY RECAP showing ok=2 changed=1 failed=0 on both hosts**

## 4. Verification Matrix

| **#** | **Check Item** | **How to Verify** | **Status** |
| --- | --- | --- | --- |
| 1 | Playbook file created in /opt/ansible/patching/ | cat filename.yml — file exists and syntax is correct | ✅  Done |
| 2 | Playbook executed with ansible-playbook -K | Terminal output shows PLAY RECAP | ✅  Done |
| 3 | PLAY RECAP shows failed=0 on dev-app | RECAP line: failed=0 | ✅  Done |
| 4 | PLAY RECAP shows failed=0 on dev-performance | RECAP line: failed=0 | ✅  Done |
| 5 | /opt/scripts/<user>/ exists on dev-app | ls -ld confirms directory present | ✅  Done |
| 6 | /opt/scripts/<user>/ exists on dev-performance | ls -ld confirms directory present | ✅  Done |
| 7 | Owner is correct user on both servers | ls -ld output: owner column = <username> | ✅  Done |
| 8 | Group is webmasters on both servers | ls -ld output: group column = webmasters | ✅  Done |
| 9 | Permissions are 775 (drwxrwxr-x) | ls -ld output: mode = drwxrwxr-x | ✅  Done |

> **Ticket 18 · Ansible: Create Shared Scripts Directory · Procore-Plus Lab***  │  Assignee: Romain Sinclair · PROCORE Infrastructure Team*
