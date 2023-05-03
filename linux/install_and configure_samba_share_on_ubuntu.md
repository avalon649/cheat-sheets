### How to configure Samba Server share on Ubuntu 22.04

Open a command line terminal and install Samba server.

```bash
     sudo apt update
     sudo apt-get install samba-server
```

We will be starting with a fresh clean configuration file, while we also keep the default config file as a backup for reference purposes. Execute the following Linux commands to make a copy of the existing configuration file and create a new /etc/samba/smb.conf configuration file:

```bash
     sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_backup
     sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf_backup | grep . > /etc/samba/smb.conf'
```

Samba has its own user management system. However, any user existing on the samba user list must also exist within the /etc/passwd file. If your system user does not exist yet, hence cannot be located within /etc/passwd file, first create a new user using the useradd command before creating any new Samba user. Once your new system user eg. linuxconfig exits, use the smbpasswd command to create a new Samba user:

```bash
    sudo smbpasswd -a linuxconfig
```

New SMB password:
Retype new SMB password:
Added user linuxconfig.

Next step is to add the home directory share. Use your favourite text editor, ex. atom, sublime, to edit our new /etc/samba/smb.conf Aamba configuration file and add the following lines to the end of the file:

```config
    [homes]
       comment = Home Directories
       browseable = yes
       read only = no
       create mask = 0700
       directory mask = 0700
       valid users = %S
```

Optionally, add a new publicly available read-write Samba share accessible by anonymous/guest users. First, create a directory you wish to share and change its access permission:

```bash
     sudo mkdir /var/samba
     sudo chmod 777 /var/samba/
```

Once ready, once again open the /etc/samba/smb.conf samba configuration file and add the following lines to the end of the file:

```config
    [public]
      comment = public anonymous access
      path = /var/samba/
      browsable =yes
      create mask = 0660
      directory mask = 0771
      writable = yes
      guest ok = yes
```

Check your current configuration. Your /etc/samba/smb.conf samba configuration file should at this stage look similar to the one below: