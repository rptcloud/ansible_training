# Lab 6: Variables

In our last lab, we successfully completed our first Ansible playbooks.  This will be the foundation that we build the rest of our labs off of.  

In this lab, we will begin to variablize our code in order to make it more robust and re-usable.  We are going to start the lab off by variablizing some of the common command line options that we've been running, and then move into refactoring parts of our existing playbook to leverage variables.

Duration: 30-40 minutes

- Task 1: Create a new ansible.cfg in our run directory to store common run arguments
- Task 2: Create group level variables for host groups and test
- Task 3: Create host level variables for individual hosts and test
- Task 4: Create playbook variables and test
- Task 5: Use `--extra-vars` to supercede all other variables


## Task 1: Create a new ansible.cfg in our run directory to store common run arguments
To recap an earlier lesson, if you run `ansible --version`, it will return information about your Ansible installation, including what your current configuraiton file is:

```shell
ansible@ansible-training-ant:/workstation/ansible > ansible --version
ansible 2.9.21
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Mar  1 2021, 11:38:31) [GCC 5.4.0 20160609]
```
As you can see, Ansible will always default for the config file located at `/etc/ansible/ansible.cfg`.  However, we can change this very easily.  Ansible will always check to see if there is a config file in your current run directory.

### Step 6.1.1
Let's go ahead and create a new ansible.cfg in our current run directory:
 ```shell
 touch /workstation/ansible/ansible.cfg
 ```
Now, let's re-run `ansible --version` to see if it picks up our new config file:
```shell
ansible@ansible-training-ant:/workstation/ansible > ansible --version
ansible 2.9.21
  config file = /workstation/ansible/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Mar  1 2021, 11:38:31) [GCC 5.4.0 20160609]
  ```
Good, our new config file is in our run directory.  Now let's add some useful information to it.

### Step 6.1.2
Let's go ahead and open our new config file with `vim`:

```shell
vim /workstation/ansible/ansible.cfg
```

And then let's add the following information to it:

```text
[defaults]
inventory = /workstation/ansible/hosts
remote_user = <your remote login username>

[privilege_escalation]
become_ask_pass = true
```
Replace your `<your remote login username>` with your actual login name.

Now that we have this, we should have a much easier time running our playbooks, as we no longer have to specify where our host file lives and which user we'll be logging in as, nor do we need to explicitly tell Ansible to prompt for our `sudo` password.  

### Step 6.1.3
Let's go ahead and run our playbook again.  Nothing should have changed at this point in time, expept for our command.  We should be able to simply run it like this now:

```shell
ansible-playbook playbook.yaml
```
If you didn't change the contents of your playbook previously, then your playbook should automatically prompt for your escalation password and then run successfully with no changes.

## Task 2: Create group level variables for host groups and test

### Step 6.2.1
Group level variables are among the lowest level of precendence in Ansible.  Let's go ahead and refactor our playbook a bit in order to get a good grasp on what happens when we start introducing various levels of variables within our code.

First of all, let's go ahead and great a new directory on the same level of our Ansible playbook.  This will have to be called `group_vars`:

```shell
mkdir /workstation/ansible/group_vars
```

### Step 6.2.2
Now that we have that, we'll also have to create a YAML file for our variables to live in.  These names must match a name of our existing node groups:

```shell
touch /workstation/ansible/group_vars/webservers.yaml;
```

### Step 6.2.3
Now let's refactor our existing code to accept the new variables that we'll be configuring for them.  If we take a look at our code, we'll notice that there are a few repeating values throughout the playbook.  Those are:
- `state: present`
- `state: started`

Let's go ahead and refactor our code by variablizing these lines like so:

```yaml
  tasks:
    - name: Install Nginx
      apt: 
        name: nginx
        state: "{{current_state}}"
    
    - name: Start Nginx
      service:
        name: nginx
        state: "{{current_status}}"
```
Make sure to do this for the `Configure Webservers` block of code.

### Step 6.2.4
Since we'll be variablizing the state and status of these services, let's go ahead and change the names to more acccurately reflect what will be happening.  Change the `name` of each of the following `tasks` to this:

- `Install Nginx` should now be called `Nginx is {{current_state}}`
- `Start Nginx` should now be called `Nginx is {{current_status}}`

### Step 6.2.5
Now that our playbook is ready to accept variables, let's go back to the host group variable files that we created earlier and add some values to them.

Open both of these files in VSCode and enter the following information into them:

```yaml
current_state: present
current_status: started
```

Your `Configure Webservers` playbook code should look like this:

```yaml
---
- name: Configure Webservers 
  hosts: linux
  become: True
 
  tasks:
    - name: Nginx is {{current_state}}
      apt: 
        name: nginx
        state: "{{current_state}}"

    - name: Insert index page
      template:
        src: /workstation/ansible/files/index.html
        dest: /usr/share/nginx/html/index.html

    - name: Nginx is {{current_status}}
      service:
        name: nginx
        state: "{{current_status}}
```
### Step 6.2.6
You can now re-run your playbook.  Your code should compile successfully, with the only difference being that the names of the individual tasks have changed to more accurately reflect what is happening in the playbook itself.

Feel free to change the values of the individual variables within the `group_vars` files to see how this changes your existing servers.

## Task 3: Create host level variables for individual hosts and test
Now that we've successfully re-factored our code to accept variables, let's see what happens when we introduce a variable file with a higher priority.

### Step 6.3.1
Create a new directory on the same level as your playbook, called `host_vars`.
```shell
mkdir /workstation/ansible/host_vars
```

### Step 6.3.2
We'll need to create some files to populate these directories.  As with group vars, the host vars must match the exact name of the individual hosts that you have in your host file.  These can be either FQDNs, IPs, or however else you have configured your host file to detect hosts.  Since we've been using IPs so far, let's stick with that.  Create a host file variable for one of your Linux servers:

```shell
touch /workstation/ansible/host_vars/54.164.149.176.yml;
```

You will need to ensure that you are using your own values for this and not the ones in this example.

### Step 6.3.3
Populate these files with the same content as the group_vars:

```yaml
current_state: present
current_status: stopped
```

Notice above that we have the value of these variables set to `stopped` for the status.  Make sure that your `group_vars` values are set to `started` so that you can see how the precedence will work here.

### Step 6.3.4
Go ahead and run your playbook again.  You should see that one set of your `linux` servers have switched statuses while the other remains the same.


## Task 4: Create playbook variables and test
Now that we've seen how we can assign individual variables to either a host group or an individual host, let's see what happens when we assign it directly to the playbook itself.

### Step 6.4.1
Our playbook is already set up to accept variables.  However, we can also define the values of the variables directly in the playbook itself.  Add the following blocks to the `Configure Webservers` code:


```yaml
---
- name: Configure Webservers 
  hosts: webservers
  become: True
  vars:
    current_state: present
    current_status: started

...
```

### Step 6.4.2
Go ahead and run your playbook again.  You should now see that the values directly within the playbook itself are being used.  If you left your host variables the same after the last run, you should see that the two that you turned off last time have been turned back on.

## Task 5: Use --extra-vars to overwrite all other variables
The highest level of priority for variables are those run directly on the command line.  Those can be used by adding the `--extra-vars` option to your run.  Let's see how that looks.

### Step 6.5.1
At this point in time, all of your services should be turned on.  Run the following command:
```shell
ansible-playbook playbook.yaml linux --extra-vars 'current_status=stopped'
```
You should see that the value for this variable superceded all others and stopped all of your Nginx services.  Because we named our variables the exact same thing, we were able to stop all of them at once.
