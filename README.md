# Configuration as Code for Ansible Automation Platform

This Ansible playbooks and roles allows for easy interaction with an Ansible Controller server via Ansible roles using the Controller collection modules.

> ⚠️ **Important Note:** AAP Version Dependency.
>
> Use the new collection `infra.aap_configuration` for AAP 2.5 or later.
>
> Be sure to use [`ansible.controller 4.5.12`](https://console.redhat.com/ansible/automation-hub/collections/published/ansible/controller/install?version=4.5.12) for AAP 2.4.


**Quick links:**

- `ansible.platform` (): This collection contains modules that can be used to automate the creation of resources on an install of Ansible Automation Platform.
- `infra.ee_utilities` [Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/infra/ee_utilities/) | [GitHub](https://github.com/redhat-cop/ee_utilities): Execution Environment creation utilities

**Table of Content**

- [Configuration as Code for Ansible Automation Platform](#configuration-as-code-for-ansible-automation-platform)
  - [What is Configuration as Code (CaC) in Ansible Automation Platform?](#what-is-configuration-as-code-cac-in-ansible-automation-platform)
  - [Requirements](#requirements)
  - [Ansible Automation Platform CaC Collections](#ansible-automation-platform-cac-collections)
  - [Other Configuration Collections for Ansible Automation Platform](#other-configuration-collections-for-ansible-automation-platform)
  - [Configuring Credential](#configuring-credential)
  - [Encrypting Sensitive data](#encrypting-sensitive-data)
  - [How to use the playbooks for configuring AAP](#how-to-use-the-playbooks-for-configuring-aap)
    - [Method 1: Using `ansible-playbook`](#method-1-using-ansible-playbook)
    - [Method 2: Using `ansible-navigator`](#method-2-using-ansible-navigator)
    - [Method 3: Using automation controller job template](#method-3-using-automation-controller-job-template)
    - [Controlling the controller configurations](#controlling-the-controller-configurations)
  - [Exporting content from AAP](#exporting-content-from-aap)
    - [AAP 2.4](#aap-24)
  - [Enable Webhook for automated CaC update (Local Testing Only)](#enable-webhook-for-automated-cac-update-local-testing-only)
    - [Using ngrok for exposing AAP and enable GitHub webhook](#using-ngrok-for-exposing-aap-and-enable-github-webhook)
  - [Troubleshooting](#troubleshooting)
  - [References](#references)


## What is Configuration as Code (CaC) in Ansible Automation Platform?

**Configuration as Code (CaC or CasC)** is the practice of managing configuration settings in source control, separate from your application logic or UI. The goal is simple: make configurations easy to version, share, audit, and deploy—just like code.

In the context of **Ansible Automation Platform**, you can bring this idea to life using automation controller features. Combine CaC with a GitOps workflow and you’ve got a solid setup for managing and replicating configurations across environments—whether it's dev, staging, or production.

This becomes especially useful when you're dealing with large or complex systems. Instead of manually tweaking settings, you define them once (in code), store them in Git, and let automation handle the rest. That means:

- **Version control**: Track every change, rollback when needed, and collaborate like you would with application code.
- **Consistency**: Avoid drift by applying the same configs across all your automation controller instances.
- **Repeatability**: Spin up identical environments without manual effort.
- **Automation**: Trigger deployments or updates directly from your Git repo.
- **Auditing & compliance**: Know exactly who made changes, when, and what changed.

Bottom line: CaC gives you control, clarity, and confidence when managing automation controller configurations—especially at scale.


## Requirements

1. `awxkit >= 9.3.0`

```shell
$ pip3 install awxkit --user
```

2. Credential to access the Ansible Automation Platform

3. Ansible Automation Platform Collections

Install required collections as per `requirements.yaml`

```yaml
---
collections:
  - name: ansible.platform
  - name: ansible.hub
  - name: ansible.controller
  - name: ansible.eda
  - name: infra.aap_configuration
  - name: infra.aap_configuration_extended
```

```shell
$ ansible-galaxy collection install -r requirements.yaml
```

(Or download and install for disconnected environment)

## Ansible Automation Platform CaC Collections

| Collection Name | Purpose  | Resources |
|:---------|:------------|:------------|
| `infra.aap_configuration` | Main collection of roles to manage Ansible Automation Platform 2.5+ with code | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/validated/infra/aap_configuration/content/), [GitHub](https://github.com/redhat-cop/infra.aap_configuration), [Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/infra/aap_configuration) |
| `infra.aap_configuration_extended` | Where other useful roles that don't fit here live | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/validated/infra/aap_configuration_extended/)|
| `ansible.platform` | gateway/platform modules | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/platform/) (no public repo for this collection) |
| `ansible.hub` | Automation hub modules | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/hub/) , [GitHub](https://github.com/ansible-collections/ansible_hub)  |
| `ansible.controller` | Automation controller modules | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/controller/), [awx.awx on GtiHub](https://github.com/ansible/awx/tree/devel/awx_collection), [GitHub](https://github.com/ansible/awx/tree/devel/awx_collection) |
| `ansible.eda` | Event Driven Ansible modules | [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/eda/), [GitHub](https://github.com/ansible/event-driven-ansible) |

## Other Configuration Collections for Ansible Automation Platform

| Collection Name  |  Purpose  | Resources |
|:------------|:-----------|:-----------|
| [EE Utilities](https://github.com/redhat-cop/ee_utilities)                                            | Execution Environment creation utilities          |
| [AAP installation Utilities](https://github.com/redhat-cop/aap_utilities)                             | Ansible Automation Platform Utilities             |
| [AAP Configuration Template](https://github.com/redhat-cop/aap_configuration_template)                | Configuration Template for this suite             |
| [Ansible Validated Gitlab Workflows](https://gitlab.com/redhat-cop/infra/ansible_validated_workflows) | Gitlab CI/CD Workflows for ansible content        |
| [Ansible Validated Github Workflows](https://github.com/redhat-cop/infra.ansible_validated_workflows) | Github CI/CD Workflows for ansible content        |
| `infra.ah_configuration` | Old collections (before AAP 2.5) | [GitHub](https://github.com/ansible/automation_hub_collection), [Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/validated/infra/ah_configuration/) |
| `infra.aap_utilities` | AAP installation Utilities | [GitHub](https://github.com/redhat-cop/aap_utilities), [Galaxy](https://galaxy.ansible.com/ui/repo/published/infra/aap_utilities/) |
| `aap_configuration_template` | AAP Configuration Template | [GitHub](https://github.com/redhat-cop/aap_configuration_template) |

## Configuring Credential

When using `ansible-playbook` or `ansible-navigator`, the credential can be passed as environment variables; configure the credential as follows.

```shell
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=secretpassword
export CONTROLLER_HOST=https://aap25.lab.iamgini.com
export CONTROLLER_VERIFY_SSL=false

export AAP_ORGANIZATION=AwesomeCorp
export AAP_ENVIRONMENT=uat
```

## Encrypting Sensitive data

You can use the vaulted files using `ansible-vault` or vaulted text as inline in the config files.

```shell
$ ansible-vault encrypt_string string_to_encrypt=Awesome123
New Vault password:            --> I used "ansible" as the vault password here
Confirm New Vault password:
Encryption successful
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          38636432353066366436633665353033626134353632343436643533353562353266626136666666
          6334656538383934336535346433646638333664646230660a646563633761316361323662333730
          33626434333534646264313331636261326532303564353933666562393366643038356536326230
          6236326566643832350a616138383932613535363130356662313666623033396438313439653266
          32393330373131643333313630643737336334343764383236383435643337663034
```

Now, add this vaulted data as inline text as follows.

e.g: `aap-cac-filetree/AwesomeCorp/env/common/aap_users.d/aap_user_accounts.yml`

```yaml
---
platform_state: present
aap_user_accounts:
  - username: jdoe
    password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          38636432353066366436633665353033626134353632343436643533353562353266626136666666
          6334656538383934336535346433646638333664646230660a646563633761316361323662333730
          33626434333534646264313331636261326532303564353933666562393366643038356536326230
          6236326566643832350a616138383932613535363130356662313666623033396438313439653266
          32393330373131643333313630643737336334343764383236383435643337663034
```


## How to use the playbooks for configuring AAP

The `playbooks/configure-aap.yaml` can be executed using `ansible-playbook`, `ansible-navigator` or using a Job template from **Ansible controller** itself.

### Method 1: Using `ansible-playbook`

For flattened configuration

```shell
$ ansible-playbook playbooks/configure-aap.yaml
```

For filetree configuration

```shell
$ ansible-playbook playbooks/configure-aap-using-filetree.yaml \
   -e "{orgs: ${AAP_ORGANIZATION}, dir_orgs_vars: ../cac_filetree, env: ${AAP_ENVIRONMENT} }"  \
   -e @orgs_vars/env/${AAP_ENVIRONMENT}/configure_connection_controller_credentials.yml \
   --tags ${CONTROLLER_OBJECT} \
   --vault-password-file ./.vault_pass.txt
```

Check  'collections/ansible_collections/infra/aap_configuration_extended/roles/filetree_read/defaults/main.yml`

### Method 2: Using `ansible-navigator`

```shell
$ ansible-navigator run playbooks/configure-aap.yaml \
  --penv CONTROLLER_USERNAME \
  --penv CONTROLLER_PASSWORD \
  --penv CONTROLLER_HOST
```

### Method 3: Using automation controller job template

Login to the automation controller (from where you are going to execute the CaC operation),

- Step 1. A Git repo for Configuration as Code - Store this content in a Git repo as it is.

- Step 2: Create the Project in Automation controller with the CaC repo as source and add Git credential if required.

- Step 3: Create a `Red Hat Ansible Automation Platform` type credential and enter the controller host, username and password for the controller.

- Step 4: Create a Job template (eg: `CaC-Controller-Configuration`) using the playbook `controller_configure.yaml`
  - Step 4.1: Attach the previously created `Red Hat Ansible Automation Platform` type credential.
  - Step 4.2: Enable `Prompt on Launch` for the `Job Tags` as we need to control the execution.
  - Step 4.3: Add a tag `none` in the `Job Tags` text box. (This is to avoid any accidental execution.)

- Step 5: Execute the playbook with appropriate tags as explained in the next section.

### Controlling the controller configurations

- If the [credential sources](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/controller/content/module/credential_input_source/?sort=-pulp_created) are enabled (automated password and token input), all of the resources can be created in a one batch.
- If the credential input (password, token and so on) needs to enter manually, then break the job execution as follows.

Step 1: Launch the job template `CaC-Controller-Configuration` with the following tags: `settings`, `organizations`, `credential_types`, `credentials`

Step 2: Login to the target automation controller (if this is a different controller), edit the source control credentials with correct password (this is for syncing the project with correct credentials). You can also update other credential passwords and tokens at this time.

Step 3. Continue the CaC configuration with the remaining tags: `projects`, `inventories`, `hosts`, `templates`, `workflows` etc.

## Exporting content from AAP

Export current AAP configurations to YAML files for version control and migration between environments.

### AAP 2.4

**Prerequisites**: Install AAP 2.4 compatible collections

```shell
$ ansible-galaxy collection install \
  -r collections/requirements-aap24.yml

# Either from Automation Hub or use the from this repo
$ ansible-galaxy collection install backup-collections/ansible-controller-4.5.12.tar.gz --force
```

**Configure credentials** as environment variables:

```shell
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=secretpassword
export CONTROLLER_HOST=https://aap24.lab.example.com
export CONTROLLER_VERIFY_SSL=false
export AAP_VERSION=2.4
```

**Export configurations**:

```shell
# Flatten output (single directory structure)
$ ansible-playbook playbooks/controller-export-24.yaml

# Filetree output (hierarchical organization-based structure)
$ ansible-playbook playbooks/controller-export-24.yaml \
  -e "flatten_output=False"
```

> **Note**: Use capital `False` (not lowercase `false`) as Ansible treats the string `"false"` as truthy. Alternatively, use JSON format: `-e '{"flatten_output": false}'`

**Output locations**:
- Flatten: `playbooks/cac_exported_aap24_flatten/`
- Filetree: `playbooks/cac_exported_aap24_filetree/`

**References**:
- [Automation Controller Export Documentation](https://github.com/redhat-cop/infra.aap_configuration/blob/devel/docs/EXPORT_README.md)
- [Export Module Documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/export_module.html)

## Migrating AAP Configurations Between Versions (2.4 to 2.5/2.6)

This section documents lessons learned from production migrations and provides a tested workflow for migrating AAP configurations from version 2.4 to 2.5/2.6.

### Migration Strategy: Flatten vs Filetree

**For Migrations**: Use **flattened configuration** approach
- Simpler, streamlined workflow
- Easier to troubleshoot and manually fix issues
- Single directory structure for all resources

**For Ongoing CaC Management**: Use **filetree configuration** approach after migration is complete
- Better organization for multi-org, multi-environment setups
- Granular control over configurations

### Known Issues in AAP 2.4 to 2.5/2.6 Migration

#### Issue 1: Variable Name Changes

The export from AAP 2.4 using `infra.controller_configuration` generates variable names that are **incompatible** with AAP 2.5/2.6 import requirements.

**Variable name mapping required**:

| AAP 2.4 Export Variable | AAP 2.5/2.6 Import Variable | File |
|-------------------------|---------------------------|------|
| `controller_organizations` | `aap_organizations` | organizations.yaml |
| `controller_teams` | `aap_teams` | teams.yaml |
| `controller_user_accounts` | `aap_user_accounts` | users.yaml |

**Automated fix**:
```bash
# After exporting from AAP 2.4, run this transformation
EXPORT_DIR="playbooks/cac_exported_aap24_flatten"

sed -i 's/^controller_organizations:/aap_organizations:/g' ${EXPORT_DIR}/organizations.yaml
sed -i 's/^controller_teams:/aap_teams:/g' ${EXPORT_DIR}/teams.yaml
sed -i 's/^controller_user_accounts:/aap_user_accounts:/g' ${EXPORT_DIR}/users.yaml
```

#### Issue 2: Malformed YAML in Exported Data

**Symptom**: The `hosts.yaml` file contains extra YAML document markers (`---` and `...`) embedded within the file content, causing parsing errors.

**Cause**: Bug in the `infra.controller_configuration` export role.

**Manual fix required**:
```bash
# Open the file and remove internal --- and ... markers
# Keep only the first --- at the top and last ... at the bottom
vi playbooks/cac_exported_aap24_flatten/hosts.yaml
```

Example:
```yaml
# INCORRECT (as exported - has bug)
---
controller_hosts:
  - name: host1
---
...
  - name: host2
...

# CORRECT (after manual fix)
---
controller_hosts:
  - name: host1
  - name: host2
...
```

#### Issue 3: Instance Groups Don't Migrate

**Symptom**: Errors like `"Unable to assign instance group 'old-instances' - does not exist"`

**Cause**: Execution instance architecture changed between AAP 2.4 and 2.6. Instance groups from 2.4 don't exist in 2.6 and cannot be automatically migrated.

**Solution**: Remove instance group references before import, then recreate manually after migration.

**Automated cleanup**:
```bash
EXPORT_DIR="playbooks/cac_exported_aap24_flatten"

# Remove instance_groups from all resource types
for file in job_templates.yaml workflow_job_templates.yaml inventories.yaml organizations.yaml
do
  if [ -f "${EXPORT_DIR}/${file}" ]; then
    # Comment out instance_groups (preserves for reference)
    sed -i 's/^  instance_groups:/#  instance_groups: # REMOVED - reconfigure after migration/g' ${EXPORT_DIR}/${file}
    sed -i 's/^    - /#    - # REMOVED/g' ${EXPORT_DIR}/${file}
  fi
done
```

**Post-migration**: Recreate instance groups in AAP 2.6 UI, then use CaC to reassign them to resources.

### Recommended Migration Workflow

#### Phase 1: Export from AAP 2.4

```bash
# Set credentials for source AAP 2.4
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=secretpassword
export CONTROLLER_HOST=https://aap24.lab.example.com
export CONTROLLER_VERIFY_SSL=false

# Export using flatten structure
ansible-playbook playbooks/controller-export-24.yaml -e "flatten_output=True"
```

#### Phase 2: Transform Exported Data

```bash
EXPORT_DIR="playbooks/cac_exported_aap24_flatten"

# Step 1: Fix variable names
sed -i 's/^controller_organizations:/aap_organizations:/g' ${EXPORT_DIR}/organizations.yaml
sed -i 's/^controller_teams:/aap_teams:/g' ${EXPORT_DIR}/teams.yaml
sed -i 's/^controller_user_accounts:/aap_user_accounts:/g' ${EXPORT_DIR}/users.yaml

# Step 2: Remove instance groups (automated)
for file in job_templates.yaml workflow_job_templates.yaml inventories.yaml organizations.yaml
do
  if [ -f "${EXPORT_DIR}/${file}" ]; then
    sed -i 's/^  instance_groups:/#  instance_groups: # REMOVED/g' ${EXPORT_DIR}/${file}
  fi
done

# Step 3: Fix malformed hosts.yaml (MANUAL - open in editor)
vi ${EXPORT_DIR}/hosts.yaml
# Remove extra --- and ... markers inside the file

# Step 4: Copy to import location
mkdir -p cac_sample/migration
cp -r ${EXPORT_DIR}/* cac_sample/migration/
```

#### Phase 3: Multi-Stage Import to AAP 2.5/2.6

**Why multi-stage?** Credentials cannot be exported with their secrets (passwords, SSH keys, tokens). They must be manually updated before dependent resources can be created.

**Set credentials for target AAP**:
```bash
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=your-password
export CONTROLLER_HOST=https://aap26.lab.example.com
export CONTROLLER_VERIFY_SSL=false
```

**Stage 1: Create foundation resources**
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags organizations,credential_types,credentials
```

Creates: Organizations, custom credential types, credential shells (without passwords), users, teams.

**Stage 2: Manual credential update (CRITICAL)**

⚠️ **Login to AAP 2.6 UI and update ALL credentials**:
- Source control credentials: Add Git passwords/tokens/SSH keys
- Machine credentials: Add SSH private keys and passphrases
- Cloud credentials: Add API tokens/access keys
- Custom credentials: Add required secret fields

**Why this matters**: Projects cannot sync without valid SCM credentials. Job templates cannot run without valid machine credentials.

**Stage 3: Create dependent resources**
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags projects,inventories,hosts,groups
```

**⚠️ Wait for all project syncs to complete** before proceeding. Check project status in AAP UI.

**Stage 4: Create automation resources**
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags templates,workflows,schedules
```

**Stage 5: RBAC and notifications**
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags roles,notifications
```

#### Phase 4: Recreate Instance Groups

Instance groups must be manually recreated in AAP 2.6:

1. In AAP UI: Navigate to **Administration → Instance Groups**
2. Create instance groups matching your architecture needs
3. Update CaC files to assign instance groups:

Create `cac_sample/migration/instance_group_assignments.yaml`:
```yaml
---
controller_job_templates:
  - name: "My Job Template"
    organization: "MyOrg"
    instance_groups:
      - "default"

controller_inventories:
  - name: "Production Inventory"
    organization: "MyOrg"
    instance_groups:
      - "production-instances"
```

4. Apply assignments:
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags inventories,templates
```

### Migration Verification Checklist

After migration, verify:

- [ ] All organizations exist in AAP 2.6
- [ ] Users can login (test a sample of users)
- [ ] All projects sync successfully (no failed status)
- [ ] Inventories show correct hosts and groups
- [ ] Test job templates execute successfully
- [ ] Workflow templates execute end-to-end
- [ ] Schedules are active and triggering correctly
- [ ] Notification integrations work (Slack, email, etc.)
- [ ] Custom execution environments are available
- [ ] RBAC permissions are correct (spot-check user access)

### Troubleshooting Migration Issues

**Debug mode for detailed output**:
```bash
ansible-playbook playbooks/configure-aap.yaml \
  -e "aap_configs_dir=cac_sample/migration" \
  --tags organizations \
  -vvv
```

**Validate YAML syntax before import**:
```bash
yamllint cac_sample/migration/*.yaml

# Or use Python
python3 -c "import yaml; yaml.safe_load(open('cac_sample/migration/organizations.yaml'))"
```

**Check AAP logs for errors**:
```bash
# On AAP controller node
sudo tail -f /var/log/tower/tower.log
sudo grep -i error /var/log/tower/tower.log | tail -20
```

**Verify collection versions**:
```bash
ansible-galaxy collection list | grep -E "(ansible.platform|ansible.controller|infra.aap)"

# Should show:
# - infra.aap_configuration (for AAP 2.5+)
# - ansible.platform (for AAP 2.5+)
# NOT ansible.controller 4.5.12 (that's for AAP 2.4)
```

### Post-Migration: Transition to Filetree for Ongoing CaC

After successful migration, consider transitioning to filetree structure for better ongoing management:

```bash
# Export current AAP 2.6 state to filetree
ansible-playbook playbooks/export-using-filetree.yaml \
  -e "orgs=MyOrg" \
  -e "env=prod"

# This creates a clean filetree structure in cac_filetree/
# Use this for future CaC updates
```

## Enable Webhook for automated CaC update (Local Testing Only)

### Using ngrok for exposing AAP and enable GitHub webhook

> In my environment, the Ansible Automation Platform is running locally (e.g: on workstation or without a public IP), and I need to create tunnel (using [ngrok](https://ngrok.com/) here) so that GitHub can reach the Ansible Automation Platform over internet.

```shell
export NGROK_AUTH_TOKEN=Your-Authtoken
export NGROK_CUSTOM_DOMAIN=Your-custom-domain

$ podman run --net=host -it \
  -e NGROK_AUTHTOKEN=$NGROK_AUTH_TOKEN \
  ngrok/ngrok:latest \
  http https://aap-rhel-92-1.lab.local \
  --domain=$NGROK_CUSTOM_DOMAIN
```

Note:
- Using Podman container to run ngrok, but you can install ngrok locally and use it.
- Using `guiding-immortal-sunbeam.ngrok-free.app` as pre-configured domain name.
- `https://aap-rhel-92-1.lab.local` is my local URL of Ansible controller.


## Troubleshooting

(If any)

## References

- [Red Hat Communities of Practice Controller Configuration Collection](https://github.com/redhat-cop/controller_configuration/tree/devel)
- [Automation controller workflow deployment as code](https://www.ansible.com/blog/automation-controller-workflow-deployment-as-code)
- [Ansible Automation Platform 2.3 Configuration as Code Improvements](https://www.ansible.com/blog/ansible-automation-platform-2.3-configuration-as-code-improvements) - blog
- [Automation Controller using a Continuous Deployment approach](https://automationiberia.github.io/controller-casc-cd/controller-casc-cd/index.html) ([GitHub](https://github.com/automationiberia/controller-casc-cd))