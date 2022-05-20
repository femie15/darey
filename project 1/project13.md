# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

Static assignments use "import" Ansible module. The module that enables dynamic assignments is "include".

In the https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

Create a new folder, name it "dynamic-assignments". Then inside this folder, create a new file and name it "env-vars.yml".
  
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., 
we will need a way to set values to variables per specific environment.
  
To keep each environment’s variables file. Create a new folder "env-vars", then for each environment, create new YAML files which we will use to set variables.
  
The layout should now look like this.
  

  
Now enter these into the "env-vars.yml" file
  
```
  ---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always 
```
  
  
### 3 things worth noting
  
1. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
  
```
include_role
include_tasks
include_vars 
```
  
In the same version, variants of import were also introduces, such as:

import_role
import_tasks
  
2. We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

### Update site.yml with dynamic assignments
 
Update "site.yml" file to make use of the dynamic assignment.
  
```
  ---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

On Jenkins-Ansible server make sure that git is installed with "git --version", then go to ‘ansible-config-artifact’ directory and run

```
git init
git pull https://github.com/femie15/ansible-config-mgt.git
git remote add origin https://github.com/femie15/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
  
if the git is not working from the root directory goto `vi .ssh/authorized_keys` and add the key to github ssh keys. then redo the process.
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
