

---

# 📘 Ansible Tomcat Role – Quick Notes

---

## 1️⃣ Project Objective

Deploy, manage, and uninstall Apache Tomcat using a **role-based, production-ready Ansible structure**.

Key goals:

* Idempotent
* Version-driven
* Modular
* OS-compatible
* Clean install/uninstall separation
* No reserved variable conflicts

---

# 2️⃣ Role Directory Structure

```
roles/
└── tomcat-role/
    ├── defaults/main.yml
    ├── handlers/main.yml
    ├── tasks/
    │   ├── main.yml
    │   ├── install.yml
    │   ├── uninstall.yml
    │   ├── install-java.yml
    │   ├── install-tomcat.yml
    │   ├── deploy-app.yml
    │   └── configure-user.yml
```

---

# 3️⃣ defaults/main.yml (Configuration Layer)

Purpose:

* Define configurable variables
* Allow CLI or inventory override

Important Variables:

```yaml
tomcat_action: install
tomcat_version: 9.0.115
tomcat_user: tomcat
tomcat_group: tomcat
```

Derived Variables:

```yaml
tomcat_install_dir: "/opt/apache-tomcat-{{ tomcat_version }}"
tomcat_symlink: /opt/tomcat
```

Design Principle:

> Change version in one place → Entire deployment updates.

---

# 4️⃣ Execution Controller – tasks/main.yml

Acts as a switch.

```yaml
- include_tasks: install.yml
  when: tomcat_action == 'install'

- include_tasks: uninstall.yml
  when: tomcat_action == 'uninstall'
```

Prevents:

* Undefined variable errors
* Repeated condition blocks

---

# 5️⃣ Installation Flow (install.yml)

Execution order:

1. Configure user
2. Install Java
3. Install Tomcat
4. Deploy application

Modular → Easy maintenance

---

# 6️⃣ User & Security Setup

Creates system user:

```yaml
system: yes
shell: /bin/false
create_home: no
```

Why?

* Security best practice
* Non-login service account

---

# 7️⃣ Java Installation Strategy

Dynamic package selection:

```yaml
java_package:
  Debian: openjdk-11-jdk
  RedHat: java-11-openjdk
```

Uses:

```yaml
{{ java_package[ansible_os_family] }}
```

Result:

* Cross-platform compatibility

---

# 8️⃣ Tomcat Installation Process

Steps:

1. Download tar file
2. Extract to `/opt`
3. Create symlink `/opt/tomcat`
4. Apply ownership

Why symlink?

```
/opt/tomcat → stable path
/opt/apache-tomcat-9.0.115 → version directory
```

Allows:

* Easy upgrades
* Minimal path changes

---

# 9️⃣ Application Deployment

Deploy WAR file:

```yaml
get_url:
  dest: "{{ tomcat_symlink }}/webapps/sample.war"
```

Demonstrates:

* Automated CI-style deployment

---

# 🔟 Handlers

```yaml
- name: Restart tomcat
  service:
    name: tomcat
    state: restarted
```

Purpose:

* Restart only when required
* Avoid unnecessary downtime

---

# 1️⃣1️⃣ Uninstallation Flow

Removes:

* Service
* Systemd file
* Installation directory
* Symlink
* User
* Group

Safe with:

```yaml
ignore_errors: yes
```

Ensures idempotent removal.

---

# 1️⃣2️⃣ Playbook Layer

```yaml
---
- name: Deploy Tomcat using role
  hosts: WebServer
  become: true

  roles:
    - tomcat-role
```

Responsibilities:

* Target inventory group
* Enable privilege escalation
* Call role

Role contains logic.
Playbook orchestrates.

---

# 1️⃣3️⃣ Execution Commands

Install:

```
ansible-playbook playbook.yml
```

Uninstall:

```
ansible-playbook playbook.yml -e "tomcat_action=uninstall"
```

