# User Management — Self-Service User Creation

Ansible playbook and demo guide for showcasing **self-service Linux user
creation** via Ansible Automation Platform Survey forms.

This is an independent demo repo (Demo 3). It pairs with two other repos:

- **webapp** — A Flask web application (the thing being deployed in Demos 1 & 2)
- **aap-automation** — AAP playbooks for provisioning and deploying the webapp (Demos 1 & 2)

---

## Architecture

```
Demo 3: Self-Service User Creation
====================================
Team lead fills in AAP Survey form
  --> Job Template: "Create User"
        --> playbooks/create-user.yml runs on RHEL 9
              --> Group created
              --> User created with home dir and shell
              --> SSH public key deployed
              --> Sudo access configured
              --> Welcome banner written
              --> Verified and summary printed
              --> User is ready to log in
```

---

## Target Environment

| Resource | Details |
|---|---|
| RHEL 9 VM | `rhel9.sjwnx.sandbox1063.opentlc.com` |
| SSH User | `lab-user` (for Ansible to connect) |

---

## What's in This Repo

| Path | Purpose |
|---|---|
| `playbooks/create-user.yml` | Main playbook — creates a Linux user from Survey variables |
| `playbooks/templates/user-welcome.j2` | Jinja2 template for the user's `~/WELCOME.txt` welcome banner |
| `inventory/hosts.yml` | Reference inventory (actual inventory is in Automation Controller) |
| `collections/requirements.yml` | Required Ansible collections (ansible.posix) |
| `DEMO-GUIDE.md` | Step-by-step demo script for presenting Demo 3 |

---

## AAP Survey Form

The Job Template uses a Survey with these fields:

| Field | Variable | Type | Required | Default | Choices |
|---|---|---|---|---|---|
| Username | `new_username` | Text | Yes | — | — |
| Full Name | `full_name` | Text | Yes | — | — |
| Group | `user_group` | Multiple Choice | Yes | `developers` | developers, ops, security, devops |
| SSH Public Key | `ssh_public_key` | Textarea | No | — | — |
| Sudo Access | `sudo_access` | Multiple Choice | Yes | `no` | yes, no |
| Shell | `user_shell` | Multiple Choice | No | `/bin/bash` | /bin/bash, /bin/zsh, /bin/sh |

---

## What the Playbook Does

1. **Validates inputs** — asserts username, full name, and group are provided
2. **Creates group** — ensures the target group exists
3. **Creates user** — with home directory, shell, GECOS (full name)
4. **Sets temporary password** — `ChangeMe@123`, forced change on first login
5. **Deploys SSH key** — to `~/.ssh/authorized_keys` (when provided)
6. **Configures sudo** — creates `/etc/sudoers.d/<user>` with NOPASSWD (when requested), validated with `visudo`
7. **Writes welcome banner** — `~/WELCOME.txt` with account details and login instructions
8. **Verifies everything** — checks user exists, home dir, SSH key file, sudoers file
9. **Prints summary** — full provisioning report with login command

---

## Quick Reference — AAP Objects

| Object | Name | Details |
|---|---|---|
| Credential | `RHEL 9 Demo VM` | Machine type, `lab-user` + SSH key (reuse from Demo 1) |
| Project | `User Management` | Points to this repo |
| Inventory | `Demo Inventory` | Host `rhel9-demo` (reuse from Demo 1) |
| Job Template | `Create User` | Uses `playbooks/create-user.yml`, Survey enabled |

---

## Demo Guide

See [DEMO-GUIDE.md](DEMO-GUIDE.md) for the full step-by-step presentation script.
