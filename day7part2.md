Got it! Here’s the **full detailed notes** including **all playbook codes** you shared, neatly organized and ready to be added to your GitHub Day7 folder/README.

---

# Day 7 Learning Notes: Ansible + AWS EC2 Automation & Conditionals

---

## Overview

Learned how to:

* Create AWS IAM user & securely store credentials.
* Automate EC2 instance creation using Ansible with loops.
* Use unique instance names to avoid duplication.
* Setup passwordless SSH.
* Selectively shutdown Ubuntu instances with conditionals.
* Gather and filter Ansible facts.
* Run commands and debug outputs on remote hosts.

---

## Task 1: AWS IAM User & Vault Credentials

* Create IAM user `ansible-user` with `AmazonEC2FullAccess`.
* Store AWS credentials securely in vault:

```bash
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```

Example `pass.yml`:

```yaml
ec2_access_key: "xxxxxxxxxxx"
ec2_secret_key: "xxxxxxxxxx"
```

---

## Task 2: EC2 Instance Creation Playbooks

### Initial (non-standard) way - repetitive code

```yaml
---
- hosts: localhost
  connection: local
  tasks:
  - name: create an EC2 instance 1
    amazon.aws.ec2_instance:
      name: "ansible-instance 1"
      key_name: "newkeypair"
      instance_type: t2.micro
      security_group: default
      region: ap-south-1
      aws_access_key: "{{ec2_access_key}}"
      aws_secret_key: "{{ec2_secret_key}}"
      network:
        assign_public_ip: true
      image_id: ami-04b70fa74e45c3917 

  - name: create an EC2 instance 2
    amazon.aws.ec2_instance:
      name: "ansible-instance 2"
      key_name: "newkeypair"
      instance_type: t2.micro
      security_group: default
      region: ap-south-1
      aws_access_key: "{{ec2_access_key}}"
      aws_secret_key: "{{ec2_secret_key}}"
      network:
        assign_public_ip: true
      image_id: ami-04b70fa74e45c3917

  - name: create an EC2 instance 3
    amazon.aws.ec2_instance:
      name: "ansible-instance 3"
      key_name: "newkeypair"
      instance_type: t2.micro
      security_group: default
      region: ap-south-1
      aws_access_key: "{{ec2_access_key}}"
      aws_secret_key: "{{ec2_secret_key}}"
      network:
        assign_public_ip: true
      image_id: ami-04b70fa74e45c3917
```

---

### Improved with loop but same name (only 2 instances created)

```yaml
---
- hosts: localhost
  connection: local
  tasks:
  - name: create an EC2 instance
    amazon.aws.ec2_instance:
      name: "ansible-instance"
      key_name: "newkeypair"
      instance_type: t2.micro
      security_group: default
      region: ap-south-1
      aws_access_key: "{{ec2_access_key}}"
      aws_secret_key: "{{ec2_secret_key}}"
      network:
        assign_public_ip: true
      image_id: "{{ item }}"
    loop:
      - "ami-0953476d60561c955"  # Amazon Linux
      - "ami-0953476d60561c955"  # Amazon Linux
      - "ami-084568db4383264d4"  # Ubuntu Linux
```

---

### Best practice: loop with unique instance names

```yaml
---
- hosts: localhost
  connection: local
  tasks:
  - name: create EC2 instances with unique names
    amazon.aws.ec2_instance:
      name: "{{ item.name }}"
      key_name: "ski-aws-keys"
      instance_type: t2.micro
      security_group: default
      region: ap-south-1
      aws_access_key: "{{ ec2_access_key }}"
      aws_secret_key: "{{ ec2_secret_key }}"
      network:
        assign_public_ip: true
      image_id: "{{ item.image }}"
    loop:
      - { image: "ami-0af9569868786b23a", name: "managed-node-amazon-1" }  # Amazon Linux
      - { image: "ami-0af9569868786b23a", name: "managed-node-amazon-2" }  # Amazon Linux
      - { image: "ami-0e35ddab05955cf57", name: "managed-node-ubuntu" }   # Ubuntu Linux
```

Run with:

```bash
ansible-playbook ec2_create.yaml --vault-password-file vault.pass
```

---

## Task 3: Passwordless SSH Setup

```bash
ssh-copy-id -f -i /path/to/sample.pem ubuntu@<ubuntu-ip>
ssh-copy-id -f -i /path/to/sample.pem ec2-user@<amazon-linux-ip>
```

---

## Task 4: Shutdown Only Ubuntu Instances (Using Conditionals)

**Inventory (`inventory2.ini`):**

```ini
ec2-user@3.109.139.180
ubuntu@65.2.189.180
ec2-user@3.111.213.87
```

**Playbook:**

```yaml
---
- hosts: all
  become: true

  tasks:
   - name: shutdown only ubuntu instances
     ansible.builtin.command: /sbin/shutdown -t now
     when: ansible_facts['os_family'] == 'Debian'
```

---

## Task 5: Gathering Ansible Facts & Filtering

### Print all facts

```yaml
---
- hosts: all
  become: true

  tasks:
    - name: print all ansible gathered facts
      ansible.builtin.debug:
        var: ansible_facts
```

---

### Print only `os_family` and `pkg_mgr`

```yaml
---
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

## Task 6: Running Commands & Showing Output

### Print OS version from `/etc/os-release`

```yaml
---
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

---

### Show sshd service status

```yaml
---
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

## Summary & Best Practices

* Use Ansible Vault for sensitive AWS keys.
* Use loops with unique instance names to create multiple EC2 instances.
* Always confirm AMI IDs match your AWS region.
* Use `when` conditionals to target tasks selectively (e.g., Ubuntu shutdown).
* Gather facts automatically with Ansible and filter output for clarity.
* Use debug module to verify variable contents and command outputs.
* Use `become: true` for tasks requiring sudo privileges.
* Use passwordless SSH for smooth automation.

---

If you want, I can help format this as a **README.md** with sections and bullet points for GitHub. Would you like that?
