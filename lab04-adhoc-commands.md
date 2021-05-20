# Lab 4: Ad Hoc Commands

Building off of the information that we went over in Lab 3, we will learn how to use modules to instruct Ansible to take actions against our remote host inventory that we built out in Lab 2.  Looking ahead, this lab will give us the foundation with which to start creating Ansible Playbooks, which is where the true power of Ansible opens up.

Duration: 20-30 minutes

- Task 1: Review Ansible module command line arugments
- Task 2: Run Ansible ad hoc commands to add/remove files to a server
- Task 3: Run Ansible ad hoc commands in install a program from a remote host
- Task 4. Run Ansible ad hoc commands to start/stop services on a remote host
- Task 5. Run Ansible ad hoc commands to remove a program from a remote host


## Task 1: Review Ansible module command line arugments
To this point, we've already ran one module command: `ansible -i ./host all -m ping -u <user>`.  Let's review the components of this command again:

### Step 4.1.1

The `-i` flag tells ansible to look for a hosts file, and entering `hosts` afterwards points it specifically to a file called `hosts` in your current directory.  `all` tells Ansible to look at all of the entries within that inventory file.

The `-m` flag tells ansible to run a `module` called `ping`.  We will go over this in greater detail in a future lab.  Just know for now that ansible is going to establish a connection to these servers over ssh and return some information if that connection is successful.

The `-u` flag tells ansible which user will be running the command.  This user must have an SSH connection to the remote host.


## Task 2: Run Ansible ad hoc commands to add/remove files to a server
### Step 4.2.1
In our last lab, we were introduced to the Ansible module inventory.  Specifically, we looked at the module `copy`.  We found that this module will copy a local file or directory and place it onto a remote host.  We also discussed that there were mandatory options for that command, specifically the `dest` option.  Let's look at how we would construct this command to include the options.

First of all, we will need to create a file that we can push to the remote host.  Lets create a files directory:

```shell
mkdir files
```

And then place a dummy text file into that directory:

```shell
echo "This file will be used to test the copy module" > files/ansibleTest.txt
```

### Step 4.2.2

Now that we have a file, let's try to copy it to one of our hosts in the `linux` group:

```shell 
ansible -i hosts linux -m copy -a 'src=/workstation/ansible/files/ansibleTest.txt dest=/tmp' -u <user>
```

This will copy the file `ansibleTest.txt` located in the `files` directory, which itself is located in our current working directory.  


### Step 4.2.2
Now that we've ran that command, let's take a moment to review the output from that command and see what it's telling us.

```shell
54.164.149.176 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": true, 
    "checksum": "b56df8ed5365fca1419818aa384ba3b5e7756047", 
    "dest": "/tmp/ansibleTest.txt", 
    "gid": 1001, 
    "group": "rpt", 
    "md5sum": "5dd39cab1c53c2c77cd352983f9641e1", 
    "mode": "0664", 
    "owner": "rpt", 
    "size": 20, 
    "src": "/home/rpt/.ansible/tmp/ansible-tmp-1620522937.27-12008-259756546684708/source", 
    "state": "file", 
    "uid": 1001
}
18.205.7.240 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": true, 
    "checksum": "b56df8ed5365fca1419818aa384ba3b5e7756047", 
    "dest": "/tmp/ansibleTest.txt", 
    "gid": 1001, 
    "group": "rpt", 
    "md5sum": "5dd39cab1c53c2c77cd352983f9641e1", 
    "mode": "0664", 
    "owner": "rpt", 
    "size": 20, 
    "src": "/home/rpt/.ansible/tmp/ansible-tmp-1620522937.28-12009-277094727393229/source", 
    "state": "file", 
    "uid": 1001
}
```

### Step 4.2.3 
Now that we've had a chance to review that, you can log into one of those servers to verify that it's in the `/tmp` directory if you'd like to see how it copies over and what file permissions it has.

```shell
ssh user@host
ls -la /tmp
```

### Step 4.2.4
Now that we've learned how to add a file, let's take a look at how we should remove that file.

First, let's do a lookup on the module that will allow us to delete a file.  To save you some time, the module that you'll be looking for is simply called `file`.

```shell
ansible-doc -l file
```

Here, you will see that this module does the following:

```text
Set attributes of files, symlinks or directories. Alternatively, remove files, symlinks or directories. Many other modules support the same options as the `file' module - including
        [copy], [template], and [assemble]. For Windows targets, use the [win_file] module instead
```

Take a moment to look through the documentation for `file` to see if you can determine which options we'll need to remove it.

![](files/waiting.gif)

If you guessed that you will need both the `path` and the `state` options, you are correct!  Your full command should look like this:

```shell
ansible -i hosts linux -m file -a 'path=/tmp/ansibleTest.txt state=absent' -u rpt
```
And you should see something alont the lines of this for your output:

```shell
54.164.149.176 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": true, 
    "path": "/tmp/ansibleTest.txt", 
    "state": "absent"
}
18.205.7.240 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": true, 
    "path": "/tmp/ansibleTest.txt", 
    "state": "absent"
}
```
## Task 3: Run Ansible ad hoc commands in install/remote programs from a remote host
Now that we know that we can successfully move some files over to the remote host, let's try to install a few services to make our web servers more robust.

### Step 4.3.1
We'll probably need to install Nginx at some point in time on our webservers if we want to properly host traffic at scale.

In order to do this, we'll need to use the `apt` module.  The reason that we are using `apt` specifically is because our target nodes are Ubuntu EC2 instances, and apt is the RPM for that Linux distribution.  

If you'd like, you can take a minute or to to look through the ansible-docs page for `apt`.  If not, this command should work for you.  Take note, because there are a few new options that we'll be introducing with this step:

```shell
ansible -i hosts linux -m apt -a 'name=nginx state=present' -u <user> -b -K
```
Notice that we have both the `-b` and `-K` options.  The `-b` option stands for `become`, which tells Ansible to become the root user when running the command.  The `-K` option adds a prompt when running the command which allows you to manually enter your password, which is necessary when becoming the root user, unless your user is currently set up to run sudo commands without the need for a password.


## Task 4. Run Ansible ad hoc commands to start/stop services on a remote host

### Step 4.4.1
Perhaps we want to ensure that a service is running on a remote host.  We can run the `service` module to start or stop services.  Further, if a service is already started or stopped (which Nginx should be), the command will simply relay that information to us.

To make sure a service is started, run the `service` module like this:

```shell
ansible -i hosts linux -m service -a 'name=nginx state=started' -u rpt -b -K
```
This should return a green status indicating that nothing was changed and that the current status already matches what you're telling Ansible to do.

### Step 4.4.2
Let's go ahead and stop the Nginx service, as we'll be more formally installing it in a future lab.  This is simply the same command, but change the state option to `state=stopped`.

```shell
ansible -i hosts linux -m service -a 'name=nginx state=stopped' -u rpt -b -K
```

## Task 5

### Step 4.5.1
Now that we stopped the service, let's go ahead and remove it altogether.  It should be the same command that we just ran to install it, but change the state to be `state=absent`.

```shell
ansible -i hosts linux -m apt -a 'name=nginx state=absent' -u <user> -b -K
```
