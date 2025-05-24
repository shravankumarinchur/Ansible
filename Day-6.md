
---

# 📘 Day 7: Working with Ansible Collections, AWS EC2 creation Automation & Variable Precedence

## 🔹 Why Collections?

Until now, we have used Ansible to communicate with managed nodes (like AWS EC2 VMs) using SSH to install and configure software. But what if we want to **interact with APIs**, like AWS, Azure, or GCP, to **provision infrastructure**?

That’s where **Ansible Collections** come in. Collections are a packaging format that includes roles, modules, and plugins for specific domains like cloud providers, monitoring tools, etc.

## 📦 What is a Collection?

* Built-in modules like `apt`, `yum`, `copy` come with Ansible by default.
* To use third-party modules (e.g., AWS EC2), you must install their collection.
* A **collection is a namespace** that contains multiple resources such as:

  * Roles
  * Modules
  * Plugins
  * Documentation

Example:

```bash
ansible-galaxy collection install amazon.aws --force
```

## 🧪 Install AWS Collection + Dependencies

1. Install the AWS collection:

   ```bash
   ansible-galaxy collection install amazon.aws
   ```
2. Install Python libraries (on the control node):

   ```bash
   pip install boto3 botocore
   ```

## 🚀 Launch an EC2 Instance Using Ansible

Use `amazon.aws.ec2_instance` module (from the installed collection).

### Playbook: `ec2_create.yaml`

```yaml
- hosts: localhost
  connection: local
  roles:
    - ec2
```

### Role Directory Structure

Initialize your role:

```bash
ansible-galaxy role init ec2
```

### Task: `roles/ec2/tasks/main.yml`

```yaml
- name: start an instance with a public IP address
  amazon.aws.ec2_instance:
    name: "ansible-instance"
    instance_type: "{{ type }}"
    image_id: ami-04b70fa74e45c3917
    region: us-east-1
    security_group: default
    aws_access_key: "{{ aws_access_key }}"
    aws_secret_key: "{{ aws_secret_key }}"
    network:
      assign_public_ip: true
```

## 🔐 Using Ansible Vault to Secure AWS Credentials

We **shouldn't** hard-code sensitive credentials in playbooks.

### Step 1: Create Vault Password File

```bash
openssl rand -base64 2048 > vault.pass
```

### Step 2: Create a Vault-Encrypted Secrets File

```bash
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```

Add content:

```yaml
aws_access_key: YOUR_ACCESS_KEY
aws_secret_key: YOUR_SECRET_KEY
```

### Step 3: Run Playbook Using Vault

```bash
ansible-playbook ec2_create.yaml --vault-password-file vault.pass
```

---

## 🔄 Fix Python/Boto3 Version Errors

Ensure `boto3` and `botocore` are compatible with your Ansible collection:

1. Install pip for Python 3.12:

```bash
/usr/bin/python3.12 get-pip.py --user
```

2. Install boto3 and botocore:

```bash
/usr/bin/python3.12 -m pip install boto3 botocore --user
```

---

## 📊 Understanding Variable Precedence in Ansible

There are 22+ places where Ansible can read variables. Here's a practical subset with real examples:

### 🥇 1. Defaults (Lowest Precedence)

Defined in `roles/ec2/defaults/main.yml`

```yaml
type: t2.micro
```

Used in task:

```yaml
instance_type: "{{ type }}"
```

### 🥈 2. Vars (Higher Precedence)

Defined in `roles/ec2/vars/main.yml`

```yaml
type: t2.medium
```

Overrides default.

### 🥉 3. Extra Vars (Highest Precedence)

```bash
ansible-playbook ec2_create.yaml -e type=t2.large --vault-password-file vault.pass
```

### 🔄 4. Inline `vars` in Tasks

```yaml
- name: Create instance
  vars:
    type: t2.micro
  amazon.aws.ec2_instance:
    instance_type: "{{ type }}"
```

### 🎯 5. Group Variables

Used when you want variables for a group of hosts.

Inventory:

```ini
[app]
app1
app2

[db]
db1
```

Group vars:

```bash
mkdir group_vars
```

Create `group_vars/app.yml`:

```yaml
name: shravan
```

---

## ✅ Summary

* Learnt what Ansible Collections are and how to use them.
* Used the `amazon.aws` collection to provision an EC2 instance.
* Secured AWS credentials with Ansible Vault.
* Explored 5 powerful ways to manage variables in Ansible.
* Installed compatible Python versions and modules to resolve boto3/botocore issues.

---


