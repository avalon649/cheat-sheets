### How to configure Samba Server share on Ubuntu 22.04

1. `Open a command line terminal and install Samba server.`

```bash
     sudo apt update
     sudo apt-get install samba-server
```

2. `Setting up Samba`

     Now that Samba is installed, we need to create a directory for it to share:

```bash
mkdir /home/<username>/sambashare/
```

     The command above creates a new folder sambashare in our home directory which we will share later.

     The configuration file for Samba is located at /etc/samba/smb.conf. To add the new directory as a share, we edit the file by running:

```bash
cp /etc/samba/smb.conf /etc /samba/smb.conf.bak
sudo nano /etc/samba/smb.conf
```

     At the bottom of the file, add the following lines:

```bash
[sambashare]
    comment = Samba on Ubuntu
    path = /home/username/sambashare
    read only = no
    browsable = yes
```

     Then press Ctrl-O to save and Ctrl-X to exit from the nano text editor.
     What we’ve just added

     Now that we have our new share configured, save it and restart Samba for it to take effect:

```bash
sudo service smbd restart
```

     Update the firewall rules to allow Samba traffic:

```bash
sudo ufw allow samba
```

3. `Setting up User Accounts and Connecting to Share`

     Since Samba doesn’t use the system account password, we need to set up a Samba password for our user account:

```bash
sudo smbpasswd -a username
```