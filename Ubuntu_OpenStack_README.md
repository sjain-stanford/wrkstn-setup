## OpenStack Ubuntu Instance Setup (WIP)

Set the correct network proxies.
```
$ sudo vim /etc/environment
```
Add this
```
http_proxy="http://abc.def.com:8080/"
https_proxy="https://abc.def.com:8080/"
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
3) Select `ext4` and hit Apply (with defaults)
4) GParted -> Refresh Devices

