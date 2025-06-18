
---

# Day 7 Learning Notes: Ansible + AWS EC2 Automation & Conditionals

---

## Overview

This session covers how to:

* Create an AWS IAM user for Ansible with programmatic access.
* Use **Ansible Vault** to securely store AWS credentials.
* Automate EC2 instance creation in AWS using the `amazon.aws.ec2_instance` module.
* Avoid duplicate instances using loops and unique instance names.
* Setup passwordless SSH authentication to EC2 instances.
* Use Ansible conditionals to selectively shut down Ubuntu instances.
* Gather and filter system facts using Ansible facts and debug module.
* Run simple commands on remote hosts and capture output.

---

## Task 1: AWS IAM User & Credentials Vaulting

* Create an IAM user (`ansible-user`) in AWS with `AmazonEC2FullAccess`.
* Generate **Access Key** and **Secret Access Key** for this user.
* Store these credentials securely in an encrypted vault file:

  ```bash
  ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
  ```
* Example vault file:

  ```yaml
  ec2_access_key: "AKIAXXXXXX"
  ec2_secret_key: "xxxxxxxxxxxx"
  ```
* Use these variables in your playbook for AWS authentication.

---

## Task 2: Automate EC2 Instance Creation with Loops

**Initial (Non-standard) approach:**

Copy-pasting EC2 creation tasks multiple times:

```yaml
- name: create EC2 instance 1
  amazon.aws.ec2_instance:
    ...
- name: create EC2 instance 2
  amazon.aws.ec2_instance:
    ...
- name: create EC2 instance 3
  amazon.aws.ec2_instance:
    ...
```

**Problems:**

* Duplicate code.
* May create fewer instances than expected due to idempotency (if all parameters identical).

---

### Improved Looping Approach

Use `loop` with just AMI IDs:

```yaml
- name: create an EC2 instance
  amazon.aws.ec2_instance:
    name: "ansible-instance"
    ...
    image_id: "{{ item }}"
  loop:
    - "ami-0953476d60561c955"   # Amazon Linux
    - "ami-0953476d60561c955"
    - "ami-084568db4383264d4"   # Ubuntu Linux
```

**Note:** This creates only **2** instances because instance name is the same (`ansible-instance`).

---

### Best Practice: Loop with Unique Instance Names

```yaml
- name: create an EC2 instance
  amazon.aws.ec2_instance:
    name: "{{ item.name }}"
    ...
    image_id: "{{ item.image }}"
  loop:
    - { image: "ami-0af9569868786b23a", name: "managed-node-amazon-1" }
    - { image: "ami-0af9569868786b23a", name: "managed-node-amazon-2" }
    - { image: "ami-0e35ddab05955cf57", name: "managed-node-ubuntu" }
```

**Result:** Creates 3 unique instances as each has a unique `name` and `image_id`.

---

### Notes:

* Make sure to use AMI IDs valid for your AWS region.
* Use ansible-vault encrypted variables for AWS keys in playbooks.
* Run playbook:

  ```bash
  ansible-playbook ec2_create.yaml --vault-password-file vault.pass
  ```

---

## Task 3: Passwordless SSH Setup to EC2 Instances

* Use `ssh-copy-id` with private key file to copy your public key to EC2 instances:

```bash
ssh-copy-id -f -i /path/to/sample.pem ubuntu@<EC2_IP>
ssh-copy-id -f -i /path/to/sample.pem ec2-user@<EC2_IP>
```

* This allows passwordless SSH login for Ansible connection.

---

## Task 4: Automate Shutdown of Ubuntu Instances Only (Using Conditionals)

* Inventory example:

```ini
ec2-user@3.109.139.180
ubuntu@65.2.189.180
ec2-user@3.111.213.87
```

* Playbook snippet:

```yaml
- hosts: all
  become: true

  tasks:
    - name: shutdown only ubuntu instances
      ansible.builtin.command: /sbin/shutdown -t now
      when: ansible_facts['os_family'] == 'Debian'
```

**Explanation:**

* `ansible_facts['os_family']` is gathered automatically.
* `when` condition ensures the shutdown runs only on Ubuntu/Debian family hosts.
* Non-Ubuntu hosts are skipped.

---

## Task 5: Using Ansible Facts

* Facts are automatically gathered system information about each host.

Example playbook to **print all facts**:

```yaml
- hosts: all
  become: true

  tasks:
    - name: print all ansible gathered facts
      ansible.builtin.debug:
        var: ansible_facts
```

---

### Filtering Specific Facts (e.g., `os_family` and `pkg_mgr`)

```yaml
- hosts: all
  become: true

  tasks:
    - name: print only os_family and pkg_mgr facts
      ansible.builtin.debug:
        msg:
          os_family: "{{ ansible_facts['os_family'] }}"
          pkg_mgr: "{{ ansible_facts['pkg_mgr'] }}"
```

---

## Task 6: Running Commands and Capturing Output

Example: Print OS release info

```yaml
- name: Print OS version on all hosts
  hosts: all
  become: true

  tasks:
    - name: print os version
      command: cat /etc/os-release
      register: os_release

    - name: show os version output
      debug:
        var: os_release.stdout
```

Example: Print SSHD service status

```yaml
- name: Print sshd status on all hosts
  hosts: all
  become: true

  tasks:
    - name: sshd status
      command: systemctl status sshd
      register: status

    - name: show sshd status output
      debug:
        var: status.stdout
```

---

## Additional Notes:

* Use `become: true` to run tasks with elevated privileges (sudo).
* Always check your region and AMI IDs for EC2 creation.
* Ansible modules like `amazon.aws.ec2_instance` are idempotent — they won't recreate existing resources with same parameters.
* Use vault for sensitive data.
* `ansible_facts` is a powerful dictionary with OS, hardware, network, and system info.
* Use `when` conditionals for selective task execution.
* Debug module helps you see variables and outputs during playbook runs.

---

## Suggested Day7 README Main Points

* Created IAM user with EC2 full access and secured keys using ansible-vault.
* Automated EC2 instance provisioning with `amazon.aws.ec2_instance` module using loops and unique naming.
* Setup passwordless SSH login to EC2 instances for smooth Ansible runs.
* Used Ansible conditionals (`when`) to shutdown Ubuntu instances selectively.
* Gathered system facts and filtered specific details using the debug module.
* Executed shell commands on remote hosts, captured output, and displayed results.
* Best practice reminders on idempotency and region-specific AMI usage.

---

If you want, I can help you format this into a **Markdown file ready for GitHub** or create a nice GitHub repo README for Day7 — just ask!
