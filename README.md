# RHEL Workstation Setup for Training Neural Networks

This document is a sequential log of my experiments (issues faced and fixes) to setup an RHEL 6.8 workstation for deep learning with NVIDIA GeForce GTX 1080 Ti and Tensorflow (built from source). It is not necessarily the quickest way to setup as it involves some back and forth and downgrade of the initially installed (latest at the time) tool versions. Due to the much restricted RHEL environment and limited package support for direct install, most packages had to be built from source. And compiling packages with interdependencies required the right combination of tool versions to get them to work. Nonetheless here is the **final configuration** of the working setup.
```
RHEL 6.8 (with default gcc 4.4.7)
CUDA 8.0
cuDNN 6.0
python 3.6.3 (Anaconda)
gcc 4.8.4 (build from source)
Bazel 0.5.4 (build from source)
Tensorflow_GPU r1.3 (build from source)
```

## CUDA 9.0 toolkit installation on RHEL (includes NVIDIA CUDA drivers v384.81, toolkit and samples)

These steps are to install CUDA 9.0, however for the final config we will only use CUDA 8.0 (use similar steps).

Reference:
[GTX 1080 Ti User Guide](cuda/GTX_1080_Ti_User_Guide.pdf)
[CUDA Installation Guide Linux](cuda/CUDA_Installation_Guide_Linux.pdf)

### CUDA Pre-installation steps

