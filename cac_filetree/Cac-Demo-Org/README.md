# Cac-Demo-Org - AAP Configuration as Code Samples

This directory contains comprehensive Configuration as Code (CaC) samples for Ansible Automation Platform (AAP) 2.5+.

## ⚠️ Ansible Vault Password

**For this PUBLIC DEMO repository only:**
- Vault password: `ansible` (stored in `.vault_pass.txt` in repo root)
- **WARNING**: This is for demo/learning purposes only!
- **NEVER commit vault passwords to git in production!**
- Production: Use secure password managers, never commit `.vault_pass.txt`

Usage:
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --vault-password-file ./.vault_pass.txt
```

## Organization Structure

```
Cac-Demo-Org/
└── env/
    ├── common/      # Shared configs across all environments (production-safe)
    ├── uat/         # UAT environment specific configs
    ├── prod/        # Production environment specific configs
    ├── aiops/       # AIOps/EDA specific configs
    └── staging/     # Staging/testing configs (requires external dependencies)
```

## Environment Guide

### `common/` - Production-Safe Samples ✅
Resources that can be safely applied without external dependencies:

- **Inventory Management**
  - `controller_inventories.d/` - Inventory definitions
  - `controller_hosts.d/` - 30+ sample hosts across tiers
  - `controller_groups.d/` - Hierarchical group structures
  - `controller_host_groups.d/` - Host-to-group assignments

- **RBAC & Teams**
  - `aap_teams.d/` - 35+ role-based teams
  - `aap_users.d/` - User account definitions
  - `aap_organizations.d/` - Organization configs
  - `gateway_role_team_assignments.d/` - Team RBAC assignments
  - `gateway_role_user_assignments.d/` - User RBAC assignments
  - `controller_roles.d/` - Controller role assignments

- **Automation Resources**
  - `controller_projects.d/` - SCM project definitions
  - `controller_job_templates.d/` - Job template samples
  - `controller_workflow_job_templates.d/` - Complex workflow examples
  - `controller_schedules.d/` - Schedule definitions
  - `controller_notifications.d/` - Notification templates
  - `controller_labels.d/` - Label definitions

- **Configuration**
  - `gateway_settings.d/` - AAP platform settings
  - `controller_settings.d/` - Controller-specific settings
  - `controller_execution_environments.d/` - EE definitions
  - `controller_credentials.d/` - Credential templates
  - `controller_credential_types.d/` - Custom credential types
  - `gateway_authenticator_maps.d/` - Authentication mappings

### `staging/` - Test Environment Only ⚠️
Resources requiring external integrations (use only for testing):

- **Hub Resources** (requires Private Automation Hub)
  - `hub_namespaces.d/` - Namespace definitions
  - `hub_collections.d/` - Collection uploads
  - `hub_ee_registries.d/` - Container registry configs

- **Vault Integration** (requires external vault systems)
  - `controller_credential_input_sources.d/` - CyberArk, HashiCorp Vault, AWS Secrets Manager, Azure Key Vault

### `uat/` & `prod/` - Environment Overrides
Environment-specific configurations that override or extend `common/` settings.

### `aiops/` - Event-Driven Automation
AIOps and EDA-specific configurations:
- `eda_projects.d/` - EDA project definitions
- `eda_rulebook_activations.d/` - Rulebook activation configs
- `eda_decision_environments.d/` - Decision environment configs
- `eda_credentials.d/` - EDA credentials

## Sample Highlights

### Enterprise Multi-Tier Inventory
**File**: `common/controller_hosts.d/enterprise_hosts.yml`

30+ hosts organized across:
- Web tier (app servers, load balancers, proxies)
- Application tier (backends, APIs, workers, messaging)
- Database tier (PostgreSQL, MySQL, MongoDB, Redis)

Each host includes realistic variables (IP addresses, tiers, datacenters, roles).

### Hierarchical Groups
**File**: `common/controller_groups.d/enterprise_groups.yml`

Parent-child group relationships:
```
web_tier
├── web_servers
├── web_load_balancers
└── web_proxy_servers

