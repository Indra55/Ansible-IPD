# IPD - Ansible Deployment Guide

> **This document serves as both documentation AND a prompt you can give to an AI assistant to implement the Ansible automation correctly.**

---

## 📋 PROJECT OVERVIEW

### What This Project Does

**IPD** is a **Federated Learning** system with:
- **Controller Node** (main machine): Orchestrates training, distributes data to edge nodes, aggregates model weights
- **Edge Nodes** (remote PCs): Receive training data, train local ML models, send weights back

### Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Controller/Edge Logic | Go | 1.22+ |
| ML Training | Python + PyTorch | Python 3.12, PyTorch 2.0+ |
| Communication | WebRTC + WebSocket | pion/webrtc, gorilla/websocket |

---

## 📁 ANSIBLE PROJECT STRUCTURE

The **simple approach** - keep entire project in one place:

```
edge-automation/
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   └── hosts.ini               # Your nodes (IPs)
├── playbooks/
│   ├── setup.yml               # Install Python 3.12 + Go 1.22
│   ├── deploy.yml              # Copy project + build + create services
│   ├── start.yml               # Start all services
│   ├── stop.yml                # Stop all services
│   ├── update.yml              # Update code + restart
│   └── status.yml              # Check health of all nodes
├── group_vars/
│   ├── all.yml                 # Common variables
│   ├── controllers.yml         # Controller-specific vars
│   └── edges.yml               # Edge-specific vars
├── templates/
│   ├── controller.service.j2   # Systemd template for Go controller
│   ├── api_server.service.j2   # Systemd template for Flask API
│   ├── edge.service.j2         # Systemd template for Go edge
│   └── ml_trainer.service.j2   # Systemd template for Python ML
└── project/                    # ← YOUR ENTIRE IPD PROJECT HERE
    ├── main/
    ├── edge/
    ├── algorithm/
    ├── networking/
    ├── server/
    ├── go.mod
    ├── go.sum
    └── ...
```

### How It Works

1. **You copy/symlink your IPD project** into `project/` directory
2. **Ansible syncs the entire `project/` folder** to each node
3. **Based on node role**, Ansible builds the right binary and starts the right services

---

## 🤖 AI PROMPT - IMPLEMENT ANSIBLE AUTOMATION

> **Copy everything between the START and END markers and give it to your AI assistant**

---

### PROMPT START ###

Create a complete Ansible automation for deploying a Federated Learning system. Here are the specifications:

#### PROJECT STRUCTURE

```
edge-automation/
├── ansible.cfg
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── setup.yml           # Install Python 3.12 + Go 1.22
│   ├── deploy.yml          # Sync project + build + setup services
│   ├── start.yml           # Start services based on role
│   ├── stop.yml            # Stop services
│   ├── update.yml          # Update code + restart
│   └── status.yml          # Health check all nodes
├── group_vars/
│   ├── all.yml
│   ├── controllers.yml
│   └── edges.yml
├── templates/
│   ├── controller.service.j2
│   ├── api_server.service.j2
│   ├── edge.service.j2
│   └── ml_trainer.service.j2
└── project/                # The entire project will be placed here
    └── (IPD project files)
```

#### INVENTORY (inventory/hosts.ini)

```ini
[controllers]
controller1 ansible_host=192.168.1.100

[edges]
edge1 ansible_host=192.168.1.101
edge2 ansible_host=192.168.1.102
edge3 ansible_host=192.168.1.103

[all:vars]
ansible_user=edgeuser
ansible_become=true
ansible_python_interpreter=/usr/bin/python3.12
```

#### ANSIBLE.CFG

```ini
[defaults]
inventory = inventory/hosts.ini
host_key_checking = False
timeout = 30

[ssh_connection]
pipelining = True
```

#### GROUP VARIABLES

**group_vars/all.yml:**
```yaml
---
# Common settings for all nodes
app_user: edgeuser
app_group: edgeuser
app_base_dir: /opt/ipd
project_src: "{{ playbook_dir }}/../project"

# Versions
go_version: "1.22.5"
go_download_url: "https://go.dev/dl/go{{ go_version }}.linux-amd64.tar.gz"
python_version: "3.12"

# Paths
go_install_dir: /usr/local/go
venv_dir: "{{ app_base_dir }}/venv"
```

