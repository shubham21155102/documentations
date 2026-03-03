# 📦 Ansible

> Frequently used Ansible commands for configuration management and automation.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Inventory](#inventory)
- [Ad-Hoc Commands](#ad-hoc-commands)
- [Playbooks](#playbooks)
- [Roles](#roles)
- [Vault (Secrets)](#vault-secrets)
- [Galaxy](#galaxy)
- [Variables & Facts](#variables--facts)
- [Common Playbook Patterns](#common-playbook-patterns)

---

## Installation

```bash
# Install Ansible (Ubuntu)
sudo apt update
sudo apt install -y ansible

# Install via pip
pip install ansible

# Verify
ansible --version

# Install additional modules
ansible-galaxy collection install community.general
```

---

## Inventory

```bash
# Default inventory: /etc/ansible/hosts

# Test connectivity to all hosts
ansible all -m ping
ansible all -m ping -i inventory.ini

# List hosts in inventory
ansible all --list-hosts -i inventory.ini
ansible webservers --list-hosts

# Inventory formats:

# INI style (inventory.ini):
# [webservers]
# web1 ansible_host=192.168.1.10
# web2 ansible_host=192.168.1.11
#
# [dbservers]
# db1 ansible_host=192.168.1.20
#
# [webservers:vars]
# ansible_user=ubuntu
# ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**YAML inventory (`inventory.yml`):**

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
      vars:
        ansible_user: ubuntu
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
```

---

## Ad-Hoc Commands

```bash
# Ping all hosts
ansible all -m ping

# Run shell command
ansible webservers -m shell -a "uptime"
ansible webservers -m command -a "hostname"

# Copy a file
ansible webservers -m copy -a "src=./file.conf dest=/etc/app/file.conf"

# Install a package
ansible webservers -m apt -a "name=nginx state=present" --become

# Start a service
ansible webservers -m service -a "name=nginx state=started enabled=yes" --become

# Gather facts
ansible web1 -m setup
ansible web1 -m setup -a "filter=ansible_distribution*"

# Reboot hosts
ansible webservers -m reboot --become

# Create a user
ansible all -m user -a "name=devops state=present groups=sudo" --become

# Transfer files
ansible webservers -m fetch -a "src=/var/log/app.log dest=./logs/"
```

---

## Playbooks

```bash
# Run a playbook
ansible-playbook site.yml
ansible-playbook site.yml -i inventory.ini
ansible-playbook site.yml -l webservers    # limit to group

# Dry-run (check mode)
ansible-playbook site.yml --check
ansible-playbook site.yml --check --diff

# Verbose output
ansible-playbook site.yml -v
ansible-playbook site.yml -vvv

# Pass extra variables
ansible-playbook site.yml -e "env=production version=1.2"
ansible-playbook site.yml -e @vars.yml

# Ask for sudo password
ansible-playbook site.yml --ask-become-pass

# Ask for SSH password
ansible-playbook site.yml --ask-pass

# Start at specific task
ansible-playbook site.yml --start-at-task="Install packages"

# Step through tasks interactively
ansible-playbook site.yml --step

# List tasks
ansible-playbook site.yml --list-tasks

# List hosts
ansible-playbook site.yml --list-hosts

# Tags
ansible-playbook site.yml --tags "install,configure"
ansible-playbook site.yml --skip-tags "deploy"
```

---

## Roles

```bash
# Create role structure
ansible-galaxy role init myrole

# Role structure:
# myrole/
# ├── defaults/main.yml    # default variables
# ├── vars/main.yml        # role variables
# ├── tasks/main.yml       # tasks
# ├── handlers/main.yml    # handlers
# ├── templates/           # Jinja2 templates
# ├── files/               # static files
# ├── meta/main.yml        # role metadata
# └── README.md

# Use role in playbook
# - hosts: webservers
#   roles:
#     - myrole

# List installed roles
ansible-galaxy role list

# Remove a role
ansible-galaxy role remove namespace.rolename
```

---

## Vault (Secrets)

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt vars.yml

# Change vault password
ansible-vault rekey secrets.yml

# Encrypt a string (inline)
ansible-vault encrypt_string 'mysecretpassword' --name 'db_password'

# Run playbook with vault password
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

---

## Galaxy

```bash
# Install a role
ansible-galaxy install geerlingguy.nginx

# Install a collection
ansible-galaxy collection install community.postgresql
ansible-galaxy collection install amazon.aws

# Install from requirements file
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install -r requirements.yml

# List installed roles/collections
ansible-galaxy role list
ansible-galaxy collection list

# Search for roles
ansible-galaxy search nginx --author geerlingguy
```

**`requirements.yml` example:**

```yaml
roles:
  - name: geerlingguy.nginx
    version: 3.1.0
  - src: https://github.com/example/role.git
    version: main

collections:
  - name: community.general
    version: ">=5.0.0"
  - name: amazon.aws
```

---

## Variables & Facts

```bash
# Display all facts for a host
ansible web1 -m setup

# Display specific facts
ansible web1 -m setup -a "filter=ansible_os_family"
ansible web1 -m setup -a "filter=ansible_default_ipv4"

# Common fact variables:
# ansible_hostname
# ansible_os_family      (Debian, RedHat)
# ansible_distribution   (Ubuntu, CentOS)
# ansible_architecture
# ansible_default_ipv4.address
# ansible_memtotal_mb
# ansible_processor_vcpus
```

---

## Common Playbook Patterns

**Basic playbook:**

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars:
    app_port: 8080

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**Loop example:**

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git
    - vim
```

**Conditional:**

```yaml
- name: Install on Debian only
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"
```

---

[← Back to Home](../README.md)