application_tier
├── app_backend_servers
├── app_api_servers
├── worker_nodes
└── messaging_queue

database_tier
├── postgresql_cluster
│   ├── postgresql_primary
│   └── postgresql_replicas
├── mysql_cluster
├── mongodb_shards
└── redis_cache
```

### Role-Based Teams
**File**: `common/aap_teams.d/role_based_teams.yml`

35+ teams organized by function:
- Platform Administration (Admins, Operators, Auditors)
- Development (Frontend, Backend, Mobile, QA)
- Infrastructure (Compute, Network, Storage, Database)
- Security (Operations, Compliance, Vulnerability Management)
- Operations (L1, L2, L3, NOC, SRE)
- Application Ownership (CRM, ERP, Portal)
- Change Management (Approvers, CAB)

### Advanced Workflows
**File**: `common/controller_workflow_job_templates.d/advanced-workflows/`

Complex workflow examples:
- DR failover workflows with approval gates
- Multi-stage deployment pipelines
- Parallel execution patterns
- Conditional branching

## Usage Examples

### Apply Common (Safe) Configurations
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  -e "dir_orgs_vars=./cac_filetree" \
  --tags settings,organizations,teams,inventories,hosts,groups,projects,templates
```

### Apply UAT Environment
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=uat" \
  -e "dir_orgs_vars=./cac_filetree" \
  --tags inventories,credentials,templates
```

### Test Hub Resources (Staging Only)
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=staging" \
  -e "dir_orgs_vars=./cac_filetree" \
  --tags hub_namespaces,hub_collections
```

### Two-Phase Deployment (Recommended)
**Phase 1**: Create credentials first
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --tags settings,organizations,credential_types,credentials
```

**Manual Step**: Login to AAP UI and update credential passwords/tokens

**Phase 2**: Create dependent resources
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --tags projects,inventories,templates,workflows,schedules,roles
```

## Configuration Patterns

### Controller Settings
- **Job Execution**: Timeouts, forks, isolation paths, task environment
- **UI Settings**: Live updates, session management, custom branding
- **Authentication**: OAuth2, session timeouts, basic auth

### Gateway Settings
- **Platform**: Session management, CSRF protection, remote host headers
- **Security**: Proxy IP allowlists, SSL settings
- **Logging**: External aggregators, log levels
- **Branding**: Custom login messages, logos
- **Analytics**: Insights tracking, analytics intervals

### Credential Input Sources
Demonstrates integration with external secret managers:
- CyberArk (object query format)
- HashiCorp Vault (KV backend, AppRole auth)
- AWS Secrets Manager
- Azure Key Vault

**Note**: These are in `staging/` as they require source credentials to be configured first.

## Best Practices Demonstrated

1. **Environment Segregation**: Common vs environment-specific configs
2. **Hierarchical Organization**: Parent-child groups, role-based teams
3. **Realistic Variables**: Actual datacenter patterns, tier assignments
4. **Naming Conventions**: Consistent, descriptive resource names
5. **Documentation**: Inline comments explaining configuration choices
6. **Safety**: Risky configs isolated to staging environment
7. **Scalability**: Patterns that work for 10 or 1000+ hosts

## Important Notes

⚠️ **Before Applying**:
- Review all variable values (especially `ansible_host` IPs, credentials)
- Update vault-encrypted values for your environment
- Test in non-production first
- Use `--tags none --check` for dry-run validation

⚠️ **Staging Environment**:
- Hub resources will fail without Private Automation Hub
- Credential input sources need external vault systems configured
- These are examples only - customize for your environment

✅ **Safe to Apply**:
- All resources in `common/` can be safely applied to demo/lab environments
- They demonstrate patterns without requiring external dependencies
- Ideal for learning AAP CaC best practices

## Related Documentation

- [Main README](../../README.md) - Repository overview
- [CLAUDE.md](../../CLAUDE.md) - Claude Code guidance for this repo
- [AAP CaC Collection Docs](https://github.com/redhat-cop/aap_configuration)
