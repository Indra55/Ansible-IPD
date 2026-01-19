<div align="center">

# 🚀 IPD – Ansible Deployment

**Federated Learning Infrastructure | Automated DevOps**

[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)

*Deploy and manage a distributed federated learning system across multiple Linux machines with a single command.*

---

[Quick Start](#-quick-start) •
[Architecture](#-architecture) •
[Playbooks](#-playbooks) •
[Configuration](#%EF%B8%8F-configuration) •
[Troubleshooting](#-troubleshooting)

</div>

---

## 📋 Overview

**IPD** (Intelligent Processing & Distribution) is a federated learning system designed for distributed ML training across a controller–edge architecture. This repository contains all the **Ansible automation** needed to:

- 🔧 **Set up** systems with Python 3.12 and Go 1.24
- 📦 **Deploy** the complete IPD stack
- ▶️ **Start/Stop** services across all nodes  
- 🔄 **Update** code and restart automatically
- 📊 **Monitor** service health and status

---

## 🏗️ Architecture

```
                    ┌─────────────────────┐
                    │     Controller      │
                    │   ┌─────────────┐   │
                    │   │  Go Binary  │   │
                    │   │ (Orchestrator)   │
                    │   └─────────────┘   │
                    │   ┌─────────────┐   │
                    │   │  API Server │   │
                    │   │  (Python)   │   │
                    │   └─────────────┘   │
                    └──────────┬──────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
            ▼                  ▼                  ▼
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │   Edge 1    │    │   Edge 2    │    │   Edge N    │
   │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │
   │ │Go Binary│ │    │ │Go Binary│ │    │ │Go Binary│ │
   │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │
   │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │
   │ │ML Train │ │    │ │ML Train │ │    │ │ML Train │ │
   │ │(Python) │ │    │ │(Python) │ │    │ │(Python) │ │
   │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │
   └─────────────┘    └─────────────┘    └─────────────┘
```

### Node Types

| Node | Components | Responsibilities |
|------|-----------|-----------------|
| **Controller** | Go orchestrator + Python API | Coordinates FL rounds, distributes tasks, aggregates models |
| **Edge** | Go node + Python ML trainer | Performs local training, sends model updates |

---

## 🛠️ Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| **Orchestration** | Ansible | Latest |
| **Core Logic** | Go | 1.24.0 |
| **ML Training** | Python + PyTorch/TensorFlow | 3.12 |
| **Computer Vision** | Ultralytics YOLO | 8.3.x |
| **Communication** | WebRTC + WebSockets | - |
| **Service Mgmt** | systemd | - |
| **Target OS** | Ubuntu | 22.04+ |

---

## 📁 Project Structure

```bash
ipd-ansible/
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   └── hosts.ini              # Target machines
├── playbooks/
│   ├── setup.yml              # Install Python & Go
│   ├── deploy.yml             # Deploy full stack
│   ├── start.yml              # Start all services
│   ├── stop.yml               # Stop all services
│   ├── status.yml             # Check health
│   └── update.yml             # Hot update code
├── group_vars/
│   ├── all.yml                # Shared variables
│   ├── controllers.yml        # Controller config
│   └── edges.yml              # Edge config
├── requirements/
│   └── edge-requirements.txt  # ML dependencies
├── templates/
│   ├── controller.service.j2  # systemd templates
│   ├── api_server.service.j2
│   ├── edge.service.j2
│   ├── edge_start.sh.j2
│   └── ml_trainer.service.j2
└── project/                   # IPD source code (Go + Python)
```

---

## ⚡ Quick Start

### Prerequisites

- **Control machine**: Ansible 2.9+, SSH access to all nodes
- **Target nodes**: Ubuntu 22.04+, sudo privileges
- **Network**: SSH connectivity between all nodes

### 1️⃣ Configure Inventory

Edit `inventory/hosts.ini` with your machines:

```ini
[controllers]
controller1 ansible_host=192.168.1.100

[edges]
edge1 ansible_host=192.168.1.101
edge2 ansible_host=192.168.1.102
edge3 ansible_host=192.168.1.103

[all:vars]
ansible_user=your_username
ansible_become=true
ansible_python_interpreter=/usr/bin/python3
```

### 2️⃣ Setup Environment

Install Python 3.12 and Go 1.24 on all nodes:

```bash
ansible-playbook playbooks/setup.yml
```

### 3️⃣ Deploy Application

Deploy the full IPD stack:

```bash
ansible-playbook playbooks/deploy.yml
```

> ⏱️ **Note**: First deployment on edges can take **15-30 minutes** due to ML dependencies (~3GB of packages including TensorFlow, PyTorch, and Ultralytics).

### 4️⃣ Start Services

```bash
ansible-playbook playbooks/start.yml
```

### 5️⃣ Verify Status

```bash
ansible-playbook playbooks/status.yml
```

---

## 📚 Playbooks

| Playbook | Description | Example |
|----------|-------------|---------|
| `setup.yml` | Install Python 3.12, Go 1.24, create users | `ansible-playbook playbooks/setup.yml` |
| `deploy.yml` | Sync code, build binaries, install deps, create services | `ansible-playbook playbooks/deploy.yml` |
| `start.yml` | Start and enable all systemd services | `ansible-playbook playbooks/start.yml` |
| `stop.yml` | Stop all services | `ansible-playbook playbooks/stop.yml` |
| `status.yml` | Check service health, ports, disk space | `ansible-playbook playbooks/status.yml` |
| `update.yml` | Hot update: sync code, rebuild, restart | `ansible-playbook playbooks/update.yml` |

### Target Specific Hosts

```bash
# Only edges
ansible-playbook playbooks/deploy.yml --limit edges

# Single host
ansible-playbook playbooks/status.yml --limit edge1

# Only controllers
ansible-playbook playbooks/start.yml --limit controllers
```

### Dry Run

```bash
ansible-playbook playbooks/deploy.yml --check
```

---

## ⚙️ Configuration

### Global Settings (`group_vars/all.yml`)

```yaml
app_base_dir: /opt/ipd       # Installation directory
go_version: "1.24.0"         # Go version to install
python_version: "3.12"       # Python version
venv_dir: /opt/ipd/venv      # Python virtual environment
```

### Controller Settings (`group_vars/controllers.yml`)

```yaml
node_role: controller
services:
  - name: controller
    type: go
    binary: controller
    build_path: ./main/
  - name: api_server  
    type: python
    script: main/api_server.py
    port: 8000
```

### Edge Settings (`group_vars/edges.yml`)

```yaml
node_role: edge
services:
  - name: ml_trainer
    type: python
    script: edge/process_images.py
    port: 8765
  - name: edge
    type: go
    binary: edge_node
    build_path: ./edge/

pip_install_timeout: 1800  # 30 min for ML packages
```

---

## 📊 Services

### Controller Services

| Service | Type | Port | Description |
|---------|------|------|-------------|
| `controller` | Go | - | Main orchestration logic |
| `api_server` | Python | 8000 | REST API for control/monitoring |

### Edge Services

| Service | Type | Port | Description |
|---------|------|------|-------------|
| `edge` | Go | - | Edge node communication |
| `ml_trainer` | Python | 8765 | ML training + inference |

### Service Management

```bash
# View logs on a specific node
ssh edge1 'journalctl -u ml_trainer -f'

# Restart a service manually
ssh edge1 'sudo systemctl restart edge'

# Check service status
ssh edge1 'sudo systemctl status ml_trainer'
```

---

## 🔧 Troubleshooting

### Common Issues

<details>
<summary><b>❌ SSH Connection Failed</b></summary>

```bash
# Test connectivity
ansible all -m ping

# Verbose output
ansible-playbook playbooks/setup.yml -vvv
```

Ensure:
- SSH keys are properly configured
- `ansible_user` has sudo access
- Firewall allows SSH (port 22)
</details>

<details>
<summary><b>❌ Pip Install Timeout</b></summary>

ML dependencies are large (~3GB). Increase timeout in `group_vars/edges.yml`:

```yaml
pip_install_timeout: 3600  # 1 hour
```

Or use a faster mirror:
```yaml
pip_extra_args: "--timeout 3600 -i https://pypi.tuna.tsinghua.edu.cn/simple"
```
</details>

<details>
<summary><b>❌ Go Build Failed</b></summary>

```bash
# SSH to the node and check manually
ssh edge1
cd /opt/ipd/project
source /etc/profile.d/go.sh
go mod tidy
go build -v ./...
```
</details>

<details>
<summary><b>❌ Service Won't Start</b></summary>

```bash
# Check logs
ssh edge1 'journalctl -u edge -n 50 --no-pager'

# Verify binary exists
ssh edge1 'ls -la /opt/ipd/project/edge_node'

# Test running manually
ssh edge1 'cd /opt/ipd/project && ./edge_node'
```
</details>

### Useful Commands

```bash
# Check all nodes
ansible all -m shell -a "hostname && uptime"

# Check disk space
ansible edges -m shell -a "df -h /"

# Check Python version
ansible all -m shell -a "python3 --version"

# Check Go version
ansible all -m shell -a "source /etc/profile.d/go.sh && go version"
```

---

## 📈 Scaling

To add more edge nodes:

1. **Add to inventory**:
   ```ini
   [edges]
   edge1 ansible_host=192.168.1.101
   edge2 ansible_host=192.168.1.102
   edge_new ansible_host=192.168.1.105  # New node
   ```

2. **Setup and deploy**:
   ```bash
   ansible-playbook playbooks/setup.yml --limit edge_new
   ansible-playbook playbooks/deploy.yml --limit edge_new
   ansible-playbook playbooks/start.yml --limit edge_new
   ```

---

## ⚠️ Important Notes

| ⚠️ Warning | Description |
|------------|-------------|
| **System Python** | Do NOT remove the system Python installation |
| **Initial Deploy** | First edge deployment can take 15-30 minutes (ML deps) |
| **SSH Access** | All nodes must have SSH key authentication configured |
| **Disk Space** | Edges need ~5GB free for ML packages + venv |
| **Project Structure** | Keep `go.mod` at project root |

---

## 🎯 Use Cases

- 🔬 **Research**: Federated learning experiments
- 🏭 **Production**: Edge computing deployments
- 🎓 **Academic**: Distributed ML training labs
- 🧪 **Testing**: Multi-node orchestration testing

---

## 📝 License

This project is for internal use.

---

<div align="center">

**Built with ❤️ using Ansible**

*Last updated: 2026-01-20*

</div>
