# Collections Requirements Files

This directory contains version-specific requirements files for different AAP versions.

## Available Requirements Files

| File | Target AAP Version | Key Collections |
|------|-------------------|-----------------|
| `requirements-aap24.yml` | AAP 2.4 | `ansible.controller: 4.5.12` (no platform/aap_configuration) |
| `requirements-aap25.yml` | AAP 2.5 | `ansible.platform`, `infra.aap_configuration` |
| `requirements-aap26.yml` | AAP 2.6 | Latest versions of all collections |
| `requirements.yml` | AAP 2.5+ (default) | Same as AAP 2.5 |

## Usage

### Install for AAP 2.4
```bash
ansible-galaxy collection install -r collections/requirements-aap24.yml
```

### Install for AAP 2.5
```bash
ansible-galaxy collection install -r collections/requirements-aap25.yml
```

### Install for AAP 2.6
```bash
ansible-galaxy collection install -r collections/requirements-aap26.yml
```

## Using with ansible-navigator

Specify different Execution Environment images per AAP version:

**AAP 2.4**:
```bash
ansible-navigator run playbooks/configure-aap.yaml \
  --eei registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8:latest
```

**AAP 2.5**:
```bash
ansible-navigator run playbooks/configure-aap.yaml \
  --eei registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
```

**AAP 2.6**:
```bash
ansible-navigator run playbooks/configure-aap.yaml \
  --eei registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest
```

## Important Notes

- **AAP 2.4**: Uses individual `ansible.controller` modules. Some playbooks may need modification to work without `infra.aap_configuration.dispatch` role.
- **AAP 2.5+**: Introduces `ansible.platform` for gateway/platform management and `infra.aap_configuration` for unified CaC.
- **AAP 2.6**: Latest version - use for newest features and improvements.

## Installation Path

By default, collections install to:
- `./collections/ansible_collections/` (local to this project)
- `~/.ansible/collections/ansible_collections/` (user-wide)

To install to a specific path:
```bash
ansible-galaxy collection install -r collections/requirements-aap25.yml -p /custom/path
```
