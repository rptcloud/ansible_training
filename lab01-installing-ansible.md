# Lab 1: Lab Setup

Duration: 10 minutes

- Task 1: Connect to the Student Workstation
- Task 2: Check your environment for the correct prequisites
- Task 3: Install Ansible
- Task 4: Verify Installation and View the Config File

## Task 1: Connect to the Student Workstation

In the previous lab, you learned how to connect to your workstation with either VSCode, SSH, or the web-based client.

One you've connected, make sure you've navigated to the `/workstation/ansible` directory. This is where we'll do all of our work for this training.

## Task 2: Check your environment for the correct prequisites

### Step 1.2.1

Run the following command to check your Python version

```shell
python --version
```

You should see something along the lines of:

```text
Python 2.7.16
```

In order to properly run Ansible, you will need a minimum of Python 2 (version 2.7) or Python 3 (versions 3.5 and higher) installed.  If you do not currently have the correct version of Python, you will need to update your Python version before proceeding.

If you need to install Python, you can run the following command:

```shell
sudo apt install python
```


### Note: For your managed nodes, Ansible makes a connection over SSH and transfers modules using SFTP. If SSH works but SFTP is not available on some of your managed nodes, you can switch to SCP in ansible.cfg. For any machine or device that can run Python, you also need Python 2 (version 2.6 or later) or Python 3 (version 3.5 or later).


## Task 3: Install Ansible

### Step 1.3.1

Now that we've verified our prequisites, we can install python.  

Run the following command:

```shell
sudo apt-add-repository ppa:ansible/ansible
```
This command will include the official project’s PPA (personal package archive) on your workstation and designate your workstation as an Ansible control node

### Step 1.3.2

Next, run this command to install the ansible software:

```shell
sudo apt update && sudo apt install ansible
```
This will install the Ansible software on your workstation.  From here, you should be able to run the following command to verify successful installation:

```shell
ansible --version
```

And your output will look like this:

```
ansible 2.9.21
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Mar  1 2021, 11:38:31) [GCC 5.4.0 20160609]
```

Note: Your output should look slightly different depending on your operating system and file structure.


You should now have a working Ansible directory set up, as well as the Ansible software up and running on your workstation.

## Task 4: Verify Installation and View the Config File

### Step 1.4.1

Looking at the output of the `ansible --version` command, we can see that Ansible already specificied a default location for our `ansible.cfg` file.  It should look something like this:

`config file = /etc/ansible/ansible.cfg`

Let's go ahead and take a look at that file in order to get a feel for what Ansible is looking for during runtime:

`cat /etc/ansible/ansible.cfg`

You will notice that almost every single line of this config file is commented out.  This is fine, as most of this information will be overridden during runtime for a typical Ansible deployment.  However, notice this block of information:

```text
#inventory      = /etc/ansible/hosts
#library        = /usr/share/my_modules/
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml
#forks          = 5
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
#remote_port    = 22
#module_lang    = C
#module_set_locale = False
```

You will see a few different examples of what some default values might look like.  Specifically, we are looking at the file location of the `inventory`.  This is a default file that is included in every Ansible installation.  We will be taking a good look at this in our next lab.

Take some time to look through the default `ansible.cfg` file.  Don't feel that you need to read all of it.  Just look through some of the commented blocks to start to get an idea of what kinds of information we can pass to Ansible for our future runs.
