
---

## 🧠 Ansible Task Execution & Error Handling – Day 8 Notes

### 🔁 **Default Task Execution Flow**

* Ansible executes **tasks sequentially** (top to bottom).
* Each task runs on **all hosts in the inventory** before moving to the next task.
* **If a task fails on a host**, the rest of the tasks are **skipped for that host** by default.

---

### ❗**Why This Can Be a Problem**

Example: You want to install Docker on 10 VMs only if:

1. `openssl` and `openssh` (if present) are up-to-date
2. Docker is not already installed

⚠️ If any host **doesn’t have openssl/openssh**, the yum task will fail → further tasks are skipped for that host.

---

### ✅ **Solution: Use `ignore_errors: yes`**

Allows playbook to **continue even if the task fails** on a host.

```yaml
- name: Verify openssl and openssh are latest
  ansible.builtin.yum:
    name: "{{ item }}"
    state: latest
  loop:
    - openssh
    - openssl
  ignore_errors: yes
```

---

### 📌 **Conditionally Installing Docker**

Check if Docker is installed using `command`, then install only if not found.

```yaml
- name: Verify if Docker is installed
  ansible.builtin.command: docker --version
  register: output
  ignore_errors: yes

- name: Install Docker if not installed
  ansible.builtin.yum:
    name: docker
    state: present
  when: output.failed
```

✅ This ensures Docker is only installed **if it’s missing**, even if earlier tasks failed.

---

### 🔍 **Inspect Output**

Use `debug` to see what's inside the registered variable:

```yaml
- ansible.builtin.debug:
    var: output
```

---

### ⚙️ **Running the Playbook**

```bash
ansible-playbook -i test_inventory.ini docker_setup.yaml
```

---

### 🧠 Bonus: **Ignore Specific Errors Only (Fine-Grained Control)**

Use `failed_when` with conditions based on `stderr`:

```yaml
- name: Check if a file exists and fail only if it does
  ansible.builtin.command: ls /tmp/this_should_not_be_here
  register: result
  failed_when:
    - result.rc == 0
    - '"No such" not in result.stderr'
```

✅ This task only fails **if the file exists**, not just because `ls` fails.

---

### ✅ Summary

| Concept              | Description                                                       |
| -------------------- | ----------------------------------------------------------------- |
| `ignore_errors: yes` | Let playbook continue on task failure                             |
| `register` + `when`  | Run conditional tasks based on previous result                    |
| `failed_when`        | Fine control over when a task is marked as failed                 |
| Task order           | Tasks run sequentially; hosts failing a task skip remaining tasks |

---

