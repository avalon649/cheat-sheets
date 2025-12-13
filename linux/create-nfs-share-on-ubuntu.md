### Install and Configure NFS Server on Ubuntu:

Source `chatgpt`

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

Create a directory that will be shared over NFS. For example, let's create a directory called "nfs_share" in the "/mnt" directory:

```bash
sudo mkdir /mnt/nfs_share
```

Edit the NFS server configuration file at "/etc/exports" to specify the directory to be shared and the clients that are allowed to access it. For example, to allow access to all clients on the network, add the following line to the end of the file:

```bash
/mnt/nfs_share *(rw,sync,no_subtree_check)
```

This configuration allows read and write access to the shared directory, and synchronizes file changes between the NFS server and clients.

If you want to restrict access to specific IP addresses or subnets, replace the "*" with the IP address or subnet in CIDR notation. For example, to allow access to only the 192.168.1.0/24 subnet, use:

```bash
/mnt/nfs_share 192.168.1.0/24(rw,sync,no_subtree_check)
```

Save and exit the file.

Export the shared directory using the exportfs command:

```bash
sudo exportfs -a
```

Start the NFS server:

```bash
sudo systemctl start nfs-kernel-server
```

To automatically start the NFS server at boot time, enable it:

```bash
sudo systemctl enable nfs-kernel-server
```

That's it! Your NFS server is now up and running, and clients on the network that are allowed to access it can mount the shared directory using the server's IP address and the directory path you specified in the "/etc/exports" file.