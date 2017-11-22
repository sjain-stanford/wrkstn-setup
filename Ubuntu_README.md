# Ubuntu Workstation Setup for Training Neural Networks


```
$ sudo update-pciids
$ lspci | grep -i nvidia

02:00.0 VGA compatible controller: NVIDIA Corporation GK107GL [Quadro K420] (rev a1)
02:00.1 Audio device: NVIDIA Corporation GK107 HDMI Audio Controller (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] (rev a1)
03:00.1 Audio device: NVIDIA Corporation GP102 HDMI Audio Controller (rev a1)
```

```
$ uname -m && cat /etc/*release

x86_64
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
```

```
$ gcc --version
gcc (Ubuntu 5.4.0-6ubuntu1~16.04.5) 5.4.0 20160609
```

```
$ uname -r
4.4.0-101-generic

$ sudo apt-get install linux-headers-4.4.0-101-generic
Reading package lists... Done
Building dependency tree
Reading state information... Done
linux-headers-4.4.0-101-generic is already the newest version (4.4.0-101.124).
```

CUDA 8.0 GA2
https://developer.nvidia.com/cuda-80-ga2-download-archive
Linux -> x86_64 -> Ubuntu -> 16.04 -> runfile

```
$ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run -P /scratch/setup/
$ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/patches/2/cuda_8.0.61.2_linux-run -P /scratch/setup/
```

```
$ lsmod | grep nouveau
nouveau              1495040  3
mxm_wmi                16384  1 nouveau
video                  40960  1 nouveau
i2c_algo_bit           16384  1 nouveau
ttm                    98304  1 nouveau
drm_kms_helper        155648  1 nouveau
drm                   364544  6 ttm,drm_kms_helper,nouveau
wmi                    20480  2 mxm_wmi,nouveau
```

Add to `/etc/modprobe.d/blacklist-nouveau.conf`
```
blacklist nouveau
options nouveau modeset=0
```

```
$ sudo update-initramfs -u
```
Reboot in text mode (run level 3)
```
$ sudo telinit 3
```

http://ubuntuhandbook.org/index.php/2014/01/boot-into-text-console-ubuntu-linux-14-04/
