# RBAC Configuration for Cac-Demo-Org

This directory contains Role-Based Access Control (RBAC) configurations for the Cac-Demo-Org organization in Ansible Automation Platform.

## Configuration Files

### Gateway Role Assignments (AAP 2.5+)

Located in respective `.d/` directories:

1. **gateway_role_user_assignments.d/rbac-user-assignments.yaml**
   - Assigns roles directly to individual users
   - Organization-level permissions
   - Team membership and admin roles
   - Platform-wide roles (if needed)

2. **gateway_role_team_assignments.d/rbac-team-assignments.yaml**
   - Assigns roles to teams for organization-wide access
   - Teams get permissions across all resources of a type
   - Multi-organization assignments supported

### Controller Role Assignments (Resource-Specific)

Located in `controller_roles.d/`:

3. **controller_roles.d/rbac-resource-assignments.yaml**
   - Granular permissions for specific resources
   - Inventory, project, job template, workflow, credential access
   - Team membership assignments
   - User-specific resource permissions

## RBAC Model (AAP 2.5+)

```
┌─────────────────────────────────────────────────────────────┐
│                    Platform Gateway                         │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Gateway Role Assignments (Organization-Level)      │    │
│  │  - Organization Admin                              │    │
│  │  - Organization Member                             │    │
│  │  - Organization <Resource> Admin                   │    │
│  │    (Project, Inventory, JobTemplate, etc.)         │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              Controller Role Assignments                    │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Resource-Specific Permissions                     │    │
│  │  - Specific Inventory: admin/use/read             │    │
│  │  - Specific Project: admin/use/read               │    │
│  │  - Specific Job Template: admin/execute/read      │    │
│  │  - Specific Credential: admin/use/read            │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## User and Team Structure

### Users
- **gineesh** - Superuser, Organization Admin
- **jsmith** - Team Admin, Organization-level admin roles
- **jdoe** - Team Member, Organization Auditor

### Teams
- **cac-team-101** - Operations team with broad access
- **cac-team-102** - Limited operations team

## Permission Hierarchy

### Organization-Level (Gateway Roles)
```yaml
Organization Admin (gineesh)
  ├── Organization Project Admin (jsmith)
  ├── Organization Inventory Admin (jsmith)
  ├── Organization JobTemplate Admin (jsmith)
  ├── Organization Credential Admin (gineesh)
  └── Organization Auditor (jdoe)
```

### Team-Level
```yaml
cac-team-101
  ├── Team Admin: jsmith
  ├── Team Member: jdoe
  └── Organization Permissions:
      ├── Organization Inventory Admin
      ├── Organization Project Admin
      ├── Organization JobTemplate Admin
      └── Organization EE Admin

cac-team-102
  ├── Team Member: jsmith
  └── Organization Permissions:
      └── Organization Member (basic access)
```

### Resource-Level (Controller Roles)
```yaml
Inventories:
  Cac-Demo-Inventory
    ├── admin: cac-team-101, jsmith
    └── use: cac-team-102, jdoe

Projects:
  Ansible-Real-Life-Cac
    ├── admin: cac-team-101, jsmith
    └── use: cac-team-102

Job Templates:
  HW-01-Daily-Morning-Check
    ├── execute: cac-team-101, cac-team-102
  HW-03-Weekly-Report
    ├── admin: cac-team-101, jsmith
