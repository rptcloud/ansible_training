## Validate a target host is reachable by Ansible

```bash
ansible all -m ping
```
127.0.0.1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

## Get Information about a target host via Ansible

```
ansible all -m gather_facts
```


