# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an **Ansible Automation Platform (AAP) Configuration as Code (CaC)** repository for managing AAP infrastructure declaratively using GitOps principles. It enables exporting, versioning, and deploying AAP configurations (organizations, projects, credentials, job templates, workflows, etc.) across environments.

**Key Concept**: Configuration as Code allows you to define AAP resources in YAML files, version them in Git, and apply them programmatically to automation controller instances - enabling consistent, auditable, and repeatable infrastructure deployments.

## Version Compatibility

- **AAP 2.5+**: Use `infra.aap_configuration` collection (required)
- **AAP 2.4**: Use `ansible.controller 4.5.12` collection
- **Python**: Requires `awxkit >= 9.3.0` for AAP API interactions

Install with: `pip3 install awxkit --user`

## Configuration Architecture

This repository supports two configuration patterns:

### 1. Flattened Configuration (`cac_sample/`)
Simple directory structure with environment-based organization:
```
cac_sample/
├── common/          # Shared configs across all environments
│   ├── gateway_organizations.yaml
│   ├── controller_projects.yaml
│   ├── controller_templates.yaml
│   └── ...
├── uat/             # UAT-specific overrides
├── prod/            # Production-specific configs
└── other/           # Additional environments
```

**When to use**: Smaller deployments, single organization, simple environment differences.

### 2. Filetree Configuration (`cac_filetree/`)
Hierarchical structure for multi-org, multi-environment deployments:
```
cac_filetree/
└── <Organization>/
    └── env/
        ├── common/              # Org-wide common configs
        │   ├── aap_organizations.d/
        │   ├── controller_projects.d/
        │   └── ...
        ├── uat/                 # Environment-specific configs
        │   ├── aap_users.d/
        │   ├── controller_hosts.d/
        │   └── ...
        └── prod/
```

**When to use**: Multiple organizations, complex environment hierarchies, granular control.

Configuration files use the `.d/` suffix pattern (e.g., `controller_projects.d/`) indicating a directory containing multiple YAML files that will be merged together by the `filetree_read` role from `infra.aap_configuration_extended`.

## Common Commands

### Prerequisites - Set Credentials

All operations require AAP credentials as environment variables:

```bash
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=your-password
export CONTROLLER_HOST=https://aap25.lab.example.com
export CONTROLLER_VERIFY_SSL=false

# Optional: Set organization and environment
export AAP_ORGANIZATION=Cac-Demo-Org
export AAP_ENVIRONMENT=uat
```

### Applying Configuration to AAP

**Flattened configuration (using cac_sample/)**:
```bash
# Using ansible-playbook
ansible-playbook playbooks/configure-aap.yaml

# Using ansible-navigator (containerized)
ansible-navigator run playbooks/configure-aap.yaml \
  --penv CONTROLLER_USERNAME \
  --penv CONTROLLER_PASSWORD \
  --penv CONTROLLER_HOST
```

**Filetree configuration (using cac_filetree/)**:
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=${AAP_ORGANIZATION}" \
  -e "dir_orgs_vars=./cac_filetree" \
  -e "env=${AAP_ENVIRONMENT}" \
  --tags settings,organizations,credentials,projects,inventories,templates

# With vault password file
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "dir_orgs_vars=./cac_filetree" \
  -e "env=uat" \
  --vault-password-file ./.vault_pass.txt
```

**Important parameters**:
- `orgs`: Organization name (must match directory under `cac_filetree/`)
- `dir_orgs_vars`: Path to filetree root (default: `../cac_filetree`)
- `env`: Environment (`common`, `uat`, `prod`, `other`)
- `--tags`: Control which AAP resources to configure (see Controlling Execution below)

### Exporting Configuration from AAP

Export current AAP state to YAML files for version control:

```bash
# Basic export
ansible-playbook playbooks/controller-export.yaml

# Filetree-based export
ansible-playbook playbooks/export-using-filetree.yaml \
  -e "orgs=${AAP_ORGANIZATION}" \
  -e "env=${AAP_ENVIRONMENT}"
```

Exported files are saved to `playbooks/cac_exported/` directory.

### Controlling Execution with Tags

Both configuration playbooks support tags to control which AAP resources are configured. This is critical for phased deployments when credentials need manual updates between stages.

**Recommended two-phase approach**:

**Phase 1**: Create foundational resources
```bash
--tags settings,organizations,credential_types,credentials
```

**Manual Step**: Login to AAP UI and update credential passwords/tokens (source control credentials, API tokens, etc.)

**Phase 2**: Create dependent resources
```bash
--tags projects,inventories,hosts,groups,templates,workflows,schedules,roles
```

**Available tags** (configure order):
- `settings` - Controller settings
- `organizations` - Organizations
- `credential_types` - Custom credential types
- `credentials` - Credentials (⚠️ may need manual token/password updates)
- `projects` - Projects (depend on credentials)
- `inventories` - Inventories
- `hosts` - Inventory hosts
- `groups` - Inventory groups
- `templates` - Job templates
- `workflows` - Workflow job templates
- `schedules` - Schedules
- `roles` - RBAC role assignments

Use `--tags none` to prevent execution (safety mechanism).

### Managing Secrets with Ansible Vault

Encrypt sensitive data (passwords, tokens) in configuration files:

```bash
# Encrypt a string
ansible-vault encrypt_string 'MySecretPassword' --name 'admin_password'