Physically install the NVIDIA GeForce GTX 1080 Ti card on PCI Express 3.0 dual width x16 slot of motherboard and connect 6-pin and 8-pin power adaptors. Verify that the system has a CUDA-capable GPU. If the GPU listed by `lspci` is listed [here](https://developer.nvidia.com/cuda-gpus), it is CUDA-capable.
```
$ update-pciids
$ lspci | grep -i nvidia

02:00.0 VGA compatible controller: NVIDIA Corporation GK107GL [Quadro K420] (rev a1)
02:00.1 Audio device: NVIDIA Corporation GK107 HDMI Audio Controller (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] (rev a1)
03:00.1 Audio device: NVIDIA Corporation GP102 HDMI Audio Controller (rev a1)
```

Verify if Linux version is supported.
```
$ uname -m && cat /etc/*release

x86_64
Red Hat Enterprise Linux Workstation release 6.8 (Santiago)
```

Verify if gcc is installed.
```
$ gcc --version

gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-17)
```

Verify the system has the correct kernel headers and development packages installed.
```
$ uname -r

2.6.32-642.el6.x86_64

$ sudo yum install kernel-devel-2.6.32-642.el6.x86_64 kernel-headers-2.6.32-642.el6.x86_64
```

### CUDA Runfile installation:
Download NVIDIA CUDA Toolkit from [here](https://developer.nvidia.com/cuda-downloads). Specs: Linux - x86_64 - RHEL - 6 - runfile (local). Filename: `cuda_9.0.176_384.81_linux.run`. Uninstall previous toolkit/driver installations to avoid conflict (not required for runfile method - see Table 2 & Table 3 [here](cuda/CUDA_Installation_Guide_Linux.pdf)). 
```
$ sudo /usr/local/cuda-X.Y/bin/uninstall_cuda_X.Y.pl
$ sudo /usr/bin/nvidia-uninstall
```

Disable Nouveau drivers prior to installing display drivers. First check if nouveau drivers are loaded.
```
$ lsmod | grep nouveau
```

Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` and add the following contents.
```
blacklist nouveau
options nouveau modeset=0
```

Regenerate the kernel initramfs.
```
$ sudo dracut --force
```

Reboot into text mode (non-GUI). This is required to completely unload Nouveau drivers and prevent the graphical interface from loading.
```
$ sudo /sbin/init 3
```

Verify that the Nouveau drivers are not loaded.
```
$ lsmod | grep nouveau
```

Run installer. Follow the on-screen prompts and specify paths for installation (unless default). The openGL libraries are selected for install since the GPU used for display is also an NVIDIA GPU (Quadro K420).
```
$ sudo sh cuda_9.0.176_384.81_linux.run
```

See installation [logfile](cuda/cuda_install_4494.log). Reboot the system to reload the graphical interface. Verify the device nodes are created properly. Check that the device files `/dev/nvidia*` exist and have correct (0666) file permissions.

### CUDA Post-installation steps

Ensure the `PATH` variable includes `/usr/local/cuda-9.0/bin` or the custom path specified during installation.
```
$ export PATH="/scratch/cuda-9.0/bin:$PATH"
```

Ensure the `LD_LIBRARY_PATH` includes `/usr/local/cuda-9.0/lib64` or the custom path specified during installation. Also include path to `libcupti.so` CUDA libraries.
```
$ export LD_LIBRARY_PATH="/scratch/cuda-9.0/lib64:/scratch/cuda-9.0/extras/CUPTI/lib64:$LD_LIBRARY_PATH"
```

Verify the NVIDIA driver version.
```
$ cat /proc/driver/nvidia/version

NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.81  Sat Sep  2 02:43:11 PDT 2017
GCC version:  gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC)
```

Verify the CUDA toolkit version.
```
$ nvcc -V

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Sep__1_21:08:03_CDT_2017
Cuda compilation tools, release 9.0, V9.0.176
```

NVIDIA-SMI
```
$ nvida-smi
```
![nvida-smi](cuda/nvidia-smi.png)

In case of two GPUs, in order to dedicate one for X display and another for CUDA compute, follow the steps listed [here](http://nvidia.custhelp.com/app/answers/detail/a_id/3029/~/using-cuda-and-x) or [PDF-1](cuda/Two_GPU_config-CUDA_Compute_and_X_Display.pdf) and [PDF-2](cuda/Two_GPU_config-StackOverflow.pdf). For instance to force GPU 1 (K420) to X Display and GPU 2 (1080 Ti) to CUDA compute, find Bus ID of K420 using `nvidia-smi -a` and add this line to `/etc/X11/xorg.conf` under 'Device' section to force K420 for X display.
```
BusID    "PCI:2:0:0"
```

Compile the CUDA sample programs by changing to `~/NVIDIA_CUDA-9.0_Samples` and type `make`. Run the resulting binaries from `~/NVIDIA_CUDA-9.0_Samples/bin`. Results from `deviceQuery` and `bandwidthTest` are shown below.

![deviceQuery](cuda/deviceQuery_Result.png)
![bandwidthTest](cuda/bandwidthTest_Result.png)

Note: If for some reason the GPU doesn't show up (ERR!) on `nvidia-smi` or isn't detected as a CUDA compatible device when running `deviceQuery`, one of the reasons is that the drivers installed by the toolkit are outdated and need to be updated. Download the latest drivers from [here](http://www.nvidia.com/Download/index.aspx?lang=en-us) and install them separately after killing GUI. E.g driver version 384.98 for Linux 64bit systems is installed.
```
$ sudo /sbin/init 3
$ sudo sh NVIDIA-Linux-x86_64-384.98.run
```

Follow the prompts to finish installation and reboot. Verify the driver version.
```
$ cat /proc/driver/nvidia/version

NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.98  Thu Oct 26 15:16:01 PDT 2017
GCC version:  gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC)
```

## cuDNN 7.0 installation on RHEL

These steps are to install cuDNN 7.0, however for the final config we will only use cuDNN 6.0 (use similar steps).

Reference:
[cuDNN Installation Guide](cuda/cuDNN-Installation-Guide.pdf)

Download the cuDNN Tar file from [here](https://developer.nvidia.com/cudnn). File version: cuDNN v7.0.3 Linux for CUDA 9.0. Filename: `cudnn-9.0-linux-x64-v7.tgz`. Unzip the cuDNN package.
```
$ tar -xzvf cudnn-9.0-linux-x64-v7.tgz
```

Copy the following files to the cuda installation directory.
```
$ sudo cp cuda/include/cudnn.h /scratch/cuda-9.0/include
$ sudo cp cuda/lib64/libcudnn* /scratch/cuda-9.0/lib64
$ sudo chmod a+r /scratch/cuda-9.0/include/cudnn.h /scratch/cuda-9.0/lib64/libcudnn*
```

## Tensorflow r1.4 installation on RHEL

These steps are to install TF r1.4, however for the final config we will only use TF r1.3 (see last section below).

Reference: 
[Tensorflow Install using Binaries](https://www.tensorflow.org/versions/master/install/install_linux)
[Tensorflow Build from Source](https://www.tensorflow.org/versions/master/install/install_sources)

Tensorflow provides compiled pre-built binaries for only a limited systems, Ubuntu being the only Linux variant supported. Nonetheless, some RHEL users have reported successfully installing from TF / TF-GPU binaries using Anaconda (`conda install tensorflow`). I confirmed this by successfully installing TF from the pre-built linux binaries using both the virtualenv and anaconda method, but it requires older CUDA versions (CUDA 8 / cuDNN 6) to run, and there are other GLIBC dependencies that make the current RHEL system incompatible as it is. In order to use the latest CUDA versions installed earlier (CUDA 9 / cuDNN 7), the only way out is to build TF from source [[ref](https://devtalk.nvidia.com/default/topic/1026198/cuda-9-0-importerror-libcublas-so-8-0/)] but that has other build issues too. Let's explore both methods.

### Prepare environment

Install Python3 packages (TF requires python3-numpy, python3-pip, python3-wheel, python3-dev). Download Anaconda 5.0.1 Linux installer for Python 3.6 version from [here](https://www.anaconda.com/download/#linux) and install. Once done, add installation directory to PATH variable.
```
$ sudo bash ./Anaconda3-5.0.1-Linux-x86_64.sh
$ export PATH="/scratch/anaconda3/bin:$PATH"
```

Check versions of installed packages.
```
$ python3 --version

