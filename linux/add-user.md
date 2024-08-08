### Add User

```bash
useradd username -m -s /bin/bash -c "comment"
usermmod -aG sudo,adm,docker username
passwd username
```