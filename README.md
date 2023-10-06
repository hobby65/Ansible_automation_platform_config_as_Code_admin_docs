# AAP Configuration As Code

For the implementation of "Configuration As Code" on the AAP service,  
The following desciption guides you though this process.  
Documentation will handle 2 parts of CaC as implemented in some organizations  
to:
1. [configure the base Automation Controller](base_config.md)
2. [configure each organization in a separate repository](org_config.md)

## Requirements

This CaC implementation uses the following collections:

- ansible.controller (or awx.awx)
- redhat_cop.controller_configuration

We will use a (Gitea) pipeline to configure the organizations and the base controller from  
the git repositories thet hold the configuration for the organizations.  

The idea is, that the configuration is stored as an inventory to feed to the playbook and build 
from there..

## Howto setup a CaC pipeline

First we need a GIT repository (we can use base_config for that..). In this repository (depending on  
the git implementation) we will setup a piece of pipelien code that will run om update of one of the branches.  

For every environment we will need to create a branch, and when you imagine a DTAP environment, we will  
create the following branches in this repository:

- prod ( or production if you will, this is the main branch!)
- accp (acceptance )
- test 
- dev 

When we create new config, just as with new code, we will push this to dev first, see if it works there and then  
go on to the next env, until we reach production.  
How is this accieved? by telling the pipeline what to do through the code.  
In the example that is listed down here, we will explain how to do this on gitea actions (is like github actions).
The code for the pipeline is as follows:

```yaml
name: Gitea Actions Config As Code for AAP
run-name: ${{ gitea.actor }} is runnig CaC for AAP
on: 
  push:
    branches:
      - dev
      - test
      - accp
      - prod
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

As you can see there are really little steps to run this pipeline that configures the controllers.
The trick is, to get your inventory configured in a way, that for every environment there is a separate  
inventory, with the name of the env in it, so the pipeline can trigger it through variable substitution.  

The contents of the inventory file is almost too simple, just this:
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

In the host_vars directory there will ba a directory for the controller for this env.
in the inventory files (1 per environment) you make the controller of that environment part of the inventory group  
that holds the specific env vars, the standard vars will come from the group_vars/all..  
As an example a tree from the inventory:

```
.
├── collections
│   └── requirements.yml
├── group_vars
│   ├── accp
│   │   ├── credentials.yaml
│   │   ├── credential_types.yaml.example
│   │   ├── execution_environments.yaml
│   │   ├── instance_groups.yaml
│   │   ├── inventory_sources.yaml
│   │   ├── inventory.yaml
│   │   ├── notification_templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── schedules.yaml
│   │   ├── settings.yaml
│   │   ├── team_roles.yaml
│   │   ├── teams.yaml
│   │   ├── user_roles.yaml
│   │   └── users.yaml
│   ├── prod
│   │   ├── credentials.yaml
│   │   ├── credential_types.yaml.example
│   │   ├── execution_environments.yaml
│   │   ├── instance_groups.yaml
│   │   ├── inventory_sources.yaml
│   │   ├── inventory.yaml
│   │   ├── notification_templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── schedules.yaml
│   │   ├── settings.yaml
│   │   ├── team_roles.yaml
│   │   ├── teams.yaml
│   │   ├── user_roles.yaml
│   │   └── users.yaml
│   ├── test
│   │   ├── credentials.yaml
│   │   ├── credential_types.yaml.example
│   │   ├── execution_environments.yaml
│   │   ├── instance_groups.yaml
│   │   ├── inventory_sources.yaml
│   │   ├── inventory.yaml
│   │   ├── notification_templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── schedules.yaml
│   │   ├── settings.yaml
│   │   ├── team_roles.yaml
│   │   ├── teams.yaml
│   │   ├── user_roles.yaml
│   │   └── users.yaml
│   ├── all
│   │   ├── credentials.yaml
│   │   ├── credential_types.yaml.example
│   │   ├── execution_environments.yaml
│   │   ├── instance_groups.yaml
│   │   ├── inventory_sources.yaml
│   │   ├── inventory.yaml
│   │   ├── notification_templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── schedules.yaml
│   │   ├── settings.yaml
│   │   ├── team_roles.yaml
│   │   ├── teams.yaml
│   │   ├── user_roles.yaml
│   │   └── users.yaml
│   └── dev
│   │   ├── credentials.yaml
│   │   ├── credential_types.yaml.example
│   │   ├── execution_environments.yaml
│   │   ├── instance_groups.yaml
│   │   ├── inventory_sources.yaml
│   │   ├── inventory.yaml
│   │   ├── notification_templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── schedules.yaml
│   │   ├── settings.yaml
│   │   ├── team_roles.yaml
│   │   ├── teams.yaml
│   │   ├── user_roles.yaml
│   │   └── users.yaml
├── host_vars
│   ├── controller_dev
│   │   └── controller_auth.yaml
│   └── controller_accp
│       └── controller_auth.yaml
├── inventory.yaml
├── main.yml
└── README.md
```

The only difference between the environments are in the settings and authentication for the controllers.  
In this example, there should be some difference in the inventories as well, but in this case they could be  
dynamic, based on the environment. For the base install there should be as little difference as possible, the organizations  
make the difference where needed. 

If you push the code through the git repository branches, all controllers will be configured identically, yust as it should.  
Just imagine how easy recovery will be after a crash, if you have this in place..

The playbook main.yml is almost too simple:
It has only 2 actions:
- First we merge the variables given in the group_vars, to vars that can be used by the collection.
- Second we run the play to configure the controller

```yaml
---
- hosts: "{{ instance }}"
  connection: local

  pre_tasks:
    - name: Set the controller vars
      ansible.builtin.set_fact:
        controller_credentials: "{{ controller_credentials_all | community.general.lists_mergeby( vars['controller_credentails_' + branch_name] | default([])) }}"
        controller_credential_types: "{{ controller_credential_types_all | community.general.lists_mergeby( vars['controller_credentail_types_' + branch_name] | default([])) }}"
        controller_execution_environments: "{{ controller_execution_environments_all | community.general.lists_mergeby( vars['controller_execution_environments_' + branch_name] | default([])) }}"
        controller_instance_groups: "{{ controller_instance_groups_all | community.general.lists_mergeby( vars['controller_instance_groups_' + branch_name] | default([])) }}"
        controller_inventories: "{{ controller_inventories_all | community.general.lists_mergeby( vars['controller_inventories_' + branch_name] | default([])) }}"
        controller_inventory_sources: "{{ controller_inventory_sources_all | community.general.lists_mergeby( vars['controller_inventory_sources_' + branch_name] | default([])) }}"
        controller_notifications: "{{ controller_notifications_all | community.general.lists_mergeby( vars['controller_notifications_' + branch_name] | default([])) }}"
        controller_organizations: "{{ controller_organizations_all | community.general.lists_mergeby( vars['controller_organizations_' + branch_name] | default([])) }}"
        controller_projects: "{{ controller_projects_all | community.general.lists_mergeby( vars['controller_projects_' + branch_name] | default([])) }}"
        controller_roles: "{{ controller_roles_all | community.general.lists_mergeby( vars['controller_roles_' + branch_name] | default([])) }}"
        controller_settings: "{{ controller_settings_all | community.general.lists_mergeby( vars['controller_settings_' + branch_name] | default([])) }}"
        controller_schedules: "{{ controller_schedules_all | community.general.lists_mergeby( vars['controller_schedules_' + branch_name] | default([])) }}"
        controller_teams: "{{ controller_teams_all | community.general.lists_mergeby( vars['controller_teams_' + branch_name] | default([])) }}"
        controller_templates: "{{ controller_templates_all | community.general.lists_mergeby( vars['controller_templates_' + branch_name] | default([])) }}"
        controller_user_accounts: "{{ controller_user_accounts_all | community.general.lists_mergeby( vars['controller_user_accounts_' + branch_name] | default([])) }}"
        controller_workflows: "{{ controller_workflows_all | community.general.lists_mergeby( vars['controller_workflows_' + branch_name] | default([])) }}"


  roles:
    - redhat_cop.controller_configuration.dispatch
```

This is all there is..