# Encrypt with specific vault password
echo "ansible" > .vault_pass.txt
ansible-vault encrypt_string 'MySecretPassword' --vault-password-file ./.vault_pass.txt
```

Use the encrypted output inline in YAML files:
```yaml
aap_user_accounts:
  - username: jdoe
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          38636432353066366436633665353033626134353632343436643533353562353266626136666666
          ...
```

**Security**: Never commit `.vault_pass.txt` to Git (already in `.gitignore`).

## Key Playbooks

| Playbook | Purpose | Key Parameters |
|----------|---------|----------------|
| `playbooks/configure-aap.yaml` | Apply flattened config from `cac_sample/` | `aap_configs_dir`, `aap_env` |
| `playbooks/configure-aap-using-filetree.yaml` | Apply filetree config from `cac_filetree/` | `orgs`, `dir_orgs_vars`, `env` |
| `playbooks/controller-export.yaml` | Export AAP config to YAML | `export_dir` |
| `playbooks/export-using-filetree.yaml` | Export to filetree structure | `orgs`, `env` |

## Custom Roles

### `roles/merge_config`
Merges configuration from multiple YAML files across environments. Used by `configure-aap.yaml` to combine:
- Common configs (`cac_sample/common/*.yaml`)
- Environment-specific configs (`cac_sample/{uat,prod}/*.yaml`)

The role reads all YAML files from specified directories and merges them into a single configuration structure that's passed to the `infra.aap_configuration.dispatch` role.

### `roles/controller_export`
Handles exporting AAP configurations using the `ansible.controller.export` module.

## Configuration File Patterns

Configuration files follow naming conventions that map to AAP resource types:

**Platform/Gateway Resources** (AAP 2.5+):
- `gateway_organizations.yaml` - Organizations
- `gateway_authenticators.yaml` - Authentication backends
- `gateway_authenticator_maps.yaml` - Authenticator mappings
- `gateway_role_user_assignments.yml` - RBAC assignments

**Controller Resources**:
- `controller_settings.yaml` - Controller settings
- `controller_projects.yaml` - SCM projects
- `controller_inventories.yaml` - Inventories
- `controller_templates.yaml` - Job templates
- `workflows.yaml` - Workflow job templates
- `credentials.yaml` - Credentials
- `credential_types.yaml` - Custom credential types

**Users and Teams**:
- `aap_user_accounts.yml` - User accounts
- `aap_teams.yml` - Teams and team memberships

**Variables in configs**: Configuration files use these patterns:
- `{{ vault_* }}` - Vaulted variables for sensitive data
- `{{ lookup('env', 'VAR') }}` - Environment variable lookups
- `platform_state: present|absent` - Resource state control

## Running from AAP Controller (Self-Configuration)

To enable AAP to configure itself via Job Templates:

1. **Create Project**: Point to this Git repository
2. **Create Credential**: Type `Red Hat Ansible Automation Platform`, enter controller URL/username/password
3. **Create Job Template**: 
   - Playbook: `playbooks/configure-aap.yaml` or `playbooks/configure-aap-using-filetree.yaml`
   - Credential: Attach the AAP credential from step 2
   - Enable "Prompt on Launch" for Job Tags
   - Set default tag to `none` (prevents accidental execution)
4. **Execute**: Launch with appropriate tags (e.g., `organizations,credentials,projects`)

## Webhook Integration

The repository supports automated CaC deployments via webhooks. When AAP is not publicly accessible, use ngrok for tunneling:

```bash
export NGROK_AUTH_TOKEN=your-token
export NGROK_CUSTOM_DOMAIN=your-domain.ngrok-free.app

podman run --net=host -it \
  -e NGROK_AUTHTOKEN=$NGROK_AUTH_TOKEN \
  ngrok/ngrok:latest \
  http https://aap-local.lab.example.com \
  --domain=$NGROK_CUSTOM_DOMAIN
```

Configure the webhook in GitHub/GitLab pointing to the ngrok URL.

## Testing and Validation

**Syntax check**:
```bash
ansible-playbook playbooks/configure-aap.yaml --syntax-check
```

**Dry run** (check mode - limited support with API modules):
```bash
ansible-playbook playbooks/configure-aap.yaml --check
```

⚠️ **Note**: Many AAP modules don't fully support `--check` mode. Test in non-production environment first.

**Validate specific resource**:
```bash
ansible-playbook playbooks/configure-aap.yaml --tags organizations -e aap_env=uat
```

## Important Notes

- **Credential Input Sources**: If using automated credential input sources, all resources can be created in one run. Otherwise, use the two-phase approach (create credentials → manual update → create dependent resources).
- **Dependencies**: The `infra.aap_configuration.dispatch` role handles resource dependency ordering automatically (e.g., projects before job templates).
- **SSL Verification**: `controller_validate_certs: false` is used in dev/lab environments. Use proper certificates in production.
- **Execution Environment**: `ansible-navigator.yml` specifies `registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8:1.0.0-395` - update for AAP 2.5+ environments.
- **Variables Precedence**: Environment-specific configs (`uat/`, `prod/`) override `common/` configs when using merge_config role.

## Required Collections

Listed in `requirements.yaml`:
- `ansible.platform` - Platform/Gateway resources (AAP 2.5+)
- `ansible.hub` - Automation Hub management
- `ansible.controller` - Controller resources
- `ansible.eda` - Event-Driven Ansible
- `infra.aap_configuration` - Main CaC collection (roles: dispatch, etc.)
- `infra.aap_configuration_extended` - Extended CaC utilities (roles: filetree_read, etc.)

Install: `ansible-galaxy collection install -r requirements.yaml`
