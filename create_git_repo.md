## Create GIT repository for new Organization

After the creation of the template repository is complete, it is time to add  
a new organization to the Ansible Automation Platform. To do this, you need to  
create a new git repository, based on the template created earlier.
The next thing is to set the name for the new organization on- and in the files  
that are copied from the template.

### What to change
In the templates there are default- and/or no values, these must be adapted for  
the new organization:

- Fill in the parameters for controller_auth
- Replace "ORG" in all files with the new organization name
- Add the appropiate keys and secrets to the credentials
- Encrypt sensitive files with vault
- Add the vault secret to GIT 
- Enable the pipeline
- push the new orgganization into git and wait for it...

The organization should be created in Ansible Automation Platform, without  
users or other objects, just some credenials. 
Then the team for that organization is given access to the repository, so they  
can start filling in the projects, templates, users and roles....

[Back](org_config.md)

