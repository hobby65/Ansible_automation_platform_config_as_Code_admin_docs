## Base Configuration of Automation Controller

As earlier mentioned, the configuration of controller is stored as an  
inventory in a GIT repository. See below for the structure to create in the
repository:

```text
.
├── collections
│   └── requirements.yml
├── group_vars
│   └── all
│       ├── credential_types.yaml.example
│       ├── execution_environments.yaml
│       ├── instance_groups.yaml
│       ├── settings.yaml
│       ├── team_roles.yaml
│       ├── credentials.yaml
│       ├── inventory_sources.yaml
│       ├── inventory.yaml
│       ├── notification_templates.yaml
│       ├── organization.yaml
│       ├── projects.yaml
│       ├── schedules.yaml
│       ├── teams.yaml
│       ├── templates.yaml
│       ├── users.yaml
│       └── user_roles.yaml
├── host_vars
│   └── controller
│       └── controller_auth.yaml
├── inventory.yaml
├── main.yml
└── README.md

```
As you can see in the above example, the playbook and inventory are in an single repository.  
First we will focus on the playbook and its dependencies:  

### Playbook config

The playbook here is main.yml and has the following contents:

```yaml
---
- hosts: controller
  connection: local
  roles:
    - redhat_cop.controller_configuration.dispatch
```

It depends on the redhat_cop.controller_configuration collection, which in turn depends on the  
ansible.controller or awx.awx collection, these are in the collections requirements.yml file:  

```yaml
collections:
  - awx.awx
  - redhat_cop.controller_configuration
```

to initialize the inventory the inventory.yml is present in the root of the git repository.  

```yaml
---
dev:
  hosts: controller_dev
test:
  hosts: controller_test
accp:
  hosts: controller_accp
prod:
  hosts: controller_prod
```

This is all that is needed to run configuration as code for the controller, once you have 
filled the inventory. The inventory is the most important part of CaC...

### Inventory step by step config

the inventory starts inthe following directory:  
**host_vars/controller_xx**

There must be a directory for each environment controller, so a controller_dev, controller_test  
and so on...

#### Authentication to controller for the playbook

First the config as code play needs to know a few things:  
- where is the controller  
- how to logon  

These settings are kept in a single file, called: controller_auth.yaml
```yaml
---
controller_hostname: __resolvable.controller.hostname__
controller_validate_certs: false
controller_username: __user_with_system_admin_rights__
controller_password: __password__
```
As there are secrets in here, it is best to keep htee safe and encrypted, at least with  
ansible-vault, or pull the secrets from other source, like git or a secret server.

#### Start filling controller items

To create the basic implementation of controller, we will run you through the files and  
discuss the entries in the files. Some may be obvious, but we will discuss them anyway.  

**./group_vars/all/execution_environments.yaml**  

Example:
```yaml
---
controller_execution_environments_all:
  - name: Control Plane Execution Environment
    description: 
    image: registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8:latest
    pull: 
    credential: Default Execution Environment Registry Credential
  - name: Default execution environment
    description: 
    image: registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8:latest
    pull: 
    credential: Default Execution Environment Registry Credential
  - name: Minimal execution environment
    description: 
    image: registry.redhat.io/ansible-automation-platform-24/ee-minimal-rhel8:latest
    pull: 
    credential: Default Execution Environment Registry Credential
  - name: Automation Hub Default execution environment
    description: 
    image: privatehub.localdomain/ee-supported-rhel8:latest
    pull: 
    credential: Default Execution Environment Registry Credential
  - name: Automation Hub Minimal execution environment
    description: 
    image: privatehub.example.com/ee-minimal-rhel8:latest
    pull: 
    credential: Default Execution Environment Registry Credential
...
```
In the above file you'll see every execution environment that is present in the controller  
at deployment time, additional there is a local defined execution environment hosted on  
a private automation hub.
Here you can add execution environments to your controller, which need to be availlable to  
the entire organization.

**./group_vars/all/credential_types.yaml**  

Example:
```yaml
---
controller_credential_types_all:
 - name: example credential
   kind: cloud
   inputs:
     - id: password
       label: Password
       help_text: A password
       type: string
       multiline: false
       secret: true
     required:
       - password
   injectors:
     env:
       EXAMPLE_CRED_PASSWD: "{{ password }}"
...
```
See the documentation for more options regarding the defition of credential_types. As only system  
administrators can create credential_types, it is not the preferred way of using credentials.  


