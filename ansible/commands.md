```bash
ansible -i ./inventory/hosts ubuntu -m ping --user someuser --ask-pass
```

```bash
ansible-playbook ./playbooks/apt.yml --user someuser --ask-pass --ask-become-pass -i ./inventory/hosts
```