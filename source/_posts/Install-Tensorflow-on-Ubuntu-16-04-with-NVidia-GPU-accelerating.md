title: Install Tensorflow on Ubuntu 16.04 with NVidia GPU accelerating
date: 2016-08-19 15:04:00
tags:
---

## Install Tensorflow on Ubuntu 16.04 with NVidia GPU accelerating

Hardware: Lenovo T450, CPU i7-5500U CPU @ 2.40GHz, Graphics Card GeForce 940m.

### Install Virtualenv

```bash
$ sudo apt-get install python-pip python-dev python-virtualenv
$ makedir  ~/env
$ virtualenv --system-site-packages ~/env/tensorflow
$ printf '\nalias tensorflow="source ~/env/tensorflow/bin/activate"' >> ~/.bashrc
$ source ~/.bashrc
$ tensorflow
(tensorflow)$ deactivate
```

### Install Other Dependencies

```bash  
$ sudo apt-get install python-numpy python-wheel python-imaging swig
```

### Install JDK (Bazel dependency)

```bash
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```

### Install Bazel

```bash  
$ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip  
$ wget https://github.com/bazelbuild/bazel/releases/download/0.3.1/bazel-0.3.1-installer-linux-x86_64.sh
$ chmod +x bazel-0.3.1-installer-linux-x86_64.sh
$ ./bazel-0.3.1-installer-linux-x86_64.sh --user
$ printf '\nexport PATH=$PATH:$HOME/bin' >> ~/.bashrc
$ source ~/.bashrc
$ bazel version
Build label: 0.3.1
Build target: bazel-out/local-fastbuild/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Fri Jul 29 09:09:52 2016 (1469783392)
Build timestamp: 1469783392
Build timestamp as int: 1469783392
```


### Install CUDA Software

Register NVIDIA [Accelerated Computer Developer Program](https://developer.nvidia.com/accelerated-computing-developer)

Download cuda-repo-ubuntu1604-8-0-rc_8.0.27-1_amd64-deb and cuda-misc-headers-8-0_8.0.27.1-1_amd64.deb from [cuda download page](https://developer.nvidia.com/cuda-downloads)  

```bash
$ sudo dpkg -i cuda-repo-ubuntu1604-8-0-rc_8.0.27-1_amd64.deb
$ sudo apt-get update
$ sudo apt-get install cuda
$ sudo dpkg -i cuda-misc-headers-8-0_8.0.27.1-1_amd64.deb
```

Download cuDNN for CUDA 8.0 RC (cuDNN v5.1 Library for Linux) from
[cuDNN download page](https://developer.nvidia.com/cudnn)

```
$ tar xvzf cudnn-8.0-linux-x64-v5.1.tgz
$ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
$ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
$ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
$ printf '\nexport LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64"' >> ~/.bashrc
```

### Install TensorFlow from source

```bash
$ git clone --recurse-submodules https://github.com/tensorflow/tensorflow
$ cd tensorflow
$ ./configure
Please specify the location of python. [Default is /usr/bin/python]:
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N] N
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with GPU support? [y/N] y
GPU support will be enabled for TensorFlow
Please specify which gcc nvcc should use as the host compiler. [Default is /usr/bin/gcc]:
Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave empty to use system default]:
Please specify the location where CUDA  toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify the Cudnn version you want to use. [Leave empty to use system default]:
Please specify the location where cuDNN  library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size.
[Default is: "3.5,5.2"]: 3.0
Setting up Cuda include
Setting up Cuda lib64
Setting up Cuda bin
Setting up Cuda nvvm
Setting up CUPTI include
Setting up CUPTI lib64
Configuration finished

$ bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
```
The build may fail several times, just need to re-run the command. And if the following error happens:

```
external/gif_archive/giflib-5.1.4/lib/gif_err.c:10:29: fatal error: gif_lib_private.h: No such file or directory
```

Just re-run the configure:

```
$ ./configure
Please specify the location of python. [Default is /usr/bin/python]:
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N] N
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with GPU support? [y/N] y
GPU support will be enabled for TensorFlow
Please specify which gcc nvcc should use as the host compiler. [Default is /usr/bin/gcc]:
Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave empty to use system default]:
Please specify the location where CUDA  toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify the Cudnn version you want to use. [Leave empty to use system default]:
Please specify the location where cuDNN  library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size.
[Default is: "3.5,5.2"]: 3.0
Setting up Cuda include
Setting up Cuda lib64
Setting up Cuda bin
Setting up Cuda nvvm
Setting up CUPTI include
Setting up CUPTI lib64
Configuration finished
```

Then re-run the build and continue to generate wheel and install it in virtualenv.

```
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package ~/tensorflow/bin
$ tensorflow
(tensorflow)$ sudo pip install ~/tensorflow/bin/~/tensorflow/bin/tensorflow-0.10.0rc0-py2-none-any.whl
```

### Verify the installation

Notice: do not try to run the following commands from tensorflow source folder. Otherwise, may encounter import tensorflow error:

```
>>> import tensorflow as tf
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "tensorflow/__init__.py", line 23, in <module>
    from tensorflow.python import *
  File "tensorflow/python/__init__.py", line 48, in <module>
    from tensorflow.python import pywrap_tensorflow
ImportError: cannot import name pywrap_tensorflow
```

Normal case should be like:

```
(tensorflow)$ python
Python 2.7.12 (default, Jul  1 2016, 15:12:24)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcublas.so locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcudnn.so locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcufft.so locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcuda.so.1 locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcurand.so locally
>>> a = tf.constant(10)
>>> b = tf.constant(22)
>>> c = a + b
>>> session = tf.Session()
>>> print session.run(c)
32
>>> 
```

