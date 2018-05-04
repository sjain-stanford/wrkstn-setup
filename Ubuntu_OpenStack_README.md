## OpenStack Ubuntu Instance Setup (WIP)

Set the correct network proxies.
```
$ sudo vim /etc/environment
```
Add these lines.
```
http_proxy="http://abc.def.com:8080/"
https_proxy="https://abc.def.com:8080/"
```
Source the file to set proxies.
```
source /etc/environment
```

Update `apt` package lists.
```
$ sudo apt-get update
```

Verify installed GPU cards.
```
$ sudo update-pciids
Downloaded daily snapshot dated 2018-04-06 03:15:02

$ lspci | grep NVIDIA
00:05.0 VGA compatible controller: NVIDIA Corporation GV100 [TITAN V] (rev a1)
00:06.0 VGA compatible controller: NVIDIA Corporation GV100 [TITAN V] (rev a1)
00:07.0 VGA compatible controller: NVIDIA Corporation GV100 [TITAN V] (rev a1)
00:08.0 VGA compatible controller: NVIDIA Corporation GV100 [TITAN V] (rev a1)
```

Install CUDA toolkit, NVIDIA drivers and TensorFlow (on the boot drive), as per [here](https://github.com/sjain-stanford/wrkstn-setup/blob/master/Ubuntu_README.md). Install gvim. Update dot files (`.bashrc`, `.bash_profile`, `.vimrc`, etc.) to set correct paths, aliases and gvim config. Take an OpenStack snapshot.

After snapshot is taken for boot drive, format NVMe drive and update `/etc/fstab` per steps below.

Verify NVMe PCIe storage drive.
```
$ lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     253:0    0   40G  0 disk
└─vda1  253:1    0   40G  0 part /
nvme0n1 259:0    0  2.9T  0 disk
```

Partition NVMe drive using `gparted`.
```
$ sudo apt-get install gparted
$ sudo gparted
```

From the GUI, do the following:

1) Device -> Create Partition Table (select gpt) 
2) Partition -> New
3) Select `ext4` filesystem and hit Apply (with defaults)
4) GParted -> Refresh Devices

Verify the new partition `nvme0n1p1`.
```
$ lsblk

NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda         253:0    0   40G  0 disk
└─vda1      253:1    0   40G  0 part /
nvme0n1     259:0    0  2.9T  0 disk
└─nvme0n1p1 259:1    0  2.9T  0 part
```

Create a mount point and mount the new partition.
```
$ sudo mkdir /data
$ sudo mount /dev/nvme0n1p1 /data
```

Verify the mount is successful.
```
$ lsblk

NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda         253:0    0   40G  0 disk
└─vda1      253:1    0   40G  0 part /
nvme0n1     259:0    0  2.9T  0 disk
└─nvme0n1p1 259:1    0  2.9T  0 part /data
```

Change ownership from root to user.
```
$ ls -ltra

total 24
drwx------  2 root root 16384 Apr 28 00:45 lost+found
drwxr-xr-x  3 root root  4096 Apr 28 00:45 .
drwxr-xr-x 24 root root  4096 Apr 28 00:50 ..

$ sudo chown -R ubuntu /data/
$ ls -ltra

total 24
drwx------  2 ubuntu root 16384 Apr 28 00:45 lost+found
drwxr-xr-x  3 ubuntu root  4096 Apr 28 00:45 .
drwxr-xr-x 24 root   root  4096 Apr 28 00:50 ..
```

To mount every time, update `/etc/fstab`. First find the UUID of NVMe drive using `blkid`.
```
$ sudo blkid

/dev/vda1: LABEL="cloudimg-rootfs" UUID="b5702e2b-9f3b-42ad-8248-878e16b622c8" TYPE="ext4" PARTUUID="c0f2ee80-01"
/dev/nvme0n1p1: UUID="6d62d888-1fe9-404f-a7e8-214ea2bae5be" TYPE="ext4" PARTUUID="f7011ea8-607c-4d17-9d7f-5f061d306d64"
```
Then edit.
```
$ sudo vim /etc/fstab
```
Add these lines.
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system>                           <mount point>   <type>  <options>       <dump>  <pass>
UUID=6d62d888-1fe9-404f-a7e8-214ea2bae5be /data           ext4    defaults        0       0
```
