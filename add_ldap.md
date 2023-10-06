# Add new Organization to LDAP mapping

Before adding the new organization to git and enable the pipeline, the  
LDAP settings must be added to the base configuration of the Ansible Automation  
Platform.

In the file below only the LDAP settings for a given org is listed.
There are 2 LDAP servers defined in this case (max 5 is possible), under each LDAP server  
the mapping for organizations are configured:

```yaml
        ORG_x:
            admins: CN=admin_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
            remove_admins: false
            remove_users: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
```

The above section needs to be added into the file, after editing the parameters:
- ORG_x 	The name of the new organization in ansible Automation Platform
- admins        Adapt the LDAP string to fit the correct administrator group in LDAP
- users		Adapt the LDAP string to fit the correct user group in LDAP


Add the section into the file under the correct LDAP server definition and save, push  
the file into the git repository. The configuration is then loaded into AAP.
After that the rest of the steps can be executed.

```yaml
---
controller_settings:
  settings:
    AUTH_LDAP_1_BIND_DN: CN=cn_name,OU=Services,OU=Accounts,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
    AUTH_LDAP_1_BIND_PASSWORD: vault_encrypted_password
    AUTH_LDAP_1_DENY_GROUP: null
    AUTH_LDAP_1_GROUP_SEARCH:
    - DC=dc_sub,DC=dc_name,DC=nl
    - SCOPE_SUBTREE
    - (objectClass=group)
    AUTH_LDAP_1_GROUP_TYPE: ActiveDirectoryGroupType
    AUTH_LDAP_1_GROUP_TYPE_PARAMS:
        name_attr: cn
    AUTH_LDAP_1_ORGANIZATION_MAP:
        ORG_1:
            admins: CN=admin_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
            remove_admins: false
            remove_users: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
        ORG_2:
            admins: CN=admin_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
            remove_admins: false
            remove_users: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
    AUTH_LDAP_1_REQUIRE_GROUP: null
    AUTH_LDAP_1_SERVER_URI: ldap://ldap_server_fqdn
    AUTH_LDAP_1_START_TLS: false
    AUTH_LDAP_1_TEAM_MAP:
        LDAP Admins:
            organization: Default
            remove: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
        LDAP_ORG_1_Admins:
            organization: ORG_1
            remove: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
        LDAP_ORG_1_Developers:
            organization: ORG_1
            remove: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
        LDAP_ORG_2_Admins:
            organization: ORG_2
            remove: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
        LDAP_ORG_2_Developers:
            organization: ORG_2
            remove: true
            users: CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
    AUTH_LDAP_1_USER_ATTR_MAP:
        email: mail
        first_name: givenName
        last_name: sn
    AUTH_LDAP_1_USER_DN_TEMPLATE: null
    AUTH_LDAP_1_USER_FLAGS_BY_GROUP:
        is_superuser:
        - CN=user_group_name,OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
    AUTH_LDAP_1_USER_SEARCH:
    - OU=ou_sub,OU=Groups,OU=ou_name,DC=dc_sub,DC=dc_name,DC=nl
    - SCOPE_SUBTREE
    - (sAMAccountName=%(user)s)
    AUTH_LDAP_BIND_DN: CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
    AUTH_LDAP_BIND_PASSWORD: vault_encrypted_password
    AUTH_LDAP_DENY_GROUP: null
    AUTH_LDAP_GROUP_SEARCH:
    - CN=group_name,OU=Mailbox,OU=Groups,OU=ou_name,OU=ALT,DC=alt,DC=dc_name,DC=nl
    - SCOPE_SUBTREE
    - (objectClass=group)
    AUTH_LDAP_GROUP_TYPE: ActiveDirectoryGroupType
    AUTH_LDAP_GROUP_TYPE_PARAMS:
        name_attr: cn
    AUTH_LDAP_ORGANIZATION_MAP: {}
    AUTH_LDAP_REQUIRE_GROUP: null
    AUTH_LDAP_SERVER_URI: ldap://ldap_server_fqdn
    AUTH_LDAP_START_TLS: false
    AUTH_LDAP_TEAM_MAP:
        LDAP Admins:
            organization: Default
            remove: true
            users: CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
        Team_1:
            organization: ORG_3
            remove: true
            users: CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
        Team_default:
            organization: Default
            remove: true
            users: CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
    AUTH_LDAP_USER_ATTR_MAP:
        email: mail
        first_name: givenName
        last_name: sn
    AUTH_LDAP_USER_DN_TEMPLATE: null
    AUTH_LDAP_USER_FLAGS_BY_GROUP:
        is_superuser:
        - CN=CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
    AUTH_LDAP_USER_SEARCH:
    - OU=Users,CN=ldap_group,OU=ou_group,OU=Accounts,OU=ou_name,DC=alt,DC=dc_name,DC=nl
    - SCOPE_SUBTREE
    - (sAMAccountName=%(user)s)

```
Add the appropiate keys and values to this file and encrypt it with vault.
then push this to the repository to configure the LDAP params.

If this is correct, add the organization repository to git..
    
[Back](org_config.md)
