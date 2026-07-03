
| **Field** | Value |
| --- | --- |
| **Ticket #** | 34 |
| **Title** | Foreman Remote Command: Create User rcamilo** *Remote Job Execution · useradd · SSH Trust · Foreman Proxy* |
| **Category** | Infrastructure / Foreman |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |
| **Status**   | **Servers**   dev-app, stage-web via Foreman |


## 1. Objective

The Security Team requires a new local user (Reuben Camilo, username: rcamilo) to be created on all infrastructure VMs. Rather than SSH-ing into each server individually, Foreman's remote job execution feature automates this across all hosts simultaneously.

> **  Task Requirements** Create user rcamilo (full name: Reuben Camilo) on ALL servers Use Foreman remote job execution — NOT manual SSH Reference: Foreman wiki (http://10.1.10.122) for remote command guidance Screenshots of Foreman UI and job results required

## 2. Step-by-Step Execution

### Step 1 — Access Foreman Web UI and Configure Remote Job

```bash
# Navigate to Foreman web interface
# URL: https://10.1.30.24/users/login
# Path: Hosts → All Hosts → select target VM
# Actions (top right) → Schedule Remote Job
# Configure the job:
#  Job Category: Commands
#  Job Template: Run Command — SSH Default
#  Search Query: name ^ (dev-app-ma3.procore.prod1)
```

> **  Screenshot 1: Foreman web UI showing remote job configuration form**

### Step 2 — Enter the useradd Command

```bash
# Click "Display Advanced Fields"
# In the Command field enter: useradd -m -c "Reuben Camilo" -s /bin/bash rcamilo
# Flags explained:
# -m  : create home directory /home/rcamilo
# -c  : comment/full name field in /etc/passwd
# -s  : default shell = /bin/bash
# Enter server password in Password field
# Click Run
```

> **  Screenshot 2: Foreman command field showing useradd command and password entry**

### Step 3 — First Attempt — 100% Failed

> **  Troubleshooting: Foreman Remote Job 100% Failed** Symptom: Foreman shows 100% Failed — red circle with 1 failure, 0 success. Cause: The foreman-proxy user did not have trusted SSH access to the target servers.        foreman-proxy/.ssh/known_hosts was missing entries for the target hosts,        and the Foreman proxy public key had not been copied to target servers. Fix (performed on stage-foreman as root):   ssh-keygen -R 10.1.30.208  (remove old/stale key)   ssh-keyscan -t ecdsa  10.1.30.209 >> /root/.ssh/known_hosts   ssh-keyscan -t ed25519 10.1.30.209 >> /root/.ssh/known_hosts   ssh-keyscan -t ecdsa  10.1.30.209 >> ~foreman-proxy/.ssh/known_hosts   ssh-keyscan -t ed25519 10.1.30.209 >> ~foreman-proxy/.ssh/known_hosts   ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub 10.1.30.209 -f

> **  Screenshot 3: Foreman job results showing 100% Failed (red) on first attempt**

### Step 4 — Fix SSH Trust and Re-run — 100% Success

```bash
# After ssh-keyscan and ssh-copy-id fix — re-run Foreman job
# dev-app result:
# PLAY RECAP: 100% Success (green)
# Target: dev-app-ma3.procore.prod1
# Repeat for stage-web (10.1.30.211): ssh-keygen -R 10.1.30.211 ssh-keyscan -t ecdsa 10.1.30.211 >> ~foreman-proxy/.ssh/known_hosts ssh-keyscan -t ed25519 10.1.30.211 >> ~foreman-proxy/.ssh/known_hosts ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub 10.1.30.211 -f
# stage-web result:
# 100% Success (green)
```

> **  Screenshot 4: Foreman job results showing 100% Success (green) on dev-app and stage-web**

## 3. Verification Matrix

| **#** | **Check Item** | **How to Verify** | **Status** |
| --- | --- | --- | --- |
| 1 | Foreman remote job configured with useradd command | Foreman UI — command field shows correct useradd flags |  Done |
| 2 | Foreman proxy SSH trust resolved (known_hosts) | ssh-keyscan output — keys added to foreman-proxy |  Done |
| 3 | Foreman proxy key copied to dev-app | ssh-copy-id: "Number of key(s) added: 1" |  Done |
| 4 | Foreman remote job succeeded on dev-app | Foreman results: 100% Success — green |  Done |
| 5 | Foreman proxy key copied to stage-web | ssh-copy-id: "Number of key(s) added: 1" |  Done |
| 6 | Foreman remote job succeeded on stage-web | Foreman results: 100% Success — green |  Done |
| 7 | User rcamilo exists on servers | id rcamilo or getent passwd rcamilo |  Done |

> **Ticket 34 · Foreman Remote Command: Create User rcamilo · Procore-Plus Lab***  │  Assignee: Romain Sinclair · PROCORE Infrastructure Team*