**./group_vars/all/instance_groups.yaml**       

Example:
```yaml
---
controller_instance_groups_all:
  - name: controlplane
    policy_instance_minimum: 0
    policy_instance_percentage: 100
    instances:
      - controller_hostname
  - name: default
    policy_instance_minimum: 0
    policy_instance_percentage: 100
    instances:
      - controller_hostname
...
```
In this file you define the instance group en resources usage for the controller and the execution nodes.  
In this example, only a single controller is present.  

**./group_vars/all/settings.yaml**

Example:
```yaml
---
controller_settings_all:
  - settings:
      ACTIVITY_STREAM_ENABLED: true
      ACTIVITY_STREAM_ENABLED_FOR_INVENTORY_SYNC: false
      AUTOMATION_ANALYTICS_GATHER_INTERVAL: 14400
      AUTOMATION_ANALYTICS_LAST_ENTRIES: ''
      DEFAULT_EXECUTION_ENVIRONMENT: null
      INSIGHTS_TRACKING_STATE: true
      INSTALL_UUID: faf8ca8f-71e2-47a4-a7ce-bd263342edfb2
      LICENSE:
          account_number: '000000'
          instance_count: 100
          license_date: 1706936399
          license_type: trial
          pool_id: 2c9404a586a120830044034da09d9
          product_name: Red Hat Ansible Automation Platform
          satellite: null
          sku: SER0496
          subscription_id: '00000000'
          subscription_name: Red Hat Ansible Automation Platform
          support_level: Self-Support
          trial: false
          usage: ''
          valid_key: true
      MANAGE_ORGANIZATION_AUTH: true
      ORG_ADMINS_CAN_SEE_ALL_USERS: true
      PENDO_TRACKING_STATE: detailed
      PROXY_IP_ALLOWED_LIST: []
      REDHAT_PASSWORD: ''
      REDHAT_USERNAME: ''
      REMOTE_HOST_HEADERS:
      - REMOTE_ADDR
      - REMOTE_HOST
      SUBSCRIPTIONS_PASSWORD: Password
      SUBSCRIPTIONS_USERNAME: subscription@example.com
      TOWER_URL_BASE: https://controler.hostname
      UI_NEXT: true
...
```

In this settings file the base settings for controller are defined, including licensing information!  
Keep this file encrypted at all times. Even the account password for the subscription holder is in  
this file..


**./group_vars/all/credentails.yaml** 

Example:
```yaml
---
controller_credentials_all:
  - name: default_infra_vault
    description: vault credential for infra
    credential_type: Vault
    organization: Default
    inputs:
      vault_id: infra
      vault_password: password

  - name: automation_hub_token_published
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: Default
    inputs:
      auth_url: ''
      token: "{{ token }}"
      url: https://privatehub.example.com/api/galaxy/content/published/

  - name: automation_hub_token_rh_certified
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: Default
    inputs:
      auth_url: ''
      token: "{{ token }}"
      url: https://privatehub.example.com/api/galaxy/content/rh_certified/

  - name: ansible
    description: 
    credential_type: Machine
    organization: Default
    inputs:
      become_method: sudo
      become_username: ''
      ssh_key_data: |
        -----BEGIN OPENSSH PRIVATE KEY-----
              key-data
        -----END OPENSSH PRIVATE KEY-----
      username: ansible

  - name: git
    description: 
    credential_type: Source Control
    organization: Default
    inputs:
      ssh_key_data: |
        -----BEGIN OPENSSH PRIVATE KEY-----
                key-data
        -----END OPENSSH PRIVATE KEY-----
      username: AAP_git_user
...
```

In this file you define the secrets for to be known by the "Default" organization, if you don't use the default  
organization for running playbooks, or checking inventories, no secrets are needed here..
It could be usefull to configure the galaxy tokens anyway, to ensure you can checkout execution env's.


**./group_vars/all/templates.yaml**

Example:
```yaml
controller_job_templates_all:

  - name: CaC_config_template
    description: Config As Code AAP
    organization: Default
    project: CaC_project
    inventory: CaC_inventory
    playbook: main.yml
    job_type: run
    fact_caching_enabled: false
    credentials:
      - ansible
      - infra_vault
    concurrent_jobs_enabled: false
    ask_scm_branch_on_launch: false
    ask_tags_on_launch: false
    ask_verbosity_on_launch: false
    ask_variables_on_launch: true
    extra_vars:
      instances: localhost
    execution_environment: Default execution environment
    survey_enabled: false
    survey_spec: {}
```

