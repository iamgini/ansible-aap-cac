# Configuration as Code for Ansible Automation Platform

This Ansible playbooks and roles allows for easy interaction with an Ansible Controller server via Ansible roles using the Controller collection modules.

> ⚠️ **Important Note:** AAP Version Dependency.
>
> Use the new collection `infra.aap_configuration` for AAP 2.5 or later.
>
> Be sure to use [`ansible.controller 4.5.12`](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/controller/) for AAP 2.4




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
  - [How to use the playbooks for Exporting content from AAP](#how-to-use-the-playbooks-for-exporting-content-from-aap)
  - [Enable Webhook for automated CaC update.](#enable-webhook-for-automated-cac-update)
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

## How to use the playbooks for Exporting content from AAP

```shell
$ ansible-playbook playbooks/controller_export.yaml
```

Refer to the [Automation Controller Export Documentation](https://github.com/redhat-cop/infra.aap_configuration/blob/devel/docs/EXPORT_README.md) and [export module documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/export_module.html)

## Enable Webhook for automated CaC update.

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