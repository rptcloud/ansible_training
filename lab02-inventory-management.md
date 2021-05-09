# Lab 2: Inventory Management

Duration: 20-30 minutes

- Task 1: Review the contents of the default host file
- Task 2: Create our working inventory
- Task 3: Push our keys to our remote hosts
- Task 4: Test access to our remote hosts through Ansible
- Task 5: Create logical groupings within our inventory file

## Task 1: Review the contents of the default host file

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


## Task 3: Push our keys to our remote hosts
Before we can begin to do anything against our remote hosts, we should first push our SSH keys to these hosts.  

### Step 2.3.1 
A public and private SSH key should have already been created as part of this workstation build.  Verify that they are currently in the `.ssh` directory of your user's home directory:

```shell
ansible@ansible-training-ant:/workstation/ansible > ls -la ~/.ssh
total 16
drwx------ 2 ansible ansible 4096 May  6 15:00 .
drwxr-xr-x 8 ansible ansible 4096 May  6 15:09 ..
-rw------- 1 ansible ansible 3243 May  6 15:00 id_rsa
-rw-r--r-- 1 ansible ansible  748 May  6 15:00 id_rsa.pub
```

### Step 2.3.2
Once verified, we will need to push our public key out to each of the four target EC2 instances:

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```
Make sure that you subsitute your `user` for your workstation's user name, and the `host` for the remote host's IP address.  

You will likely be prompted for a password during this step.  Enter your password to complete the transaction.

Repeat this step for the other three target hosts.

### Step 2.3.3
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


## Task 4: Test access to our remote hosts through Ansible
Now that we can remotely connect to these servers manually through SSH, this also means that Ansible should be able to remotely connect as well.  Let's test this out.

### Step 2.4.1
Run the following command to ping the hosts:
```shell
ansible -i ./hosts all -m ping -u <user>
```

The `-i` flag tells ansible to look for a hosts file, and entering `hosts` afterwards points it specifically to a file called `hosts` in your current directory.  `all` tells Ansible to look at all of the entries within that inventory file.

The `-m` flag tells ansible to run a `module` called `ping`.  We will go over this in greater detail in a future lab.  Just know for now that ansible is going to establish a connection to these servers over ssh and return some information if that connection is successful.

The `-u` flag tells ansible which user will be running the command.  This user must have an SSH connection to the remote host.  For this example, the user will be the one that you pushed your ssh keys to in step 2.3.2.

### Step 2.4.2
Review your results to make sure that you ansible was able to successfully return the correct results.   You should see the word `pong` returned for each server where a connection was successful.

## Task 5: Create logical groupings within our inventory file
Now that we've verified that Ansible is able to connect to our hosts, let's edit our host file a bit to logically group our servers in way that makes sense.
Going forward, we will be provisioning two of these servers to be web servers and two of them to be database servers.  Let's go ahead and denote that in our hosts file.
### Step 2.5.1
In VSCode, edit your host file to include a named block of hosts, denoted by a set of square brackets:

```shell
[webservers]
172.17.0.1 
172.17.0.2

[dbservers]
172.17.0.3 
172.17.0.4 
```

This will create two blocks of hosts that can be called simply by stating the name of the host block.  For example:

```shell
ansible -i ./hosts webservers -m ping
```

This should only run the ping module on the two hosts in the `[webservers]` block.

### Step 2.5.2
While we can logically denote these hosts by whether or not they are web servers or database servers, what if one of each host is production and one is development?  In a typical workflow, we'll usually be running updates on development first in order to ensure that the changes won't break anything.  However, there are times when we want to apply changes wholesale to just webservers regardless of environment.  Ansible allows us to make multiple logical groupings of servers using the same hosts.

Let's create two new host blocks using the same four hosts, one for production and one for development.  Let's include one web host and one database host in each of the new blocks:

```shell
[webservers]
172.17.0.1 
172.17.0.2

[dbservers]
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

### Step 2.5.3
Assuming everything in our host file is correct and currently reflects our environment, let's assume that all of these servers are in the `us-east-1` region of AWS.  Let's also assume that this host file will be appended in the future to include hosts that are in `us-east-2`.  Let's create one last host block in our host file to denote that.

This time, we are going to use the concept of `ranges` in order to simplify our new host block.  As you probably noticed, your web servers were created with sequential IP addresses.  Let's leverage this in our next code block and save ourselves some time:

```shell
[webservers]
172.17.0.1 
172.17.0.2

[dbservers]
172.17.0.3 
172.17.0.4

[production]
172.17.0.1 
172.17.0.3 

[development]
172.17.0.2
172.17.0.4

[us-east-1]
172.17.0.[1:4]
```
The `[1:4]` block tells Ansible to match every host that falls with the range specified within the square brackets.  This will match every host that we've used in this lab thus far.  

Here are two more common methods of matching ranges in a host file:
- match letters: `[A:D]`
- match zeros: `[01:07]`

### Step 2.5.4
Let's assume that we use ephemeral servers in our AWS environment, and we can't count on the IP addresses remaining constant.  A better way of matching hosts would be with hostnames. Luckily, Ansible also allows us to match by hostname. 

Using the hostnames that you copied down in `Step 2.3.3`, let's replace a few of these hosts with their hostname instead and test access again.