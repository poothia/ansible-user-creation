# Demo 3 — Self-Service User Creation

**Story:** "A team lead needs a new Linux account on the server. They don't
file a ticket. They don't call the sysadmin. They open the AAP portal, fill
in a simple form — username, group, SSH key, sudo — and click Launch. Thirty
seconds later, the user is created, configured, and ready to log in. No one
SSH-ed into the server."

**Duration:** ~8-10 minutes live

---

## What You Need Open

| Tab / Window | What |
|---|---|
| Browser — AAP | Automation Controller dashboard |
| Terminal | Ready to SSH into the VM as the new user (for the verify step) |

---

## Pre-Demo Setup (do this before the audience arrives)

1. The RHEL 9 VM is running at `rhel9.sjwnx.sandbox1063.opentlc.com`
2. This `user-management` repo is pushed to GitHub as its own repo
3. The `Demo Inventory` and `RHEL 9 Demo VM` credential exist in AAP
4. (Optional) Generate a throwaway SSH key pair to paste during the demo:
   ```bash
   ssh-keygen -t ed25519 -f /tmp/demo-user-key -N "" -C "demo-user"
   cat /tmp/demo-user-key.pub
   ```

---

## Demo Script

### Step 1 — Set the scene

> **Say:** "A new developer just joined the team and needs access to this
> server. Normally that means a ticket, a wait, and a sysadmin SSH-ing in
> to create the account. Let's do it differently."

---

### Step 2 — Create the AAP Project

> **Say:** "First I point AAP at our user-management automation repo."

**Action:**

1. Go to **Resources --> Projects --> Add**
2. Fill in:
   - Name: `User Management`
   - Source Control Type: Git
   - Source Control URL: `https://github.com/<YOU>/user-management.git`
   - Source Control Credential: `GitHub SCM` (if private)
   - Options: check **Update Revision on Launch**
3. Click **Save**
4. Wait for sync to complete (green checkmark)

---

### Step 3 — Create the Job Template with a Survey

> **Say:** "Now I create a Job Template with a Survey form. The Survey turns
> this into a self-service operation — anyone with permission can create a
> user by filling in a form."

**Action — Create the Job Template:**

1. Go to **Resources --> Templates --> Add --> Add Job Template**
2. Fill in:
   - Name: `Create User`
   - Job Type: Run
   - Inventory: `Demo Inventory`
   - Project: `User Management`
   - Playbook: `playbooks/create-user.yml`
   - Credentials: `RHEL 9 Demo VM`
3. Click **Save**

**Action — Add the Survey:**

4. Go to the **Survey** tab on the Job Template
5. Click **Add** to create each field:

   **Field 1 — Username:**
   - Question: `Username`
   - Description: `Linux username (lowercase, no spaces)`
   - Answer Variable Name: `new_username`
   - Answer Type: Text
   - Required: Yes

   **Field 2 — Full Name:**
   - Question: `Full Name`
   - Description: `First and last name for the account`
   - Answer Variable Name: `full_name`
   - Answer Type: Text
   - Required: Yes

   **Field 3 — Group:**
   - Question: `Group`
   - Description: `Which team does this user belong to?`
   - Answer Variable Name: `user_group`
   - Answer Type: Multiple Choice (single select)
   - Multiple Choice Options:
     ```
     developers
     ops
     security
     devops
     ```
   - Default Answer: `developers`
   - Required: Yes

   **Field 4 — SSH Public Key:**
   - Question: `SSH Public Key`
   - Description: `Paste the user's public key (optional — leave blank to use password)`
   - Answer Variable Name: `ssh_public_key`
   - Answer Type: Textarea
   - Required: No

   **Field 5 — Sudo Access:**
   - Question: `Sudo Access`
   - Description: `Should this user have passwordless sudo?`
   - Answer Variable Name: `sudo_access`
   - Answer Type: Multiple Choice (single select)
   - Multiple Choice Options:
     ```
     yes
     no
     ```
   - Default Answer: `no`
   - Required: Yes

   **Field 6 — Shell:**
   - Question: `Shell`
   - Description: `Login shell`
   - Answer Variable Name: `user_shell`
   - Answer Type: Multiple Choice (single select)
   - Multiple Choice Options:
     ```
     /bin/bash
     /bin/zsh
     /bin/sh
     ```
   - Default Answer: `/bin/bash`
   - Required: No

6. Toggle the Survey to **Enabled**

> **Say:** "Six fields. That is all anyone needs to know to create a fully
> configured Linux account. No playbook knowledge, no SSH, no command line."

---

### Step 4 — Launch the Job Template (fill in the Survey live)

> **Say:** "Now let me be a team lead. I need to create an account for a
> new developer who just joined."

**Action:**

1. Click **Launch** on the `Create User` template
2. The Survey form appears. Fill it in live:

   | Field | Value |
   |---|---|
   | Username | `jdoe` |
   | Full Name | `Jane Doe` |
   | Group | `developers` |
   | SSH Public Key | Paste the key from pre-demo setup (or leave blank) |
   | Sudo Access | `yes` |
   | Shell | `/bin/bash` |

