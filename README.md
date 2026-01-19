# IPD – Ansible Deployment Guide

## 📋 Project Overview

**IPD** is a **Federated Learning system** designed to run across multiple Linux machines using a controller–edge architecture.
The project uses **Ansible** to automate environment setup, deployment, updates, and service management across all nodes.

The goal is to make deploying and operating a distributed ML system **repeatable, fast, and consistent**, even when managing many machines.

---

## 🧠 System Architecture

IPD consists of two types of nodes:

### Controller Node

* Acts as the central orchestrator
* Coordinates federated learning rounds
* Distributes data or tasks to edge nodes
* Aggregates trained model weights
* Exposes an API server for control and monitoring

### Edge Nodes

* Receive training tasks from the controller
* Perform local ML training
* Send model updates back to the controller
* Can run on heterogeneous hardware

This setup allows training to happen close to the data while keeping coordination centralized.

---

## 🧰 Technology Stack

| Component                  | Technology                      |
| -------------------------- | ------------------------------- |
| Orchestration & Automation | Ansible                         |
| Core Logic                 | Go (Controller & Edge binaries) |
| ML Training                | Python + PyTorch                |
| Communication              | WebRTC & WebSockets             |
| Service Management         | systemd                         |
| OS Target                  | Ubuntu-based Linux systems      |

---

## 📁 Repository Structure

The repository is structured to keep **automation and application code together**:

```
edge-automation/
├── ansible.cfg
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── setup.yml
│   ├── deploy.yml
│   ├── start.yml
│   ├── stop.yml
│   ├── update.yml
│   └── status.yml
├── group_vars/
│   ├── all.yml
│   ├── controllers.yml
│   └── edges.yml
├── templates/
│   ├── controller.service.j2
│   ├── api_server.service.j2
│   ├── edge.service.j2
│   └── ml_trainer.service.j2
└── project/
    └── IPD source code (Go + Python)
```

* **playbooks/**: High-level operational workflows
* **group_vars/**: Role-based configuration (controller vs edge)
* **templates/**: systemd service definitions
* **project/**: The actual IPD application codebase

---

## ⚙️ What Ansible Automates

This project automates the entire lifecycle of the system:

### Environment Setup

* Installs required system packages
* Installs Python 3.12 and Go 1.22
* Configures users and permissions

### Deployment

* Syncs the IPD project to all nodes
* Builds Go binaries
* Creates Python virtual environments
* Installs ML dependencies
* Generates systemd services

### Runtime Operations

* Start and stop services
* Restart services after updates
* Check health and status across all nodes

### Updates

* Syncs code changes
* Rebuilds binaries if needed
* Restarts affected services automatically

---

## 🚀 Typical Workflow

1. Add controller and edge machines to the Ansible inventory
2. Run the setup playbook to prepare all systems
3. Deploy the project to all nodes
4. Start services on controllers and edges
5. Monitor status and logs centrally
6. Push code updates and redeploy with one command

---

## 🔧 Operational Notes

* The controller and edges run different services based on their role
* Services are managed using systemd for reliability
* Python ML workloads are isolated using virtual environments
* Go binaries are rebuilt automatically when code changes
* The system is designed to scale by simply adding more edge nodes

---

## ⚠️ Important Considerations

* Do not remove the system Python installation
* Initial ML dependency installation can take significant time
* All nodes must have SSH access configured
* Project structure must remain consistent with `go.mod`

---

## 🧩 Use Case

This setup is ideal for:

* Federated learning experiments
* Edge computing research
* Distributed ML training
* Managing many Linux machines consistently
* Academic or production-grade orchestration testing

---

*Last updated: 2026-01-19*