Python 3.6.3 :: Anaconda, Inc.

$ pip -V

pip 9.0.1 from /scratch/anaconda3/lib/python3.6/site-packages (python 3.6)
```

Install virtualenv using pip.
```
$ sudo su -
$ pip install virtualenv
```

### Install TF from binary - virtualenv method

Create a virtualenv.
```
$ virtualenv --system-site-packages -p python3 /scratch/tensorflow

Running virtualenv with interpreter /scratch/anaconda3/bin/python3
Using base prefix '/scratch/anaconda3'
New python executable in /scratch/tensorflow/bin/python3
Also creating executable in /scratch/tensorflow/bin/python
Installing setuptools, pip, wheel...done.
```

Install TF within the virtualenv using pip.
```
$ source /scratch/tensorflow/bin/activate
(tensorflow) $ pip3 install --upgrade tensorflow-gpu
```

Validate the installation by running the following on python:
```
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

TF is successfully installed, however on running python `import tensorflow as tf` it errors out: 
```
ImportError: libcublas.so.8.0: cannot open shared object file. No such file or directory.
```

So it is looking for CUDA 8, but the installed version is CUDA 9. The pre-built binaries are old and we would have to either downgrade to older CUDA/cuDNN versions to continue using this TF installation, or build TF from source [[ref](https://devtalk.nvidia.com/default/topic/1026198/cuda-9-0-importerror-libcublas-so-8-0/)].

Downgrade to CUDA 8 / cuDNN 6 by installing them respectively (might need to install latest CUDA drivers since the versions attached to CUDA toolkit 8 might be old) and redefining the `PATH` and `LD_LIBRARY_PATH` to point to this version. 

Now upon python `import tensorflow as tf` it has a new error: 
```
ImportError: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.19' not found (required by /scratch/tensorflow/lib/python3.6/site-packages/tensorflow/python/_pywrap_tensorflow_internal.so)
```

Check the available versions of GLIBC* in `/usr/lib64/libstdc++.so.6`.
```
$ strings /usr/lib64/libstdc++.so.6 | grep GLIBC*

GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBC_2.2.5
GLIBC_2.3
GLIBC_2.4
GLIBC_2.3.2
```

The pre-built TF binary has a hard dependency on `GLIBCXX_3.4.19` which isn't present in `/usr/lib64/libstdc++.so.6` but is present in `/scratch/anaconda3/lib/libstdc++.so.6`.
```
$ strings /scratch/anaconda3/lib/libstdc++.so.6 | grep GLIBC*

GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBC_2.3
GLIBC_2.2.5
GLIBC_2.3.2
```

To pick the `glibcxx` from the newer Anaconda env, try to next install TF using anaconda method.

### Install TF from binary - anaconda method

Since Anaconda was already installed in the previous steps, simply create a new conda env named `tensorflow` and activate it.
```
$ conda create -n tensorflow pip python=3.6
$ source activate tensorflow
```

Install TF inside the conda env.
```
(tensorflow) $ pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.4.0-cp36-cp36m-linux_x86_64.whl
```

Validate the installation as done previously. Upon running python `import tensorflow as tf` there is a new error.
```
ImportError: /lib64/libc.so.6: version `GLIBC_2.16' not found (required by /home/sambhavj/.conda/envs/tensorflow/lib/python3.6//tensorflow/python/_pywrap_tensorflow_internal.so)
```
```
$ strings /lib64/libc.so.6 | grep GLIBC*

GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
```
```
$ ldd --version

ldd (GNU libc) 2.12
```

Current version is `GLIBC_2.12`, but the installation requires `GLIBC_2.16`. This is much harder to fix on a cluster machine, and often requires upgrade of OS/kernels. Installing a separate version of `glibc` and including in `LD_LIBRARY_PATH` is not recommended as it can affect shell usage and render the machine unusable. 

Hence the only option remaining is to build TF from source. For this, we require Bazel (which is a super efficient build tool by Google).

### Install Bazel 0.7.0

These steps are to install Bazel 0.7.0, however for the final config we will only use Bazel 0.5.4 (use similar steps).

Reference: 
[Bazel Compile from Source](https://docs.bazel.build/versions/master/install-compile-source.html)
[My Bazel Build Issue on RHEL 6.8](https://github.com/bazelbuild/bazel/issues/4107)

Install JDK8 dependency (for Bazel).
```
$ sudo yum install java-1.8.0-openjdk-devel.x86_64 java-1.8.0-openjdk-headless.x86_64
```

#### Bazel from binary

Download Bazel 0.7.0 binary (for Ubuntu Linux) from [here](https://github.com/bazelbuild/bazel/releases) and run.
```
chmod +x bazel-0.7.0-installer-linux-x86_64.sh
./bazel-0.7.0-installer-linux-x86_64.sh --prefix=/scratch/bazel
```

Error:
```
Uncompressing....../scratch/bazel/bin/bazel: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /scratch/bazel/bin/bazel)
/scratch/bazel/bin/bazel: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.14' not found (required by /scratch/bazel/bin/bazel)
/scratch/bazel/bin/bazel: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.19' not found (required by /scratch/bazel/bin/bazel)
/scratch/bazel/bin/bazel: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.15' not found (required by /scratch/bazel/bin/bazel)
/scratch/bazel/bin/bazel: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.18' not found (required by /scratch/bazel/bin/bazel)
```

This results in a similar issue as earlier. The binary has a hard dependency on `GLIBC_2.14` which is a prohibitive change to do on a cluster machine. The solution is to build Bazel from source.

#### Bazel from source [[issue](https://github.com/bazelbuild/bazel/issues/4107)]

Download Bazel 0.7.0 distribution archive from [here](https://github.com/bazelbuild/bazel/releases), unzip and compile.
```
$ unzip bazel-0.7.0-dist.zip
$ bash compile.sh
```

Error (snipped):
```
🍃  Building Bazel from scratch......
🍃  Building Bazel with Bazel.
ERROR: /scratch/bazel/build1/third_party/ijar/BUILD:47:1: C++ compilation of rule '//third_party/ijar:zlib_client' failed (Exit 1): gcc failed: error executing command
  (cd /tmp/bazel_42EuRdvP/out/execroot/io_bazel && \
  exec env - \
    LD_LIBRARY_PATH=/scratch/cuda-9.0/lib64:/scratch/cuda-9.0/extras/CUPTI/lib64: \
    PATH=/scratch/anaconda3/bin:/scratch/cuda-9.0/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/puppetlabs/bin:/tools/xgs/bin:/home/sambhavj/bin \
    PWD=/proc/self/cwd \
  /usr/bin/gcc -U_FORTIFY_SOURCE -fstack-protector -Wall -B/usr/bin -B/usr/bin -Wunused-but-set-parameter -Wno-free-nonheap-object -fno-omit-frame-pointer -g0 -O2 '-D_FORTIFY_SOURCE=1' -DNDEBUG -ffunction-sections -fdata-sections '-std=c++0x' -MD -MF bazel-out/local-opt/bin/third_party/ijar/_objs/zlib_client/third_party/ijar/zlib_client.d '-frandom-seed=bazel-out/local-opt/bin/third_party/ijar/_objs/zlib_client/third_party/ijar/zlib_client.o' -iquote . -iquote bazel-out/local-opt/genfiles -iquote external/bazel_tools -iquote bazel-out/local-opt/genfiles/external/bazel_tools -isystem third_party/zlib -isystem bazel-out/local-opt/genfiles/third_party/zlib -isystem external/bazel_tools/tools/cpp/gcc3 -Wno-builtin-macro-redefined '-D__DATE__="redacted"' '-D__TIMESTAMP__="redacted"' '-D__TIME__="redacted"' -c third_party/ijar/zlib_client.cc -o bazel-out/local-opt/bin/third_party/ijar/_objs/zlib_client/third_party/ijar/zlib_client.o).
ERROR: Could not build Bazel
```

As it is, build fails with error `C++ compilation of rule ... failed` since the default `gcc-4.4.7` which comes with RHEL 6.8 is too old [[ref1](https://github.com/bazelbuild/bazel/issues/1900)] [[ref2](https://stackoverflow.com/questions/42982735/how-to-compile-bazel-on-centos-6-x)]. This requires installing newer gcc at a custom path and helping Bazel pick the right path (quite tricky). Previous Bazel releases requires modifying `tools/cpp/CROSSTOOL` [[ref](https://github.com/bazelbuild/bazel/issues/760)]. In newer released, a `cc_configure.bzl` method is in place [[ref](https://groups.google.com/forum/#!msg/bazel-discuss/MSunz2ZUOq0/U5tE7uQLJQAJ)] for auto-config for custom gcc toolchain paths, which still requires defining the right flags (`JAVA_HOME`, `CC`, `CXX`, `LDFLAGS`, `CXXFLAGS`). Refer to this [Wiki](https://github.com/bazelbuild/bazel/wiki/Building-with-a-custom-toolchain) for building older releases of bazel with a custom toolchain (`CROSSTOOL`).

To proceed, install a newer `gcc-4.8.4` at custom path. The following steps are taken from [ref1](https://gcc.gnu.org/wiki/InstallingGCC), [ref2](https://stackoverflow.com/questions/25433142/installing-gcc-4-8-2-on-red-hat-enterprise-linux-6-5). Download `gcc-4.8.4.tar.gz` from [here](https://ftp.gnu.org/gnu/gcc/) and install.

```
$ tar xzf gcc-4.8.4.tar.gz
$ cd gcc-4.8.4
$ ./contrib/download_prerequisites
$ cd ../
$ mkdir objdir
$ cd objdir
$ ../gcc-4.8.4/configure --prefix=/scratch/gcc-4.8.4 --enable-languages=c,c++,fortran,go
$ make
$ make install
```

Output (snipped):
```
Libraries have been installed in:
   /scratch/gcc-4.8.4/lib/../lib
Libraries have been installed in:
   /scratch/gcc-4.8.4/lib/../lib64

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'
```

Once installed, add the custom binary and library paths to point to `gcc-4.8.4`. We add `/scratch/gcc-4.8.4/lib64` to the `LD_LIBRARY_PATH` since the default `/usr/lib64` doesn't have all the required versions of `GLIBCXX*`.
```
$ export PATH="/scratch/gcc-4.8.4/bin:$PATH"
$ export LD_LIBRARY_PATH="/scratch/gcc-4.8.4/lib64:$LD_LIBRARY_PATH"
```

Now that `gcc-4.8.4` is installed at `/scratch/gcc-4.8.4`, set the following flags (only in current shell, not in `~/.bashrc`) and build bazel.
```
$ readlink -f /usr/bin/java | sed 's/\/jre\/bin\/java//'
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.91-1.b14.el6.x86_64

$ export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.91-1.b14.el6.x86_64"
$ export CC="/scratch/gcc-4.8.4/bin/gcc"
$ export CXX="/scratch/gcc-4.8.4/bin/c++"
$ export LDFLAGS="-L/scratch/gcc-4.8.4/lib -L/scratch/gcc-4.8.4/lib64"
$ export CXXFLAGS="-L/scratch/gcc-4.8.4/lib -L/scratch/gcc-4.8.4/lib64"

$ bash compile.sh
```

Output (snipped):
```
🍃  Building Bazel from scratch......
🍃  Building Bazel with Bazel.

Target //src:bazel up-to-date:
  bazel-bin/src/bazel
INFO: Elapsed time: 143.167s, Critical Path: 53.32s

Build successful! Binary is here: /scratch/bazel/build/output/bazel
```

We can copy the binary `bazel` to a directory on the PATH (such as `/usr/local/bin` on Linux) or use it in-place.
```
$ cp /scratch/bazel/output/bazel /scratch/bazel/bin/
$ export PATH="/scratch/bazel/bin:$PATH"
```

### Build TF from source (final config)

This is the **final configuration** that worked. Ensure the correct versions of all dependencies are installed.

```
RHEL 6.8 (with default gcc 4.4.7)
CUDA 8.0
cuDNN 6.0
python 3.6.3 (Anaconda)
gcc 4.8.4 (build from source)
Bazel 0.5.4 (build from source)
Tensorflow_GPU r1.3 (build from source)
```

Check env for correct paths and package versions.
```
$ echo $PATH
/scratch/gcc-4.8.4/bin:/scratch/bazel/bin:/scratch/anaconda3/bin:/scratch/cuda-8.0/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/puppetlabs/bin:/tools/xgs/bin:/home/sambhavj/bin

$ echo $LD_LIBRARY_PATH
/scratch/gcc-4.8.4/lib64:/scratch/cuda-8.0/lib64:/scratch/cuda-8.0/extras/CUPTI/lib64:

$ which nvcc
/scratch/cuda-8.0/bin/nvcc

$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2016 NVIDIA Corporation
Built on Tue_Jan_10_13:22:03_CST_2017
Cuda compilation tools, release 8.0, V8.0.61

$ which python
/scratch/anaconda3/bin/python

$ python --version
Python 3.6.3 :: Anaconda, Inc.

$ which gcc
/scratch/gcc-4.8.4/bin/gcc

$ gcc -v
gcc version 4.8.4 (GCC)

$ which bazel
/scratch/bazel/bin/bazel

$ bazel version
Build label: 0.5.4- (@non-git)
Build target: bazel-out/local-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Sat Nov 18 06:33:51 2017 (1510986831)
```

Clone TF repository.
```
$ git clone https://github.com/tensorflow/tensorflow
$ cd tensorflow
$ git checkout r1.3
```

#### Prepare before running `configure` step

Set `/tmp` for Bazel extraction, since the default home location is on another NFS mount which leads to indeterministic behavior when running Bazel (`WARNING: Output base '/home/sambhavj/.cache/bazel/_bazel_sambhavj/d41d8cd98f00b204e9800998ecf8427e' is on NFS. This may lead to surprising failures and undetermined behavior.`)
```
$ export TEST_TMPDIR="/tmp"
```

Check if jemalloc support can be enabled. The Linux kernel version has to be greater than >= 2.6.38 for jemalloc to work [ref](https://github.com/tensorflow/tensorflow/issues/7268). Since it's not, disable jemalloc during `configure` to avoid `C++ compilation of rule '@jemalloc//:jemalloc' failed`.
```
$ uname -r
2.6.32-642.el6.x86_64
```

Check the compute capability for the installed NVIDIA GPU [here](https://developer.nvidia.com/cuda-gpus). It is 6.1 for GeForce GTX 1080 Ti. We will also avoid choosing `clang` as CUDA compiler and use the default `nvcc` instead. Specify the custom CUDA path. And if the configuration complains about missing cuDNN 6.0, just set the correct symbolic link `$ sudo ln -sf libcudnn.so.6.0.21 /scratch/cuda-8.0/lib64/libcudnn.so.6.0`. Run `configure`.

```
$ ./configure

WARNING: ignoring http_proxy in environment.
INFO: $TEST_TMPDIR defined: output root default is '/tmp'.
.........
You have bazel 0.5.4- installed.
Please specify the location of python. [Default is /scratch/anaconda3/bin/python]:
Found possible Python library paths:
  /scratch/anaconda3/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/scratch/anaconda3/lib/python3.6/site-packages]

