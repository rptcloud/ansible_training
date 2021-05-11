# Lab 7: Roles

In our last lab, we took a look at how to variablize our existing code, and refactored it a bit to allow for easier reusability.  In this lab, we are going to take that concept one step further with the introduction of roles.

Roles allow a far greater amount of reusability with our code by allowing us to fully modularize and reuse existing bits of code between multiple teams and environments.

Duration: 30-40 minutes

- Task 1: Create new roles directories for webserver and database
- Task 2: Populate webserver role directories with needed information
- Task 3: Populate database role directories with needed information
- Task 4: Create new main.yml file that handles running the roles

## Task 1: Create new roles directories for webservers and dbservers
The first thing that we'll have to do is create directories for our new roles to live in.  Ansible expects and interprets a specific directory structure for roles.
### Step 7.1.1
In the same directory level as your `playbook.yaml` file, create a new `roles` directory

```shell
mkdir /workstation/ansible/roles
```

### Step 7.1.2
In this directory, we will be making two new directories, one for each role: `webserver` and `database`.

```shell
cd roles; mkdir webserver database
```
### Step 7.1.3
Next, we will have to create the correct role subdirectories for each new role:

```shell
cd /workstation/ansible/roles/webserver; mkdir files handlers meta tasks vars
```

```shell
cd /workstation/ansible/roles/database; mkdir files handlers meta tasks vars
```

## Task 2: Populate webserver role directories with needed information
Now that we have the correct directories, let's start populating them one by one with the needed yaml files.  We'll begin with `webserver`.
### Step 7.2.1
We'll go one directory at a time and explain their purpose (and whether or not they are needed)

`Files` - This directory is for any files that we need to include with the roles.  If you recall back to our earlier labs, we already have one file that we want to include here.  This is the `index.html` file that we used earlier.  Let's go ahead and copy it to the new files directory:

```shell
cp /workstation/ansible/files/index.html /workstation/ansible/roles/webserver/files/index.html
```

### Step 7.2.2
`Handlers` is a slightly more advanced concept that we have not covered yet.  Handlers essentially will trigger an action only when a change is made on a machine.  We don't need to know much about them right now, but we can fill in a bit of information here.

Let's say that we want Nginx to restart if any change is made to the configuration.  We can make a handler for that.  First, we'll need to make a yaml file for it:

```shell
touch /workstation/ansible/roles/webserver/handlers/main.yml
```

Let's go ahead and populate that with this code, which will prompt nginx to restart upon any updates:

```yaml
---
- name: Start Nginx
  service: 
    name: nginx 
    state: started

- name: Reload Nginx
  service: 
    name: nginx
    state: reloaded
```

### Step 7.2.3
`Meta` allows you to attach meta arguments to your roles at runtime.  One of the most common meta arguments are to add `dependencies` to your roles, such as ensuring that one role has already ran before this one runs.  For this step, we do not need any dependencies, but let's set up the file in case that changes in the future.

```shell
touch workstation/ansible/roles/webserver/meta/main.yml
```

The contents of the file can simply be an empty dependency set:
```yaml
---
dependencies: []
```
### Step 7.2.4
`Tasks` are where the bulk of the new code will be going.  Much like the tasks in our previous playbook, these are the tasks that will define what this role will be doing.  Here, we can move the code from our other playbook for the webservers over to this new yaml file.  

Let's create the file:

```shell
touch workstation/ansible/roles/webserver/tasks/main.yaml
```

And let's add the specific webserver tasks from our last playbook to this file:

```yaml
---
- name: Nginx is {{current_state}}
  apt: 
    name: nginx
    state: "{{current_state}}"

- name: Insert index page
  template:
    src: /workstation/ansible/roles/webserver/files/index.html
    dest: /usr/share/nginx/html/index.html

- name: Nginx is {{current_status}}
  service:
    name: nginx
    state: "{{current_status}}"
```
Note that we changed the relative file path of the `index.html` file to the new `files` directory for the role.

### Step 7.2.5
`Vars` are for variables.  As you can see in our `tasks` step, we are still declaring our variables in our tasks.  We will need to create a new file to contain our variables and their values:

```shell
touch /workstation/ansible/roles/webserver/vars/main.yml
```

And here, we will declare our variables as we did before:

```yaml
---
current_state: present
current_status: started
```

## Task 3: Populate database role directories with needed information
We will continue building out our new roles by repeating this process with the `database` role.  Let's go through the directories again and see if there is anything that we need from our previous steps:

### Step 7.3.1
`Files` - There are no database files that have been used so far, so we can skip this step.

