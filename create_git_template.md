## Create template git repo for CaC

In git you first create a standard repository and fill this repository  
with the template files needed to configure an organization.  

Fill the repository with the files that are listed below.
Change the repository type to template repository.
If these steps are completed, you can create new repositories with the standard  
content as created in this template.

The files needed in this template repository are:

```
.
├── collections
│   └── requirements.yml
├── group_vars
│   └── all
│       ├── credentials.yaml
│       ├── inventory.yaml
│       ├── organization.yaml
│       ├── projects.yaml
│       ├── roles.yaml
│       ├── teams.yaml
│       ├── templates.yaml
│       └── users.yaml
├── host_vars
│   └── controller
│       └── controller_auth.yaml
├── inventory.yaml
├── main.yml
└── README.md
```

The files inside the all directory are already discussed and are the same as in the   
base config. These files will be encrypted with ansible-vault the password is not shared. 
As this is a template repository, the files in this repository are not complete and can   
be used as templates to fill the organization.

We will discuss each file and list its content... as template.  
In the user documentation we discuss how to fill the templates with real data.  

**credentials.yaml**

```yaml
---
controller_credentials_all:

  - name: ORG_automation_hub_token_published
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: ORG
    inputs:
      auth_url: ''
      token: {{ token }}
      url: https://privatehub.localdomain/api/galaxy/content/published/

  - name: ORG_automation_hub_token_rh_certified
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: ORG
    inputs:
      auth_url: ''
      token: {{ token }}
      url: https://privatehub.localdomain/api/galaxy/content/rh_certified/


  - name: ORG_ansible
    description: 
    credential_type: Machine
    organization: ORG
    inputs:
      become_method: sudo
      become_username: ''
      ssh_key_data: |
        -----BEGIN OPENSSH PRIVATE KEY-----
              key-data
        -----END OPENSSH PRIVATE KEY-----        
      username: ansible

  - name: ORG_git
    description: 
    credential_type: Source Control
    organization: ORG
    inputs:
      ssh_key_data: |
        -----BEGIN OPENSSH PRIVATE KEY-----
                key-data
        -----END OPENSSH PRIVATE KEY-----        
      username: git_user
...
```

**inventory.yaml**

```yaml
---
controller_inventories_all:
  - name: ORG_inventory
    description: ORG inventory
    organization: ORG

controller_inventory_sources:
  - name: ORG_inventory
    description: 
    organization: ORG
    source: scm
    source_project: ORG_inventory
    source_path: hosts.yaml
    inventory: ORG_inventory
    update_on_launch: true
    overwrite: true
...
```

**job_templates.yaml**

```yaml
---
controller_job_templates_all:
---
  - name: ORG_template1
    description: description for the template
    organization: ORG
    project: ORG_project1
    inventory: ORG_inventory
    playbook: main.yml
    job_type: run
    fact_caching_enabled: false
    credentials:
      - ansible
    concurrent_jobs_enabled: false
    ask_scm_branch_on_launch: false
    ask_tags_on_launch: false
    ask_verbosity_on_launch: false
    ask_variables_on_launch: true
    extra_vars:
      var_name: var_content
    execution_environment: Default execution environment
    survey_enabled: false
    survey_spec: {}
...
```

**organization.yaml**

```yaml
---
controller_organizations_all:
  - name: NEW_ORG
    description: New Organization description
    galaxy_credentials:
      - ORG_automation_hub_token_rh_certified
      - ORG_automation_hub_token_published
    assign_galaxy_credentials_to_org: true

...
```

**projects.yaml**

```yaml
---
controller_projects_all:

  - name: ORG_project1
    description: project1
    organization: NEW_ORG
    scm_type: git
    scm_url: ssh_url
    scm_credential: ORG_git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: true
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0

  - name: ORG_project2
    description: project2
    organization: NEW_ORG
    scm_type: git
    scm_url: ssh_url
    scm_credential: ORG_git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: true
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0

  - name: ORG_inventory
    description: inventory project
    organization: NEW_ORG
    scm_type: git
    scm_url: ssh_url
    scm_credential: ORG_git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: true
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0
...
```

**roles.yaml**

```yaml
---
controller_roles_all:
  - user: user1
    organizations:
      - NEW_ORG
    role: member

  - user: user1
    target_teams:
      - ORG_devel
    role: member

  - team: ORG_devel
    job_templates: 
      - ORG_template1
    role: execute

  - team: ORG_devel
    credentials:
      - ORG_git
      - ORG_ansible
    role: use
      
  - team: ORG_devel
    inventory: ORG_inventory
    role: use

  - team: ORG_devel
    projects:
      - ORG_project1
      - ORG_project2
    role: use
...
```

**teams.yaml**

```yaml
---
controller_teams_all:
  - name: ORG_devel
    description: development users
    organization: NEW_ORG
...
```

**users.yaml**  (not used in AD or LDAP)

```yaml
---
controller_user_accounts_all:
- username: ORG_devel
  password: password
  email: 
  first_name: developer
  last_name: org
  auditor: false
  superuser: false
  update_secrets: false
...
```

There is one other file needed for the pipeline to work, and that is (in case of gitea)
.gitea/workflows/CaC_pipeline.yaml

```yaml
name: Gitea Actions Config As Code for AAP
run-name: ${{ gitea.actor }} is runnig CaC for AAP
on: 
  push:
    branches: [master]
jobs:
  CaC-Pipeline-Run:
    runs-on: ubuntu-latest
    steps:
      - name: Run apt-get update
        run: apt-get update
      - name: Install pip3
        run: apt-get install -y python3-pip
      - name: Install ansible
        run: pip3 install ansible
      - name: Install collections
        run: |
          ansible-galaxy collection install awx.awx
          ansible-galaxy collection install redhat_cop.controller_configuration
      - name: Checkout config
        uses: actions/checkout@v3
      - name: Run CaC to configure controller
        run: |
          ansible-playbook main.yml -i inventory.yaml -e instance=controller_${{ gitea.ref_name }} -e branch_name=${{ gitea.ref_name }}
```

in case of GitLab this will be a gitlab ci runner file..
The steps are almost equal..

With this template in place it is very easy to deploy a new organization, just create a new repository,
based on this template. The files will be copied and the CI_runner is already in place.  
Just edit the files in the repository to reflect the new organization and push them to the repository.
After that enable the pipeline runner, and ensure the pipeline is run...on update.
Then every update to the repository will trigger the pipeline and push the configuration into AAP.

[Back](org_config.md)