```

## Role Definitions Reference

### Gateway Roles (Organization-Scoped)

| Role Definition | Scope | Description |
|-----------------|-------|-------------|
| Organization Admin | Organization | Full control over organization and all resources |
| Organization Member | Organization | Basic access to organization |
| Organization Auditor | Organization | Read-only access to organization |
| Organization Project Admin | Organization | Manage all projects in organization |
| Organization Inventory Admin | Organization | Manage all inventories in organization |
| Organization JobTemplate Admin | Organization | Manage all job templates in organization |
| Organization Credential Admin | Organization | Manage all credentials in organization |
| Organization Workflow Admin | Organization | Manage all workflows in organization |
| Organization Execution Environment Admin | Organization | Manage all EEs in organization |
| Team Admin | Team | Manage team and its members |
| Team Member | Team | Basic team membership |

### Controller Roles (Resource-Scoped)

| Resource Type | Available Roles | Description |
|---------------|-----------------|-------------|
| Inventory | admin, use, update, adhoc, read | Inventory-specific permissions |
| Project | admin, use, update, read | Project-specific permissions |
| Job Template | admin, execute, read | Job template-specific permissions |
| Workflow | admin, execute, approval, read | Workflow-specific permissions |
| Credential | admin, use, read | Credential-specific permissions |
| Team | member, admin | Team membership |

## Applying RBAC Configuration

### Full RBAC Setup
```bash
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=your-password
export CONTROLLER_HOST=https://aap.example.com
export CONTROLLER_VERIFY_SSL=false

ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "dir_orgs_vars=./cac_filetree" \
  -e "env=common" \
  --tags gateway_role_user_assignments,gateway_role_team_assignments,controller_roles
```

### Apply in Phases

**Phase 1: Users and Teams**
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --tags organizations,users,teams
```

**Phase 2: Gateway Role Assignments (Organization-level)**
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --tags gateway_role_user_assignments,gateway_role_team_assignments
```

**Phase 3: Controller Role Assignments (Resource-specific)**
```bash
ansible-playbook playbooks/configure-aap-using-filetree.yaml \
  -e "orgs=Cac-Demo-Org" \
  -e "env=common" \
  --tags controller_roles
```

## Best Practices

1. **Use Gateway Roles for Organization-Level Access**
   - Prefer `Organization <Resource> Admin` roles over individual resource assignments
   - Cleaner RBAC model, fewer assignments to manage

2. **Use Controller Roles for Granular Access**
   - When users/teams need access to specific resources only
   - For fine-grained permission control

3. **Team-Based Access Control**
   - Assign permissions to teams, not individual users
   - Add users to teams for role inheritance
   - Easier to manage as organization grows

4. **Principle of Least Privilege**
   - Start with minimal permissions
   - Grant additional access as needed
   - Use `read` and `use` roles before `admin`

5. **Auditing**
   - Assign `Organization Auditor` role for compliance
   - Separate audit accounts from operational accounts

## Troubleshooting

### Common Issues

**Permissions not working after applying:**
- Check that users exist before applying role assignments
- Verify organization name matches exactly
- Ensure team exists before assigning team roles

**Role assignment fails:**
- Gateway roles require AAP 2.5+
- Some role combinations are not valid (e.g., Team Admin to Team)
- Check role definition name spelling

**Users can't see resources:**
- Organization membership is required first
- Check both gateway and controller role assignments
- Verify resource exists in the correct organization

## Verification

After applying RBAC, verify in AAP UI:

1. **Login as each user** (jsmith, jdoe) to verify access
2. **Check Access** → Navigate to Resources → verify can/cannot see expected items
3. **Test Execute** → Try launching job templates
4. **Check Administration** → Verify admin users can manage resources

Or via API/CLI:
```bash
# Check user's role assignments
ansible-playbook -m ansible.platform.role_user_assignment \
  -a "user=jsmith state=exists" localhost

# List all role assignments
awx role list
```

## Migration from AAP 2.4 to 2.5+

If migrating from AAP 2.4:
- Old `controller_user_accounts` with `organization` field → `aap_user_accounts`
- Old controller organization roles → `gateway_role_user_assignments` with `Organization <Resource> Admin`
- Resource-specific `controller_roles` remain the same

## Additional Resources

- [AAP 2.5+ RBAC Documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/access_management_and_authentication/)
- [infra.aap_configuration Collection Docs](https://github.com/redhat-cop/infra.aap_configuration)
- [ansible.platform Collection](https://github.com/ansible/ansible-platform)
