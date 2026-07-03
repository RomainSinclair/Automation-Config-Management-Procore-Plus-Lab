
| **Field** | Value |
| --- | --- |
| **Ticket #** | 33 |
| **Title** | Register VMs to Foreman Server** *Configuration Management · SSH Trust · Foreman Proxy · Security Compliance* |
| **Category** | Infrastructure / Configuration Management |
| **Prepared by** | Romain Sinclair |
| **Environment** | Procore-Plus Lab (CentOS Stream / RHEL-based) |


## 1. Objective

The Security Team requires all infrastructure servers to be registered with the Foreman configuration management server for centralized patching and compliance tracking. This ticket registers dev-app, dev-performance, and stage-web to Foreman.

> **  Task Requirements** All servers registered to Foreman: dev-app, dev-performance, stage-web Foreman proxy must have SSH trust to all managed hosts Use Foreman Wiki (http://10.1.10.122) for setup guidance Provide screenshots of all work completed

## 2. Why Foreman? The Engineer's Rationale

| **Concept** | **Explanation** |
| --- | --- |
| Centralized Patching | Apply security patches to all servers from one control plane — no SSH-ing into each server individually. |
| Compliance Tracking | Foreman maintains patch history and compliance reports — essential for audit readiness. |
| Remote Job Execution | Run commands on multiple hosts simultaneously via Foreman's UI — scales to hundreds of servers. |
| SSH Proxy Architecture | Foreman communicates through a foreman-proxy service — isolates the control plane from direct server access. |

## 3. Step-by-Step Execution

### Step 1 — Confirm Foreman Server IP Address

```bash
# On dev-app — confirm foreman server IP in hosts file cat /etc/hosts │ grep foreman
# Output: 10.1.30.24    stage-foreman.procore.prod
# SSH into the Foreman server ssh tshank@10.1.30.24
```

> **  Screenshot 1: cat /etc/hosts │ grep foreman output and SSH login to Foreman**

### Step 2 — Configure SSH to Allow Root Login on Target Servers

The Foreman proxy needs to SSH into managed hosts as root. Edit sshd_config to allow this.

```bash
# On dev-app — switch to root
sudo -i
# Edit SSH config to allow root login vi /etc/ssh/sshd_config
# → Set: PermitRootLogin yes
# Restart SSH service systemctl restart sshd
```

> **  Engineer's Note** PermitRootLogin yes is required temporarily for Foreman proxy key exchange. In production hardened environments, certificate-based auth replaces password-based root login after initial setup.

> **  Screenshot 2: sshd_config with PermitRootLogin yes and systemctl restart sshd**

### Step 3 — Add Foreman Proxy SSH Keys to Known Hosts

From the stage-foreman server, scan the target server keys and add the Foreman proxy public key.

```bash
# On stage-foreman — remove stale key (if host was previously registered) ssh-keygen -R 10.1.30.209
# Scan and add target server keys for root ssh-keyscan -t ecdsa  10.1.30.209 >> /root/.ssh/known_hosts ssh-keyscan -t ed25519 10.1.30.209 >> /root/.ssh/known_hosts
# Scan and add target server keys for foreman-proxy user ssh-keyscan -t ecdsa  10.1.30.209 >> ~foreman-proxy/.ssh/known_hosts ssh-keyscan -t ed25519 10.1.30.209 >> ~foreman-proxy/.ssh/known_hosts
# Copy Foreman proxy public key to target server ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub 10.1.30.209 -f
```

> **  Troubleshooting: SSH Known Hosts Mismatch** Symptom: Foreman remote jobs fail — SSH host key verification fails. Cause: The foreman-proxy user's known_hosts doesn't have entries for the target servers. Fix: Run ssh-keyscan for both ecdsa and ed25519 key types, into BOTH /root/.ssh and ~foreman-proxy/.ssh. Critical: Must copy the FOREMAN-PROXY key (not root key) using ssh-copy-id with the -f force flag.

> ** Screenshot 3: ssh-keyscan commands and ssh-copy-id output showing key added**

### Step 4 — Verify Clean SSH Access

```bash
# Verify foreman-proxy can SSH cleanly to dev-app ssh 10.1.30.209
# from stage-foreman
# Expected: Clean login without password or key warnings
# Repeat entire process for stage-web (10.1.30.211) ssh-keygen -R 10.1.30.211 ssh-keyscan -t ecdsa  10.1.30.211 >> /root/.ssh/known_hosts ssh-keyscan -t ed25519 10.1.30.211 >> /root/.ssh/known_hosts ssh-keyscan -t ecdsa  10.1.30.211 >> ~foreman-proxy/.ssh/known_hosts ssh-keyscan -t ed25519 10.1.30.211 >> ~foreman-proxy/.ssh/known_hosts ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub 10.1.30.211 -f
```

> ** Screenshot 4: Clean SSH login to dev-app and stage-web from Foreman proxy**

## 4. Verification Matrix

| **#** | **Check Item** | **How to Verify** | **Status** |
| --- | --- | --- | --- |
| 1 | Foreman server IP confirmed in /etc/hosts | cat /etc/hosts │ grep foreman | Done |
| 2 | PermitRootLogin yes set on all target servers | grep PermitRootLogin /etc/ssh/sshd_config | Done |
| 3 | ssh-keyscan run for both ecdsa and ed25519 keys | known_hosts files updated for root and foreman-proxy | Done |
| 4 | Foreman proxy public key copied to dev-app | ssh-copy-id output: "Number of key(s) added: 1" | Done |
| 5 | Foreman proxy public key copied to stage-web | ssh-copy-id output: "Number of key(s) added: 1" | Done |
| 6 | Clean SSH access verified to dev-app | SSH from Foreman without password/warnings | Done |
| 7 | Clean SSH access verified to stage-web | SSH from Foreman without password/warnings | Done |

> **Ticket 33 · Register VMs to Foreman Server · Procore-Plus Lab***  │  Assignee: Romain Sinclair · PROCORE Infrastructure Team*
