# Day 3: Pooling Layers and Dimensionality Reduction

Welcome to Day 3. Convolutions are powerful because they generate dozens of unique "Feature Maps" highlighting edges, textures, and patterns. 

But passing that massive amount of data perfectly preserved into a deeper network causes severe memory issues. Furthermore, if the AI memorizes the exact pixel integer location of every edge, the model will Overfit!

We solve this using **Pooling**. 

## What is a Pooling Layer?
Pooling is an aggressive downsampling technique. Like a Convolution, it uses a sliding window (usually $2 \times 2$), but it performs no complex multiplication. 

Instead, it looks at the 4 pixels inside the window and forces a drastic summary:
1.  **MaxPooling:** It looks at the 4 pixels and simply throws away the 3 weakest ones. It only keeps the single *brightest* (highest integer) pixel!
2.  **AveragePooling:** It looks at the 4 pixels and calculates their mathematical average, creating a blurred, smoothened single pixel.

## The Result of Pooling
Because a $2 \times 2$ window summarizes 4 pixels into 1, applying a `MaxPooling(2, 2)` layer instantly shrinks the physical width and height of an image by **half**! ($32 \times 32$ becomes $16 \times 16$).

This systematically strips out noise, brutally reduces the Parameter count, and forces the network to only focus on the absolute strongest mathematical activations!

```python
# day3_ex.py
import tensorflow as tf
import torch.nn as nn

# 1. TensorFlow Implementation
model_tf = tf.keras.Sequential([
    tf.keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3)),
    # Instantly cut the image dimensions in half!
    tf.keras.layers.MaxPooling2D((2,2)),
])

# 2. PyTorch Implementation
class SimpleCNN(nn.Module):
    def __init__(self):
        super(SimpleCNN, self).__init__()
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3)
        # Instantly cut the image dimensions in half!
        self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2)
# Output:
# 2026-03-09 23:56:30.274349: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/keras/src/layers/convolutional/base_conv.py:113: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.
#   super().__init__(activity_regularizer=activity_regularizer, **kwargs)
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773114995.154578  744458 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6086 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
# 2026-03-09 23:56:35.451773: W external/local_xla/xla/service/gpu/llvm_gpu_backend/default/nvptx_libdevice_path.cc:41] Can't find libdevice directory ${CUDA_DIR}/nvvm/libdevice. This may result in compilation or runtime failures, if the program we try to run uses routines from libdevice.
# Searched for CUDA in the following directories:
#   ./cuda_sdk_lib
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_f571e376.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day3_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow Implementation
# model_tf = tf.keras.Sequential([
#     tf.keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3)),
#     # Instantly cut the image dimensions in half!
#     tf.keras.layers.MaxPooling2D((2,2)),
# ])
# 
# # 2. PyTorch Implementation
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3)
#         # Instantly cut the image dimensions in half!
#         self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2).runfiles/cuda_nvcc
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_f571e376.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day3_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow Implementation
# model_tf = tf.keras.Sequential([
#     tf.keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3)),
#     # Instantly cut the image dimensions in half!
#     tf.keras.layers.MaxPooling2D((2,2)),
# ])
# 
# # 2. PyTorch Implementation
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3)
#         # Instantly cut the image dimensions in half!
#         self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2).runfiles/cuda_nvdisasm
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_f571e376.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day3_ex.py
# import tensorflow as tf
# import torch.nn as nn
# 
# # 1. TensorFlow Implementation
# model_tf = tf.keras.Sequential([
#     tf.keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(32, 32, 3)),
#     # Instantly cut the image dimensions in half!
#     tf.keras.layers.MaxPooling2D((2,2)),
# ])
# 
# # 2. PyTorch Implementation
# class SimpleCNN(nn.Module):
#     def __init__(self):
#         super(SimpleCNN, self).__init__()
#         self.conv1 = nn.Conv2d(3, 32, kernel_size=3)
#         # Instantly cut the image dimensions in half!
#         self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2).runfiles/nvidia_nvshmem
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

## Max vs. Average
In 95% of modern Deep Learning, you will exclusively use **MaxPooling**. 

Max Pooling acts as an extreme contrast filter. If a Convolutional Kernel successfully detects a sharp vertical edge, that pixel value will be extremely high. Max Pooling instantly grabs that specific high-value pixel and throws away the empty background noise around it! 

Average Pooling dilutes the sharp edge by blurring it with the surrounding noise.

## Wrapping Up Day 3
The classic structure of an architecture is born:
1. Convolution (Extract Features)
2. Max Pooling (Compress Image)
3. Convolution (Extract Deeper Features)
4. Max Pooling (Compress Image)

Tomorrow, on **Day 4: CNNs in TensorFlow**, we string these blocks together to build a fully capable image classifier in Keras.
