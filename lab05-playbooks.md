# Lab 5: Playbooks

Now that we've familiarized ourselves with how Ansible runs commands through modules, as well as how we need to connect and pass credentials through SSH via Ansible, it's time that we take a look at Ansible Playbooks.  

Playbooks allow us to write "plays" into persistent code.  These plays store the same types of ad hoc commands that we have been using up to this point.  The main differences between running an ad hoc command vs. a playbook is that playbooks allow us to use multiple modules within one playbook, and playbooks themselves serve as a persistent point of reference for what should be configured within our provisioned hardware.

In this lab, we will be taking some of the commands that we ran in previous labs and condensing them into playbooks.  We will be using these playbooks to turn our vanilla servers into web servers and database servers.

Duration: 30-40 minutes

- Task 1: Create a playbook to configure our web servers
- Task 2: Review output
- Task 3: Edit our playbook to configure our database servers


## Task 1: Create a play to configure our web servers

### Step 5.1.1
The first thing that we will need to do is create the file that will house all our the code for our playbook.  In the `/workstation/ansible` directory, create a file called `playbook.yaml`.

```shell
touch /workstation/ansible/playbook.yaml
```

### Step 5.1.2
We'll now need to start editing our yaml file with the information needed to configure this web server.  Let's start by using `vim` (or your preferred Linux text editor) to get our playbook set up.

```shell
vi playbook.yaml
```

### Step 5.1.3
Now let's begin adding our code.  Best practice is to begin every playbook with three dashes.  Let's go ahead and add them here along with the beginnings of our code block to install Nginx:

```yaml
---
- name: Configure Webservers
  hosts: linux
  become: true
```
Thinking back to our ad hoc commands, we know that we want to target the host group webservers and that we'll need to become root to complete the install.  The code block above illustrates what that will look like in a playbook.  

Also, keep in mind that this is written in YAML.  YAML is a markdown language that relies on proper spacing to compile correctly.  Do no use tabs, as it will break the code.

### Step 5.1.4
The last code block that we added simply defined what the first code block will be targeting and what permission level we will be using.  Now, let's go ahead and define some tasks involved in getting our webserver up and running.  These next few lines of code will start on the same indent level as the previous lines (2 spaces):

```yaml
  tasks:
    - name: Add Nginx
      apt: 
        name: nginx
        state: present
```

### Step 5.1.5

Now, let's make a simple index.html file and upload that as part of our playbook:

```shell
vi /workstation/ansible/files/index.html
```
and add the following code to the index.html file:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to your Ansible Webserver!</title>
</head>
</html>
```
And lastly, let's go ahead and add another task to our playbook that uses the `template` module to move this file up.  This will sit on the same indent level as the previous subtask.  The full code as of right now should look like this:

```yaml
---
- name: Configure Webservers
  hosts: linux
  become: true

  tasks:
    - name: Add Nginx
      apt: 
        name: nginx
        state: present

    - name: Insert index page
      template:
        src: /workstation/ansible/files/index.html
        dest: /usr/share/nginx/html/index.html
```
### Step 5.1.6
Last thing that we will do is add a block of code that tells Ansible to make sure that Nginx is turned on.  Our last lesson on ad hoc commands had this same command.  Let's turn it into playbook code.  Our full playbook should now look like this:

```yaml
---
- name: Configure Webservers
  hosts: linux
  become: true

  tasks:
    - name: Add Nginx
      apt: 
        name: nginx
        state: present

    - name: Insert index page
      template:
        src: /workstation/ansible/files/index.html
        dest: /usr/share/nginx/html/index.html

    - name: Start Nginx
      service:
        name: nginx
        state: started
```
 ### Step 5.1.7
 Let's go ahead and apply these changes to make sure that our code worked properly.  Remember, we have a `become` declaration in our code, so we will need to include that option as well.  

 The command for deploying Ansible playbooks is simple `ansible-playbook`.  Let's take a look at what our command will be:

 ```shell
 ansible-playbook playbook.yaml -i hosts -u rpt -K
 ```
 Go ahead and run that code.  You should see output indicating that your changes applied correctly, with a breakdown for each individual step


## Task 2: Review Output

### Step 5.2.1
Now that we've ran that code, let's take a look at our output:

```shell
ansible@ansible-training-ant:/workstation/ansible > ansible-playbook playbook.yaml -i hosts -u rpt -K
BECOME password: 

PLAY [Configure Webservers] *****************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************
ok: [54.164.149.176]
ok: [18.205.7.240]

TASK [Add Nginx] *********************************************************************************************************************************************************************
changed: [18.205.7.240]
changed: [54.164.149.176]

TASK [Insert index page] *************************************************************************************************************************************************************
changed: [54.164.149.176]
changed: [18.205.7.240]

TASK [Start Nginx] *******************************************************************************************************************************************************************
ok: [54.164.149.176]
ok: [18.205.7.240]

PLAY RECAP ***************************************************************************************************************************************************************************
18.205.7.240               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
54.164.149.176             : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Looking at these one at a time:

- `PLAY [Configure Webservers]` - This is telling you which of your configured plays is currently running.
  - As we only have one play currently, this is the only one that ran.
- `TASK [Gathering Facts]` - This is checking your Ansible code to ensure that everything compiles correctly and your hosts are reachable.
  - As you can see, this should be returning just the IP addresses of the two hosts that were labeled as `linux`.  Your `windows` should not have been touched.
  - The `ok` status indicates that these hosts were reaachable.
- `TASK [Add Nginx]` - This is the task that configured in our playbook.  This is installing Nginx on the remote webserver.
  - As you can see, the output here has both hosts listed as `changed`.  This is because we did not have Nginx installed on our host, and Ansible went out and applied that for us.
- `TASK [Insert index page]` - This task moved our index.html file onto the server as a template.  This is simliar functionality to both the `file` and `copy` modules that we previously used, however this one designates this as a template.  There are other labs down the road that go over this a bit more in depth.  Just know for now that we could have used `copy` instead with mostly the same result.
  - Again, we see that the status is `changed`.  This is because we moved the file over for the first time.
- `TASK [Start Nginx]` - Again, this is ensuring that our service is started
  - Since `nginx` is started upon installation, the status simply returned `ok` instead of `changed`.  This is because there was nothing for Ansible to do here.
- `PLAY RECAP` - This is simply just a conslidation of all of the information that we saw above.
  - If you go down to the very bottom, it will tell you useful information, such as how many things were changed, ok, failed, etc on a per-host basis

### Step 5.2.2
Let's go ahead and run our `ansible-playbook` command again.  You should see every status return as `ok` rather than changed.  This is because Ansible is idempotent.  It will recognize that the needed changes are already present.

## Task 3: Edit our playbook to configure our database servers
Now that we have our web servers configured, let's go ahead and take a look at our database servers.  We do not need to create an entirely new playbook for these servers.  We can simply add them directly into our current playbook.

### Step 5.3.1
Let's go ahead and edit our existing `playbook.yaml` file and add a new code block at the end.  This is very similar to the last process.  As always with YAML, it's important to make sure that our indentation level is accurate, though.  This new code block is an entirely new play, so it will have to go on the same level as the previous play (`Configure Webservers`)

```yaml
---
- name: Configure Webservers
  hosts: linux
  become: true

  tasks:
    - name: Add Nginx
      apt: 
        name: nginx
        state: present

    - name: Insert index page
      template:
        src: /workstation/ansible/files/index.html
        dest: /usr/share/nginx/html/index.html

    - name: Start Nginx
      service:
        name: nginx
        state: started

- name: Configure Windows Servers
  hosts:  windows
  become: true
  ```
  
### Step 5.3.2
Let's go ahead and add the rest of our code.  This should be pretty familiar to you at this point in time.  We'll just be adding code blocks in to download and install the `Apache` application with the `win_get_url` and `win_package` modules.

```yaml
---
- name: Configure Webservers
  hosts: webservers
  become: true

  tasks:
    - name: Add Nginx
      apt: 
        name: nginx
        state: present

    - name: Insert index page
      template:
        src: /workstation/ansible/files/index.html
        dest: /usr/share/nginx/html/index.html

    - name: Start Nginx
      service:
        name: nginx
        state: started

- name: Installing Apache MSI 
  hosts: windows
  tasks:
    - name: Download the Apache installer
      win_get_url:
        url: https://archive.apache.org/dist/httpd/binaries/win32/httpd-2.2.25-win32-x86-no_ssl.msi
        dest: C:\Users\your_user_name\Desktop\httpd-2.2.25-win32-x86-no_ssl.msi

    - name: Install MSI
      win_package: 
        path: C:\Users\your_user_name\Desktop\httpd-2.2.25-win32-x86-no_ssl.msi
        arguments:
          - /install
          - /passive
          - /norestart
```

### Step 5.3.3.
At this point in time, you can re-run your code.  You can use the exact same command that you used previously.

### Step 5.3.4
Let's go ahead and review the output again.  We'll only be looking at the relevant bits of information to avoid redundancy:

- `PLAY [Configure Database Servers]` - As you can see, we now have a second play added to our output
- `TASK [Gathering Facts]` - Pay attention here.  The IP addresses listed for this play should be the IPs configured in your `windows` host group.
- `TASK [Download the Apache installer]` - Again, this should have returned a `changed` instead of `ok`, as we are downloading Apache for the first time.
- `TASK [Install MSI]` - This should be changed, since we are installing Apache for the first time.
- `PLAY RECAP` - You should now see all four of your servers listed in the output here.  

At this point, we should have a fully functioning Ansible playbook that configures both your web servers and your database servers.  More to the point, we now have usable code that we can store in source control and use as a part of our provisioning pipelines.  This is the first step towards turning Ansible from a useful command line utility into a robust Infrastructure as Code tool.  In future labs, we will discuss how to make this even more robust.
