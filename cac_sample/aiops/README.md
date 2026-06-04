# AIOps Configuration as Code

This directory contains AAP Configuration as Code (CaC) for the Intelligent Remediation AIOps system.

## Components

### Controller (AAP) Configuration

| File | Purpose |
|------|---------|
| `controller_projects.yaml` | Git project for remediation playbooks |
| `controller_inventories.yaml` | Inventories for remediation targets |
| `credential_types.yaml` | Custom credential type for MCP/Git integration |
| `credentials.yaml` | Controller credentials (AAP, MCP, Git, SSH) |
| `controller_templates.yaml` | 5 job templates (4 remediation + 1 AI) |
| `apply-aiops-cac.yml` | **Master playbook to apply Controller config** |

### EDA Configuration

| File | Purpose |
|------|---------|
| `eda_credentials.yaml` | EDA credentials (AAP Controller token) |
| `eda_decision_environments.yaml` | Decision environments for rulebooks |
| `eda_projects.yaml` | EDA project for rulebooks |
| `eda_rulebook_activations.yaml` | Rulebook activation configuration |
| `apply-eda-cac.yml` | **Master playbook to apply EDA config** |

## Architecture

```
Event Sources → EDA Rulebook → AAP Job Templates → Remediation
                    ↓
            intelligent-remediation.yml
                    ↓
        ┌───────────┴────────────┐
        │                        │
   Known Events            Unknown Events
        │                        │
   Job Templates            AI Intelligence
   (Cases 1-4)              (Case 5)
```

## Deployment

### Step 1: Configure Variables

Edit the YAML files and update:
- Organization names
- Project URLs (if using your fork)
- Credential values (use vault for secrets)
- Inventory names

### Step 2: Apply Configuration

```bash
cd ~/ansible/ansible-aap-cac

# Configure controller connection
export CONTROLLER_HOST="https://controller.example.com"
export CONTROLLER_USERNAME="admin"
export CONTROLLER_PASSWORD="password"
export CONTROLLER_VERIFY_SSL="false"

# Apply all AIOps CaC
ansible-playbook configure-controller.yml \
  -e @cac_sample/aiops/controller_projects.yaml \
  -e @cac_sample/aiops/controller_inventories.yaml \
  -e @cac_sample/aiops/credential_types.yaml \
  -e @cac_sample/aiops/credentials.yaml \
  -e @cac_sample/aiops/controller_templates.yaml

# Apply EDA configuration
ansible-playbook configure-eda.yml \
  -e @cac_sample/aiops/eda_credentials.yaml \
  -e @cac_sample/aiops/eda_decision_environments.yaml \
  -e @cac_sample/aiops/eda_projects.yaml \
  -e @cac_sample/aiops/eda_rulebook_activations.yaml
```

### Step 3: Verify

```bash
# List created job templates
awx job_templates list --name "Remediate Disk Space"
awx job_templates list --name "AI Intelligence"

# List EDA rulebook activations
awx-eda rulebook_activations list
```

## Order of Application

1. ✅ `credential_types.yaml` - Custom credential types first
2. ✅ `credentials.yaml` - Credentials (depends on types)
3. ✅ `controller_projects.yaml` - Projects
4. ✅ `controller_inventories.yaml` - Inventories
5. ✅ `controller_templates.yaml` - Job templates (depends on all above)
6. ✅ `eda_credentials.yaml` - EDA credentials
7. ✅ `eda_decision_environments.yaml` - Decision environments
8. ✅ `eda_projects.yaml` - Rulebook projects
9. ✅ `eda_rulebook_activations.yaml` - Activate rulebooks

## Integration with ansible-aiops

These CaC files reference playbooks from:
- **Repository:** `https://github.com/iamgini/ansible-aiops`
- **Playbooks:**
  - `playbooks/remediation/disk-cleanup.yml`
  - `playbooks/remediation/restart-service.yml`
  - `playbooks/remediation/investigate-cpu.yml`
  - `playbooks/remediation/renew-certificate.yml`
  - `playbooks/intelligent-aiops-workflow.yml`
- **Rulebook:** `rulebooks/intelligent-remediation.yml`

## Customization

### Change Organization

Update all files:
```yaml
organization: "Your-Org-Name"
```

### Use Different Inventory

Update `controller_templates.yaml`:
```yaml
inventory: "Your-Production-Inventory"
```

### Customize Thresholds

Update extra_vars in `controller_templates.yaml`:
```yaml
extra_vars:
  cpu_threshold: 90  # Instead of default 80
  log_retention_days: 60  # Instead of default 30
```

## Testing

After applying CaC:

1. **Test Job Templates Manually:**
   ```bash
   awx job_templates launch "Remediate Disk Space" \
     --limit "test-server-01" \
     --extra_vars '{"partition": "/tmp"}'
   ```

2. **Test EDA Rulebook:**
   ```bash
   curl -X POST https://eda-controller:5000/webhook \
     -H "Content-Type: application/json" \
     -d @~/ansible/ansible-aiops/test-events/case1-disk-full.json
   ```

3. **Monitor in AAP UI:**
   - Jobs → All jobs (see job executions)
   - Automation Decisions → Rulebook Activations (see event routing)

## Maintenance

### Update Playbook Repository

```bash
# Update project in AAP
awx projects update "AIOps Remediation"
```

### Modify Job Templates

Edit `controller_templates.yaml` and re-apply:
```bash
ansible-playbook configure-controller.yml \
  -e @cac_sample/aiops/controller_templates.yaml
```

### Update Rulebook

1. Update rulebook in git repository
2. Update EDA project:
   ```bash
   awx-eda projects update "AIOps Rulebooks"
   ```
3. Rulebook activation will automatically use new version

## Troubleshooting

### Issue: Job template creation fails

**Check:**
- Projects exist and synced?
- Credentials configured?
- Inventory exists?

### Issue: EDA rulebook activation fails

**Check:**
- Decision environment configured?
- EDA credentials correct?
- Rulebook syntax valid?

### Issue: Jobs don't launch from EDA

**Check:**
- AAP credential attached to EDA activation?
- Job template names match exactly?
- Webhook port (5000) accessible?

## Documentation

See also:
- `~/ansible/ansible-aiops/AAP-JOB-TEMPLATES-SETUP.md`
- `~/ansible/ansible-aiops/DEPLOYMENT-GUIDE.md`
- `~/ansible/ansible-aiops/REMEDIATION-PLAYBOOKS-SUMMARY.md`
