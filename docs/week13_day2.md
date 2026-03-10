# Day 2: Transfer Learning in Computer Vision

Welcome to Day 2. Yesterday, we learned the syntax to freeze a model's weights. Today, we deploy this in a fully functional Computer Vision pipeline.

When dealing with images, researchers use famous backbone models. 
*   **VGG16:** A simple, deep vertical stack of convolutions.
*   **ResNet50:** Utilizes "Residual Connections" to skip layers, preventing Vanishing Gradients on extremely deep photo analysis.
*   **EfficientNet:** Scales depth and width dynamically for mobile environments.

Today, we use **ResNet50** as our frozen backbone. 

## The Intuition
The frozen ResNet50 will act as a "Camera Lens." It looks at a picture of a dog, analyzes it through 50 layers of pre-trained convolutional math, and outputs a highly compressed, brilliant Array of features. 

We then take that Array, and feed it into our *own* custom Dense Neural Network (The Classification Head) that we build from scratch!

## Hands-On: TensorFlow Custom Head Implementation
Look at `day2_tf.py`. We pull `ResNet50` from Keras Applications. Notice the `include_top=False` parameter! This explicitly tells TensorFlow to amputate ResNet's output layer before downloading it!

```python
# day2_tf.py
from tensorflow.keras.applications import ResNet50
from tensorflow.keras.layers import Dense, Flatten
from tensorflow.keras.models import Model

# 1. Download ResNet50 WITHOUT its head (include_top=False)!
base_model = ResNet50(weights="imagenet", include_top=False, input_shape=(224, 224, 3))

# 2. Freeze the base model
for layer in base_model.layers:
    layer.trainable = False

# 3. BUILD THE CUSTOM HEAD!
x = Flatten()(base_model.output)            # Flatten ResNet's feature map Array
x = Dense(256, activation='relu')(x)        # Build our own hidden layer
output = Dense(5, activation="softmax")(x)  # Output exactly 5 Custom Classes!

# 4. Bind the Frozen base and the Custom Head together into a single Model!
model = Model(inputs=base_model.input, outputs=output)

model.compile(optimizer='adam', loss="categorical_crossentropy", metrics=['accuracy'])
# Output:
# 2026-03-09 23:51:54.875464: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773114718.572968  742846 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6083 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
# 2026-03-09 23:51:58.927143: W external/local_xla/xla/service/gpu/llvm_gpu_backend/default/nvptx_libdevice_path.cc:41] Can't find libdevice directory ${CUDA_DIR}/nvvm/libdevice. This may result in compilation or runtime failures, if the program we try to run uses routines from libdevice.
# Searched for CUDA in the following directories:
#   ./cuda_sdk_lib
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_5dbb74da.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day2_tf.py
# from tensorflow.keras.applications import ResNet50
# from tensorflow.keras.layers import Dense, Flatten
# from tensorflow.keras.models import Model
# 
# # 1. Download ResNet50 WITHOUT its head (include_top=False)!
# base_model = ResNet50(weights="imagenet", include_top=False, input_shape=(224, 224, 3))
# 
# # 2. Freeze the base model
# for layer in base_model.layers:
#     layer.trainable = False
# 
# # 3. BUILD THE CUSTOM HEAD!
# x = Flatten()(base_model.output)            # Flatten ResNet's feature map Array
# x = Dense(256, activation='relu')(x)        # Build our own hidden layer
# output = Dense(5, activation="softmax")(x)  # Output exactly 5 Custom Classes!
# 
# # 4. Bind the Frozen base and the Custom Head together into a single Model!
# model = Model(inputs=base_model.input, outputs=output)
# 
# model.compile(optimizer='adam', loss="categorical_crossentropy", metrics=['accuracy']).runfiles/cuda_nvcc
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_5dbb74da.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day2_tf.py
# from tensorflow.keras.applications import ResNet50
# from tensorflow.keras.layers import Dense, Flatten
# from tensorflow.keras.models import Model
# 
# # 1. Download ResNet50 WITHOUT its head (include_top=False)!
# base_model = ResNet50(weights="imagenet", include_top=False, input_shape=(224, 224, 3))
# 
# # 2. Freeze the base model
# for layer in base_model.layers:
#     layer.trainable = False
# 
# # 3. BUILD THE CUSTOM HEAD!
# x = Flatten()(base_model.output)            # Flatten ResNet's feature map Array
# x = Dense(256, activation='relu')(x)        # Build our own hidden layer
# output = Dense(5, activation="softmax")(x)  # Output exactly 5 Custom Classes!
# 
# # 4. Bind the Frozen base and the Custom Head together into a single Model!
# model = Model(inputs=base_model.input, outputs=output)
# 
# model.compile(optimizer='adam', loss="categorical_crossentropy", metrics=['accuracy']).runfiles/cuda_nvdisasm
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_5dbb74da.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day2_tf.py
# from tensorflow.keras.applications import ResNet50
# from tensorflow.keras.layers import Dense, Flatten
# from tensorflow.keras.models import Model
# 
# # 1. Download ResNet50 WITHOUT its head (include_top=False)!
# base_model = ResNet50(weights="imagenet", include_top=False, input_shape=(224, 224, 3))
# 
# # 2. Freeze the base model
# for layer in base_model.layers:
#     layer.trainable = False
# 
# # 3. BUILD THE CUSTOM HEAD!
# x = Flatten()(base_model.output)            # Flatten ResNet's feature map Array
# x = Dense(256, activation='relu')(x)        # Build our own hidden layer
# output = Dense(5, activation="softmax")(x)  # Output exactly 5 Custom Classes!
# 
# # 4. Bind the Frozen base and the Custom Head together into a single Model!
# model = Model(inputs=base_model.input, outputs=output)
# 
# model.compile(optimizer='adam', loss="categorical_crossentropy", metrics=['accuracy']).runfiles/nvidia_nvshmem
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

### Training the Pipeline
When you execute `model.fit()`, the image passes through the completely frozen `base_model` instantly. Backpropagation *only* touches the final `Dense` layers we attached. 

Because we are only training 2 layers instead of 50, the model rockets to $90\%+$ accuracy in a fraction of the time, even on tiny datasets! 

## Wrapping Up Day 2
You have successfully orchestrated a classic Transfer Learning architecture. 

However, sometimes the dataset you are classifying (e.g., microscopic cells) looks radically different than the dataset the backbone was originally trained on (e.g., airplanes and dogs). If they look too different, the frozen feature extractor might fail.

Tomorrow, on **Day 3: Fine-Tuning Techniques**, we will learn how to selectively *unfreeze* layers to force the algorithm to adapt its base math to our custom reality.