**group_vars/controllers.yml:**
```yaml
---
node_role: controller
services:
  - name: controller
    description: "IPD Controller Service"
    type: go
    binary: controller
    build_path: ./main/
    working_dir: "{{ app_base_dir }}/project"
  - name: api_server
    description: "IPD API Server"
    type: python
    script: main/api_server.py
    port: 8000
    working_dir: "{{ app_base_dir }}/project"

python_requirements:
  - flask>=3.0.0
  - torch>=2.0.0
  - numpy>=1.21.0
```

**group_vars/edges.yml:**
```yaml
---
node_role: edge
services:
  - name: edge
    description: "IPD Edge Node Service"
    type: go
    binary: edge_node
    build_path: ./edge/
    working_dir: "{{ app_base_dir }}/project"
  - name: ml_trainer
    description: "IPD ML Trainer Service"
    type: python
    script: edge/process_images.py
    port: 8765
    working_dir: "{{ app_base_dir }}/project"

# Edge requires heavy ML dependencies
python_requirements_file: "{{ app_base_dir }}/project/edge/requirements.txt"
pip_install_timeout: 1800  # 30 min for PyTorch download
```

#### PLAYBOOK: setup.yml

```yaml
---
- name: Setup Python 3.12 and Go 1.22 on all nodes
  hosts: all
  become: yes
  tasks:
    - name: Install required system packages
      apt:
        name:
          - software-properties-common
          - curl
          - git
          - build-essential
          - ca-certificates
        state: present
        update_cache: yes

    - name: Add deadsnakes PPA for Python 3.12
      apt_repository:
        repo: ppa:deadsnakes/ppa
        state: present

    - name: Install Python 3.12
      apt:
        name:
          - python3.12
          - python3.12-venv
          - python3.12-dev
        state: present

    - name: Set Python 3.12 as default alternative
      alternatives:
        name: python3
        path: /usr/bin/python3.12
        priority: 100

    - name: Check if Go is already installed
      stat:
        path: "{{ go_install_dir }}/bin/go"
      register: go_binary

    - name: Remove old Go installation if exists
      file:
        path: "{{ go_install_dir }}"
        state: absent
      when: go_binary.stat.exists

    - name: Download Go {{ go_version }}
      get_url:
        url: "{{ go_download_url }}"
        dest: /tmp/go{{ go_version }}.linux-amd64.tar.gz
        mode: '0644'

    - name: Extract Go to /usr/local
      unarchive:
        src: /tmp/go{{ go_version }}.linux-amd64.tar.gz
        dest: /usr/local
        remote_src: yes

    - name: Add Go to system PATH
      copy:
        dest: /etc/profile.d/go.sh
        content: |
          export PATH=$PATH:/usr/local/go/bin
          export GOPATH=$HOME/go
        mode: '0644'

    - name: Create app user if not exists
      user:
        name: "{{ app_user }}"
        shell: /bin/bash
        create_home: yes

    - name: Verify installations
      shell: |
        source /etc/profile.d/go.sh
        echo "Python: $(python3.12 --version)"
        echo "Go: $(go version)"
      args:
        executable: /bin/bash
      register: version_check
      changed_when: false

    - name: Display versions
      debug:
        var: version_check.stdout_lines
```

#### PLAYBOOK: deploy.yml

