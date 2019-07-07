---
title: Limit GPU memory usage in Keras with Tensorflow Backend
date: 2018-05-18 17:26:19
tags:
---

## 问题：

我使用Keras 2.1.6和Tensorflow 1.8.0做一个OCR实验，这个OCR分为两步，第一步Text Localization没有使用Keras，直接使用Tensorflow实现的网络，单独运行正常；第二步Text Recognition使用Keras实现的另一个网络，执行时出现了Out of Memory的警告：

```
2018-05-18 17:01:53.022692: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:1053] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 11264 MB memory) -> physical GPU (device: 0, name: GeForce GTX 1080 Ti, pci bus id: 0000:01:00.0, compute capability: 6.1)
2018-05-18 17:01:53.028636: E T:\src\github\tensorflow\tensorflow\stream_executor\cuda\cuda_driver.cc:936] failed to allocate 11.00G (11811160064 bytes) from device: CUDA_ERROR_OUT_OF_MEMORY
2018-05-18 17:01:53.031124: E T:\src\github\tensorflow\tensorflow\stream_executor\cuda\cuda_driver.cc:936] failed to allocate 9.90G (10630043648 bytes) from device: CUDA_ERROR_OUT_OF_MEMORY
```

最后Python进程挂起，并且出现以下错误，创建cudnn handle失败：

```
2018-05-18 16:47:14.594352: E T:\src\github\tensorflow\tensorflow\stream_executor\cuda\cuda_dnn.cc:455] could not create cudnn handle: CUDNN_STATUS_ALLOC_FAILED
2018-05-18 16:47:14.596108: E T:\src\github\tensorflow\tensorflow\stream_executor\cuda\cuda_dnn.cc:427] could not destroy cudnn handle: CUDNN_STATUS_BAD_PARAM
2018-05-18 16:47:14.597622: F T:\src\github\tensorflow\tensorflow\core\kernels\conv_ops.cc:713] Check failed: stream->parent()->GetConvolveAlgorithms( conv_parameters.ShouldIncludeWinogradNonfusedAlgo<T>(), &algorithms)
```

我使用的主机有一块GTX 1080Ti 11G的显卡，而从错误信息里看，Keras/Tensorflow试图分配全部11G GPU内存，但由于桌面显示也使用同一块显卡，只剩下9.10G显存，因此进程挂起。显然不是根据task所需来分配GPU内存的。查阅资料，发现Keras缺省是要分配全部GPU内存。

## 解决方法：

Keras本身没有设置GPU内存限制的入口，但Tensorflow的session可以设置，设置以后可以再把session设置给Keras。

```
# limit GPU memory usage, otherwise it will try to allocate all memory and failed with out of memory.
import tensorflow as tf
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.7
from keras.backend.tensorflow_backend import set_session
set_session(tf.Session(config=config))
```

上面代码设置每个GPU内存使用上限是70%。

这段代码需要在每次train和inference之前执行，并且在Keras 2.0.9及以上版本，必须要在程序的最开始执行，一旦有Keras的import，它就会query可用的GPU，而在query的过程中会试图分配所有的GPU内存。

看Keras的github issues讨论，Keras团队认为这是Tensorflow的bug，有人认为是Keras 2.0.9引入的bug。没有下文。

测试环境：Windows 10 64bit，Anaconda3 5.1.0，Python 3.6.5，Tensorflow 1.8.0，Keras 2.1.6。

