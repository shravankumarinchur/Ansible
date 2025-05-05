
---

## 🗓️ Day 02 – SSH Authentication Methods & Passwordless Setup

---

### 🔹 **Type 1: Key-based Authentication (Common in Cloud Environments)**

**✅ Use Case:**
When managing cloud servers like AWS EC2, SSH keys (`.pem`) are the default authentication method.

---

**📌 Connect from Control Node (Initial Login):**

```bash
ssh -i /path/to/key.pem ec2-user@<INSTANCE-PUBLIC-IP>
```

> This uses the PEM file (private key) to log in.

---

**📌 Set Up Passwordless SSH (Copy Public Key to Target Server):**

```bash
ssh-copy-id -f "-o IdentityFile=/path/to/key.pem" ec2-user@<INSTANCE-PUBLIC-IP>
```

> This uploads your **control node’s** public key (`~/.ssh/id_rsa.pub`, etc.) to the **target server**, enabling key-based login.
  i,e control node's public key (typically ~/.ssh/id_rsa.pub or similar) is copied to this file on the remote server : ~/.ssh/authorized_keys
---

**✅ After Passwordless Setup – Login Command:**

```bash
ssh ec2-user@<INSTANCE-PUBLIC-IP>
```

> Now you no longer need the PEM file for this host — SSH will use your local public key.

---

### 🔹 **Type 2: Password-based Authentication (On-prem or Non-cloud VMs)**

**✅ Use Case:**
Used in on-premises or legacy systems where password login is enabled instead of SSH keys.

---

**📌 Enable Password Auth:**

Edit the SSH daemon config:

```ini
/etc/ssh/sshd_config
```

Ensure these settings:

```ini
PasswordAuthentication yes
PermitRootLogin yes
ChallengeResponseAuthentication no
UsePAM yes
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

---

**📌 Push Public Key to Server Using Password:**

```bash
ssh-copy-id root@<PUBLIC-IP>
```

> You'll be prompted for the root password once. After that, key-based login will work.

---

**✅ After Passwordless Setup – Login Command:**

```bash
ssh root@<PUBLIC-IP>
```

> This now works **without entering the password**.

---

### 🔹 **Note for Cloud Instances (like AWS EC2)**

Cloud-init may override `sshd_config`. Check:

```bash
cd /etc/ssh/sshd_config.d/
```

Edit:

```bash
sudo vi 50-cloud-init.conf
```

Set:

```ini
PasswordAuthentication yes
```

Then restart:

```bash
sudo systemctl restart sshd
```

---

## 💡 Summary

| Step                         | Command                                                           |
| ---------------------------- | ----------------------------------------------------------------- |
| Initial login with PEM       | `ssh -i /path/to/key.pem ec2-user@<ip>`                           |
| Copy key (cloud)             | `ssh-copy-id -f "-o IdentityFile=/path/to/key.pem" ec2-user@<ip>` |
| Copy key (on-prem/password)  | `ssh-copy-id root@<ip>`                                           |
| Login after passwordless set | `ssh ec2-user@<ip>` or `ssh root@<ip>`                            |

---

Absolutely! Here's a short, clean version you can append to your GitHub notes:

---
### NOTE:
### 🔹 Passwordless SSH for Specific Users

* Passwordless SSH is **user-specific** — the public key is stored in:

  ```
  /home/<username>/.ssh/authorized_keys
  ```

#### ✅ Steps to Set Up for Any User (e.g., `john`):

1. **Ensure user exists** on the remote machine:

   ```bash
   ssh root@<REMOTE-IP> "id john || useradd john"
   ```

2. **Copy SSH public key** to that user:

   ```bash
   ssh-copy-id john@<REMOTE-IP>
   ```

3. **Login without password**:

   ```bash
   ssh john@<REMOTE-IP>
   ```

> SSH will now use your key for `john` instead of prompting for a password.

---



