## Ansible Variables

Variables are another thing that makes Ansible powerful. By being able to write generic commands, with decision trees based on variables, we can automate almost anything. This lab will walk through a couple of things we can do with variables to help burn in the concept.

We want to only install tcpdump on servers with admin in their hostnames. And we want Git installed, but only on servers running RedHat. We can do it using Ansible variables.

## Write a Playbook That Removes `tcpdump` When a Server's Name Does Not Contain `admin`

Let's verify whether or not tcpdump is installed:

```
rpm -q tcpdump
```

Let's write a playbook to remove tcpdump if the server does not have admin in their host name.

```ansible
---
# Variables playbook

- name: This playbook will remove tcpdump (if installed) from servers without admin in their hostnames
  hosts: all
  become: yes
  tasks:
   - name: Remove tcpdump from all but admin servers
     yum:
      name: tcpdump
      state: absent
     when: "'admin' not in inventory_hostname"
```

```bash
hostname
Server1
```

As you can see our server does not have `admin` in it's hostname so it will remove `tcpdump`

```bash
ansible-playbook variables.yml
```

```bash
rpm -q tcpdump
package tcpdump is not installed
```

## Modify That Playbook to Install Git on RedHat Servers

Check to see if the server you are running on is a RedHat server
```
ansible all -m gather_facts | grep ansible_os_family

"ansible_os_family": "RedHat",
```

Check to see if the server you are running has git installed.  If so, manually remove it.

```
rpm -q git
yum remove git -y
```

Install `git` with Ansible playbook.  Update the `variables.yml` playbook to include the install git task.

Since it is, let's validate our playbook will succesfully install `git`
```txt
   - name: Make sure git is installed only on Red Hat servers
     yum:
      name: git
      state: present
     when: ansible_facts['os_family'] == 'RedHat'
```

In the output, we'll see that something changed. Let's run `rpm -q` git again, and we'll see that the playbook installed the software correctly.