3. Click **Next**, then **Launch**

> **Say:** "That is it. I filled in a form and clicked Launch. Let's watch
> what happens on the server."

---

### Step 5 — Watch the job run (narrate each task)

**Action:** Watch the real-time output in the Controller UI. Narrate:

> "It is validating the inputs... creating the 'developers' group...
> creating the user 'jdoe' with a home directory... deploying the SSH
> public key... creating the sudoers file... writing the welcome banner...
> and now verifying everything was created correctly."

When the job completes, point out the **provisioning summary** at the end:

> **Say:** "Look at the summary — username, group, shell, SSH key deployed,
> sudo granted. All the details in one place. And notice — it shows the
> temporary password and says 'must change on first login'. Security built in."

---

### Step 6 — Verify: SSH as the new user (the "wow" moment)

> **Say:** "Let's prove it. I am going to SSH into the server as the user
> we just created."

**Action:**

1. Open a terminal and SSH in:

   ```bash
   ssh jdoe@rhel9.sjwnx.sandbox1063.opentlc.com
   ```

   If you deployed an SSH key:
   ```bash
   ssh -i /tmp/demo-user-key jdoe@rhel9.sjwnx.sandbox1063.opentlc.com
   ```

2. Show the welcome file:
   ```bash
   cat ~/WELCOME.txt
   ```

3. Show sudo works:
   ```bash
   sudo whoami
   ```
   (should return `root`)

4. Show group membership:
   ```bash
   id
   ```

> **Say:** "The account is fully configured. SSH key works. Sudo works.
> There is even a welcome file telling the user everything they need to know.
> All of this — without anyone SSH-ing into the server to set it up."

---

## Key Talking Points

| Point | What to say |
|---|---|
| **Self-service** | "A team lead creates users through a form. No tickets, no waiting, no sysadmin bottleneck." |
| **Guardrails** | "The Survey constrains the inputs — they can only pick valid groups, valid shells. You cannot mistype a command." |
| **RBAC** | "I can give the 'Team Lead' role permission to launch this template but not edit it. They get the button, not the playbook." |
| **Audit trail** | "Every user creation is logged — who requested it, what values they entered, when it ran, on which server." |
| **Consistency** | "Every user gets the same setup: SSH directory permissions, sudoers validated with visudo, welcome file. No human error." |
| **Scalable** | "This works for 1 user or 100 servers. Change the inventory to a group of hosts and the same form creates the user everywhere." |

---

## What Just Happened (Technical Summary)

```
Team Lead fills in AAP Survey form
  |
  v
Job Template: "Create User"
  - Project: User Management (this repo)
  - Playbook: playbooks/create-user.yml
  - Survey vars: new_username, full_name, user_group,
                 ssh_public_key, sudo_access, user_shell
  |
  v
RHEL 9 VM (rhel9.sjwnx.sandbox1063.opentlc.com):
  1. Validate inputs (assert required fields)
  2. Create group "developers" (if not exists)
  3. Create user "jdoe" with home dir, bash shell, GECOS
  4. Set temporary password (force change on first login)
  5. Deploy SSH public key to ~/.ssh/authorized_keys
  6. Create /etc/sudoers.d/jdoe (NOPASSWD, validated with visudo)
  7. Write ~/WELCOME.txt with account details
  8. Verify: id, home dir, SSH key file, sudoers file
  9. Print summary with login instructions
```

---

## Troubleshooting During Demo

| Problem | Quick Fix |
|---|---|
| Job fails on "Validate required inputs" | Check the Survey fields — variable names must match exactly: `new_username`, `full_name`, `user_group` |
| User already exists | The playbook is idempotent — it will update the existing user. If you want a clean demo, pick a unique username |
| SSH key authentication fails | Verify the key format (must start with `ssh-ed25519` or `ssh-rsa`). Check `/home/jdoe/.ssh/authorized_keys` permissions |
| Sudo does not work | Check `/etc/sudoers.d/jdoe` exists and has mode `0440`. The playbook validates with `visudo` before writing |
| Password prompt on SSH (no key) | Expected if no SSH key was provided. Use the temp password `ChangeMe@123` — you will be forced to change it |

---

## Wrap-Up (All Three Demos)

> **Say:** "Let me recap what we have shown today:
>
> In **Demo 1**, we took a blank RHEL 9 VM and deployed a web application
> entirely from the AAP portal — without ever SSH-ing into the server.
>
> In **Demo 2**, we changed the code and pushed to GitHub. We never opened
> the AAP portal. Event-Driven Ansible picked up the webhook and deployed
> the update automatically in about 30 seconds.
>
> In **Demo 3**, we created a fully-configured Linux user account through a
> simple Survey form — SSH key, sudo, welcome banner, password policy — all
> without touching the command line.
>
> Three different use cases — infrastructure provisioning, continuous
> deployment, and day-2 operations — all powered by one platform:
> **Ansible Automation Platform**."
