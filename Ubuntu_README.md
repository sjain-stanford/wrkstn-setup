# Ubuntu Workstation Setup for Training Neural Networks
## Work in Progress (WIP)!!

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

Latest NVIDIA drivers (v384.98, released 2017.11.2)
http://www.nvidia.com/Download/index.aspx
GeForce -> GeForce 10 -> GeForce GTX 1080 Ti -> Linux 64-bit



```
$ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run -P /scratch/setup/cuda
$ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/patches/2/cuda_8.0.61.2_linux-run -P /scratch/setup/cuda
$ wget http://us.download.nvidia.com/XFree86/Linux-x86_64/384.98/NVIDIA-Linux-x86_64-384.98.run -P /scratch/setup/cuda
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

Add to `/etc/modprobe.d/blacklist-nouveau.conf` (open as root)
```
blacklist nouveau
options nouveau modeset=0
```
```
$ sudo update-initramfs -u
```

Reboot
```
reboot
```
http://ubuntuhandbook.org/index.php/2014/01/boot-into-text-console-ubuntu-linux-14-04/
Exit GUI (X server) and enter terminal mode (CTRL+ALT+F1), then kill GUI (lightdm stop) then install CUDA toolkit 8.0, patch and NVIDIA driver v384.98
```
$ lsmod | grep nouveau
$ sudo service lightdm stop
$ sudo sh cuda_8.0.61_375.26_linux-run
$ sudo sh cuda_8.0.61.2_linux-run
$ sudo sh NVIDIA-Linux-x86_64-384.98.run
```

Add 
```
$ export PATH="/scratch/cuda-8.0/bin:$PATH"
$ export LD_LIBRARY_PATH="/scratch/cuda-8.0/lib64:/scratch/cuda-8.0/extras/CUPTI/lib64:$LD_LIBRARY_PATH"
```

```
$ sudo service lightdm start
Ctrl + Alt + F7 to get back to X server (GUI)
$ reboot
```
Verify the NVIDIA driver version.
```
$ cat /proc/driver/nvidia/version

NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.98  Thu Oct 26 15:16:01 PDT 2017
GCC version:  gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.5)
```

Verify the CUDA toolkit version.
```
$ nvcc -V

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2016 NVIDIA Corporation
Built on Tue_Jan_10_13:22:03_CST_2017
Cuda compilation tools, release 8.0, V8.0.61
```

$ nvidia-smi

In case of two GPUs, in order to dedicate one for X display and another for CUDA compute, follow the steps listed here or PDF-1 and PDF-2. For instance to force GPU 1 (K420) to X Display and GPU 2 (1080 Ti) to CUDA compute, find Bus ID of K420 using nvidia-smi -a and add this line to /etc/X11/xorg.conf under 'Device' section to force K420 for X display.

BusID    "PCI:2:0:0"


cuDNN 6.0 (sign up)
https://developer.nvidia.com/rdp/cudnn-download

cuDNN 7.0 installation on RHEL

These steps are to install cuDNN 7.0, however for the final config we will only use cuDNN 6.0 (use similar steps).

Reference: cuDNN Installation Guide

Download the cuDNN Tar file from here. Unzip the cuDNN package.
```
$ tar -xzvf cudnn-8.0-linux-x64-v6.0.tgz
```
Copy the following files to the cuda installation directory.
```
$ sudo cp cuda/include/cudnn.h /scratch/cuda-8.0/include/
$ sudo cp cuda/lib64/libcudnn* /scratch/cuda-8.0/lib64/
$ sudo chmod a+r /scratch/cuda-8.0/include/cudnn.h /scratch/cuda-8.0/lib64/libcudnn*
```

Install Tensorflow (Ubuntu Prebuilt binary -> virtualenv)

https://www.tensorflow.org/install/install_linux

```
$ sudo apt-get install libcupti-dev
$ sudo apt-get install python3-pip python3-dev python-virtualenv
```

```
$ which python3
/usr/bin/python3

$ python3 --version
Python 3.5.2
```

```
$ sudo su -
$ apt install virtualenv
```
$ virtualenv --system-site-packages -p python3 /scratch/tensorflow
$ source /scratch/tensorflow/bin/activate
$ pip3 install --upgrade tensorflow-gpu

```
(tensorflow) sambhavj@xsjsambhavj50:/scratch$ pip3 install --upgrade tensorflow-gpu
Collecting tensorflow-gpu
  Downloading tensorflow_gpu-1.4.0-cp35-cp35m-manylinux1_x86_64.whl (170.1MB)
    100% |████████████████████████████████| 170.1MB 10kB/s
Collecting tensorflow-tensorboard<0.5.0,>=0.4.0rc1 (from tensorflow-gpu)
  Downloading tensorflow_tensorboard-0.4.0rc3-py3-none-any.whl (1.7MB)
    100% |████████████████████████████████| 1.7MB 522kB/s