Using python library path: /scratch/anaconda3/lib/python3.6/site-packages
Do you wish to build TensorFlow with MKL support? [y/N] n
No MKL support will be enabled for TensorFlow
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
Do you wish to use jemalloc as the malloc implementation? [Y/n] n
jemalloc disabled
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N] n
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with Hadoop File System support? [y/N] n
No Hadoop File System support will be enabled for TensorFlow
Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N] n
No XLA JIT support will be enabled for TensorFlow
Do you wish to build TensorFlow with VERBS support? [y/N] n
No VERBS support will be enabled for TensorFlow
Do you wish to build TensorFlow with OpenCL support? [y/N] n
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N] y
CUDA support will be enabled for TensorFlow
Do you want to use clang as CUDA compiler? [y/N] n
nvcc will be used as CUDA compiler
Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 8.0]: 8.0
Please specify the location where CUDA 8.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: /scratch/cuda-8.0
Please specify which gcc should be used by nvcc as the host compiler. [Default is /scratch/gcc-4.8.4/bin/gcc]:
Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 6.0]: 6.0
Please specify the location where cuDNN 6.0 library is installed. Refer to README.md for more details. [Default is /scratch/cuda-8.0]:
Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size.
[Default is: "6.1,3.0"]: 6.1
Do you wish to build TensorFlow with MPI support? [y/N] n
MPI support will not be enabled for TensorFlow
Configuration finished
```

The regular installation command is
```
$ bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
```

However, 

But there are issues with --config=opt (avx / sse / fma) C++ library detection


```
$ bazel build -c opt --config=cuda --verbose_failures //tensorflow/tools/pip_package:build_pip_package
...
...
...
At global scope:
cc1plus: warning: unrecognized command line option "-Wno-self-assign" [enabled by default]
Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package
INFO: Elapsed time: 1055.219s, Critical Path: 171.70s
```

```
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /scratch/tf

