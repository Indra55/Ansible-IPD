# IPD Ansible - Command Reference

Quick reference for deploying and managing IPD across nodes.

---

## Pre-requisites: Tailscale Setup

Run on each node to get a Tailscale IP address for networking.

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Connect to Tailscale network (adjust hostname per node)
sudo tailscale up \
  --authkey  \
  --hostname edge-01 \
  --advertise-tags=tag:edge

# Get assigned IP
tailscale ip -4
```

---

## Controller Commands

All commands run from the Ansible control machine.

### Core Playbooks

```bash
# Install Python 3.12 and Go 1.24 on all nodes
ansible-playbook playbooks/setup.yml

# Deploy full stack (code, dependencies, systemd services)
ansible-playbook playbooks/deploy.yml

# Start all services
ansible-playbook playbooks/start.yml

# Stop all services
ansible-playbook playbooks/stop.yml

# Check service status, ports, disk space
ansible-playbook playbooks/status.yml

# Update code and restart services
ansible-playbook playbooks/update.yml
```

### Target Specific Hosts

```bash
ansible-playbook playbooks/deploy.yml --limit edges
ansible-playbook playbooks/deploy.yml --limit controllers
ansible-playbook playbooks/status.yml --limit edge1
```

### Pre-flight Checks

```bash
ansible all -m ping
ansible-playbook playbooks/deploy.yml --check
ansible-playbook playbooks/deploy.yml --syntax-check
```

### Ad-hoc Commands

```bash
ansible all -m shell -a "hostname && uptime"
ansible edges -m shell -a "df -h /"
ansible all -m shell -a "python3 --version"
ansible all -m shell -a "source /etc/profile.d/go.sh && go version"
ansible all -m shell -a "ss -tlnp | grep -E ':(8000|8765)'"
```

---

## Edge Node Commands

Run directly on edge machines via SSH.

### Service Management

```bash
sudo systemctl status ml_trainer
sudo systemctl status edge
sudo systemctl restart ml_trainer
sudo systemctl restart edge
sudo systemctl stop ml_trainer
sudo systemctl stop edge
```

### View Logs

```bash
journalctl -u ml_trainer -f
journalctl -u edge -f
journalctl -u ml_trainer -n 50 --no-pager
```

### Manual Testing

```bash
# Test Go binary
cd /opt/ipd/project
./edge_node

# Test Python script
cd /opt/ipd/project
source ../venv/bin/activate
python edge/process_images.py
```

### Debug Go Build

```bash
cd /opt/ipd/project
source /etc/profile.d/go.sh
go mod tidy
go build -v ./edge/
```

### Check Python Packages

```bash
source /opt/ipd/venv/bin/activate
pip list | grep -E "torch|tensorflow|ultralytics"
```

---

## Controller Node Commands

Run directly on controller machines via SSH.

### Service Management

```bash
sudo systemctl status controller
sudo systemctl status api_server
sudo systemctl restart controller
sudo systemctl restart api_server
```

### View Logs

```bash
journalctl -u controller -f
journalctl -u api_server -f
```

---

## Typical Deployment Flow

```bash
# 1. Setup environment (one-time)
ansible-playbook playbooks/setup.yml

# 2. Deploy application (15-30 min on edges)
ansible-playbook playbooks/deploy.yml

# 3. Start services
ansible-playbook playbooks/start.yml

# 4. Verify
ansible-playbook playbooks/status.yml

# 5. After code changes
ansible-playbook playbooks/update.yml
```

---

## Troubleshooting

### SSH Issues

```bash
ansible all -m ping
ansible-playbook playbooks/setup.yml -vvv
```

### Stuck pip Install

Increase timeout in `group_vars/edges.yml`:

```yaml
pip_install_timeout: 3600
```

### Go Build Fails

```bash
ssh edge1
cd /opt/ipd/project
source /etc/profile.d/go.sh
go mod tidy
go build -v ./...
```

### Service Wont Start

```bash
ssh edge1 'journalctl -u edge -n 50 --no-pager'
ssh edge1 'ls -la /opt/ipd/project/edge_node'
```

### Kill Stuck Processes

```bash
ansible edges -m shell -a "pkill -9 -f edge_node || true"
ansible edges -m shell -a "pkill -9 -f process_images || true"
```

---

## Notes

- First edge deployment takes 15-30 minutes (ML packages are ~3GB)
- Edges need ~5GB free disk space
- All nodes need SSH key authentication configured
- Do not remove system Python