```yaml
---
- name: Deploy IPD project to all nodes
  hosts: all
  become: yes
  vars:
    project_dest: "{{ app_base_dir }}/project"
  tasks:
    - name: Create application directory
      file:
        path: "{{ app_base_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_group }}"
        mode: '0755'

    - name: Sync project files to node
      synchronize:
        src: "{{ project_src }}/"
        dest: "{{ project_dest }}"
        delete: yes
        rsync_opts:
          - "--exclude=venv"
          - "--exclude=__pycache__"
          - "--exclude=*.pyc"
          - "--exclude=.git"
          - "--exclude=data"
      notify: Rebuild binaries

    - name: Set project directory ownership
      file:
        path: "{{ project_dest }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_group }}"
        recurse: yes

    - name: Create Python virtual environment
      command: python3.12 -m venv {{ venv_dir }}
      args:
        creates: "{{ venv_dir }}/bin/activate"
      become_user: "{{ app_user }}"

    - name: Upgrade pip in venv
      pip:
        name: pip
        state: latest
        virtualenv: "{{ venv_dir }}"
      become_user: "{{ app_user }}"

    # For controllers - install from list
    - name: Install Python requirements (controllers)
      pip:
        name: "{{ python_requirements }}"
        virtualenv: "{{ venv_dir }}"
      become_user: "{{ app_user }}"
      when: node_role == 'controller'

    # For edges - install from requirements.txt (includes torch)
    - name: Install Python requirements (edges)
      pip:
        requirements: "{{ python_requirements_file }}"
        virtualenv: "{{ venv_dir }}"
        extra_args: "--timeout {{ pip_install_timeout }}"
      become_user: "{{ app_user }}"
      when: node_role == 'edge'
      async: 2400  # 40 minute timeout for torch
      poll: 30

    - name: Build Go binaries
      shell: |
        source /etc/profile.d/go.sh
        cd {{ project_dest }}
        go mod download
        {% for svc in services if svc.type == 'go' %}
        go build -o {{ svc.binary }} {{ svc.build_path }}
        {% endfor %}
      args:
        executable: /bin/bash
        chdir: "{{ project_dest }}"
      become_user: "{{ app_user }}"

    - name: Create systemd service files
      template:
        src: "templates/{{ item.name }}.service.j2"
        dest: "/etc/systemd/system/{{ item.name }}.service"
        mode: '0644'
      loop: "{{ services }}"
      notify: Reload systemd

  handlers:
    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Rebuild binaries
      shell: |
        source /etc/profile.d/go.sh
        cd {{ project_dest }}
        {% for svc in services if svc.type == 'go' %}
        go build -o {{ svc.binary }} {{ svc.build_path }}
        {% endfor %}
      args:
        executable: /bin/bash
      become_user: "{{ app_user }}"
```

#### PLAYBOOK: start.yml

```yaml
---
- name: Start IPD services
  hosts: all
  become: yes
  tasks:
    - name: Start and enable services
      systemd:
        name: "{{ item.name }}"
        state: started
        enabled: yes
      loop: "{{ services }}"

    - name: Wait for services to be ready
      wait_for:
        port: "{{ item.port }}"
        timeout: 30
      loop: "{{ services }}"
      when: item.port is defined
```

#### PLAYBOOK: stop.yml

```yaml
---
- name: Stop IPD services
  hosts: all
  become: yes
  tasks:
    - name: Stop services
      systemd:
        name: "{{ item.name }}"
        state: stopped
      loop: "{{ services }}"
      ignore_errors: yes
```

#### PLAYBOOK: update.yml

```yaml
---
- name: Update code and restart services
  hosts: all
  become: yes
  tasks:
    - name: Sync updated project files
      synchronize:
        src: "{{ project_src }}/"
        dest: "{{ app_base_dir }}/project"
        delete: yes
        rsync_opts:
          - "--exclude=venv"
          - "--exclude=__pycache__"
          - "--exclude=*.pyc"
          - "--exclude=.git"
          - "--exclude=data"
      register: sync_result

    - name: Rebuild Go binaries if code changed
      shell: |
        source /etc/profile.d/go.sh
        cd {{ app_base_dir }}/project
        {% for svc in services if svc.type == 'go' %}
        go build -o {{ svc.binary }} {{ svc.build_path }}
        {% endfor %}
      args:
        executable: /bin/bash
      become_user: "{{ app_user }}"
      when: sync_result.changed

    - name: Restart services if code changed
      systemd:
        name: "{{ item.name }}"
        state: restarted
      loop: "{{ services }}"
      when: sync_result.changed
```

#### PLAYBOOK: status.yml

```yaml
---
- name: Check status of all nodes
  hosts: all
  become: yes
  tasks:
    - name: Check service status
      systemd:
        name: "{{ item.name }}"
      loop: "{{ services }}"
      register: service_status

    - name: Display service status
      debug:
        msg: "{{ item.item.name }}: {{ item.status.ActiveState }}"
      loop: "{{ service_status.results }}"

    - name: Check listening ports
      shell: ss -tlnp | grep -E ':(8000|8765)' || true
      register: ports
      changed_when: false

    - name: Display listening ports
      debug:
        var: ports.stdout_lines

    - name: Check disk space
      shell: df -h / | tail -1
      register: disk
      changed_when: false

    - name: Display disk space
      debug:
        var: disk.stdout
```

#### SYSTEMD TEMPLATES