The only template at the moment in the Default organization is the CaC template, to run configuration updates.
Just add more templates as needed...

**./group_vars/all/projects.yaml**

Example:
```yaml
---
controller_projects_all:

  - name: CaC_project
    organization: Default
    scm_branch: master
    scm_type: git
    scm_update_on_launch: true
    scm_credential: git
    scm_url: git@git.example.com:project/aap_cac_automation.git

  - name: inventory
    description: inventory project
    organization: Default
    scm_type: git
    scm_url: git@git.example.com:project/inventory_test.git
    scm_credential: git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: true
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0
...
```
Define the projects needed by the job_templates/inventories here. Be sure to note the names,  
you need them to add the job_templates/inventories.

**./group_vars/all/inventory.yaml**

Example:
```yaml
---
controller_inventories_all:
  - name: inventory_test
    description: Default inventory
    organization: Default

  - name: CaC_inventory
    description: CaC inventory
    organization: Default

controller_inventory_sources_all:
  - name: inventory
    description: 
    organization: Default
    source: scm
    source_project: inventory
    source_path: hosts.yaml
    inventory: inventory_test
    update_on_launch: true
    overwrite: true

  - name: CaC_inventory
    description: 
    organization: Default
    source: scm
    source_project: CaC_project
    source_path: inventory.yaml
    inventory: CaC_inventory
    update_on_launch: true
    overwrite: true
...
```

Here 2 inventories are defined, each with a different inventory source, both inventories  
are sourced from a project.

**./group_vars/all/schedules.yaml**

Example:
```yaml
---
controller_schedules_all:
  - name: Sync Private Hub
    description: Sync Private hub repos
    unified_job_template: sync_private_hub
    rrule: "DTSTART:20230711T110000Z RRULE:FREQ=DAILY;INTERVAL=1;BYDAY=TU,TH"
...
```

Scheduled jobs can be defined as above, the tricky part here is to get the RRULE correct.  
Its not well documented, but you can find it on the net.  


**./group_vars/all/users.yaml**

Example:
```yaml
---
controller_user_accounts_all:
- username: deploy
  password: password
  email: 
  first_name: deploy
  last_name: 
  auditor: false
  superuser: false
  update_secrets: false

- username: super
  password: superpass
  email: su.per@example.com
  first_name: su
  last_name: per
  auditor: false
  superuser: true
  update_secrets: false
...
```

Define users for the Default organization, the superusers should be defined in any case,  
so you can set the password to a initial value when update_secrets is set to true.
At least one should be defined, so you wont lose access if AD fails...
The admin user is defined on installation and could be overuled here..


**./group_vars/all/teams.yaml**

Example:
```yaml
---
controller_teams_all:
  - name: admins
    description: Admin users
    organization: Default

  - name: deploy
    description: deployment users
    organization: Default
...
```
Define teams you need in the default organization, there shouldn't be many.  

**./group_vars/all/team_roles.yaml**

Example:
```yaml
---
controller_roles_all:
  - team: deploy
    job_template: CaC_config_template
    role: execute
...
```
In this file you define what a non-superuser can run as templates, or even administer..  
Here a member of the deploy team can run the config template, nothing else..

**./group_vars/all/user_roles.yaml**

Example:
```yaml
---
controller_roles_all:
  - user: deploy
    organizations:
      - Default
    role: member
  - user: deploy
    target_teams:
      - deploy
    role: member
...
```

In this file the deploy user is made a member of the deploy team in the default organization..  

This concludes the base configuration of the controller..

## Per environment config
All the above configurations are global, they will be applied to all instances of AAP in your environment.
To configure items for an environment specific, create a branch for the environment and create the files needed.
For every file in the all directory, there should be a file in every branch for an environment.
If a file is missing, you will get an error while running the playbook.
The keys in the files should be adapted for the branch_name...
**example**
In the example we created a dev branch and add credentials for the controller in this branch.
```yaml
---
controller_credentials_dev: []


...
```

You can use this schema for each of the controller files.

[Back](README.md)