Collecting numpy>=1.12.1 (from tensorflow-gpu)
  Downloading numpy-1.13.3-cp35-cp35m-manylinux1_x86_64.whl (16.9MB)
    100% |████████████████████████████████| 16.9MB 88kB/s
Requirement already up-to-date: wheel>=0.26 in ./tensorflow/lib/python3.5/site-packages (from tensorflow-gpu)
Collecting protobuf>=3.3.0 (from tensorflow-gpu)
  Downloading protobuf-3.5.0-cp35-cp35m-manylinux1_x86_64.whl (6.4MB)
    100% |████████████████████████████████| 6.4MB 196kB/s
Collecting enum34>=1.1.6 (from tensorflow-gpu)
  Using cached enum34-1.1.6-py3-none-any.whl
Collecting six>=1.10.0 (from tensorflow-gpu)
  Using cached six-1.11.0-py2.py3-none-any.whl
Collecting bleach==1.5.0 (from tensorflow-tensorboard<0.5.0,>=0.4.0rc1->tensorflow-gpu)
  Using cached bleach-1.5.0-py2.py3-none-any.whl
Collecting werkzeug>=0.11.10 (from tensorflow-tensorboard<0.5.0,>=0.4.0rc1->tensorflow-gpu)
  Using cached Werkzeug-0.12.2-py2.py3-none-any.whl
Collecting markdown>=2.6.8 (from tensorflow-tensorboard<0.5.0,>=0.4.0rc1->tensorflow-gpu)
  Using cached Markdown-2.6.9.tar.gz
Collecting html5lib==0.9999999 (from tensorflow-tensorboard<0.5.0,>=0.4.0rc1->tensorflow-gpu)
  Using cached html5lib-0.9999999.tar.gz
Requirement already up-to-date: setuptools in ./tensorflow/lib/python3.5/site-packages (from protobuf>=3.3.0->tensorflow-gpu)
Building wheels for collected packages: markdown, html5lib
  Running setup.py bdist_wheel for markdown ... done
  Stored in directory: /home/sambhavj/.cache/pip/wheels/bf/46/10/c93e17ae86ae3b3a919c7b39dad3b5ccf09aeb066419e5c1e5
  Running setup.py bdist_wheel for html5lib ... done
  Stored in directory: /home/sambhavj/.cache/pip/wheels/6f/85/6c/56b8e1292c6214c4eb73b9dda50f53e8e977bf65989373c962
Successfully built markdown html5lib
Installing collected packages: six, html5lib, bleach, werkzeug, markdown, protobuf, numpy, tensorflow-tensorboard, enum34, tensorflow-gpu
  Found existing installation: six 1.10.0
    Not uninstalling six at /usr/lib/python3/dist-packages, outside environment /scratch/tensorflow
  Found existing installation: html5lib 0.999
    Not uninstalling html5lib at /usr/lib/python3/dist-packages, outside environment /scratch/tensorflow
Successfully installed bleach-1.5.0 enum34-1.1.6 html5lib-0.9999999 markdown-2.6.9 numpy-1.13.3 protobuf-3.5.0 six-1.11.0 tensorflow-gpu-1.4.0 tensorflow-tensorboard-0.4.0rc3 werkzeug-0.12.2

```


Install Keras inside virtualenv

```
(tensorflow) sambhavj@xsjsambhavj50:/scratch$ pip install keras
Collecting keras
  Downloading Keras-2.1.1-py2.py3-none-any.whl (302kB)
    100% |████████████████████████████████| 307kB 1.2MB/s
Collecting pyyaml (from keras)
  Downloading PyYAML-3.12.tar.gz (253kB)
    100% |████████████████████████████████| 256kB 1.2MB/s
Requirement already satisfied: numpy>=1.9.1 in ./tensorflow/lib/python3.5/site-packages (from keras)
Requirement already satisfied: six>=1.9.0 in ./tensorflow/lib/python3.5/site-packages (from keras)
Collecting scipy>=0.14 (from keras)
  Downloading scipy-1.0.0-cp35-cp35m-manylinux1_x86_64.whl (49.6MB)
    100% |████████████████████████████████| 49.6MB 34kB/s
Building wheels for collected packages: pyyaml
  Running setup.py bdist_wheel for pyyaml ... done
  Stored in directory: /home/sambhavj/.cache/pip/wheels/2c/f7/79/13f3a12cd723892437c0cfbde1230ab4d82947ff7b3839a4fc
Successfully built pyyaml
Installing collected packages: pyyaml, scipy, keras
Successfully installed keras-2.1.1 pyyaml-3.12 scipy-1.0.0
```
