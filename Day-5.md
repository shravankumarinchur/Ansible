Here's a detailed write-up for **Day 5 – Working with Ansible Galaxy**, covering all the points you've mentioned. You can directly add this to your GitHub repo's `README.md` or as a `day-5.md` file in a docs folder.

---

# 📘 Day 5 – Working with Ansible Galaxy

## 🌐 What is Ansible Galaxy?

**Ansible Galaxy** is a community hub where DevOps engineers share Ansible roles. You can:

* Use roles created by others
* Publish your own roles
* Share roles internally within organizations or publicly

Ansible Galaxy hosts **collections** and **roles**, but our focus here is on **roles**.

---

## 🛠️ Why Use Roles from Galaxy?

Consider this scenario:

You need to install and configure **Docker** on multiple operating systems:

* RedHat
* Debian
* Ubuntu
* Windows
* Alpine

Each OS has its own:

* Package manager
* Docker install command
* User group configurations
* Service handling (e.g., `systemctl` vs `service`)

Instead of writing all of this from scratch, you can:
✅ Search for an existing, tested role on Ansible Galaxy
✅ Use it in your playbook
✅ Save time and reduce errors

---

## 🔍 Searching for Roles on Galaxy
 
 Select appropriate role
 
 
---

## ✅ Example: Installing Docker Using a Galaxy Role

### 🔹 Step 1: Install Role

```bash
ansible-galaxy role install bsmeding.docker
```

This installs the role in:

```bash
~/.ansible/roles/bsmeding.docker
```

### 🔹 Step 2: Create Playbook

Create a playbook to use the downloaded role.

**docker-playbook.yaml**

```yaml
---
- hosts: all
  become: true
  roles:
    - bsmeding.docker
```

### 🔹 Step 3: Run the Playbook

```bash
ansible-playbook -i inventory.ini docker-playbook.yaml
```

### ✅ Docker is now installed on managed nodes (EC2 instances):

Check with:

```bash
rpm -qa | grep -i docker
docker --version
```

The role supports multiple OSes and internally handles conditions like:

```yaml
when: ansible_os_family == 'RedHat'
```

---

## 🧑‍💻 Publishing Your Role to Galaxy

You can share your custom roles on Ansible Galaxy for others to use.

### ✅ Step-by-Step:

#### 1. Create a GitHub repo

Let’s say your role is called `test`.

```bash
cd test/
git init
git remote add origin https://github.com/<your-username>/ansible-roles.git
git add .
git commit -m "Initial commit"
git push origin main
```

> ✅ Replace `<your-username>` with your actual GitHub username.

---

#### 2. Update `meta/main.yml`

This file contains metadata about your role.

Example:

```yaml
galaxy_info:
  author: shravankumarinchur
  description: A role to install and configure Apache server
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions:
        - 8
        - 9
    - name: Ubuntu
      versions:
        - focal
        - jammy
```

---

#### 3. Link GitHub with Galaxy

* Go to [https://galaxy.ansible.com/](https://galaxy.ansible.com/)
* Log in using your GitHub account
* Go to **My Content → Add Content**
* Note your **Namespace** (usually your GitHub username)
* Create an **API Token** from Galaxy

---

#### 4. Import Your Role

```bash
ansible-galaxy role import <github-username> <repo-name> --token <your-token>
```

Example:

```bash
ansible-galaxy role import shravankumarinchur ansible-roles --token ghp_XXXXXXXXXXXXXXXXXXXX
```

You’ll get a success message once it’s published.

---

## ✅ Benefits of Galaxy Roles

* Saves development time
* Promotes reusable and modular code
* Helps manage OS-specific complexities
* Easy to share across teams or the community

---

## 📁 Recap – Directory Structure After Installing a Galaxy Role

```bash
~/.ansible/roles/bsmeding.docker/
├── defaults/
├── files/
├── handlers/
├── meta/
├── tasks/
├── templates/
├── tests/
├── vars/
```

Each folder serves a purpose:

* **tasks/**: The main automation logic (main.yml)
* **handlers/**: Tasks triggered by events (e.g., restart service)
* **vars/**: Variable definitions
* **defaults/**: Default variable values
* **files/**: Static files to be copied (e.g., index.html)
* **templates/**: Dynamic files using Jinja2 (e.g., config files with variables)
* **meta/**: Role metadata
* **tests/**: Test playbooks for role validation

---

## ✅ Commands Reference

```bash
# Install a role
ansible-galaxy role install <role_name>

# List installed roles
ansible-galaxy role list

# Remove a role
ansible-galaxy role remove <role_name>

# Import your own GitHub-hosted role
ansible-galaxy role import <github-username> <repo-name> --token <your-token>
```

---

## 🧠 Summary

* Ansible Galaxy is a hub for sharing reusable roles.
* Roles help modularize large playbooks.
* You can use roles from the community or publish your own.
* Galaxy simplifies complex setups like Docker installation across multiple OSes.

---

Let me know if you'd like this exported as a Markdown file or styled specifically for GitHub.
