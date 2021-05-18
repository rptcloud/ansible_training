# Lab 2: Inventory Management

Duration: 20-30 minutes

- Task 1: Review the contents of the default host file
- Task 2: Create our working inventory
- Task 3: Set up Windows Hosts for Ansible Access
- Task 4: Push our keys to our remote Linux hosts
- Task 5: Test access to our remote hosts through Ansible
- Task 6: Create logical groupings within our inventory file

## Task 1: Review the contents of the default host file.

In the previous lab, we installed Ansible and took a look at the default configuration file.  Within that configuration file, we found that Ansible creates a default host file as well.  Let's take a look at the contents of that host file:

```shell
ansible@ansible-training-ant:/workstation/ansible > cat /etc/ansible/hosts 
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
## 
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com
```
This file will provide the basis for all of the custom inventory files that we will be using going forward.  Pay special attention to the examples 2 and 3, as this will be how we will begin to build out our own inventory file.

## Task 2: Create our working inventory

### Step 2.2.1

Let's start by creating our inventory file in our working directory.  Run the following command to create the file:

```shell
touch /workstation/ansible/hosts
```

### Step 2.2.2

When we spun up our workstations for this lab, we also created four EC2 instances in parallel to work with.  You should be able to find the information within the Google Sheet that we used to assign workstations at the beginning of this course.

Let's take those servers and add them to the host file that we just created.

Open the `hosts` file you just crated in VSCode.  Add the following lines to the file, substituting your IPs with the ones in this example:

```shell
172.17.0.1 
172.17.0.2
172.17.0.3 
172.17.0.4 
```

This is a perfectly good host file to begin running tests against.

### Step 2.2.3 

We'll need to edit this host file a bit in order to allow the Windows hosts to connect properly.

Add a group called [windows] and place the IP addresses of your Windows hosts after it:

```shell
172.17.0.1 
172.17.0.2

[windows]
172.17.0.3 
172.17.0.4 
```

Step 2.2.4
We'll now need to tell Ansible to connect to the host over WinRM.  Let's add a few lines of code to accomplish this:

```shell
172.17.0.1 
172.17.0.2

[windows]
172.17.0.3 
172.17.0.4 

[windows:vars]
ansible_user=your_user_name_here
ansible_password=your_password_here
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

## Task 3: Set up Windows Hosts for Ansible Access

### Step 2.3.1
First, we will need to log into our Windows hosts in AWS.

Grab one of the Windows IP addresses in the Google Sheets document pertaining to your workstation IPs.  Place that IP address into your Remote Desktop Client and connect to the host.

### Step 2.3.2

There is a file in the `files` directory of this repository called `configureWinRM.ps1`.  Copy and paste the contents of that script into a new Notepad file on your remote Windows host.  Save it, making sure that it has the `.ps1` file extension on it.

### Step 2.3.3.

Open PowerShell as an administrator.  Navigate to the file that you just saved, and execute the script:

```powershell
./configureWinRM.ps1
```

You should now have WinRM set up on your Windows host.

### Step 2.3.4

We must now install `Pywinrm` on the Windows host as well:

```powershell
pip install pywinrm
```



## Task 4: Push our keys to our remote Linux hosts
Before we can begin to do anything against our remote hosts, we should first push our SSH keys to these hosts.  

### Step 2.4.1 
A public and private SSH key should have already been created as part of this workstation build.  Verify that they are currently in the `.ssh` directory of your user's home directory:

```shell
ansible@ansible-training-ant:/workstation/ansible > ls -la ~/.ssh
total 16
drwx------ 2 ansible ansible 4096 May  6 15:00 .
drwxr-xr-x 8 ansible ansible 4096 May  6 15:09 ..
-rw------- 1 ansible ansible 3243 May  6 15:00 id_rsa
-rw-r--r-- 1 ansible ansible  748 May  6 15:00 id_rsa.pub
```

### Step 2.4.2
Once verified, we will need to push our public key out to each of the four target EC2 instances:

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```
Make sure that you subsitute your `user` for your workstation's user name, and the `host` for the remote host's IP address.  

You will likely be prompted for a password during this step.  Enter your password to complete the transaction.

Repeat this step for the other three target hosts.

### Step 2.4.3
Verify that you are able to access these servers without the need to enter a password.

```shell 
ssh user@host
```
Make sure that you subsitute your `user` for your workstation's user name, and the `host` for the remote host's IP address.

While you're here, copy this server's hostname and paste it into a notepad for later on in this lesson:

```shell
ansible@ansible-training-ant:/workstation/ansible > hostname
ansible-training-ant.node.consul
```

Do this step for every server in your host file to ensure that we are able to access everything.


## Task 5: Test access to our remote hosts through Ansible
Now that we can remotely connect to these servers manually through SSH, this also means that Ansible should be able to remotely connect as well.  Let's test this out.

### Step 2.5.1
Run the following command to ping the hosts:
```shell
ansible -i ./hosts all -m ping -u <user>
```

The `-i` flag tells ansible to look for a hosts file, and entering `hosts` afterwards points it specifically to a file called `hosts` in your current directory.  `all` tells Ansible to look at all of the entries within that inventory file.

The `-m` flag tells ansible to run a `module` called `ping`.  We will go over this in greater detail in a future lab.  Just know for now that ansible is going to establish a connection to these servers over ssh and return some information if that connection is successful.

The `-u` flag tells ansible which user will be running the command.  This user must have an SSH connection to the remote host.  For this example, the user will be the one that you pushed your ssh keys to in step 2.3.2.

### Step 2.5.2
Review your results to make sure that you ansible was able to successfully return the correct results.   You should see the word `pong` returned for each server where a connection was successful.

## Task 6: Create logical groupings within our inventory file
Now that we've verified that Ansible is able to connect to our hosts, let's edit our host file a bit to logically group our servers in way that makes sense.
Going forward, we will be provisioning two of these servers to be web servers and two of them to be database servers.  Let's go ahead and denote that in our hosts file.
### Step 2.6.1
In VSCode, edit your host file to include a named block of hosts for your Linux servers.

```shell
[linux]
172.17.0.1 
172.17.0.2

[windows]
172.17.0.3 
172.17.0.4 
```

This will create two blocks of hosts that can be called simply by stating the name of the host block.  For example:

```shell
ansible -i ./hosts linux -m ping
```

This should only run the ping module on the two hosts in the `[linux]` block.

### Step 2.6.2
While we can logically denote these hosts by whether or not they are web servers or database servers, what if one of each host is production and one is development?  In a typical workflow, we'll usually be running updates on development first in order to ensure that the changes won't break anything.  However, there are times when we want to apply changes wholesale to just webservers regardless of environment.  Ansible allows us to make multiple logical groupings of servers using the same hosts.

Let's create two new host blocks using the same four hosts, one for production and one for development.  Let's include one web host and one database host in each of the new blocks:

```shell
[linux]
172.17.0.1 
172.17.0.2

[windows]
172.17.0.3 
172.17.0.4

[production]
172.17.0.1 
172.17.0.3 

[development]
172.17.0.2
172.17.0.4
```

We can now test our changes against just the `[production]` or `[development]` hosts.  

```shell
ansible -i ./hosts production -m ping
```
