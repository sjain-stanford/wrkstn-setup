## Workstation Setup

### CUDA toolkit installation on RHEL (includes NVIDIA CUDA drivers v384.81, toolkit and samples)
Reference:
[GTX 1080 Ti User Guide](cuda/GTX_1080_Ti_User_Guide.pdf)
[CUDA Installation Guide Linux](cuda/CUDA_Installation_Guide_Linux.pdf)

#### Pre-installation steps:
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
$ sudo yum install kernel-devel-2.6.32-642.el6.x86_64 kernel-headers-2.6.32-642.el6.x86_64
```

#### Runfile installation:
6. Download NVIDIA CUDA Toolkit from [here](https://developer.nvidia.com/cuda-downloads). Specs: Linux - x86_64 - RHEL - 6 - runfile (local). Filename: ```cuda_9.0.176_384.81_linux.run```

7. Uninstall previous toolkit/driver installations to avoid conflict.
```
$ sudo /usr/local/cuda-X.Y/bin/uninstall_cuda_X.Y.pl
$ sudo /usr/bin/nvidia-uninstall
```

8. Disable Nouveau drivers prior to installing display drivers.
* First check if nouveau drivers are loaded:
```
$ lsmod | grep nouveau
```
* Create a file at ```/etc/modprobe.d/blacklist-nouveau.conf``` and add the following contents:
```
blacklist nouveau
options nouveau modeset=0
```
* Regenerate the kernel initramfs:
```
$ sudo dracut --force
```

9. Reboot into text mode (non-GUI). This is required to completely unload Nouveau drivers and prevent the graphical interface from loading.
```
$ sudo /sbin/init 3
```

10. Verify that the Nouveau drivers are not loaded.
```
$ lsmod | grep nouveau
```

11. Run installer. Follow the on-screen prompts and specify paths for installation (unless default). The openGL libraries are selected for install since the GPU used for display is also an NVIDIA GPU (Quadro K420).
```
$ sudo sh cuda_9.0.176_384.81_linux.run
```
See installation logfile [here](cuda/cuda_install_4494.log).

12. Reboot the system to reload the graphical interface.

13. Verify the device nodes are created properly. Check that the device files ```/dev/nvidia*``` exist and have correct (0666) file permissions.

#### Post-installation steps:
14. Ensure the ```PATH``` variable includes ```/usr/local/cuda-9.0/bin``` or the custom path specified during installation.
```
$ export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
```

15. Ensure the ```LD_LIBRARY_PATH``` includes ```/usr/local/cuda-9.0/lib64``` or the custom path specified during installation.
```
$ export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64\
${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

16. Verify the NVIDIA driver version.
```
$ cat /proc/driver/nvidia/version
```
```
NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.81  Sat Sep  2 02:43:11 PDT 2017
GCC version:  gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC)
```

17. In case of two GPUs, in order to dedicate one for X display and another for CUDA compute, follow the steps listed in [Doc-1](cuda/Two_GPU_config-CUDA_Compute_and_X_Display.pdf) and [Doc-2]((cuda/Two_GPU_config-StackOverflow.pdf)).
