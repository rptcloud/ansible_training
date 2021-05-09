## Building and Running Ansible Playbook

## Objective
We're in the midst of a proof of concept project. Now, we've been tasked with writing a playbook that will set our machine up as a web server, with the normal software that our company's web servers typically have running on them. The playbook must install the following software:

```
httpd
git
tcpdump
php
```

Then we need to set up a second playbook that creates the following users:

```
security
devs
admins
```

All of those users should be members of the web group.

## Creating a User
```ansible
---
- name: New user is created
  hosts: webservers
  become: true

  tasks:
    - name: User gets created
      user:
        name: test
        state: present
```
ansible-playbook example.yml

Update `example.yml` to set state to `absent`

ansible-playbook --limit web02 example.yml

## Installing Software
ansible-playbook software.yml --become


## Creating Users

```
ansible-playbook users.yml --become
```

If you re-run the ansible-playbook command you will see that items return as `ok` as ansible is idemponent and the users already exist.

```
ansible-playbook users.yml --become
```

```bash
PLAY [Create required users] *******************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [127.0.0.1]

TASK [group] ***********************************************************************************************
ok: [127.0.0.1]

TASK [user] ************************************************************************************************
ok: [127.0.0.1] => (item=devs)
ok: [127.0.0.1] => (item=security)
ok: [127.0.0.1] => (item=admins)

PLAY RECAP *************************************************************************************************
127.0.0.1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


### Validate Membership
Our output should show things that happened. We can check with `id admins` (which will print out a list of all the groups that admins is a member of). We can check the other users the same way.

### Remove Membership
Change the `user` task `state` to `absent` and re-run the ansible playbook command.

```
ansible-playbook users.yml --become
```

```bash
PLAY [Create required users] *******************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [127.0.0.1]

TASK [group] ***********************************************************************************************
ok: [127.0.0.1]

TASK [user] ************************************************************************************************
changed: [127.0.0.1] => (item=devs)
changed: [127.0.0.1] => (item=security)
changed: [127.0.0.1] => (item=admins)

PLAY RECAP *************************************************************************************************
127.0.0.1                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

We can check with `id admins` (which will print out a list of all the groups that admins is a member of). We can check the other users the same way.

```
id admins
id: ‘admins’: no such user
```