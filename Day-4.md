
---

## 🗓️ Day 04 – Ansible Roles (Modular Playbook Design)

---

### 🔹 Why Roles?

Large Ansible playbooks (e.g., 2000+ lines with 40+ tasks and dozens of variables) become hard to maintain.
**Roles** help break down a complex playbook into smaller, logical pieces. This improves:

* 🔍 Readability
* 🧩 Modularity
* ♻️ Reusability
* 📦 Shareability

---

### 🔹 What Are Roles?

A **Role** is a structured format for organizing Ansible content — variables, tasks, files, templates, handlers, etc.

Using roles:

* Keeps playbooks clean and modular.
* Makes it easier to test and maintain.
* Allows sharing and reuse (e.g., via [Ansible Galaxy](https://galaxy.ansible.com/)).

---

### 🔹 Creating a Role

```bash
ansible-galaxy role init <role-name>
```

Example:

```bash
ansible-galaxy role init test
```

Structure created:

```bash
test/
├── defaults/
├── files/
├── handlers/
├── meta/
├── tasks/
├── templates/
├── tests/
├── vars/
└── README.md
```

---

## 🧩 Components of a Role (with Examples)

---

### 📌 1. `tasks/` – Define What to Do

This is the **heart of a role**, where the main instructions go.

📄 `tasks/main.yml`:

```yaml
---
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: latest

- name: Deploy website
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/
    mode: '0644'
    owner: root
    group: root

- name: Restart Apache
  ansible.builtin.service:
    name: httpd
    state: restarted
```

---

### 📌 2. `files/` – Store Static Files

Use this folder to store **files that need to be copied as-is** to the remote hosts.

📄 `files/index.html`:

```html
<!DOCTYPE html>
<html>
  <head><title>Welcome</title></head>
  <body><h1>Deployed by Ansible</h1></body>
</html>
```

✅ Used in task:

```yaml
src: files/index.html
```

---

### 📌 3. `templates/` – Dynamic File Generation (Jinja2)

Templates use **Jinja2** syntax for injecting variables into configuration files.

📄 `templates/config.j2`:

```ini
[webserver]
server_name = {{ server_name }}
listen_port = {{ port }}
```

📄 Example Task in `tasks/main.yml`:

```yaml
- name: Generate Apache config
  ansible.builtin.template:
    src: config.j2
    dest: /etc/httpd/conf.d/custom.conf
  notify:
    - Restart Apache
```

> 📌 Use `vars/` or `defaults/` to define `server_name` and `port`.

---

### 📌 4. `handlers/` – Run on Change Notification

Handlers are tasks triggered by `notify:` from other tasks, typically for restarting services.

📄 `handlers/main.yml`:

```yaml
---
- name: Restart Apache
  ansible.builtin.service:
    name: httpd
    state: restarted
```

📄 Called in a task:

```yaml
notify:
  - Restart Apache
```

---

### 📌 5. `defaults/` – Default Variables (Lowest Precedence)

Default values for variables that can be overridden by `vars/` or command line.

📄 `defaults/main.yml`:

```yaml
server_name: "example.com"
port: 80
```

---

### 📌 6. `vars/` – Variables (Higher Precedence)

Same as defaults, but overrides them. Used when you need to hard-code values.

📄 `vars/main.yml`:

```yaml
port: 8080  # Will override the value in defaults
```

---

### 📌 7. `meta/` – Metadata for the Role

Metadata such as author name, license, dependencies, and more.

📄 `meta/main.yml`:

```yaml
---
galaxy_info:
  author: Your Name
  description: Deploy Apache web server
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions:
        - 8
```

---

### 📌 8. `tests/` – Optional Tests

You can include example playbooks to test your role here.

---

## 🛠️ Main Playbook Using the Role

📄 `install_http.yaml`:

```yaml
---
- hosts: all
  become: true
  roles:
    - test   # Refers to the 'test' role directory
```

📄 `inventory.ini`:

```ini
[web]
192.168.1.100
```

---

### ✅ Run It

```bash
ansible-playbook -i inventory.ini install_http.yaml
```

---

## 💡 Ansible is Idempotent

Ansible won’t repeat tasks that are already completed i.e if the desired state already exists, it won’t re-apply the task.

Example:

* If `index.html` is already present and unchanged, Ansible will skip the `copy` task.
* Output: `changed: 0`

You can test this by deleting the file and re-running the playbook:

```bash
sudo rm /var/www/html/index.html
ansible-playbook -i inventory.ini install_http.yaml
```

---

## 🚀 Benefits of Using Roles

| Feature         | Benefit                                                 |
| --------------- | ------------------------------------------------------- |
| ✅ Clean Code    | Structured and easy to maintain                         |
| ♻️ Reusable     | Reuse across multiple playbooks                         |
| 📚 Shareable    | Upload to [Ansible Galaxy](https://galaxy.ansible.com/) |
| 🧪 Testable     | Easier to write and run unit tests                      |
| 🌐 Discoverable | Use community-contributed roles from Galaxy             |
| 🛒 Marketplace  | Like DockerHub for Ansible roles                        |

---

## 🧠 Real Use Cases for Roles

* Kubernetes setup (control plane, kubelet, worker nodes)
* Setting up Oracle DB
