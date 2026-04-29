1. Multiple Requirements Files (Simplest)

  Create version-specific requirements files:
  - requirements-aap24.yaml - with ansible.controller: 4.5.12
  - requirements-aap25.yaml - with infra.aap_configuration
  - requirements-aap26.yaml - with latest collections

  Install based on target:
  ansible-galaxy collection install -r requirements-aap24.yaml
  # or
  ansible-galaxy collection install -r requirements-aap25.yaml

  2. Different Execution Environments (Best for ansible-navigator)

  Use different EE images per AAP version in ansible-navigator.yml or via CLI:
  # AAP 2.4
  ansible-navigator run playbook.yml --eei
  registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8:latest

  # AAP 2.5
  ansible-navigator run playbook.yml --eei
  registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest

  Each EE has the correct collections pre-installed.

  3. Different Collections Paths

  Install collections to version-specific directories:
  ansible-galaxy collection install -r requirements-aap24.yaml -p ./collections/aap24
  ansible-galaxy collection install -r requirements-aap25.yaml -p ./collections/aap25

  Then specify the path when running:
  ANSIBLE_COLLECTIONS_PATH=./collections/aap24 ansible-playbook playbook.yml

  4. Environment Variables + Conditional Logic

  export AAP_VERSION=2.4  # or 2.5, 2.6



  5. Separate Playbooks/Directories

  Structure like:
  playbooks/
  ├── aap24/
  │   └── configure-aap.yaml
  ├── aap25/
  │   └── configure-aap.yaml

  Recommended Combination

  Most teams use #1 + #2 together:
  - Multiple requirements files for local development
  - Different EE images for each AAP version (containerized consistency)
  - Environment variable (AAP_VERSION) to switch behavior in shared playbooks if needed

  This gives you flexibility to test locally (ansible-playbook + right requirements file) or in
  CI/CD (ansible-navigator + right EE image).