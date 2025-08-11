# Ansible
## Install Ansible on Ubuntu

1. Install PIP

```bash
sudo apt install python3-pip
```

2. Install Ansible

```bash
pip3 install ansible
```

3. Add execution path

```bash

ansible -i ./inventory/hosts ubuntu -m ping --user someuser --ask-pass
```

```bash
ansible-playbook ./playbooks/apt.yml --user someuser --ask-pass --ask-become-pass -i ./inventory/hosts
```