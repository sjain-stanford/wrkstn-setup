## Workstation Setup

### CUDA toolkit installation (includes NVIDIA driver, CUDA toolkit and samples)
Reference:
[GTX 1080 Ti User Guide](cuda/GTX_1080_Ti_User_Guide.pdf)
[CUDA Installation Guide Linux](cuda/CUDA_Installation_Guide_Linux.pdf)

1. Physically install the NVIDIA GeForce GTX 1080 Ti card on PCI Express 3.0 dual width x16 slot of motherboard and connect 6-pin and 8-pin power adaptors.

2. Verify that the system has a CUDA-capable GPU. If the GPU listed by ```lspci``` is listed [here](https://developer.nvidia.com/cuda-gpus), it is CUDA-capable.
```
$ update-pciids
$ lspci | grep -i nvidia
```
```
02:00.0 VGA compatible controller: NVIDIA Corporation GK107GL [Quadro K420] (rev a1)
02:00.1 Audio device: NVIDIA Corporation GK107 HDMI Audio Controller (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] (rev a1)
03:00.1 Audio device: NVIDIA Corporation GP102 HDMI Audio Controller (rev a1)
```

3. Verify if Linux version is supported.
```
$ uname -m && cat /etc/*release
```
```
x86_64
Red Hat Enterprise Linux Workstation release 6.8 (Santiago)
```

4. Verify if gcc is installed.
```
$ gcc --version
```
```
gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-17)
```

5. Verify the system has the correct kernel headers and development packages installed.
```
$ uname -r
```
```
2.6.32-642.el6.x86_64
```
```
yum install kernel-devel-2.6.32-642.el6.x86_64 kernel-headers-2.6.32-642.el6.x86_64
```

6. 
>> Check if nouveau drivers are present (should list something):
lsmod | grep nouveau

>> In the file "/etc/modprobe.d/blacklist-nouveau.conf", enter the following:
blacklist nouveau
options nouveau modeset=0

>> Run this to regenerate kernel initramfs after blacklisting nouveau:
sudo dracut --force

>> Reboot into text mode (non-GUI) (run as root)
/sbin/init 3

>> Verify nouveau drivers are not loaded (should list nothing):
lsmod | grep nouveau

>> Run installer
sudo sh cuda_<version>_linux.run

>> Follow on screen prompts
>> Installation Logfile: /tmp/cuda_install_4494.log

3) Post installation
>> PATH should include /scratch/cuda-9.0/bin
>> LD_LIBRARY_PATH should include /scratch/cuda-9.0/lib64

>> Verify driver version:
cat /proc/driver/nvidia/version
