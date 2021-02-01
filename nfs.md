# Setup NFS Server in a Raspberry Pi 4

These are some simple step-by-step instructions to set an NFS server on a Raspberry Pi 4 running Ubuntu 20.04 LTS.

## 1. Mount device

Create a folder where we will mount the partition on the device

```bash
$ sudo mkdir /mnt/usb
$ sudo chown -R nobody:nogroup /mnt/usb
$ sudo chmod 777 /mnt/usb
```

Mount the device. To see the list of devices run `lsblk`.

```bash
$ sudo mount /dev/sda1 /mnt/usb
```

Next, we need to mount it on startup automatically. 

Lets find out the unique id of our disks

```bash
$ sudo blkid
```

Edit file `/etc/fstab` and add the mount at the end of the file.

Example:

```
UUID=<uuid> /mnt/usb ext4 defaults 0 0
```

Reboot the pi and check if it was mounted automatically.

## 2. NFS Server

Install packages:

```bash
$ sudo apt-get install -y nfs-kernel-server
```

Check a new service `nfs-server` is running

```bash
$ sudo systemctl status nfs-server
```

Edit the config file `/etc/exports`.

Once you have opened the file, you can allow access to:

- A single client by adding the following line in the file:

```
/mnt/sharedfolder clientIP(rw,sync,no_subtree_check)
```

- Multiple clients by adding the following lines in the file:

```
/mnt/sharedfolder client1IP(rw,sync,no_subtree_check)
/mnt/sharedfolder client2IP(rw,sync,no_subtree_check)
```

- Multiple clients, by specifying an entire subnet that the clients belong to:

```
/mnt/sharedfolder subnetIP/24(rw,sync,no_subtree_check)
```



The permissions “rw,sync,no_subtree_check” permissions defined in this file mean that the client(s) can perform:

- **rw**: read and write operations
- **sync**: write any change to the disc before applying it
- **no_subtree_check**: prevent subtree checking



In our example, we want to grant access to our mounted drive to an entire subnet:

```
/mnt/usb 192.168.0.0/24(rw,sync,no_subtree_check)
```

After making all the above configurations in the host system, now is the time to export the shared directory through the following command as sudo:

```bash
$ sudo exportfs -a
```

and restart `ntfs-kernel-server` service:

```bash
$ sudo systemctl restart nfs-kernel-server
```

## 3. Mount NFS  on client

Install apt package:

```bash
$ sudo apt install nfs-common
```

Create a mount point for the NFS host's shared folder:

```bash
$ sudo mkdir -p /mnt/nfs/pi
```

Mount shared directory on the client:

```bash
$ sudo mount <serverIP>:/<exportFolder> /mnt/nfs/pi
```