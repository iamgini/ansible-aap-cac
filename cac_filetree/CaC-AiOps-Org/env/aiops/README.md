# AIOps Configuration as Code

This directory contains AAP Configuration as Code (CaC) for the Intelligent Remediation AIOps system.

## Directory Structure

Following the standard CaC filetree pattern - each resource type in its own `.d` directory.

**Controller Resources:** 5 directories
**EDA Resources:** 4 directories
**Apply Playbooks:** 2 files

## How to Apply

### Controller Configuration:
```bash
cd /home/gmadappa/ansible/ansible-aap-cac/cac_filetree/Cac-Demo-Org/env/aiops
export CONTROLLER_HOST="https://controller.example.com"
export CONTROLLER_USERNAME="admin"
export CONTROLLER_PASSWORD="password"
ansible-playbook apply-aiops-cac.yml
```

### EDA Configuration:
```bash
export EDA_CONTROLLER_HOST="https://eda-controller.example.com"
ansible-playbook apply-eda-cac.yml
```

## Related Documentation
- Main AIOps project: ~/ansible/ansible-aiops/
- Deployment guide: ~/ansible/ansible-aiops/DEPLOYMENT-GUIDE.md