**templates/controller.service.j2:**
```ini
[Unit]
Description=IPD Controller Service
After=network.target

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_base_dir }}/project
ExecStart={{ app_base_dir }}/project/controller
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment="PATH=/usr/local/go/bin:/usr/bin"

[Install]
WantedBy=multi-user.target
```

**templates/api_server.service.j2:**
```ini
[Unit]
Description=IPD API Server
After=network.target

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_base_dir }}/project
ExecStart={{ venv_dir }}/bin/python main/api_server.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**templates/edge.service.j2:**
```ini
[Unit]
Description=IPD Edge Node Service
After=network.target ml_trainer.service

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_base_dir }}/project
ExecStart={{ app_base_dir }}/project/edge_node
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment="PATH=/usr/local/go/bin:/usr/bin"

[Install]
WantedBy=multi-user.target
```

**templates/ml_trainer.service.j2:**
```ini
[Unit]
Description=IPD ML Trainer Service
After=network.target

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_base_dir }}/project
ExecStart={{ venv_dir }}/bin/python edge/process_images.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

#### USAGE COMMANDS

After creating all files, use these commands:

```bash
# 1. Test connectivity
ansible all -m ping

# 2. Setup environments (Python 3.12 + Go 1.22)
ansible-playbook playbooks/setup.yml

# 3. Deploy project (copy, build, create services)
ansible-playbook playbooks/deploy.yml

# 4. Start all services
ansible-playbook playbooks/start.yml

# 5. Check status
ansible-playbook playbooks/status.yml

# 6. Update code after changes
ansible-playbook playbooks/update.yml

# 7. Stop all services
ansible-playbook playbooks/stop.yml

# Target specific groups
ansible-playbook playbooks/deploy.yml --limit edges
ansible-playbook playbooks/deploy.yml --limit controllers
```

### PROMPT END ###

---

## 📝 QUICK START GUIDE

### Step 1: Create the Ansible Directory

```bash
mkdir -p ~/edge-automation/{inventory,playbooks,group_vars,templates}
cd ~/edge-automation
```

### Step 2: Copy/Symlink Your Project

```bash
# Option A: Symlink (recommended for development)
ln -s /path/to/IPD ./project

# Option B: Copy
cp -r /path/to/IPD ./project
```

### Step 3: Setup Control Machine

```bash
# Install Ansible
sudo apt install -y ansible

# Generate SSH key
ssh-keygen -t ed25519

# Copy to all nodes
ssh-copy-id edgeuser@192.168.1.101
ssh-copy-id edgeuser@192.168.1.102
# ... repeat for all nodes
```

### Step 4: Create Files

Either manually create the files above, or give the AI prompt to an assistant.

### Step 5: Run Playbooks

```bash
cd ~/edge-automation
ansible-playbook playbooks/setup.yml
ansible-playbook playbooks/deploy.yml
ansible-playbook playbooks/start.yml
```

---

## 🔧 COMMON OPERATIONS

### Add a New Edge Node

1. On new node: `sudo apt install openssh-server`
2. Create user: `sudo adduser edgeuser && sudo usermod -aG sudo edgeuser`
3. Copy SSH key: `ssh-copy-id edgeuser@NEW_IP`
4. Add to `inventory/hosts.ini` under `[edges]`
5. Run:
   ```bash
   ansible-playbook playbooks/setup.yml --limit NEW_HOSTNAME
   ansible-playbook playbooks/deploy.yml --limit NEW_HOSTNAME
   ansible-playbook playbooks/start.yml --limit NEW_HOSTNAME
   ```

### Update Code

After making changes to your project:
```bash
ansible-playbook playbooks/update.yml
```

### View Logs

```bash
# All edges
ansible edges -m shell -a "journalctl -u edge -u ml_trainer -n 50 --no-pager"

# Controller
ansible controllers -m shell -a "journalctl -u controller -u api_server -n 50 --no-pager"
```

### Restart Services

```bash
ansible all -m systemd -a "name=edge state=restarted" --limit edges
```

---

## ⚠️ IMPORTANT NOTES

1. **Don't remove system Python** - Ubuntu needs it
2. **PyTorch is ~2GB** - Edge deployment will take 10-30 minutes first time
3. **Go version** - Use 1.22.x (1.24 doesn't exist yet)
4. **Project structure must match go.mod** - The module is `github.com/Omkardalvi01/IPD`

---

*Last updated: 2026-01-19*
