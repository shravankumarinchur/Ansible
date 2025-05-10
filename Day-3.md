
---

## 🗓️ Day 03 – YAML, Ansible Playbook Structure, and Apache Setup

---

### 🔹 What is YAML?

**YAML** (YAML Ain't Markup Language) is a human-readable data serialization language often used for configuration files and in applications where data is being stored or transmitted.

---
Why not plain text?

* Text files have no fixed standard, leading to inconsistent formatting.

Why YAML over JSON?

* YAML is easier to read and write for humans.

* Follows a strict syntax, reducing errors.



---

### 🔹 YAML Basics

#### ✅ Scalars (Strings, Numbers, Booleans)

```yaml
name: "John"
age: 30
active: true
```

#### ✅ List

```yaml
packages:
  - git
  - nginx
  - curl
```

#### ✅ Dictionary

```yaml
user:
  name: alice
  uid: 1010
```

#### ✅ List of Dictionaries

```yaml
users:
  - name: bob
    shell: /bin/bash
  - name: alice
    shell: /bin/zsh
```

---

### 🔹 Ansible Playbook Overview

An **Ansible Playbook** is a YAML file consisting of **one or more plays**.

#### ✅ Anatomy of a Play:

* `- hosts`: Target group of machines
* `become`: Run as sudo
* `tasks`: List of actions/modules

> Each play always begins with `-` (a dash)

---

### ✅ Example – Multiple Plays (Each Starts with `-`)

```yaml
---
- hosts: web
  become: true
  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present

- hosts: db
  become: true
  tasks:
    - name: Install MySQL
      ansible.builtin.dnf:
        name: mysql-server
        state: present
```

---

### 🔹 Apache Setup via Playbook

#### Folder Structure

```bash
ansible/
├── index.html
├── install_http.yaml
├── inventory.ini
```

#### `install_http.yaml`

```yaml
---
- hosts: all
  become: true
  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: latest

    - name: Copy HTML page
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/
        owner: root
        group: root
        mode: '0644'
```

#### Run the Playbook

```bash
ansible-playbook -i inventory.ini install_http.yaml
```

---

### 🔹 Ad-hoc Service Management

Start or stop Apache using ad-hoc commands:

```bash
ansible -i inventory.ini -u ec2-user --become -m ansible.builtin.service -a "name=httpd state=started" all
ansible -i inventory.ini -u ec2-user --become -m ansible.builtin.service -a "name=httpd state=stopped" all
```

---




### 🔹 Summary

* YAML is the language Ansible uses for playbooks and config.
* Each play begins with `-`, followed by target hosts and tasks.
* Apache setup automated using `dnf` and `copy` modules.
* Ad-hoc commands are quick one-liners for on-the-fly changes.
* Think of playbooks like scripts for managing remote servers.

---