Sun Nov 19 19:02:27 PST 2017 : === Using tmpdir: /tmp/tmp.IQLZJxFVzc
/scratch/setup/tf/tensorflow/bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles /scratch/setup/tf/tensorflow
/scratch/setup/tf/tensorflow
/tmp/tmp.IQLZJxFVzc /scratch/setup/tf/tensorflow
Sun Nov 19 19:02:27 PST 2017 : === Building wheel
/scratch/setup/tf/tensorflow
Sun Nov 19 19:02:40 PST 2017 : === Output wheel file is in: /scratch/tf
```

```
$ virtualenv --system-site-packages -p python3 tensorflow
$ source tensorflow/bin/activate
(tensorflow) $ pip install --upgrade /scratch/tf/tensorflow-1.3.1-cp36-cp36m-linux_x86_64.whl
```



```
C++ compilation of rule '@boringssl//:crypto' failed (Exit 1): 
https://github.com/tensorflow/tensorflow/issues/12579
```
bazel build --config=opt --config=cuda --verbose_failures //tensorflow/tools/pip_package:build_pip_package --copt=-O


Tried with bazel 0.5.4, cuda 8 / cudnn 6
C++ compilation of rule '@grpc//third_party/nanopb:nanopb' failed (Exit 1):

Solutions!!!!!!!!!!!!!!!!!
http://thelazylog.com/install-tensorflow-with-gpu-support-on-sandbox-redhat/
https://github.com/tensorflow/tensorflow/issues/110#issuecomment-201834137

https://ctmakro.github.io/site/on_learning/tf1c.html 
https://github.com/tensorflow/tensorflow/issues/7449


```
bazel build --config=opt --config=cuda --verbose_failures //tensorflow/tools/pip_package:build_pip_package --copt=-O --linkopt '-lrt' --linkopt '-lm' --linkopt '-lz' --linkopt '-Wl,-rpath,/scratch/cuda-9.0/lib64'
```
https://github.com/tensorflow/tensorflow/issues/110#issuecomment-304106970
