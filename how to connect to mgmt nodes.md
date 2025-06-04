
---

## 🗓️ Day 02 – SSH Authentication Methods, Passwordless Setup & Ansible Inventory

---

### 🔹 **Type 1: Key-based Authentication (Cloud Environments)**

#### ✅ Use Case:

Cloud servers like AWS EC2 use SSH keys (`.pem`) for secure login.

#### 📌 Initial Login from Control Node:

```bash
ssh -i /path/to/key.pem ec2-user@<INSTANCE-PUBLIC-IP>
```

> Uses the PEM file (private key) for login.

#### 📌 Set Up Passwordless SSH:

```bash
ssh-copy-id -f "-o IdentityFile=/path/to/key.pem" ec2-user@<INSTANCE-PUBLIC-IP>
```

> This uploads the control node’s **public key** (typically `~/.ssh/id_rsa.pub`) to the remote node’s file:
> `~/.ssh/authorized_keys`

#### ✅ Login After Setup:

```bash
ssh ec2-user@<INSTANCE-PUBLIC-IP>
```

> You can now SSH without the PEM file.

---

### 🔹 **Type 2: Password-based Authentication (On-premise VMs)**

#### ✅ Use Case:

Used where password auth is enabled instead of PEM.

#### 📌 Enable Password Authentication:

Edit the SSH config:

```ini
/etc/ssh/sshd_config
```

Ensure the following entries:

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

#### 📌 Set Up Passwordless SSH (Using Password):

```bash
ssh-copy-id root@<PUBLIC-IP>
```

> You'll be prompted for the password once, then key-based login will work.

#### ✅ Login After Setup:

```bash
ssh root@<PUBLIC-IP>
```

---

### 🔹 **Note for Cloud Instances (e.g., AWS EC2)**

In some cloud environments, `cloud-init` may override `sshd_config`. In that case:

```bash
cd /etc/ssh/sshd_config.d/
sudo vi 50-cloud-init.conf
```

Update the line:

```ini
PasswordAuthentication yes
```

Then restart SSH:

```bash
sudo systemctl restart sshd
```

---

### 🔹 Passwordless SSH for Specific Users

Passwordless SSH is **user-specific**. The public key gets stored in:

```bash
/home/<username>/.ssh/authorized_keys
```

#### ✅ Setup Example for a User (e.g., `john`):

```bash
# Ensure user exists
ssh root@<REMOTE-IP> "id john || useradd john"

# Copy your key
ssh-copy-id john@<REMOTE-IP>

# Login
ssh john@<REMOTE-IP>
```

---

## 📘 Ansible Inventory Basics

---

### 🧾 What is Ansible Inventory?

An **Ansible Inventory** is a file listing the managed nodes (IP addresses/usernames), which Ansible uses to execute tasks remotely.

---

### 📂 Inventory File Types

1. **INI Format (Most common)** – `hosts.ini`, `inventory.ini`
2. **YAML Format** – Also supported but less used

> INI is preferred for its readability and ease of use.

---

### 📍 Inventory File Location

* Use a **custom inventory file** and pass its path:

  ```bash
  ansible -i /path/to/inventory.ini ...
  ```

* Or use the **default inventory**:

  ```bash
  /etc/ansible/hosts
  ```

  > If not present, create it. Ansible will use this if `-i` isn't passed.

---

### 🧪 Example Inventory (INI Format)

```ini
root@10.168.5.33
root@10.168.5.84
```

---

### ▶️ Running Ad-hoc Commands

**Syntax**:

```bash
ansible -i <inventory_file> -m <module> -a <arguments> <hosts>
```

#### Example – Ping All Nodes:

```bash
ansible -i inventory.ini -m ping all
```

#### Output:

```json
root@10.168.5.84 | SUCCESS => { "ping": "pong" }
root@10.168.5.33 | SUCCESS => { "ping": "pong" }
```

---

### ❌ Host Not in Inventory

```bash
ansible -i inventory.ini -m ping root@10.168.4.50
```

```plaintext
[WARNING]: Could not match supplied host pattern, ignoring: root@10.168.4.50
[WARNING]: No hosts matched, nothing to do
```

---

### 🧭 Grouping Hosts in Inventory

Use groups to organize nodes:

```ini
[app]
root@10.168.5.33

[db]
root@10.168.5.84
```

#### Run Command on Specific Group:

```bash
ansible -i inventory.ini -m ping db
```

---

### 🧰 How to Run Instructions in Ansible

1. **Ad-hoc Commands** – For quick, one-liner tasks
2. **Playbooks (YAML)** – Reusable scripts for complex automation

---

### 🧪 Example Task – List Files in `/etc`

#### On `app` Group:

```bash
ansible -i inventory.ini -m shell -a "ls -l /etc/" app
```

#### On All Nodes:

```bash
ansible -i inventory.ini -m shell -a "ls -l /etc/" all
```

---

### 📌 Ad-hoc Command Syntax Summary

```bash
ansible -i <inventory_file> -m <module> -a "<args>" <host/group>
```

#### Example:

```bash
ansible -i inventory.ini -m shell -a "apt update" all
```

---

## 📚 References

* 🔗 [Ansible Official Documentation](https://docs.ansible.com/)
* 🔗 [Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)

---

