1. Install Ubuntu server (Minimal)
2. Install updates and upgrades
```
sudo apt install nfs-kernel-server
```
3. At the root of the server
```
cd
```

 ```
 mkdir -p /mnt/nfs_share
 ```

4. Setup directory permissions to the new folder
 ```
 chown -R nobody:nogroup /mnt/nfs_share
 ```

5. Setup file permissions

```
chmod 777 /mnt/nfs_share
```

6. Install nano (Optional if using minimal server)

 ```
 apt install nano
 ```

7. Grant NFS access

```
nano /etc/exports
```
8. Add:

```
/mnt/nfs_share <IP ADDRESS>/24(rw,sync,no_subtree_check,crossmnt,fsid=0)
```

9. Export nfs directory

 ```
 exportfs -a
 ```
 
```
exportfs -v
```

10. Restart the service

```
systemctl restart nfs-kernel-server
```

11. Update the IP address of the server

```
cd /etc/netplan
```

```
nano <filename>config.yaml
```

```
network:

  ethernets:

    enp0s18:

      dhcp4: false

      addresses: [<IP ADDRESS>/24]

      routes:

        - to: default

          via: <ROUTER IP ADDRESS>

     nameservers:

       addresses: [8.8.8.8 , 1.1.1.1]

  version: 2

```

```
netplan apply
```

- Reboot

12. Install firewall stuff
```
apt install ufw
```

```
ufw allow from <ROUTER IP ADDRESS>/224 to any port nfs
```

```
ufw enable
```

```
ufw status
```

13. Connect from host to test the share

14. Create a test file on the share from another machine and connect to NFS server to verify the contents

15. Set permissions on server for folder

```
chmod -R 777 'folder location'/
```

## Troubleshooting

```
Systemctl start nfs-server.service
```

```
Systemctl start rpcbind.service
```

```
Mountstats
```

```
Nano /etc/nfs.conf (manage-gids=y)
```

- Check server side if running: 
```
rpcinfo -p
```

Check client side: 
```
rpcinfo -p <IP ADDRESS>
```

- ADD Client side in /etc/fstab

- NFS share location

- Run: 
```
mount -a
```
- Service status: 
```
service nfs status
```

> Replace any <IP ADDRESS> with your local IP address for the intended host IP or Router IP. replace <filename> with intended file name.
