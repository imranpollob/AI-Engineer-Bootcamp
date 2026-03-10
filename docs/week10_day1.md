# Day 1: Introduction to Convolutional Neural Networks

Welcome to Week 10. Last week, we used `Dense` Networks to classify images. 

If we feed a `Dense` network a picture of a dog perfectly centered in the frame, it learns to look for "Dog Pixels" in the middle of the photograph. 

If we show that exact same AI a picture of the *same dog*, but the dog is standing in the top-left corner of the frame... the `Dense` network will fail completely. It doesn't understand that the object moved. It just sees that the middle pixels are empty.

This is a lack of **Translation Invariance**.

## The Convolutional Neural Network (CNN)
The CNN solves Translation Invariance. Instead of assigning a unique Weight to every single pixel permanently, a CNN uses a tiny sliding window (a "Kernel") that scans the entire image left-to-right, top-to-bottom. 

Because the window scans everywhere, it doesn't matter if the dog is in the center, the top-left, or the bottom-right. The window will eventually slide over it and detect the mathematical "texture" of the dog!

### The CNN Pipeline
Almost all Modern Computer Vision models follow a strict 3-Part pipeline:
1.  **Convolutional Layers:** These slide over the image and extract raw localized features (Edges, Curves, Colors).
2.  **Pooling Layers:** These downsize the image to save RAM and focus only on the brightest, most important features.
3.  **Fully Connected (Dense) Layers:** After the image is heavily processed by Convolutions, it is finally passed to a traditional Dense layer which acts as the final "Voting Classifier".

## Hands-On Let's Initialize!
Look at `day1_ex.py`. We prepare placeholders for both architectures!

```python
# day1_ex.py
import tensorflow as tf
import torch.nn as nn

# 1. TensorFlow CNN Blueprint
tf_model = tf.keras.Sequential([
    # The Sliding Window!
    tf.keras.layers.Conv2D(32, (3, 3), activation="relu", input_shape=(32, 32, 3)),
    # The Downsizing Layer!
    tf.keras.layers.MaxPooling2D((2, 2)),
    # ... [Flatten and Dense Classifiers]
])

# 2. PyTorch CNN Blueprint
class SimpleCNN(nn.Module):
    def __init__(self):
        super(SimpleCNN, self).__init__()
        # The Sliding Window!
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, activation='relu')
        # The Downsizing Layer!
        self.pool = nn.MaxPool2d(2, 2)
# Output:
# 2026-03-09 23:57:03.056518: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/keras/src/layers/convolutional/base_conv.py:113: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.
#   super().__init__(activity_regularizer=activity_regularizer, **kwargs)
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773115028.003668  745067 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6086 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
# 2026-03-09 23:57:08.297381: W external/local_xla/xla/service/gpu/llvm_gpu_backend/default/nvptx_libdevice_path.cc:41] Can't find libdevice directory ${CUDA_DIR}/nvvm/libdevice. This may result in compilation or runtime failures, if the program we try to run uses routines from libdevice.
# Searched for CUDA in the following directories:
#   ./cuda_sdk_lib
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_e9adde52.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day1_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow CNN Blueprint
# tf_model = tf.keras.Sequential([
#     # The Sliding Window!
#     tf.keras.layers.Conv2D(32, (3, 3), activation="relu", input_shape=(32, 32, 3)),
#     # The Downsizing Layer!
#     tf.keras.layers.MaxPooling2D((2, 2)),
#     # ... [Flatten and Dense Classifiers]
# ])
# 
# # 2. PyTorch CNN Blueprint
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         # The Sliding Window!
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3, activation='relu')
#         # The Downsizing Layer!
#         self.pool = nn.MaxPool2d(2, 2).runfiles/cuda_nvcc
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_e9adde52.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day1_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow CNN Blueprint
# tf_model = tf.keras.Sequential([
#     # The Sliding Window!
#     tf.keras.layers.Conv2D(32, (3, 3), activation="relu", input_shape=(32, 32, 3)),
#     # The Downsizing Layer!
#     tf.keras.layers.MaxPooling2D((2, 2)),
#     # ... [Flatten and Dense Classifiers]
# ])
# 
# # 2. PyTorch CNN Blueprint
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         # The Sliding Window!
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3, activation='relu')
#         # The Downsizing Layer!
#         self.pool = nn.MaxPool2d(2, 2).runfiles/cuda_nvdisasm
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_e9adde52.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day1_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow CNN Blueprint
# tf_model = tf.keras.Sequential([
#     # The Sliding Window!
#     tf.keras.layers.Conv2D(32, (3, 3), activation="relu", input_shape=(32, 32, 3)),
#     # The Downsizing Layer!
#     tf.keras.layers.MaxPooling2D((2, 2)),
#     # ... [Flatten and Dense Classifiers]
# ])
# 
# # 2. PyTorch CNN Blueprint
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         # The Sliding Window!
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3, activation='relu')
#         # The Downsizing Layer!
#         self.pool = nn.MaxPool2d(2, 2).runfiles/nvidia_nvshmem
#   
# import/cuda_nvcc
#   
# import/cuda_nvdisasm
#   
# import/nvidia_nvshmem
#   
#   /usr/local/cuda
#   /opt/cuda
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../nvidia/cuda_nvcc
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../nvidia/cuda_nvcc
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../cuda
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../../..
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../../../..
#   .
# You can choose the search directory by setting xla_gpu_cuda_data_dir in HloModule's DebugOptions.  For most apps, setting the environment variable XLA_FLAGS=--xla_gpu_cuda_data_dir=/path/to/cuda will work.
```

## Wrapping Up Day 1
CNNs require drastically fewer Parameters than Dense networks because a sliding Kernel (which only contains 9 weights) is re-used thousands of times across the entire image! It is beautifully efficient.

But how does a set of 9 random numbers actually extract an "Edge" from a photograph? 

Tomorrow on **Day 2: Convolutional Layers and Filters**, we drop down into the raw Numpy mathematics to manually trace an Edge-Detection matrix!
