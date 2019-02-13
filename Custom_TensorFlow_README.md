## TensorFlow 1.12 GPU (w/ patch)

Includes patch for indefinite stalling when using `ignore_errors` with corrupt TFRecords.

### Reference
- [Issue](https://github.com/tensorflow/tensorflow/issues/25700)
- [PR](https://github.com/tensorflow/tensorflow/pull/25705)

### Build from Source
See [this page](https://www.tensorflow.org/install/source) for official instructions on building TensorFlow from source.

Here are the custom steps to build r1.12 including the patch.

#### Install Python (Anaconda)
```
wget https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh
bash ./Anaconda3-5.3.1-Linux-x86_64.sh

conda create -n my_tf pip python=3.6
```

#### Install Bazel (binary)
```
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python
export TEST_TMPDIR=/tmp/bazel/

wget https://github.com/bazelbuild/bazel/releases/download/0.15.0/bazel-0.15.0-installer-linux-x86_64.sh
chmod +x bazel-0.15.0-installer-linux-x86_64.sh

./bazel-0.15.0-installer-linux-x86_64.sh --prefix=/scratch/bazel
```

#### Build TensorFlow
Install dependencies.
```
conda activate my_tf

pip install -U pip six numpy wheel mock
pip install -U keras_applications==1.0.6 --no-deps
pip install -U keras_preprocessing==1.0.5 --no-deps
```
Clone and checkout branch `v1.12_ignore_errors_fix` containing the patch on r1.12.
```
git clone https://github.com/sjain-stanford/tensorflow.git
cd tensorflow/
git checkout v1.12_ignore_errors_fix
```
Configure the bazel build with the following options.
```
$ ./configure

You have bazel 0.15.0 installed.
Please specify the location of python. [Default is /scratch/anaconda3/envs/my_tf/bin/python]:

Found possible Python library paths:
  /scratch/anaconda3/envs/my_tf/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/scratch/anaconda3/envs/my_tf/lib/python3.6/site-packages]

Do you wish to build TensorFlow with Apache Ignite support? [Y/n]: n
No Apache Ignite support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [Y/n]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]:
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]:
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use. [Leave empty to default to CUDA 9.0]:

Please specify the location where CUDA 9.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: /scratch/cuda-9.0

Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7]:

Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /scratch/cuda-9.0]:

Do you wish to build TensorFlow with TensorRT support? [y/N]:
No TensorRT support will be enabled for TensorFlow.

Please specify the NCCL version you want to use. If NCCL 2.2 is not installed, then you can use version 1.3 that can be fetched automatically but it may have worse performance with multiple GPUs. [Default is 2.2]: 1.3

Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,7.0]: 7.0

Do you want to use clang as CUDA compiler? [y/N]:
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]:

Do you wish to build TensorFlow with MPI support? [y/N]:
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:

Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]:
Not configuring the WORKSPACE for Android builds.

Configuration finished
```
Bazel build. If using `GCC 5`, provide the switch `--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0"` to be consistent with older ABI used to build official TensorFlow binaries.
```
bazel build --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
```
Build the package.
```
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```
Install the package.
```
pip install /tmp/tensorflow_pkg/tensorflow-1.12.0-cp36-cp36m-linux_x86_64.whl
```
Done!