### Step 7.3.2
`Handlers` - We can follow the same steps as we did with the Nginx portion of this lab.  Please note that you may not actually want to have your databases restart if they are in production.  This is just a sample excercise.

```shell
touch /workstation/ansible/roles/database/handlers/main.yml
```

```yaml
---
- name: Start Postgres
  service: 
    name: postgres 
    state: started

- name: Reload Postgres
  service: 
    name: postgres
    state: reloaded
```
### Step 7.3.3
`Meta` Though we did not include this last time, we can configure a meta argument for the Postgres databse for the sake of learning.

```shell
touch /workstation/ansible/roles/database/meta/main.yml
```

```yaml
---
dependencies:
  - {role: webservers}
```
Again, this will not really change the functionality.  But this is what this would look like should you have dependent roles

### Step 7.3.4
`Tasks` - Much like the webserver tasks, we will need to copy out the database tasks from the previous playbook and paste them into a new yaml file:

```shell
touch /workstation/ansible/roles/database/tasks/main.yml
```

```yaml
---
- name: Postgres is {{db_state}}
  apt:
    name: postgresql
    state: "{{db_state}}"

- name: Postgres is {{db_status}}
  service:
    name: postgresql
    state: "{{db_status}}"
```

Remember from the end of the last lab how we discussed the potential room for error that we gave ourselves by applying the same variable in inappropriate places?  This is a good place to fix that.  Note how we changed the variables above:

- `current_state` changed to `db_state`
- `current_status` changed to `db_status`


### Step 7.3.5
`Vars` - As before, we will need to create a new variable file.  As mentioned in the last step, we adjusted the names of the variables, so that will need to be reflected in this variables file as well.

```shell
touch /workstation/ansible/roles/database/vars/main.yml
```

```yaml
---
db_state: present
db_status: started
```

## Task 4: Create new main.yml file that handles running the roles
Now that we have successfully built out our roles, it's time that we made the final yaml file that calls them.  This will live in the same directory that your previous playbook lived in (/workstation/ansible)

### Step 7.4.1
We will first need to create the file.  You can call this file whatver you'd like, as long as it ends in .yml.  We can just call this `main_playbook.yml` for the lab.

```shell
touch /workstation/ansible/main_playbook.yml
```

### Step 7.4.2 
Now we will need to build out the information for the webserver role within this file.  The code for that should read:

```yaml
---
- hosts: webservers
  become: yes
  user: rpt
  roles:
    - webserver
```
Much like before, we are declaring a few things here:
1. `hosts` - these are simply the names of the host groups that we would like to apply our changes to.
2. `become` - as before, we need to declare above that we will be executing these as root
3. `user` - This will supercede any information that you have in your `ansible.cfg` file.  This will allow you to run commands in a more granular fashion if you have strict policies on which users can run which tasks.
4. `roles` - Here, you can define any and all roles you would like to apply to each of the above `hosts` groups.  As this is simply our small Nginx installation, we only need to run the `webserver` role.

### Step 7.4.3
As with the `webserver` role, let's add the same with the `database` role.  The full code for your `main_playbook.yml` file should read:

```yaml
---
- hosts: webservers
  become: yes
  user: rpt
  roles:
    - webserver

- hosts: dbservers
  become: yes
  user: rpt
  roles:
    - dbservers
```

### Step 7.4.4
At this point, you should be ready to run your new roles.  Much like a normal playbook, they are simply ran with the `ansible-playbook` command:

```shell
ansible-playbook main_playbook.yml
```

You should now see the full run along with all of the steps from both of the roles cycling through.

### Step 7.4.5
If you remember back in step 7.3.3, we put a meta argument in for the database role.  We made that role contingent up the `webserver` role running.  If you pay close attention to the output of your last command, you'll see that the `webserver` role is running twice because of this.  There are two ways we can handle this.  

1. We can remove that meta argument and keep our `main_playbook.yml` the same
2. We can remove the block in `main_playbook.yml` that calls the webserver role, since it is now redundant.  

In production, we would probably decouple the two roles by removing the meta argument, since it's just an example of what can be done here.  However, for the sake of showing how these roles can be nested, let's go ahead and remove that block from `main_playbook.yml` and re-run the code.  This should essentially revert the entire playbook run to only the steps that were present before the refactor:

```yaml
---
- hosts: dbservers
  become: yes
  user: rpt
  roles:
    - dbservers
```

Go ahead and run your code one last time and pay attention to the output.  Everything should now be running exactly as expected.