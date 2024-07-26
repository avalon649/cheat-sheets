### Creating a Samba Share on Ubuntu

1. Update Your System:
  
    Start by ensuring your Ubuntu system is up-to-date:

```bash
    sudo apt update
    sudo apt upgrade
```

2. Install Samba:

```bash
    sudo apt install samba
```

3. Create a Shared Directory:
  
   Create a directory where you want to share files. For example:

```bash
    sudo mkdir /srv/shared
```
  
  Make the directory accessible to the user who will manage the Samba share:

```bash
    sudo chown -R yourusername:yourusername /srv/shared
```

4. Configure Samba's Global Options (smb.conf):

    Open the Samba configuration file:

```bash
    sudo nano /etc/samba/smb.conf
```

  Add the following global settings (adjust as needed):

```text
    [global]
    workgroup = WORKGROUP
    security = user
    log level = info
```

   workgroup: Set your Samba workgroup (e.g., "WORKGROUP").

   security: Use "user" for basic security or "ntlm" for more robust Windows authentication.

   log level: Control the amount of logging.

   Save and close the file.


5. Configure a Share:

   Add a share definition within the [global] section (or create a new section):

```text
    [shared_folder]
    path = /srv/shared
    valid users = yourusername
    read only = no
    guest ok = no
```

   shared_folder: The name of your share.

   path: The directory you created in step 3.

   valid users: List the usernames allowed to access this share. Use @all for all users.

   read only: Set to "no" for read/write access.

   guest ok: Set to "no" to disable guest access.


6. Update Firewall Rules:

   Allow Samba traffic through your firewall (e.g., ufw):

```bash
    sudo ufw allow samba
```



7. Restart Samba:

    Restart the Samba service:

```bash
    sudo systemctl restart smbd
```

8. Connect to the Share:

```text
    From a Windows machine, use File Explorer to connect to the share: \\your_ubuntu_ip_address\shared_folder
```

